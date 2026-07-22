# Model Signing with Sigstore and Registry-Ingestion Gates

Chapter 04 assembled the provenance package. This chapter locks it down: *sign* the model weights and the provenance documents so that a downstream consumer can verify that what they load is what the training pipeline produced, and *gate* the model registry so that anything without a valid signature — or without a passing malicious-artefact scan, per chapter 06 — cannot land in production.

The tooling anchor is **Sigstore**: a Linux Foundation project (originally sponsored by the OpenSSF) that provides free code signing via short-lived keys, a transparency log (Rekor), a certificate authority (Fulcio), and a signing CLI (cosign). Sigstore has become the de-facto software-signing stack in the cloud-native ecosystem, and the model-signing use case is a direct extension.

## Motivation

Three failure modes that pull production down when weights are unsigned:

- **The "unsigned weights loaded in prod" anti-pattern.** A model file appears in the production registry. Nobody knows who put it there — the CI system, a manual upload, a script an engineer ran on their laptop, a copy from another environment. When it starts producing outputs that fail evaluation, tracing back to "who built this and how" takes days.
- **Silent swap by an attacker with registry write access.** An adversary with credentials for the model bucket replaces `weights.safetensors` with a look-alike file — either poisoned or backdoored (see chapter 06). Inference workers pick it up on their next warm restart. Without signature verification, the swap is invisible; with it, the swap fails signature check on load and the worker refuses to start.
- **Silent swap by an insider under time pressure.** No adversary involved — a well-meaning engineer copies a "known good" weights file from staging into production to unblock a launch, bypassing the pipeline entirely. Without signature enforcement, the file lands. With signature enforcement wired to the provenance record, the file has no matching Rekor entry and the load fails.

All three go away when *loading a model without a valid signature is a runtime error, not a warning*. This chapter is how you get there.

## Sigstore in six sentences

Sigstore was designed to let developers sign software artefacts without managing long-lived signing keys. The **cosign** CLI (part of Sigstore) generates a short-lived key pair, obtains a signing certificate from **Fulcio** (the Sigstore certificate authority) by proving OIDC identity (e.g., the CI job's OIDC token), signs the artefact with the ephemeral private key, records the signature in **Rekor** (the Sigstore transparency log), and destroys the private key. Verification retrieves the certificate and the Rekor entry and confirms that the artefact was signed by the claimed identity at the claimed time. The result is a verifiable signature without any long-lived key material to be stolen and without any single-signer bottleneck. For AI, the same flow signs model weights, provenance documents, and evaluation attestations. Sigstore does not itself decide *what* to sign or *what to trust* — those are policy decisions the registry-gate policy expresses.

## What you sign, at minimum

For one model release, a viable signing surface includes:

| Artefact | Signature purpose |
|---|---|
| Weights files (`*.safetensors`, `*.bin`, `*.gguf`, `*.onnx`, `*.pt`) | Confirms the file loaded at inference time is the file the training pipeline produced |
| Model Card (`model-card.md`) | Confirms the metadata claimed about the model is the metadata the pipeline produced |
| ML-BOM (`mlbom.cyclonedx.json`) | Confirms the component inventory has not been tampered with |
| SLSA provenance record | Confirms the "how was this made" record itself |
| Evaluation reports | Confirms the evaluation results were produced by the validation pipeline, not written by hand |
| The scan reports from chapter 06 | Confirms the malicious-artefact scan output is genuine |

The signing identity is the *pipeline*, not a human. The pipeline's OIDC identity (its service account, its workload identity in a cloud IAM system) proves who signed; Rekor makes the signing event auditable. A human should almost never be the direct signer of a production model artefact; when they are, it should be a very deliberate exception with its own audit trail.

## The Rekor transparency log

Rekor is a signed append-only log. Every signature made via Sigstore is optionally (and by default, in the public deployment) written to Rekor as an entry that includes the signing certificate, the artefact hash, and the signature. Verifiers can query Rekor by artefact hash to retrieve the signature history for a given artefact.

For AI, Rekor gives you two properties you would otherwise have to build yourself:

- **Non-repudiation.** The signer cannot later claim not to have signed; the log has a signed record of the event.
- **Non-falsifiability of the signing timeline.** Because Rekor is append-only and its checkpoints are witnessed, you can prove *when* an artefact was signed and thereby detect antedated signatures.

Some organisations run a private Rekor deployment for internal artefacts (weights that must not be publicly indexed). The tradeoff is that you inherit responsibility for the log's uptime and integrity properties. In most cases the public Rekor deployment is sufficient for signing internal model artefacts — Rekor stores hashes and signatures, not the artefacts themselves.

<!-- needs-research: confirm current guidance on private Rekor deployment vs public log usage for internal AI artefacts, and reference the Sigstore project's own recommended patterns. -->

## The registry-ingestion gate

The gate is the mechanism that decides whether an artefact appearing at the registry boundary is allowed to enter. For an AI model registry — whether it is Hugging Face-style, MLflow-style, an artefact bucket in a cloud provider, a specialised model registry (Vertex AI Model Registry, SageMaker Model Registry, Databricks Unity Catalog), or a self-hosted OCI registry — the gate needs to enforce a small set of policies before the artefact is considered *live*.

A shippable ingestion-gate policy for an ML/AI model:

| Policy check | Pass rule |
|---|---|
| Signature present | Every weights file, Model Card, and provenance document has a cosign signature |
| Signature valid | The signature verifies against the expected identity (the CI pipeline's service account) |
| Rekor entry present | The signature is recorded in Rekor (public or private log as configured) |
| SLSA provenance present | A SLSA provenance record accompanies the artefact and is signed |
| SLSA level ≥ organisation floor | The provenance record meets the SLSA level the organisation has adopted as the minimum for models in production |
| ML-BOM present and valid | A CycloneDX ML-BOM (and SPDX 3.0 record if the organisation requires both) is present and syntactically valid |
| Data Cards for every training / eval corpus | Every dataset referenced from the Model Card has a corresponding Data Card in the provenance folder |
| ModelScan clean (chapter 06) | ModelScan output shows no findings above the block threshold |
| picklescan clean (chapter 06) | picklescan output shows no findings above the block threshold |
| Evaluation gate | Required evaluations (per mod-101 sub-category mapping) have passed |
| Human sign-off | A validation-owner signature (or the equivalent policy attestation) is present |

Each check is *blocking*. Failing checks stop the ingestion; the artefact does not become discoverable to inference workloads. Warnings are for lower-severity findings that require review but do not block automatically; the taxonomy of "block" vs "warn" is a risk-council decision (mod-112), not an engineer's.

The gate is implemented in whatever pipeline your registry uses — a Kubernetes admission controller for OCI-backed registries, a pre-ingest hook in the registry service, a policy engine like Open Policy Agent (OPA) evaluating the artefact metadata, or a CI job that runs before publishing. What matters is that *bypass is impossible without an auditable exception*.

## The "unsigned weights loaded in prod" anti-pattern

The most common failure this chapter targets is not sophisticated attack; it is a *default* that permits unsigned weights to load. Any of the following collapse to that anti-pattern:

- The model loader (in inference code) fetches a URL and loads whatever bytes come back, without verifying a signature.
- The registry publishes artefacts without requiring a signature; only *some* teams sign, and only when they remember.
- The signature is required for publication but not for consumption — inference workers do not verify, only humans do (in a dashboard nobody reads).
- The signature is verified but the verifier trusts *any* signature, not signatures from the expected identity. An attacker with any Sigstore-issued certificate can sign and pass.
- The signature-verification key is a long-lived key stored in a secret manager, and it has been shared across teams for years.

The fix is a consistent inference-side pre-load check that (a) verifies the signature, (b) verifies the signing identity is on the allow-list, (c) verifies the Rekor entry exists and is recent, and (d) refuses to load on any of the above failing. Some model-serving stacks now bundle this; where they do not, it is a wrapper you write once and apply everywhere.

## SLSA level uplift as the registry matures

A useful maturity ladder for the registry gate, keyed to SLSA level:

| Maturity stage | What the registry enforces | SLSA level |
|---|---|---|
| Stage 0 — inventory | Registry knows what artefacts exist; no signing enforced | none |
| Stage 1 — signing enforced | Registry rejects artefacts without a cosign signature and Rekor entry | SLSA 1 (build produces provenance) |
| Stage 2 — hosted-signer enforced | Signatures come from a hosted build service under organisation OIDC (no laptop signing) | SLSA 2 |
| Stage 3 — provenance non-falsifiable | Build platform is isolated; provenance cannot be forged from a compromised build step | SLSA 3 |
| Stage 4 — cross-verified pipeline | Two-party review on build pipeline changes; hermetic builds | (See SLSA v1.0 for the current top-level definition) |

Stage 1 is the *minimum viable* posture. Stages 2 and 3 are progressively harder and are typically owned by the security platform team (`ai-infra-security`, level 35) rather than by risk engineering. Your job at level 25 is to keep the gate *turned on*, to keep the register aware of what level the platform is currently providing, and to escalate when the platform's assurance level regresses.

## Signature verification failure — the runtime story

A signature-verification failure is an operational event with a defined response:

- The inference worker refuses to load the artefact and emits a structured event.
- The event fires a monitor (mod-109) that alerts on-call within a tight SLA.
- The monitor's playbook (mod-111) branches on cause: was the signing pipeline changed (expected — verify and roll a new signature), was the artefact modified (unexpected — investigate as potential integrity incident), is the signing service down (a platform incident that requires the L35 security team).
- The inference layer either fails closed (traffic returns an error) or fails to the previous known-good version (canary / rollback), per the risk-council-approved policy.

Signature failures should be *rare* by construction. When they happen, they are a signal — the response cannot be "disable the check."

## Cross-references

- **mod-101** frameworks pin cryptographic integrity of provenance as an implicit MEASURE / MANAGE requirement (particularly under MEASURE 2.7 security and resilience).
- **mod-103** register rows on supply-chain-tampering scenarios have their residual score reduced by signing; the control-defeated column names the "signature verification disabled" mode.
- **mod-108** guardrail catalogue includes the signature-verification pre-load hook as a control credited on rows targeting supply-chain-tampering scenarios.
- **mod-109** monitors watch for signature-verification failures at ingest and at inference; the metric feeds the register.
- **mod-111** incident-response playbook has a "signature-verification failure" branch.
- **mod-112** cross-functional programme provides the risk-council escalation path for signing-service outages and exception requests.
- Chapter 04 produced the provenance package this chapter signs.
- Chapter 06 adds malicious-artefact scanning to the same gate.

## Summary

- Sigstore (cosign + Fulcio + Rekor) is the anchor tooling for signing AI artefacts without long-lived keys.
- Sign the weights, the Model Card, the ML-BOM, the SLSA provenance record, the evaluation reports, and the scan reports; the signer is the pipeline (via OIDC identity), not a human.
- The registry-ingestion gate must enforce a small set of blocking checks: signature present, signature valid, Rekor entry present, SLSA provenance present, SLSA level at or above the organisation floor, ML-BOM valid, Data Cards present, scanners clean, evaluations passed, validation-owner sign-off present.
- The "unsigned weights loaded in prod" anti-pattern is a default failure, not a sophisticated attack; the fix is inference-side signature verification with a specific-identity allow-list.
- SLSA level uplift is a security-platform maturity ladder owned above this level; risk engineering keeps the gate on and the register accurate.
- Exercise 05 signs one model's weights with cosign, records the signature in Rekor, and wires the signature gate into the registry.
