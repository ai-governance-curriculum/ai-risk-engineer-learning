# Lab 03 — Ingestion-Gate Policy Authoring Drill (Nine Ordered Checks + Diagnostics + Exception Template)

**Duration:** 45 minutes · **Format:** guided practice, markdown + policy-pseudocode authoring on a fixed fabricated LLM application

## Objective

Internalise the chapter 05-06 **registry-ingestion-gate policy** — the nine ordered checks (format allow-list → picklescan → ModelScan → cosign signature → Rekor entry → SLSA provenance → ML-BOM / Data Cards → evaluations passed → validation-owner sign-off), the identity allow-list, the pass / fail diagnostic shape, and the five-step exception template — by authoring the gate policy for a fixed fabricated running system (`HelpDesk`). The lab drills the shape so that when **exercise 05** wires real cosign + Fulcio + Rekor and **exercise 06** adds ModelScan + picklescan, the engineering budget goes into the tool wiring and the real block-threshold decisions — not into inventing the policy structure, the ordering, the fail-closed defaults, or the exception vocabulary.

The chapter-05/06 anti-patterns the shape forecloses: the **"unsigned weights loaded in prod"** default (publish-side signing only; inference-side verification missing); the **"trust any Sigstore cert"** verifier; the **"shared long-lived signing key"**; the **"scanners after signatures"** ordering that wastes operator time; the **"permanent exception"** waiver that never expires; and the **"fail-open on signing-service outage"** default that is the same as no gate.

## Prerequisites

- Chapter 05 read — specifically "What you sign, at minimum" (weights, Model Card, ML-BOM, SLSA, evaluation reports, scan reports — signer is the pipeline via OIDC, not a human), "The Rekor transparency log" (non-repudiation + non-falsifiability of signing timeline), "The registry-ingestion gate" (the 11-row policy table), "The 'unsigned weights loaded in prod' anti-pattern" (the five collapse-to-anti-pattern defaults), "SLSA level uplift as the registry matures" (stages 0 / 1 / 2 / 3), and "Signature verification failure — the runtime story" (the three cases the playbook branches on).
- Chapter 06 read — specifically "The pickle-and-format threat surface" (the format risk table), "The scanner suite" (ModelScan + picklescan as the open-source baseline), "The ingestion-gate policy" (the ordered 9-check composition with scanners *before* signature checks — cheap and destructive first; exception handling as a controlled path), and "The exception path — because reality intrudes" (the five-step exception process).
- Lab 02 completed — the provenance folder `provenance/helpdesk-finetune/v1.3.0/` exists with the Data Card, Model Card, ML-BOM, SLSA placeholder. The gate policy's file-path references assume that folder layout.
- A terminal where you can create a working directory and author markdown files. No external access, no cosign, no ModelScan install required — this lab drills the policy *shape* (what the policy file says; what the diagnostics shape looks like; what the exception template looks like). The real tool wiring is exercise 05 (cosign) and exercise 06 (ModelScan / picklescan).

## The fixed example system

Use this system throughout the lab — do *not* substitute your own. Continuing the `HelpDesk` running example from the mod-109 labs, this module's lab 01 (MRM packet), and this module's lab 02 (provenance package).

> **`HelpDesk`** — customer-support RAG assistant; English-language B2B SaaS customer base; in-house fine-tune `helpdesk-finetune` v1.3.0 on base `FoundationCo-LLM-3.5`; safetensors weights + safetensors LoRA adapter; internal model registry is an OCI-backed registry at `oci://internal-registry.example/helpdesk-finetune/v1.3.0`; signing is enforced by a Kubernetes admission controller (Kyverno) *at the OCI registry* and additionally by an inference-side pre-load wrapper (`inference/verify_on_load.py` reserved for exercise 05).
>
> **Signing identity model:** keyless via Sigstore; the OIDC issuer is the internal GitLab CI identity provider; the training-pipeline identity is `gitlab-oidc://internal.example/groups/helpdesk/pipelines/train.yaml@refs/tags/v1.3.0`.
>
> **SLSA floor:** `v1.0 Build L2` as the organisation-wide floor for production models (chapter 05 §"SLSA level uplift as the registry matures" stage 2 — hosted-signer enforced under organisation OIDC; chapter 04 is where the floor is pinned).

### Fixed artefact paths (do not revise)

The gate policy references these paths. They match lab 02's output folder layout.

| Artefact | Path |
|---|---|
| Weights | `provenance/helpdesk-finetune/v1.3.0/weights.safetensors` |
| LoRA adapter | `provenance/helpdesk-finetune/v1.3.0/adapter.safetensors` |
| Model Card | `provenance/helpdesk-finetune/v1.3.0/model-card.md` |
| ML-BOM | `provenance/helpdesk-finetune/v1.3.0/mlbom.cyclonedx.json` |
| SLSA | `provenance/helpdesk-finetune/v1.3.0/slsa-provenance.json` |
| Evaluation report | `provenance/helpdesk-finetune/v1.3.0/evaluation/eval-report.md` |
| picklescan report (clean) | `provenance/helpdesk-finetune/v1.3.0/attestations/scan-report-picklescan.json` |
| ModelScan report (clean) | `provenance/helpdesk-finetune/v1.3.0/attestations/scan-report-modelscan.json` |
| Signing manifest | `provenance/helpdesk-finetune/v1.3.0/attestations/signing-manifest.json` |
| Data Cards folder | `provenance/helpdesk-finetune/v1.3.0/datasets/` |

### Fixed identity allow-list (do not revise)

Three identities are on the production allow-list for signing `helpdesk-finetune`. One is for human exception-signing, time-bound.

| Identity | OIDC issuer | Claim shape | Purpose |
|---|---|---|---|
| `gitlab-oidc://internal.example/groups/helpdesk/pipelines/train.yaml@refs/tags/*` | internal GitLab | repo + pipeline + tag | Training pipeline (keyless) |
| `gitlab-oidc://internal.example/groups/helpdesk/pipelines/validate.yaml@refs/tags/*` | internal GitLab | repo + pipeline + tag | Validation pipeline (keyless) — signs validator memo + eval reports; the mod-104/105/106/107 evidence files come from this identity |
| `oidc://internal.example/users/L35-eval-eng-alex@human-exception-until-2027-01-15` | internal OIDC | user + expiry claim | Time-bound human exception-signer; expires 2027-01-15; waiver approved by mod-112 risk council |

## Steps

### Step 1 — Create your working directory and the gate-policy scaffold

```bash
mkdir -p mod110-lab03/gates mod110-lab03/playbooks
cd mod110-lab03
touch gates/registry-ingestion-policy.md \
      gates/identity-allowlist.md \
      gates/demo-pass.log \
      gates/demo-fail.log \
      gates/exception-template.md \
      playbooks/signature-and-scan-failure.md
```

The gate policy is authored as a markdown spec with pseudocode blocks for each check — the real implementation in exercise 05/06 is OPA/Kyverno/CI-script per the team's stack choice. The policy's semantics and ordering are what the lab drills.

### Step 2 — Fill the identity allow-list (`gates/identity-allowlist.md`)

Chapter 05 §"The 'unsigned weights loaded in prod' anti-pattern" names "the verifier trusts *any* signature, not signatures from the expected identity" as one of the five anti-pattern defaults. The identity allow-list is the file that forecloses it; the gate's check-4 reads this list.

```markdown
# Identity Allow-List — helpdesk-finetune production

Chapter 05 §"What you sign, at minimum" rule: signer is the pipeline (OIDC), not a human. Human exception-signing is time-bound and recorded here.

| # | Identity | OIDC issuer | Claim shape | Purpose | Valid until |
|---|---|---|---|---|---|
| 1 | `gitlab-oidc://internal.example/groups/helpdesk/pipelines/train.yaml@refs/tags/*` | internal GitLab | repo + pipeline + tag | Training pipeline (keyless) | no expiry — pipeline identity |
| 2 | `gitlab-oidc://internal.example/groups/helpdesk/pipelines/validate.yaml@refs/tags/*` | internal GitLab | repo + pipeline + tag | Validation pipeline (keyless); signs validator memo + eval reports | no expiry — pipeline identity |
| 3 | `oidc://internal.example/users/L35-eval-eng-alex@human-exception-until-2027-01-15` | internal OIDC | user + expiry claim | Time-bound human exception-signer for the research-collaborator-checkpoint waiver filed 2026-10-30 | 2027-01-15 |

## Rules

- A signature from an identity **not on this list** is a rejected signature per gate check 4; the diagnostic cites the mismatched identity.
- Every human identity is time-bound; the `Valid until` column must carry a specific date. A row without an expiry is permitted only for pipeline identities.
- Updates to this list are a mod-112 risk-council decision; the change-log at the bottom records the approving minutes.
- Chapter 05 §"The 'unsigned weights loaded in prod' anti-pattern" explicitly rules out "trust-any-Sigstore-cert" as the identity model.

## Change log

- 2026-10-18 — initial allow-list for helpdesk-finetune v1.3.0 production (pipeline identities 1 + 2); approved by mod-112 minutes 2026-Q3.
- 2026-10-30 — temporary human identity 3 added for a 90-day research-collaborator exception; approved by mod-112 minutes 2026-10-30 extraordinary session; expires 2027-01-15.
```

Reading check: three identities; two pipeline (no expiry) and one human (specific expiry date); every row has issuer + claim shape; the rules section names the chapter-05 anti-pattern the list forecloses; the change log attributes each update to a mod-112 risk-council decision.

### Step 3 — Author the gate policy (`gates/registry-ingestion-policy.md`)

Chapter 06 §"The ingestion-gate policy" names the 9-row ordered policy, with scanners *before* signature checks and signatures before the heavier policy checks. Chapter 05 §"The registry-ingestion gate" lists the signing-related sub-checks. Fill the policy file:

```markdown
# Registry-Ingestion Gate Policy — helpdesk-finetune

Policy version: v0.1.0 (lab 03 initial authoring)
Scope: artefacts under the OCI registry prefix `oci://internal-registry.example/helpdesk-finetune/`
Enforcement points: Kyverno admission policy at the OCI registry (primary); `inference/verify_on_load.py` pre-load wrapper (secondary; exercise-05 wiring); CI publish-step check (tertiary; defence in depth).
Fail-closed default: on any exception, platform outage, or unresolvable list — **do not admit**.
Allow-list file: `gates/identity-allowlist.md` (version-pinned in this policy below).

## Ordered checks

Order per chapter 06 §"The ingestion-gate policy": **cheap and destructive checks first**; the gate fails fast so operators do not wait five minutes for a signature check to time out on an artefact that was going to be rejected on format. Every failure is blocking; exceptions follow the `gates/exception-template.md` five-step process.

### Check 1 — Artefact format is on the allow-list

- Allow-list of formats: `safetensors`, `gguf`, `onnx` (no custom operators), `safetensors+adapter.safetensors` (for LoRA composites).
- Deny-list (reject outright unless on a chapter-06 §"Exception path" waiver): `pt`, `pth`, `bin` produced by `torch.save`, `pkl`, `pickle`, `joblib`, `npy`/`npz` with `allow_pickle`, Keras `.h5` with Lambda layers.
- Diagnostic shape on fail: `check=1 format decision=reject detected_format=<ext> expected={allow-list} reason="format not on allow-list; see chapter 06 §'The pickle-and-format threat surface'"`.

### Check 2 — picklescan clean (runs iff a pickle-based artefact is present)

- Trigger: any artefact detected in Check 1 as pickle-based (relevant for waiver cases only; the clean allow-list avoids pickle entirely).
- Tool: `picklescan` (Hugging Face); run with the organisation's safe-deserialiser allow-list.
- Block threshold: any high-severity finding → reject; medium-severity findings are logged and reviewed on cadence; the review is a mod-112 action not an engineer action.
- Diagnostic shape: `check=2 picklescan decision=<pass|reject> findings=<list of (class, callable, severity)>`.

### Check 3 — ModelScan clean

- Trigger: every ML-model component in the ML-BOM (chapter 06 — format-swap does not eliminate the surface; scan regardless of format).
- Tool: `modelscan` (Protect AI, open-source baseline).
- Block threshold: any high-severity finding → reject; medium-severity → logged and reviewed; low-severity → logged only.
- Diagnostic shape: `check=3 modelscan decision=<pass|reject> findings=<list of (operator, severity, component)>`.

### Check 4 — Cosign signature present, valid, from allow-listed identity

- Required for every file in chapter 05's minimum-signing-surface: `weights.safetensors`, `adapter.safetensors`, `model-card.md`, `mlbom.cyclonedx.json`, `slsa-provenance.json`, `evaluation/eval-report.md`, `attestations/scan-report-picklescan.json`, `attestations/scan-report-modelscan.json`, `attestations/signing-manifest.json`.
- Allow-list source: `gates/identity-allowlist.md` pinned at git commit `<lab03-commit>` — the gate verifies against the list as it exists in the pinned commit, not whatever is currently in the repo.
- Diagnostic shape on fail: `check=4 cosign decision=reject file=<path> reason=<not-present | invalid | identity-not-on-allowlist> observed_identity=<identity-claim>`.
- Anti-pattern foreclosures (chapter 05): no trust-any-Sigstore-cert; no long-lived shared signing key (keyless only); human-signer identities must carry an expiry claim.

### Check 5 — Rekor entry present

- Every cosign signature verified in Check 4 must have a corresponding Rekor entry.
- Public Rekor by default; a private Rekor is permitted per the organisation's current posture memo (chapter 05 §"The Rekor transparency log" tradeoff decision — not a lab-03 call).
- Diagnostic shape on fail: `check=5 rekor decision=reject file=<path> rekor_entry=<not-found | mismatched-hash> expected_sha256=<hash>`.

### Check 6 — SLSA provenance record present and at or above organisation floor

- Floor: **v1.0 Build L2** for production models (hosted-signer under organisation OIDC; chapter 05 §"SLSA level uplift" stage 2).
- Diagnostic shape on fail: `check=6 slsa decision=reject record=<not-present | malformed | below-floor> observed_level=<observed> expected_floor=v1.0 Build L2`.

### Check 7 — ML-BOM valid; Data Cards present for every dataset cited

- ML-BOM parses as valid CycloneDX 1.6; every `type=data` component has an `externalReferences` pointer to a `.datacard.yaml` file that exists; Model Card cross-references the same identifiers.
- Diagnostic shape on fail: `check=7 mlbom decision=reject reason=<parse-error | missing-datacard | mismatched-reference> details=<path>`.

### Check 8 — Required evaluations passed

- The mod-101 sub-category to eval-report mapping must show pass on every required evaluation (fairness per `tenant_tier`; adversarial robustness; PII recall on the `pii-egress-detector` at ingress; red-team suite above pass threshold).
- Diagnostic shape on fail: `check=8 evaluations decision=reject failing_evaluations=<list>`.

### Check 9 — Validation-owner sign-off recorded

- A signed validator-memo from the validation-pipeline identity (`gitlab-oidc://.../pipelines/validate.yaml@refs/tags/*`) is present at `07-independent-validation-memo.md` *and* the memo's disposition line is `approve` or `conditional approve` with gap-closures named.
- Diagnostic shape on fail: `check=9 validator decision=reject reason=<missing-memo | missing-signature | disposition-reject>`.

## Diagnostics format (overall)

On a passing artefact, the gate emits one line per check (`check=<n> <name> decision=pass`). On a failing artefact, the gate emits all failing checks with their diagnostic shapes; the primary diagnostic is the first-failing check by ordering (chapter 06 — "the *first-failing* check by policy ordering is prominent in the diagnostics"). The diagnostic lines are structured so that `grep '^check=<n>'` on the log identifies the failing check cheaply.

## Fail-closed default

If any check cannot evaluate — signing service unreachable; Rekor down; allow-list unresolvable; scanner service outage; SLSA parser error — the gate decision is `reject`. Chapter 05 §"The runtime story" rule: "fail-open ('we could not verify but let it through') is the same as 'no gate at all' for a determined attacker." The incident response for a gate-side outage is at `playbooks/signature-and-scan-failure.md` branch 3.

## Exception handling

Any failing check that a team argues should not block is a candidate for the chapter-06 exception path. See `gates/exception-template.md`. Exceptions are **time-bound**, approved by **mod-112 risk council**, backed by a **compensating control** named on a register row, and **monitored** by a mod-109 signal-contract row.

## Change log

- v0.1.0 (lab 03, 2026-11-24) — initial authoring. Nine ordered checks with scanner → signature → policy ordering per chapter 06 §"The ingestion-gate policy". Scanner-related checks (2 + 3) are placeholders for exercise 06 wiring; signature-related checks (4 + 5) are placeholders for exercise 05 wiring. Identity allow-list pinned at the current commit.
```

Reading check: nine ordered checks numbered 1-9; every check has a diagnostic shape; scanners (2, 3) come before signatures (4, 5) per chapter 06; the SLSA floor (`v1.0 Build L2`) is pinned; fail-closed is explicit on the default; the exception process is named and its file is cross-referenced; the change log attributes the authoring and names the exercises that wire the real tools.

### Step 4 — Author the pass-run log (`gates/demo-pass.log`)

Chapter 06 §"The ingestion-gate policy" says a passing gate run emits one line per check. Fabricate a plausible pass log against the clean lab-02 provenance folder:

```
2026-11-24T10:14:02Z gate=registry-ingestion-policy version=v0.1.0 artefact=oci://internal-registry.example/helpdesk-finetune/v1.3.0 run_id=gate-20261124-101402
2026-11-24T10:14:02Z check=1 format decision=pass detected_formats={safetensors, safetensors+adapter.safetensors}
2026-11-24T10:14:02Z check=2 picklescan decision=pass note=no pickle-based artefact present; check skipped per trigger rule
2026-11-24T10:14:11Z check=3 modelscan decision=pass findings=[] scanner_version=modelscan-0.8.1
2026-11-24T10:14:18Z check=4 cosign decision=pass files_verified=9 identities={gitlab-oidc://internal.example/groups/helpdesk/pipelines/train.yaml@refs/tags/v1.3.0, gitlab-oidc://internal.example/groups/helpdesk/pipelines/validate.yaml@refs/tags/v1.3.0}
2026-11-24T10:14:19Z check=5 rekor decision=pass rekor_entries=9 log=public-rekor
2026-11-24T10:14:19Z check=6 slsa decision=pass observed_level=v1.0 Build L2 floor=v1.0 Build L2
2026-11-24T10:14:20Z check=7 mlbom decision=pass components=7 datasets_with_datacards=2 cyclonedx_spec_version=1.6
2026-11-24T10:14:21Z check=8 evaluations decision=pass required=[fairness-tenant-tier, adversarial-robustness, pii-recall, red-team-suite] failing=[]
2026-11-24T10:14:21Z check=9 validator decision=pass memo=07-independent-validation-memo.md signer=L35-eval-eng-alex disposition=conditional-approve
2026-11-24T10:14:21Z gate decision=pass all_checks_passed=9 total_duration_ms=19200
```

Reading check: timestamps progress forward; every check emits one line; the final `gate decision=pass` summarises the result; durations reflect scanner-before-signature ordering (ModelScan at 9 seconds dominates the budget; cosign checks run fast after that).

### Step 5 — Author the fail-run log (`gates/demo-fail.log`)

Chapter 06 §"The ingestion-gate policy" and the exercise-05 failing-demonstration rubric name a fail log that shows the **first-failing** check prominently with distinct diagnostics. Fabricate a fail log where a well-intended engineer dropped a `.pt` file (pickle-based, not on the allow-list) into the provenance folder and an attacker replaced the ML-BOM with a tampered copy.

```
2026-11-24T10:22:07Z gate=registry-ingestion-policy version=v0.1.0 artefact=oci://internal-registry.example/helpdesk-finetune/v1.3.0-candidate run_id=gate-20261124-102207
2026-11-24T10:22:07Z check=1 format decision=reject detected_formats={safetensors, pt} unexpected_formats={pt} reason="pt detected; .pt (torch.save pickle) is not on the allow-list; see chapter 06 §'The pickle-and-format threat surface'"
2026-11-24T10:22:07Z note="gate ordering: Check 1 failed; downstream checks continue only to surface additional findings — the first-failing check by ordering is the primary diagnostic and would be sufficient to block admission"
2026-11-24T10:22:07Z check=2 picklescan decision=reject file=weights.pt findings=[{class=os.system, callable=rm -rf /tmp/pwned, severity=high}] block_threshold=high reason="pickle __reduce__ enumerates os.system call; high severity finding"
2026-11-24T10:22:16Z check=3 modelscan decision=pass findings=[] note="ModelScan does not flag this particular payload shape; picklescan caught it — chapter 06 rule: run both; complementary coverage"
2026-11-24T10:22:16Z check=4 cosign decision=reject file=weights.pt reason=not-present observed_identity=<none> note="no signature on tampered file; chapter 05 §'The unsigned weights loaded in prod' anti-pattern — first signature failure"
2026-11-24T10:22:17Z check=4 cosign decision=reject file=mlbom.cyclonedx.json reason=invalid observed_identity=gitlab-oidc://internal.example/groups/helpdesk/pipelines/train.yaml@refs/tags/v1.3.0 reason_detail="signature does not verify against file hash; file has been modified since signing"
2026-11-24T10:22:17Z check=5 rekor decision=reject file=weights.pt rekor_entry=not-found
2026-11-24T10:22:17Z check=6 slsa decision=pass observed_level=v1.0 Build L2 floor=v1.0 Build L2 note="SLSA record still parses; the tampered state is caught at check 4/5, not here"
2026-11-24T10:22:17Z check=7 mlbom decision=pass components=7 note="tampered ML-BOM still parses as valid CycloneDX; the semantic tamper is caught at check 4 (signature invalid); if an attacker could also re-sign the ML-BOM, Check 7 would not catch it alone — this is why the signature check is load-bearing"
2026-11-24T10:22:18Z check=8 evaluations decision=pass required=[fairness-tenant-tier, adversarial-robustness, pii-recall, red-team-suite] failing=[]
2026-11-24T10:22:18Z check=9 validator decision=pass memo=07-independent-validation-memo.md signer=L35-eval-eng-alex disposition=conditional-approve
2026-11-24T10:22:18Z gate decision=reject first_failing_check=1 all_failing_checks={1,2,4,5} total_duration_ms=11000 playbook=playbooks/signature-and-scan-failure.md
```

Reading check: four distinct failing checks reported (1 format, 2 picklescan, 4 cosign on two files, 5 rekor); the first-failing check (`check=1`) is called out in the summary line; the `note=` lines explain why ModelScan passed where picklescan caught it (chapter-06 complementary-coverage rule); the overall `gate decision=reject` names the playbook that would fire.

### Step 6 — Author the exception template (`gates/exception-template.md`)

Chapter 06 §"The exception path" names the five-step process and §"Where this module hands off" names the mod-112 risk council as the decision owner. Fill the template:

```markdown
# Ingestion-Gate Exception Request — Template

Chapter 06 §"The exception path" five-step process. Exceptions are **not a bypass** — they are a controlled, time-bound path with a compensating control, a monitor, and an expiry review.

## 1. Written request from the owner

- Requester name and role:
- Artefact path:
- Check(s) that would fail (name by number from `gates/registry-ingestion-policy.md`):
- Reason the failure is acceptable (one paragraph — a factual statement about the artefact, not a timeline argument):
- Compensating control proposed (name the specific mod-108 control / mod-109 monitor that will substitute for the failing check; a bare "we'll be careful" is not a compensating control):
- Proposed waiver duration (specific days / weeks; absolute end date):

## 2. Risk-council decision (mod-112)

- Approver: mod-112 risk council — minutes pointer:
- Decision: `approve` | `reject` | `approve-with-amendment`
- Amendments (if any) — tighter waiver duration, additional compensating control, more frequent review cadence:
- Approved end date (specific date, no "until further notice"):

## 3. Register update

- mod-103 register row(s) affected:
- Residual-risk elevation (what residual moves and by how much):
- Register `evidence_pointer` added to this exception document:

## 4. Compensating-control monitor

- mod-109 signal-contract row id for the compensating monitor:
- Metric, threshold, cadence, routing, action-on-fire (one line each):
- Rule: if the compensating control degrades (monitor fires), the waiver is suspended and the artefact is pulled; this is the mod-111 incident-response branch.

## 5. Waiver review at expiration

- Review date (one calendar entry 7 days before the approved end date; one at the end date):
- Review question: can the artefact now clear the gate cleanly? If yes, close the waiver. If no, is there a changed compensating control and a changed residual the council will approve for a further defined duration? (Chapter 06 rule: *no exception is permanent by default*.)

## Change log

- Waiver draft date:
- Council approval date (mod-112 minutes pointer):
- Compensating-monitor live date:
- Expiry date:
- Close / extension decision date:
```

Reading check: five numbered steps; each step has concrete fields; the "no exception is permanent by default" rule from chapter 06 is explicit; the mod-112 risk-council decision is named as the approver, not an engineer; the compensating control must be a named mod-108 control or mod-109 monitor, not a prose claim.

### Step 7 — Author the runtime-failure playbook stub (`playbooks/signature-and-scan-failure.md`)

Chapter 05 §"Signature verification failure — the runtime story" names three branches; chapter 06 §"The malicious-artefact detection lifecycle" adds a fourth branch for re-scan flip. Fill the stub with the five branches and their routing:

```markdown
# Playbook — signature / scan failure at ingestion or re-scan

This playbook branches on the cause of a gate-side failure. It is cross-referenced from `gates/registry-ingestion-policy.md` and from the mod-111 incident-response playbook top-level.

## Branch 1 — Signing pipeline changed (expected)

- Trigger: check 4 (cosign) decision=reject reason=invalid *and* the pipeline source has a recent change that re-signed the artefact under a new identity.
- First response: ML on-call confirms the pipeline change is intentional via the mod-112 risk-council change-log; verifies the new identity is on `gates/identity-allowlist.md`; if yes, re-sign and re-run the gate.
- Escalation: if the identity is not on the allow-list, escalate to L35 security platform and to the mod-112 council (the allow-list is a mod-112 artefact).
- Routing: ML on-call → L35 security platform (on identity question) → mod-112 council (on allow-list update).

## Branch 2 — Artefact modified (unexpected)

- Trigger: check 4 or 5 decision=reject on an artefact whose pipeline has not changed; signature or Rekor entry does not match.
- First response: isolate the artefact; do not retry; treat as a potential integrity incident. Open an incident candidate per mod-111; capture the artefact hash and the Rekor entry (or its absence).
- Escalation: security team within 15m; L35 platform team for the signing surface; mod-112 council digest on the next cycle.
- Routing: ML on-call + security + L35 platform; DPO on digest if any PII pathway is in scope.

## Branch 3 — Signing service outage (platform)

- Trigger: check 4 or 5 or any gate dependency cannot evaluate; fail-closed default is in effect.
- First response: gate rejects admission; traffic continues on the previously known-good version (canary / rollback per the model-serving stack). Do **not** disable the gate as the remediation; chapter 05 §"The runtime story" rule — "disable the check" is not a valid response.
- Escalation: L35 security platform owns the signing stack (Sigstore / Fulcio / Rekor deployment); their incident-response playbook branch covers the platform recovery.
- Routing: ML on-call → L35 platform; mod-112 digest if the outage exceeds 1h.

## Branch 4 — Scanner finding at ingestion (malicious artefact caught)

- Trigger: check 2 (picklescan) or check 3 (ModelScan) decision=reject with high-severity findings.
- First response: block the artefact at ingestion; capture the findings and the artefact hash; do not load, do not retry.
- Escalation: security team (the finding may indicate a compromised upstream source); L35 platform team for the scanner deployment; mod-112 council on the next cycle.
- Routing: ML on-call + security + L35.

## Branch 5 — Re-scan flip (previously-passing artefact flagged under new scanner rules)

- Trigger: periodic re-scan against the *current* scanner version flags a previously-admitted artefact. Chapter 06 §"The malicious-artefact detection lifecycle" rule — new detection rules follow new attacks.
- First response: the artefact is in production; isolate the inference workload serving it (canary down to 0%); open an incident; capture findings + scanner version diff + the hash.
- Escalation: security team + mod-111 incident-response branch for malicious-artefact-in-prod; L35 platform on the scanner update; mod-112 council within 24h.
- Routing: on-call + security + L35 + mod-112 council digest; DPO if any PII pathway is in scope.
```

Reading check: five branches (chapter 05 three + chapter 06 two); each branch names trigger, first response, escalation, and routing; the "disable the check" anti-pattern is explicitly ruled out on Branch 3; the re-scan flip branch (Branch 5) cross-references the chapter 06 lifecycle rule.

### Step 8 — Final checkpoint

Re-read the six files. Confirm:

- **Identity allow-list** has pipeline identities with no expiry and human identities with a specific expiry date; the rules section names the chapter-05 "trust-any-cert" anti-pattern; updates are attributed to mod-112 minutes.
- **Gate policy** has exactly nine ordered checks numbered 1-9; scanners (2, 3) are **before** signatures (4, 5); every check has a diagnostic shape; the SLSA floor (`v1.0 Build L2`) is pinned on check 6; fail-closed on dependency failure is explicit; the exception path is named and cross-referenced.
- **Pass log** emits one line per check and a final `gate decision=pass`; timestamps progress forward; the scanner-before-signature ordering is visible in the duration profile.
- **Fail log** reports the first-failing check prominently; at least three distinct failing checks are shown; the `note=` lines explain why ModelScan and picklescan disagree on a particular payload (complementary coverage); the summary line names the playbook branch.
- **Exception template** has five numbered steps; the compensating control must be a named mod-108 control or mod-109 monitor; the approved end date is specific (no "until further notice"); the "no exception is permanent by default" rule is explicit.
- **Playbook** has the three chapter-05 branches (signing pipeline changed; artefact modified; signing service outage) plus the two chapter-06 branches (scanner finding at ingestion; re-scan flip); routing names on-call + security + L35 platform + DPO where chapter 03 / 05 / 06 require; the "disable the check" fix is explicitly ruled out.

## Expected output

`mod110-lab03/gates/` and `mod110-lab03/playbooks/` — six populated files totalling three to four pages. The load-bearing artefacts are `gates/registry-ingestion-policy.md` (the ordered nine-check spec) and `gates/exception-template.md` (the five-step process); the pass + fail logs demonstrate the diagnostic shape; the playbook routes the failure branches.

## Checkpoint questions (self-check)

1. The gate policy puts picklescan and ModelScan (checks 2 and 3) *before* the cosign signature check (check 4). An engineer argues the cosign check is faster (milliseconds) than ModelScan (seconds) and should therefore run first. Why does the chapter-06 ordering override the latency argument? (*Chapter 06 §"The ingestion-gate policy" is explicit: "**cheap and destructive checks first** — the gate should fail *fast* on cheap checks so that operators do not wait five minutes for a signature check to time out on an artefact that was going to be rejected on format." The ordering principle is *destructiveness of the finding*, not raw latency. A malicious pickle whose `__reduce__` executes `os.system` is **more destructive** than a missing signature: the pickle would run code in the inference process; the missing signature only indicates the artefact would not have been admitted anyway. If a signature check were ordered first and the scanner were ordered after, a well-intended engineer could reasonably interpret "signature valid but scanners pending" as "safe to touch for debugging" and load the file locally before the scanner fires — which is exactly the chapter-06 motivation failure (`torch.load` in a Python process with production credentials). The ordering foreclosures the ambiguity: a pickle-based artefact never reaches the signature stage.*)

2. The exception template says the compensating control "must be a named mod-108 control or mod-109 monitor, not a prose claim." Why is the artefact identifier load-bearing, and what breaks if the control is described in prose only? (*Chapter 06 §"The exception path" step 3 and step 4 require that the exception route through the register (residual-risk elevation; evidence pointer to the exception) and that the compensating control have a monitor that fires the incident-response playbook if it degrades. Without a named mod-108 control, the register row cannot credit the control (chapter 01 §"Ongoing monitoring" — "a model without live monitoring cannot be credited as 'in scope of ongoing validation'"). Without a named mod-109 monitor, there is no file that fires when the compensating control degrades, which collapses step 4 into "we'll watch the dashboard" — the chapter-01 ad-hoc-dashboard anti-pattern. The exception then becomes a **default bypass** (chapter 06's exact anti-pattern), not a **controlled path**. The named-artefact rule forces the exception's compensating control to be a real file with an owner, a revision history, and a routing that reaches the register when things change.*)

3. The playbook's Branch 3 (signing service outage) rules out "disable the check" as the fix. What is the chapter-05 rationale, and what is the correct response? (*Chapter 05 §"Signature verification failure — the runtime story" closes with "Signature failures should be *rare* by construction. When they happen, they are a signal — the response cannot be 'disable the check.'" The chapter also pins §"The 'unsigned weights loaded in prod' anti-pattern" that fail-open ("we could not verify but let it through") is the same as "no gate at all." If the signing service (Fulcio / Rekor or a private mirror) is down and the team disables signature verification "temporarily" to admit an urgent release, they create exactly the fail-open behaviour an attacker with registry write access depends on; the "temporary" disablement persists longer than the outage because nobody remembers to re-enable it. The correct response is **fail-closed**: traffic continues on the previously known-good version (canary / rollback), the inference layer serves either the prior model or an error, the signing-platform outage is treated as an L35 platform incident with its own incident-response branch, and the mod-112 council digests the outage. The outage is *the signal* that the platform needs attention — not the gate.*)

4. The identity allow-list has a human identity (`L35-eval-eng-alex@human-exception-until-2027-01-15`) with an explicit expiry claim. Chapter 05 is emphatic that "a human should almost never be the direct signer of a production model artefact." Why is the human identity on the list at all, and how does the expiry claim prevent the long-lived-shared-key anti-pattern? (*Chapter 05 §"What you sign, at minimum" permits human exception-signing but conditions it: "when they are, it should be a very deliberate exception with its own audit trail." Chapter 06 §"The exception path" step 2 (mod-112 time-bound waiver) and step 5 (expiry review) are what operationalise that discipline. The human identity exists on the list so that a specific, council-approved exception (the research-collaborator checkpoint waiver in the lab-03 change log) can be verified by the gate rather than silently bypassed — but the identity's claim shape includes an expiry (`until-2027-01-15`) so the gate itself refuses to accept the signature past that date. This closes the chapter-05 "long-lived key shared across teams for years" failure mode by construction: the identity cannot be "silently shared" because it is scoped to one person, and it cannot "persist for years" because the expiry claim is a gate-side check. When 2027-01-15 passes, the identity is dropped from the allow-list by the automated expiry review; a renewed exception would require a fresh mod-112 decision. The audit trail is in the change log (which attributes the row to minutes 2026-10-30) and in the Rekor log (which records every signing event).*)

## Why this lab matters

Exercise 05 asks you to sign the provenance package end-to-end with cosign + Fulcio + Rekor, wire the signature-related checks into the gate, and demonstrate a passing and a failing run. Exercise 06 extends the same gate with ModelScan + picklescan, crafts a sandboxed malicious pickle, and demonstrates a blocked artefact. If the gate structure — the ordering, the diagnostic shape, the identity allow-list, the exception template, the playbook branches — is new at exercise-05/06 authoring time, the budget blows on inventing the policy skeleton, arguing about where scanners sit relative to signatures, and re-authoring the exception vocabulary. This lab drills the policy *shape* on a fixed fabricated system so that exercise-05 time goes into the real cosign wiring, the real Rekor query, the real inference-side verification wrapper, and the real failing-run tampers — and exercise-06 time goes into the real malicious-artefact crafting, the real scanner outputs, and the real re-scan-cadence design. The shape it drills is also what the chapter-04 provenance package (lab 02) is signed and scanned against — this is the terminal artefact of the module.
