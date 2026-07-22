# Malicious-Artefact Detection and the Level Ladder

A signed artefact from a trusted signer can still contain code that runs on load. Model file formats — particularly those built on Python pickle — are Turing-complete deserialisers by design; loading a maliciously crafted checkpoint can execute arbitrary code inside the inference process before any inference happens. Chapter 05's signing gate proves *where the artefact came from*; this chapter's scanning gate proves *what the artefact will do when loaded*. Together they close the runtime supply-chain surface. This chapter also draws the boundary lines to the roles above and below yours — the level-50 architect who owns the control-library architecture and the level-35 security engineer who owns the runtime signing / scanning platform.

## Motivation

Two failure modes to eliminate:

- **Pickle-based checkpoint from Hugging Face loads code on `torch.load`.** A team fetches a fine-tune from a public hub, calls `torch.load` in a Python process with production credentials, and the pickle's `__reduce__` executes an attacker's payload — exfiltrating environment variables, planting a persistent shell, or writing a backdoor into the model weights before the process even prints a log line. This is not hypothetical: multiple public checkpoints containing malicious pickle payloads have been observed in the wild.
- **A converted ONNX / SavedModel is trusted because it is "not pickle."** ONNX has custom-operator escape hatches; TensorFlow SavedModel embeds Python-callable ops in some configurations; Keras H5 can load Lambda layers that execute arbitrary code. Format-swapping does not eliminate the code-execution surface — it only changes which scanner needs to catch it.

The engineering fix is a scanner suite that runs *before* the artefact is loaded, on every ingestion, and blocks on findings above a policy threshold. The scanner suite is a *layered* defence — different scanners catch different classes — and it composes with the signing gate from chapter 05.

## The pickle-and-format threat surface

Every ML framework has an on-disk model format. Several of those formats permit code execution during load. A working inventory:

| Format | Underlying mechanism | Code-execution risk |
|---|---|---|
| Python pickle (`.pkl`, `.pickle`, `.pt`, `.pth`, `.bin` when produced by `torch.save`) | Python `pickle` module — Turing-complete, executes `__reduce__` on load | High. Arbitrary code executes on `pickle.load` / `torch.load` by default |
| Joblib (`.joblib`) | Wraps pickle with efficient array serialisation | Same as pickle |
| Numpy `.npy` / `.npz` | Numpy binary format; `.npy` header can specify pickle | Medium — depends on load flags; `allow_pickle=True` re-introduces pickle risk |
| TensorFlow SavedModel (`saved_model.pb`, `variables/`) | Protobuf graph + variables; may include Python-callable ops in certain configurations | Medium; check for `tf.py_function` and custom-op escape hatches |
| Keras H5 (`.h5`) | HDF5-based; can include Lambda layers whose code is serialised | Medium-to-high — Lambda layers can pickle-serialise arbitrary Python |
| ONNX (`.onnx`) | Protobuf graph; standardised operator set | Low by default; custom-operator plugins re-introduce risk |
| Safetensors (`.safetensors`) | Header + tensor buffers, no code | Very low by design; the format is *specifically* designed to eliminate arbitrary code execution on load |
| GGUF (`.gguf`) | Header + tensor buffers used by llama.cpp and its downstream tooling | Very low by design; format is data-only |
| TorchScript (`.pt` produced by `torch.jit.save`) | Serialised graph; distinct from pickle-based `torch.save` | Lower than pickle; still requires careful handling — some ops can escape |

The direction of travel is clear: **safetensors and GGUF are safe-by-design and should be preferred for weights** whenever the framework supports them; pickle-based formats are legacy and must be scanned aggressively when they cannot be avoided.

## The scanner suite

Two tools are the current open-source baseline. A mature programme runs both, in that order.

### Protect AI ModelScan

**ModelScan** (Protect AI, open source) scans model files across multiple formats — PyTorch (pickle-based), TensorFlow SavedModel, Keras H5, and others — looking for operators or serialised objects that indicate arbitrary-code-execution capability. It reports findings with severity levels and produces machine-readable output suitable for gating decisions. ModelScan is the *cross-format* scanner in the suite.

For the ingestion gate, ModelScan runs against every model artefact and emits a JSON report. Findings above a policy threshold are blocking; findings below are logged and reviewed.

### Hugging Face picklescan

**picklescan** (originally developed by Hugging Face) is a narrower tool focused specifically on the pickle format. It statically analyses pickle streams (without executing them) to enumerate the classes and callables the pickle would invoke on load, and flags any that are not on an allow-list of safe deserialisers. It catches pickle payloads that ModelScan might miss and gives you a fine-grained view of exactly what a pickle stream will do.

picklescan is the *deep pickle* scanner. For any pickle-based artefact, run both — ModelScan for cross-format coverage and picklescan for pickle-specific depth.

### Commercial and enterprise scanners

Commercial scanning platforms — HiddenLayer's AISec Platform, Protect AI's enterprise Guardian offering, and others — extend the open-source baseline with additional coverage: framework-specific heuristics, model-behaviour scanning, embedded-secret detection, and centralised reporting. In a mature programme, open-source scanners run in every pipeline as the free floor and a commercial scanner runs centrally in the model registry as an additional layer. The commercial choice is an above-your-level decision made by the security platform team.

<!-- needs-research: confirm the current stable feature scope and licensing model of HiddenLayer AISec Platform, Protect AI Guardian, and any other production-grade AI-model scanners this section should cite; keep the list to actively maintained products and cite their vendor documentation pages. -->

## The ingestion-gate policy

Combining chapter 05's signing gate with this chapter's scanning gate, the shippable ingestion-gate policy is:

| Order | Check | Effect on failure |
|---|---|---|
| 1 | Artefact format is on the allow-list | Reject if format outside the allow-list unless an exception is on file |
| 2 | If pickle-based, picklescan clean or findings ≤ block threshold | Reject if above threshold |
| 3 | ModelScan clean or findings ≤ block threshold | Reject if above threshold |
| 4 | Cosign signature present, valid, and from allow-listed identity | Reject if failing |
| 5 | Rekor entry present | Reject if failing |
| 6 | SLSA provenance record present and at or above organisation floor | Reject if failing |
| 7 | ML-BOM present, syntactically valid, and citing Data Cards for every dataset | Reject if failing |
| 8 | Required evaluations passed | Reject if failing |
| 9 | Validation-owner sign-off recorded | Reject if failing |

Order matters: cheap and destructive checks first (format allow-list), then scanning, then signing / provenance, then policy checks that assume the artefact is otherwise sound. The gate should fail *fast* on cheap checks so that operators do not wait five minutes for a signature check to time out on an artefact that was going to be rejected on format.

Exception handling — the escape valve for a legitimate finding that needs to be waived — is a controlled path: exception request, risk-council decision (mod-112), time-bounded waiver, monitor to confirm the waiver's assumptions still hold. Exceptions are logged and reviewed on cadence; no exception is permanent by default.

## The malicious-artefact detection lifecycle

The lifecycle wraps the scanner suite in a monitoring shape:

- **On every ingestion**, scanners run and produce reports; the reports are signed (chapter 05) and stored in the provenance folder.
- **On a periodic cadence**, scanners re-run against every artefact in the registry with the *current* scanner versions — new detection rules catch things that were missed at initial ingestion. Re-scan findings that flip a previously-passing artefact are treated as incidents (mod-111).
- **On scanner version bumps**, the re-scan cadence is triggered immediately for high-severity registries (production).
- **On new-format arrival** (a team wants to introduce an artefact of a format not currently on the allow-list), the format is evaluated: what scanner covers it, what code-execution surface it has, and whether it can be transformed to a safer format (e.g., converting a pickle to safetensors) before ingestion.

The re-scan discipline is important. A model that was clean when ingested six months ago against then-current scanners might not be clean against today's scanners — attackers develop new payloads, and detection rules follow.

## Where this module hands off — the level ladder

The three anchor roles adjacent to level 25 for this module:

### Level 15 — `ai-governance-analyst`

The analyst layer maintains the *inventory* of models, third parties, and datasets. Your ingestion gate consumes their inventory (to know what artefacts exist and are in scope) and produces an updated inventory as an output (to record what was ingested, blocked, or waived).

### Level 35 — `security-learning` / `ai-infra-security`

The runtime security platform team owns:

- the build system's SLSA level (chapter 04);
- the Sigstore / cosign / Fulcio deployment (public infrastructure or a private mirror);
- the scanner-tool deployment and version management;
- the OPA policy engine (or equivalent) that enforces the ingestion gate;
- the inference-time signature-verification wrapper;
- the incident response for signing-service or scanner outages.

Your role at level 25 is *policy consumer*: you specify the block thresholds, the identity allow-lists, the required checks, and the risk-register impact of degraded platform assurance. You do not run the signing service. When something is wrong with the platform — the signing service is down, the scanner has a known false-negative on a specific format, Rekor is unavailable — you escalate to level 35 and mark the register accordingly.

### Level 50 — `senior-ai-governance-architect`

The governance architect layer owns the *control-library architecture*: the taxonomy of controls, the mapping from framework requirement to control ID, the review cadence for the control library, the pattern for how controls are documented and versioned. Your ingestion gate is *one control assembly* out of many that plug into that library. When you find a gap in the library — a framework obligation that has no control in the library, a control in the library that has no implementation — you escalate to level 50.

The level 50 architect is who you talk to about *whether the ingestion gate should exist as one control or several*, *how it should be described in the library*, *how it composes with other controls in the library*, and *what evidence format the library expects*. They do not decide the block threshold for ModelScan on a Tuesday afternoon — you do that. But the taxonomy under which that decision is recorded is theirs.

### Level 40 — `agentic-safety-engineer` (adjacent)

For agentic systems with tool-use topology, the artefact set expands to include tool definitions, sub-agent orchestration configurations, and credential-propagation policies. Level 40 owns the frontier-agent depth of that supply chain; the interaction pattern is: this module's scanners cover the model artefacts; the agentic-safety-engineer's scope covers the *tool* artefacts and the *agent-graph* artefacts. Where they overlap, coordinate.

## The exception path — because reality intrudes

No policy is absolute. A team will show up with:

- a pickle-based checkpoint from a research collaborator that has no safetensors equivalent yet;
- a Model Card that is missing a field because the base-model publisher never populated it;
- a Data Card that is a draft because the corpus was licensed from a vendor whose questionnaire is not returned;
- a scanner finding on a false positive the tool is known to over-fire on.

Each case has an *exception process*, not a *default bypass*. The exception process:

1. Written request from the owner naming the check that would fail, why the failure is acceptable, and the compensating control.
2. Risk-council decision (mod-112) with time-bound waiver — waivers are dated and expire.
3. Register update recording the residual risk elevation and the compensating control.
4. Monitor confirming the compensating control is present and effective; monitor fires the incident-response playbook if the compensating control degrades.
5. Waiver review at expiration — either extended (with fresh justification) or the artefact must clear the gate cleanly by the deadline.

An organisation whose registry has a large exception backlog is one where the gate is misconfigured — either the thresholds are too tight for reality, the exception process is too easy, or the underlying artefact quality is too low. All three are addressable; ignoring them is not.

## Cross-references

- **mod-101** MEASURE 2.7 (security and resilience) closes when the ingestion gate is in place; MANAGE 4.1 (post-deployment monitoring) closes when the re-scan cadence and signature-failure monitors are in place.
- **mod-102** harm-model rows on "upstream data supplier" and "insider" personas are what the scanning-plus-signing gate closes.
- **mod-103** register rows for supply-chain compromise reduce residual score when the gate is credited; the control-defeated column names "signature verification bypassed" and "scanners silently disabled" as the top-two failure modes to monitor.
- **mod-108** guardrail catalogue lists the ingestion gate as a compound control with sub-rows per check.
- **mod-109** monitors watch for signature-verification failures, scanner-finding rates, and exception-request volume; the exception-request volume is a leading indicator of gate misconfiguration.
- **mod-111** incident-response playbook has a "signature or scan failure at ingestion" branch and a "malicious artefact detected during re-scan" branch.
- **mod-112** cross-functional programme owns the risk council that approves exceptions and reviews the exception backlog.
- Chapter 04 produced the provenance package; chapter 05 signed it; this chapter scans it.

## Summary

- Model file formats vary widely in code-execution surface; pickle-based formats are highest risk, safetensors and GGUF are safe-by-design and should be preferred where possible.
- The open-source scanning baseline is Protect AI ModelScan (cross-format) plus Hugging Face picklescan (pickle depth); commercial scanners layer additional detection above the baseline.
- The registry ingestion gate composes chapter 05's signing checks with this chapter's scanning checks, in a specific order, with each check blocking on failure and a controlled exception path for legitimate waivers.
- Re-scan cadence catches artefacts that pass at ingestion but that new scanner versions flag later.
- Level 50 (`senior-ai-governance-architect`) owns the control-library architecture; level 35 (`security-learning` / `ai-infra-security`) owns the runtime platform; you own the policy, the thresholds, and the register consequences.
- Exercise 06 wires ModelScan and picklescan into the registry ingestion gate and demonstrates a blocked malicious artefact.
