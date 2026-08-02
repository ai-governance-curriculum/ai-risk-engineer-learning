# Exercise 06 — ModelScan and picklescan Ingestion Gate with a Blocked Malicious Artefact

**Estimated effort:** 2 hours

## Objective

Extend the exercise-05 registry-ingestion gate with **Protect AI ModelScan** and **Hugging Face picklescan**, and demonstrate the gate *blocking* a deliberately crafted malicious artefact. The outcome is a full end-to-end gate — signing plus scanning — that a determined attacker (or a well-meaning engineer who copied a bad file) cannot bypass without an audited exception, plus a re-scan cadence that catches artefacts that pass at ingestion but that new detection rules later flag.

## Prerequisites

- Chapter 06 read in full. The pickle-and-format threat surface, the scanner suite (ModelScan + picklescan + commercial context), the ordered ingestion-gate policy, the re-scan lifecycle, the exception process, and the level ladder to L35 / L50 / L40 are the reference shapes.
- Exercise 05 completed. The gate policy file, the identity allow-list, the passing and failing demonstrations, and the runtime failure playbook stub are the base this exercise extends. Do not rewrite exercise 05's gate — extend it in place.
- ModelScan installed (`modelscan --version` runs) and picklescan installed (`picklescan --version` or the Python-module equivalent runs). The commercial-scanner slot may remain a placeholder — that is an L35 platform-team decision, not this exercise's scope.
- A safe environment for producing a deliberately-malicious pickle. Use an isolated container or VM with no network access to production credentials. The malicious payload for this exercise is a *loud, obvious* pickle whose `__reduce__` writes a file to `/tmp/`; do not craft anything actually harmful.
- A copy of the exercise-04 provenance package and a "sandbox" copy you can mutate to produce the malicious variant. Do not tamper with the real provenance package.

## Problem statement

Wire ModelScan and picklescan into the existing exercise-05 gate. Then produce a deliberately-malicious artefact (a pickle whose deserialisation would execute code) and demonstrate that the gate blocks it at multiple checks. Finally, define the re-scan cadence and the exception process for legitimate waivers.

### The scanner wiring

The ingestion-gate policy from exercise 05 grows two new blocking checks:

- **picklescan clean or findings ≤ block threshold** — runs on any pickle-based artefact (`*.pkl`, `*.pickle`, `*.pt`, `*.pth`, `*.bin` produced by `torch.save`, `.joblib`, `.npy` / `.npz` if `allow_pickle=True` is a possibility). Findings above the block threshold reject the artefact.
- **ModelScan clean or findings ≤ block threshold** — runs on every model artefact regardless of format (PyTorch pickle, TensorFlow SavedModel, Keras H5, ONNX with custom operators, joblib). Findings above the block threshold reject the artefact.

Both scanners run *before* the signature checks (chapter 06 §"The ingestion-gate policy" — cheap and destructive checks first; scanning is destructive-enough because the alternative is running unverified code). The full ordered policy is now:

1. Artefact format is on the allow-list — reject if not.
2. If pickle-based, picklescan clean or ≤ block threshold — reject if above.
3. ModelScan clean or ≤ block threshold — reject if above.
4. Cosign signature present, valid, and from allow-listed identity — reject if failing.
5. Rekor entry present — reject if failing.
6. SLSA provenance record present and at or above organisation floor — reject if failing.
7. ML-BOM present, syntactically valid, and citing Data Cards for every dataset — reject if failing.
8. Required evaluations passed — reject if failing.
9. Validation-owner sign-off recorded — reject if failing.

### The malicious artefact

Craft a small pickle in a sandbox whose `__reduce__` performs an obvious, safe side-effect (e.g., writes `pwned` to `/tmp/modelscan-drill.txt`). Save it as `weights.pt` inside a copy of the provenance package. Do not put the file anywhere near a production system, and do not use a payload with any real damage potential.

Confirm the payload fires by loading it (inside a disposable container) and observing the side-effect file. Then confirm that both picklescan and ModelScan flag the file with high-severity findings when run against it. Only after those two confirmations, feed the tampered package into the gate.

### The re-scan cadence

The scanning lifecycle (chapter 06 §"The malicious-artefact detection lifecycle") requires re-scanning every artefact in the registry on a periodic cadence with current scanner versions. Define this cadence, name the trigger conditions (scheduled + scanner-version-bump), and specify the incident response when a re-scan flips a previously-passing artefact.

### The exception process

Chapter 06 §"The exception path" describes a controlled exception process for legitimate waivers. Define the exception process's shape as a file that a team can fill when they have a pickle-based checkpoint from a research collaborator with no safetensors equivalent.

## Requirements

Produce all seven items:

1. **The extended gate policy file** — the exercise-05 gate policy is now updated to include the two scanner checks in the ordered position above (checks 2 and 3). The file's version bumps and the change is logged.
2. **The scanner reports for the clean artefact** — running picklescan and ModelScan against the (safetensors-based, per exercise 04) clean provenance package produces reports with zero high-severity findings. Store them as `attestations/scan-report-picklescan.json` and `attestations/scan-report-modelscan.json` in the provenance folder. Sign them (per exercise 05's process). Update the signing manifest.
3. **The malicious artefact and its confirmation** — the sandbox-crafted malicious pickle exists at `drills/malicious-artefact/weights.pt` (outside the real provenance folder), with a `README.md` describing what the payload does, how the safety of the payload was ensured (isolated container, no network, benign side-effect), and the output of running it in the sandbox. Do not commit the payload to a shared branch without the container isolation stated on the file itself.
4. **The blocked-artefact demonstration** — a recorded run of the extended gate against the malicious package that reports "fail" on at least the two scanner checks (picklescan and ModelScan), with clear diagnostics. Record the output as `gates/demo-scanner-block.log`. Cross-check that the ordering is correct — the gate should fail on picklescan / ModelScan *before* the signature checks are even attempted.
5. **The re-scan cadence spec** (`gates/rescan-cadence.md`) — names the cadence (e.g., weekly for production registries, daily for high-severity tiers), the scanner-version-bump trigger, and the incident-response branch when a re-scan flips a previously-passing artefact. Cross-references the mod-111 incident-response playbook.
6. **The exception process document** (`gates/exception-process.md`) — the five-step exception process from chapter 06 §"The exception path": written request from owner, risk-council decision, register update, compensating monitor, waiver review at expiration. Includes the template a requester fills.
7. **The updated runtime-failure playbook** (`playbooks/malicious-artefact-detection.md` — new branch, or a section added to the exercise-05 playbook) covering the two chapter-06 branches: signature or scan failure at ingestion (block; alert on-call; investigate) and malicious artefact detected during re-scan (isolate; treat as integrity incident; escalate to L35 and to the risk council).

## Starter guidance

- **Do not use a real malicious payload.** A `__reduce__` that writes `pwned` to `/tmp/modelscan-drill.txt` is enough. Never use a payload that touches production credentials, network endpoints, or persistent storage. If your organisation has an offensive-security team, they will happily supply a safer test corpus than you can craft from a first-time drill.
- **Confirm the payload fires before you scan.** If you cannot demonstrate that the pickle's `__reduce__` actually executes on load in an isolated container, the scanners have nothing to catch and the exercise is trivial. The demonstration flow is: (a) craft, (b) execute in sandbox and observe the side-effect file, (c) run scanners and confirm they flag it, (d) feed into gate and confirm it blocks. Any step that fails is a real finding — fix it before the next step.
- **Order matters, and the gate must actually enforce order.** Chapter 06 is emphatic: cheap and destructive checks first. If your gate implementation runs all checks in parallel and reports the union, that is fine as long as the *first-failing* check by policy ordering is prominent in the diagnostics. If it is not, refactor.
- **safetensors is the format the exercise-04 provenance package should already prefer.** The clean provenance package should have `weights.safetensors`, not `weights.pt`. If exercise 04 shipped a pickle-based weights file, this exercise is where you flag that as a finding to fix — either re-export the weights as safetensors or accept the increased scanning burden per artefact and note it in the register.
- **picklescan and ModelScan are complementary, not redundant.** Run both. picklescan gives you depth on pickle-specific payloads (the exact classes and callables the pickle would invoke); ModelScan gives you breadth across formats. Their outputs together are what a triage engineer reads.
- **Re-scan cadence is where the "clean at ingestion" assumption ages.** A model that passed six months ago against then-current scanners might fail today's scanner version. Without re-scan, that risk sits silently in the registry. The re-scan cadence is small operationally and large in value.
- **The exception process is not a bypass.** A vendor with an unavoidable pickle checkpoint gets a waiver only after the risk council names the compensating control, the register is updated, and the monitor is in place. Waivers expire and require re-justification. A registry that accumulates permanent exceptions is misconfigured.
- **Scanner findings below the block threshold are not silent.** Findings below the threshold are logged and reviewed at cadence; the reviewer decides whether to raise or lower the threshold or to file a bug against the scanner. Silent below-threshold findings are how false-negative rates creep up.
- **This exercise does not deploy commercial scanners.** Commercial coverage (HiddenLayer AISec, Protect AI Guardian, others) is an L35 decision. Reserve the slot in the gate policy for a `commercial_scanner` check that returns `not_enforced` today; when L35 wires it, the gate file changes and the register is updated.

## Acceptance criteria

The submission is accepted when:

- The gate policy is extended to include picklescan and ModelScan as blocking checks in positions 2 and 3, and its version is bumped.
- Scanner reports for the clean provenance package exist in the `attestations/` folder, are signed, and are recorded in the signing manifest.
- The malicious artefact exists in a labelled `drills/` folder with a `README.md` that documents the payload, the isolation used, and the confirmed side-effect.
- The blocked-artefact demonstration log shows the gate failing on at least the two scanner checks with diagnostics; the ordering (scanners before signature checks) is visible in the log.
- The re-scan cadence spec is present, names cadence and trigger conditions, and cross-references the mod-111 incident-response playbook.
- The exception process document is present with the five-step shape and a fill-in template.
- The updated playbook covers the two chapter-06 branches with on-call roles and escalation SLAs.
- The exercise-01 MRM packet's implementation-testing section and the exercise-04 provenance package's `README.md` how-to-verify block are updated to reference the extended gate.

## Stretch goals

- **Second malicious variant** — craft a second variant using a *non-pickle* code-execution surface (a Keras H5 with a Lambda layer, a TensorFlow SavedModel with a `tf.py_function` op, an ONNX model with a custom operator). Confirm ModelScan flags it. This exercises the "format-swap does not eliminate the surface" point from chapter 06.
- **Silent-regression drill** — take an artefact that passes today, wait for a ModelScan version bump (or manually update the scanner's rule set), and re-run the scan. Record any newly-flagged findings. If none show up, the exercise is complete; if any do, follow the incident-response branch as if in production.
- **Cross-track review with L40** — for the agentic-system slice of your work, hand the extended gate and playbook to a peer working through the `agentic-safety-engineer` (level 40) material and ask whether they would consume it as-is for tool artefacts and agent-graph artefacts. Their disagreement is signal on where the scanner scope needs to expand.
- **L35 hand-off memo** — write a one-page memo to the L35 platform team naming (a) what your gate does today, (b) what L35 would need to change to run the gate at organisation scale (centralised scanner deployment, OPA policy engine, scanner-version tracking, exception-backlog dashboard), and (c) which parts of the gate you will keep owning as risk engineering (thresholds, identity allow-list, exception-process shape). The memo is the boundary document between L25 and L35 for this control.
- **Exception-backlog audit** — for any registry that already has scanner exceptions in place, count them by age and severity and produce a one-slide summary. A backlog trending up is a gate-misconfiguration signal.
- **Register update** — in the mod-103 register, on every row whose residual is credited to scanning controls, add an `evidence_pointer` to the scanner-report UUID / hash and the scanner version at scan time. Bump the row's `staleness` field when a scanner-version bump happens without a re-scan.

## Deliverable format

A single markdown document `mod-110-ex06-scanner-gate.md` that indexes the artefacts. The index contains:

1. *Package and scanner summary* — one paragraph naming the provenance package extended, the two scanners' versions used, and the malicious artefact type crafted.
2. *Gate policy diff* — the two new check rows plus their ordering position, quoted from the updated policy file.
3. *Clean-scan report summary* — one line per scanner: findings count, highest severity, block threshold, decision.
4. *Malicious-artefact README excerpt* — payload description, isolation, confirmed side-effect.
5. *Blocked demonstration* — the demo-scanner-block log summarised, one line per failing check.
6. *Re-scan cadence* — the cadence, trigger conditions, and incident-response branch summarised.
7. *Exception process* — the five-step shape summarised, with the template header quoted.
8. *Updated playbook* — the two-branch structure summarised.
9. *Cross-references* — pointers to the exercise-01 MRM packet's implementation-testing section, the exercise-04 provenance package, the exercise-05 gate policy the extension modifies, and the L35 platform contact.
10. *Open items* — commercial-scanner slot, format-swap coverage gaps, exception-backlog concerns, anything blocked on the L35 platform.

## Submission

Attach the index markdown plus the updated `gates/` folder, the updated `playbooks/` folder, the `drills/malicious-artefact/` folder, and the signed scanner reports inside the provenance package. Confirm in the index that the malicious artefact was crafted and executed only in isolation, that no production system was involved, and that the payload's side-effect was benign. The extended gate is the module's terminal artefact: it composes chapters 04, 05, and 06 into the single ingestion boundary that turns "the model is signed and scanned" from a claim into a runtime property.
