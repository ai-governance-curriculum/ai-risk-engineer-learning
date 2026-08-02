# Exercise 05 — Sigstore Model Signing and Registry-Ingestion Gate

**Estimated effort:** 2 hours

## Objective

Sign the model weights and the provenance documents produced in exercise 04 with **cosign**, record the signatures in **Rekor** (public or private log per your organisation's posture), and wire the signature check into the model registry as a *blocking* ingestion gate. The outcome is a demonstrated instance of the "no unsigned weights in production" rule — an artefact that fails ingestion on signature-verification failure — plus the policy documentation the L35 security platform team consumes to run the gate at scale.

## Prerequisites

- Chapter 05 read in full. Sigstore's cosign / Fulcio / Rekor flow, the minimum signing surface, the registry-gate policy table, the "unsigned weights loaded in prod" anti-pattern, and the SLSA level uplift ladder are the reference shapes.
- Exercise 04 completed. The provenance package for one deployed model is what you will sign; the layout and the how-to-verify block are the anchors this exercise extends.
- Cosign installed locally (`cosign version` runs) and an OIDC identity available for keyless signing — GitHub Actions runner identity, GitLab CI identity, Google / Microsoft / GitHub personal identity for local drills, or an internal workload-identity token for organisation-scoped work. Local ephemeral signing for the drill is acceptable; production signing must use a pipeline identity.
- A test model registry to gate — this can be an existing internal registry, a local Docker registry configured for OCI-artefact storage, a Kubernetes admission-controller sandbox (Gatekeeper / Kyverno), or an OPA policy evaluated in a CI job. The exercise does not require a production registry; it requires a *demonstrable* gate.
- A pointer to the L35 security-platform contact (`ai-infra-security` / `security-learning`) who owns the runtime signing / verification stack in your organisation. If no such contact exists, name the peer who would inherit the responsibility.

## Problem statement

Sign the exercise-04 provenance package end-to-end and wire the registry-ingestion gate so that an unsigned or improperly signed artefact cannot land. The exercise has two halves: the *signing* half (cosign against Fulcio + Rekor) and the *gating* half (a policy that verifies signatures and blocks on failure).

### The signing surface

Chapter 05 §"What you sign, at minimum" names the six artefact classes. For this exercise, sign at minimum:

- The weights file(s) — `weights.safetensors` or the equivalent for your model's format.
- The Model Card — `model-card.md`.
- The ML-BOM — `mlbom.cyclonedx.json`.
- The SLSA provenance record — `slsa-provenance.json`.
- The evaluation report — `evaluation/eval-report.md` (or `.json`).
- A placeholder for the scan report (`attestations/scan-report.json`) — this file will exist after exercise 06; sign it if you have run exercise 06 already, or reserve the slot.

Each signed artefact produces a `.sig` file (and an optional `.pem` certificate reference) alongside it in the provenance folder, plus a Rekor entry with a searchable UUID recorded in the signing manifest.

### The gate policy

The registry-ingestion gate implements the check list from chapter 05 §"The registry-ingestion gate" (the subset that applies to signing — the scanner subset is added in exercise 06). For each artefact appearing at the registry boundary:

1. Signature present? — reject if absent.
2. Signature verifies against the expected identity? — reject if identity does not match the allow-list.
3. Rekor entry present and consistent? — reject if absent or if the entry does not correspond to the artefact's hash.
4. SLSA provenance record present? — reject if absent.
5. SLSA level meets the organisation floor? — reject if below floor.
6. ML-BOM present and syntactically valid? — reject if not.
7. Data Cards referenced by the Model Card exist in the provenance folder? — reject if any are missing.
8. (Placeholder for exercise 06) Scanner reports present and clean? — reserve the slot.

The gate is implemented once (per the tooling of your choice: OPA + rego policy, Kyverno / Gatekeeper policy, a CI job, an admission-webhook-shaped script) and demonstrated against both a passing and a failing artefact.

## Requirements

Produce all seven items:

1. **The signed provenance package** — the exercise-04 folder now has `.sig` files (and, where applicable, `.pem` certificate references) for every artefact in the signing-surface list. The signing was done using cosign with keyless OIDC identity (or a documented alternative flow if OIDC is not available in your environment).
2. **The signing manifest** (`provenance/<model>/<release>/attestations/signing-manifest.json`) — a machine-readable inventory of every signature made, keyed to the artefact hash. Each entry records: artefact path, artefact SHA-256, signer identity, signing time, Rekor entry UUID (or "n/a — private log" if using a private Rekor). The manifest itself is signed.
3. **The gate policy** — as a file: `gates/registry-ingestion-policy.rego` (OPA), `gates/registry-ingestion-policy.yaml` (Kyverno / Gatekeeper), or `gates/registry-ingestion-check.py` (a CI-job checker) — implementing the seven signing-related checks above. The policy is stored alongside the provenance package and is itself version-pinned.
4. **A passing-artefact demonstration** — a recorded run of the gate against the fully signed provenance package that reports "pass" on every check. Record the output as `gates/demo-pass.log`.
5. **A failing-artefact demonstration** — a recorded run of the gate against a deliberately tampered variant of the provenance package that reports "fail" on at least three distinct checks. Suggested tampers (pick at least three): remove the weights signature; replace the Model Card with an unsigned copy; corrupt the SLSA record so it no longer parses; alter a weights byte so its hash no longer matches the signature; sign the weights with an identity not on the allow-list. Record the output as `gates/demo-fail.log`.
6. **The identity-allow-list document** (`gates/identity-allowlist.md`) — the list of signing identities the gate accepts, keyed to purpose (training-pipeline service account, validation-pipeline service account, exception-signing human identities). Each entry cites the OIDC issuer and the identity claim shape.
7. **The runtime failure playbook stub** (`playbooks/signature-verification-failure.md`) — a short branch of the mod-111 incident-response playbook covering the three cases from chapter 05: signing pipeline changed (expected; verify and re-sign), artefact modified (unexpected; investigate as integrity incident), signing service down (platform incident; escalate to L35). Each branch names the on-call role and the escalation SLA.

## Starter guidance

- **Use safetensors or GGUF for weights where you can.** Chapter 06 will lean on this preference for the scanner suite; chapter 05 benefits too, because safe-by-design formats reduce the "did the file execute code on load" concern that runs in parallel with signature checks. If the weights are pickle-based, sign them, but flag the format as a gap in the runtime-failure playbook.
- **Keyless signing is the default; keyed signing is the exception.** Cosign's keyless flow via Fulcio + Rekor is what the exercise expects. Long-lived signing keys in a secret manager are the anti-pattern chapter 05 named ("shared across teams for years"). If your environment cannot yet do keyless, document the exception and note the gap for the L35 platform team.
- **The signing identity must match the pipeline, not the human.** For a training pipeline running in GitHub Actions, the signer is the workflow identity (repo + workflow file + run ID). For a validation pipeline running in Jenkins, it is the Jenkins service account. Human signatures are exceptional and get their own audit trail.
- **Rekor entries are the audit trail.** After signing, retrieve the Rekor entry UUID and record it in the signing manifest. `cosign search --rekor` and `rekor-cli search --sha` are the retrieval commands. Signatures without recorded Rekor entries are half-signed for audit purposes.
- **The gate is one policy expressed in multiple places.** The same seven checks may be implemented as an admission controller at the OCI registry, as a pre-load check in the inference process, and as a CI check on the training pipeline's publish step. The policy source of truth is one file; the enforcement points are many.
- **The failing demonstration is where the gate earns its keep.** A passing gate is easy; a gate that catches five distinct failure modes with clear diagnostics is what makes it credible. When you pick tampers, aim for coverage across the check list, not five variants of the same failure.
- **Fail-closed is the correct default.** If the gate cannot verify a signature — signing service unreachable, Rekor down, allow-list unresolvable — the default action is to *not admit the artefact* and fire the runtime-failure playbook. Fail-open ("we could not verify but let it through") is the same as "no gate at all" for a determined attacker.
- **The gate does not decide policy; it enforces policy.** Threshold decisions (what SLSA level is the organisation floor, which identities are on the allow-list, which exceptions have been granted) are risk-council decisions (mod-112). Your job is to encode them in the gate file and update it as the council decides.
- **Placeholder the scanner checks for exercise 06.** The gate policy should have the scanner check slots defined and returning "not yet enforced" so that exercise 06's wiring is additive rather than a rewrite.

## Acceptance criteria

The submission is accepted when:

- Every artefact in the signing-surface list has a valid cosign signature and (where the public Rekor is used) a Rekor entry.
- The signing manifest is present, is itself signed, and records artefact hash, signer identity, signing time, and Rekor UUID per entry.
- The gate policy file exists, expresses the seven signing-related checks, and is version-pinned alongside the provenance package.
- The passing-artefact demonstration log shows all applicable checks passing.
- The failing-artefact demonstration log shows at least three distinct check failures with clear diagnostics.
- The identity-allow-list document exists and cites OIDC issuer plus identity claim shape per accepted identity.
- The runtime-failure playbook stub is present and branches on the three chapter-05 cases with named on-call roles and escalation SLAs.
- The exercise-01 MRM packet's implementation-testing section and the exercise-04 provenance package's `README.md` how-to-verify block are updated to reference the signing and the gate.

## Stretch goals

- **Inference-side verification wrapper** — write a small Python wrapper (`inference/verify_on_load.py` or equivalent) that verifies the weights signature before `torch.load` / `safetensors.load` runs, checks the Rekor entry, and refuses to load on failure. Wire it into a demonstration inference pod. The wrapper is the runtime enforcement point chapter 05 names.
- **Rekor query drill** — for one of the signed artefacts, run a Rekor query (`rekor-cli search --sha <hash>`) and paste the retrieved entry into `attestations/rekor-entry.json`. This is the audit-time drill: given an artefact hash, retrieve the signing event.
- **Private Rekor evaluation memo** — if your organisation is considering a private Rekor deployment, write a one-page memo weighing the tradeoffs from chapter 05 §"The Rekor transparency log": operational overhead of running the log vs the confidentiality benefit of not indexing internal artefact hashes publicly. Cite the current Sigstore project recommendation.
- **SLSA-level uplift plan** — draft a short memo naming the current SLSA level of your training pipeline and the two next steps that would raise it. This is the input to the L35 platform team's roadmap.
- **Register update** — in the mod-103 register, on every row whose residual is credited to signing or provenance controls, add an `evidence_pointer` to the signing manifest UUID or the Rekor entry. This is the two-way lookup between the register and the runtime evidence.
- **Exception drill** — draft a signing-exception request for a hypothetical scenario (a research collaborator's checkpoint that cannot be signed by your pipeline). Follow chapter 06's exception process shape: written request, risk-council decision, time-bound waiver, monitor, expiry review.

## Deliverable format

A single markdown document `mod-110-ex05-signing-and-gate.md` that indexes the artefacts. The index contains:

1. *Package and signing summary* — one paragraph naming the model, the release tag, the number of artefacts signed, and the signing identity used.
2. *Signing manifest excerpt* — the manifest's header plus at least three entries quoted verbatim.
3. *Gate policy summary* — the seven checks listed with the tool used to enforce each.
4. *Passing demonstration* — the demo-pass log quoted or summarised.
5. *Failing demonstration* — the tampers applied and the demo-fail log summarised, one line per failing check.
6. *Identity allow-list* — the list quoted verbatim.
7. *Runtime failure playbook stub* — the three-branch structure quoted or summarised.
8. *Cross-references* — pointers to the exercise-01 MRM packet's implementation-testing section, the exercise-04 provenance package, and the L35 platform contact who owns the runtime enforcement.
9. *Open items* — anything blocked on the L35 platform team, on keyless-signing infrastructure, or on the scanner slots that exercise 06 will fill.

## Submission

Attach the index markdown plus the updated provenance package folder (with signatures) and the `gates/` and `playbooks/` folders. If your environment does not permit signing with the public Sigstore instance for this drill, document the alternative flow (a private Sigstore, a local test key with a labelled-and-time-bounded exception) in the index — but preserve the manifest, the gate policy, and the demonstration logs. The gate is the load-bearing artefact for chapters 05 and 06; exercise 06 extends it in-place.
