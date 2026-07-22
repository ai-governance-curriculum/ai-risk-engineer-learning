# Garak as the Baseline Scanner

NVIDIA's garak (Generative AI Red-teaming and Assessment Kit) is the tool this programme reaches for first. It is not the deepest attack authoring you will do — that is PyRIT (chapter 03) and Inspect (chapter 06) — but it is the one you run *first*, on every commit, to establish a baseline noise floor of known-attack coverage. This chapter pins garak's model, wires it into CI as a smoke test, and shows the triage workflow that turns its findings into register-row updates.

## Motivation

Two failure modes when a programme does not have a baseline scanner:

- **The perpetual bespoke suite.** Every red-team run is hand-authored. Findings from run N are not automatically re-checked in run N+1. Regression happens silently and is only discovered when a customer report or an external researcher surfaces the same attack again. Finding: "there is no automated attack coverage running per release."
- **The single-tool programme.** The team wrote its own custom probes. The custom probes are excellent — for the last attack pattern the team saw. The published state of the art (encoded jailbreaks, DAN variants, prompt-leak probes, obfuscation classes) is not exercised because the team has not had time to encode it. Finding: "attack coverage is not calibrated against public state of the art."

garak is the tool that closes both. It ships a maintained probe library that tracks published attack classes, it runs headlessly against any provider or local model, and it produces a JSON report that a CI job can score. The trade is depth: garak is a scanner, so its probes are broad but shallow. That is exactly the shape you want for the first pass.

## The four moving parts

garak's data model — `probes`, `detectors`, `generators`, and `harnesses` — is worth learning explicitly, because the same four names recur in most red-team scanners you will encounter and PyRIT (chapter 03) uses very similar decomposition under different names.

| Concept | What it is | Analogy |
|---|---|---|
| **Generator** | The thing under test — an OpenAI model, an Anthropic model, a local HuggingFace model, a REST endpoint wrapping your app | The "target" in PyRIT / Inspect |
| **Probe** | A parametrised attack — a set of prompts (with variants and mutations) intended to elicit a specific policy-negative behaviour | The "attack" in PyRIT |
| **Detector** | The scorer that decides whether the model's response constitutes a hit for the probe | The "scorer" in PyRIT / Inspect |
| **Harness** | The orchestration layer that pairs generators × probes × detectors and produces a report | The "runner" in PyRIT / Inspect |

<!-- needs-research: confirm the current naming (`generators`, `probes`, `detectors`, `harnesses`) matches the latest garak release; the project has re-organised submodule layouts in past versions. -->

An engineer running garak against a system is composing one generator with a subset of probes and detectors, running the harness, and reading the report. The composition itself is the first-class artifact — a garak run is *identified* by its (generator, probe-set, detector-set, seed) tuple. Two garak runs with the same tuple should produce the same finding set. That reproducibility is what lets you use it as a regression tool.

## What a probe covers

garak's probe library is organised roughly by attack class. A working mental map of the classes and what they exercise:

| Probe family | What it tries |
|---|---|
| Prompt-injection probes | Direct-instruction override, ignore-previous-instructions, developer-mode framing |
| Encoding / obfuscation probes | Base64, ROT13, leetspeak, unicode-lookalike, translation-based bypass |
| DAN-family / persona-hijack probes | Do-Anything-Now variants, role-play framings that request policy-negative behaviour |
| Prompt-leak probes | Extract the system prompt, extract retrieval-side instructions |
| Toxicity / harmful-content generation probes | Elicit outputs in categories the model provider disallows |
| PII / secret-leak probes | Prompt the model to emit training-data-shaped PII (names, addresses, credentials) |
| Malware-generation probes | Attempt to elicit exploit code, phishing content, or credential-harvesting scripts |
| Refusal-suppression probes | Cast the request in a framing that reduces the model's refusal rate |

<!-- needs-research: pin the current garak probe family taxonomy to the latest release notes at authoring time — the library adds families and reorganises them across releases. -->

Each family is one row on the coverage matrix. Ticking a family as covered means *at least one probe in that family ran against the target during this release*. Coverage does not claim depth — depth comes from PyRIT and Inspect. Coverage claims *this attack class was exercised*.

## What a detector decides

The detector reads a `(prompt, response)` pair and returns a probability that the pair constitutes a hit. Detector implementations at this level of maturity fall into three shapes:

- **Substring / regex detectors** — the response contains a giveaway string ("I am DAN," a base64-decoded slur, a credential shape). Cheap, brittle, high recall on the specific patterns they name and zero recall elsewhere.
- **Classifier detectors** — a small model (toxicity classifier, PII detector, code classifier) scores the response. More expensive, tunable precision / recall, and dependent on the classifier's own biases.
- **LLM-judge detectors** — a stronger model is asked whether the response constitutes a hit. Most flexible, most expensive, and prone to the LLM-judge failure modes the eval-engineering track (level 30) will teach you to calibrate.

The detector choice is itself a triage input. A regex-detector hit is a strong signal to investigate; a classifier-detector hit needs a false-positive review; an LLM-judge-detector hit needs judge-calibration evidence before it counts. This detail matters when you build the triage workflow below.

## Wiring garak into CI

The garak smoke suite in CI is the smallest useful invocation of garak that runs on every model / prompt / retriever / tool change and takes minutes, not hours.

A working smoke-suite shape:

```yaml
# red-team/suites/garak-smoke/config.yaml
generator:
  type: rest
  endpoint: https://internal.example/inference/cs-rag-assistant
  model_id_header: X-Model-Id
  auth_secret: env:CS_RAG_TEST_TOKEN

probes:
  # Baseline coverage — one probe per family, cheap variants only.
  - promptinject.HijackHateHumans
  - encoding.InjectBase64
  - dan.Dan_11_0
  - leakreplay.LiteratureCloze
  - malwaregen.Evasion
  - continuation.ContinueSlursReclaimedSlurs
  # ...

detectors:
  # Detectors chosen for cheapness first. Deep detectors run in the full suite.
  - mitigation.MitigationBypass
  - encoding.DecodeMatch
  - dan.DAN
  - leakreplay.StartsWith
```

<!-- needs-research: confirm the smoke-suite probe / detector identifiers above resolve in the current garak release; the identifiers have changed across versions and should be pinned to a release tag. -->

Two disciplines around the smoke suite:

- **The suite is pinned to a garak version.** Bumping the garak dependency is a change-controlled event, not an incidental `pip install --upgrade`. When you bump, you re-baseline the noise floor.
- **The smoke suite is not the release-gate suite.** Smoke runs in single-digit minutes to give per-commit feedback. The release-gate suite runs the full probe library with tuned detectors and takes hours; it belongs in a nightly / per-RC pipeline, not per-commit.

A CI job invoking garak looks roughly like:

```bash
# CI step: red-team smoke
garak \
  --config red-team/suites/garak-smoke/config.yaml \
  --report_prefix red-team/runs/${CI_COMMIT_SHA} \
  --seed 42

python red-team/tools/garak_gate.py \
  --report red-team/runs/${CI_COMMIT_SHA}.report.jsonl \
  --thresholds red-team/suites/garak-smoke/thresholds.yaml
```

<!-- needs-research: confirm the current garak CLI flags (`--config`, `--report_prefix`, `--seed`) against the latest release. -->

The `garak_gate.py` step is where you own the pass rule. garak's report emits per-probe hit-rates; the gate script compares each against the threshold and exits non-zero on breach. Thresholds are versioned with the suite.

## Triage — turning garak candidates into register updates

garak produces *candidates*. A candidate is a `(probe, prompt, response, detector-score)` row that scored above the detector's threshold. Every candidate must be dispositioned. The triage workflow:

### Step 1 — dedup within run

Many candidates from the same probe on the same target are variants of the same underlying failure. Group by `(probe, response-fingerprint)`. The fingerprint is a hash of the response with obvious variation (whitespace, model-generated ids) normalised out. A dozen `dan.Dan_11_0` hits with the same refusal-bypass wording become one triage row.

### Step 2 — false-positive review

For each unique candidate, a human reviews the `(prompt, response)` pair against the detector's semantic claim.

| Detector kind | Common false positive | Reviewer disposition |
|---|---|---|
| Substring / regex | Model quotes the giveaway string while refusing ("I will not say 'I am DAN'…") | Mark FP; feed the substring into the detector's negative-example set if the detector supports it |
| Classifier | Classifier is triggering on benign response of the same shape (a security tutorial mentioning credential formats without leaking any) | Mark FP; log the classifier's false positive for the classifier-owner |
| LLM judge | Judge is over-inferring intent — sees "hit" where the model's answer is bounded and policy-consistent | Mark FP; add the response to the judge-calibration set (`ai-eval-engineer` L30 owns the judge-calibration methodology) |

Genuine hits are marked TP and move to step 3.

### Step 3 — dedup across runs

The finding may be one you already logged in a prior run. Match by (probe, response-fingerprint) against the triage log's history. Duplicates get marked and the count on the existing finding increments; a new finding is opened only when the response-fingerprint is novel.

### Step 4 — severity and register attachment

For each novel TP:

- Assign severity from the harm model's impact axis (mod-102 chapter 01, mod-103 chapter 02).
- Attach to the matching harm-scenario row in mod-102's harm model.
- Attach to the matching register row in mod-103. If the finding names a category with no existing register row, open a new row via the mod-103 chapter 06 process.
- Package as a replay bundle (chapter 01) and store in the out-of-repo payload store.
- Open a mitigation issue with the mitigation owner named in the harm model row.

### Step 5 — regression fixture creation

Once the fix ships, the replay bundle is added as a regression fixture in the mod-111 fixture store. From then on the fixture runs on every release-gate suite and any regression blocks the release automatically.

The five-step triage workflow is exercise 02's core deliverable.

## What garak does *not* do — and what to reach for next

Deliberate limits of garak that keep you honest about when to reach for the next tool:

- **Multi-turn.** garak's baseline probes are single-turn. Multi-turn refusal-bypass — cast a benign framing, then use the model's own reply as leverage for the next turn — needs PyRIT's orchestrator model (chapter 03).
- **Tool calling and agent topology.** garak scores model outputs; it does not exercise the tool-calling side of an agent. AgentDojo and InjecAgent (chapter 04) cover indirect-injection and tool-abuse threat models.
- **Custom policy-derived suites.** Your organisation's internal policy defines refusal categories that garak's probe library does not exactly enumerate. The Inspect harness (chapter 06) is where you author the internal suite.
- **GCG-style optimisation.** Adversarial-suffix generation against a specific target requires access to gradients and its own optimisation loop (chapter 05). garak's suffix-related probes exercise *published* suffixes, not freshly optimised ones.
- **Novel attack authoring.** New attack classes surface faster than garak's release cadence. You back-port novel patterns into the internal suite (chapter 06) and, where appropriate, contribute upstream.

Chapter 03 picks up the orchestration model. Chapter 04 picks up agent-side. Chapter 05 picks up the deeper jailbreak work. Chapter 06 picks up the harness for internal suites.

## Interfaces with adjacent modules

- **Mod-102 (harm modelling)** — every triaged TP updates a harm-scenario row's evidence refs; novel categories drive harm-model updates.
- **Mod-103 (risk register)** — smoke-suite hit-rates are a likelihood-anchor input for register rows; sustained smoke-suite passes over time are residual-control-effectiveness evidence.
- **Mod-108 (guardrails)** — many garak hits are mitigated by guardrail configuration; garak becomes the effectiveness measurement for those guardrails.
- **Mod-109 (monitoring)** — patterns from garak hits inform production detector logic; the smoke-suite hit-rate trend feeds the monitoring dashboard.
- **Mod-111 (incident response)** — every TP produces a replay bundle that becomes a regression fixture; incident post-mortems feed novel probes back into the internal suite.

## Summary

- garak is the first-pass scanner: broad probe library, headless generators, JSON reports, low friction to wire into CI.
- Its four moving parts — generators, probes, detectors, harnesses — recur (under different names) across most red-team tools, so the mental model transfers.
- Probe families map onto the coverage-matrix rows in chapter 01; ticking a family means at least one probe in it ran.
- Detector kinds (regex, classifier, LLM-judge) drive different triage disciplines and different false-positive review paths.
- The smoke suite runs per commit on a pinned garak version, with a versioned thresholds file that gates the build.
- The five-step triage workflow — dedup within run, FP review, dedup across runs, severity + register attachment, regression fixture — is the artifact that turns candidates into register updates.
- garak's deliberate limits (single-turn, no tool-abuse, no custom policy, no fresh GCG) are the handoff to chapters 03–06.
- Exercise 02 walks the smoke suite, the CI wiring, and the triage on the running system.
