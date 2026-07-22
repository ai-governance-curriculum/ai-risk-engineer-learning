# mod-110 — Model Risk Management (MRM) for ML/AI and AI Supply Chain

**Estimated effort:** 16 hours
**Position in the track:** tenth module; assumes mod-101 through mod-109 are done. You bring a harm model (mod-102), a risk register (mod-103), evaluation evidence (mods 104-107), a guardrail catalogue (mod-108), and live monitors (mod-109). This module packages that engineering into the shape US financial-services supervisors, FDA reviewers, and auditors reading provenance evidence expect to see.

The regulated-sector packaging is *not* new engineering work — it is *labelling*, *routing*, and *evidence assembly* built on the artefacts you already produce. This module teaches the two dominant sector packages (SR 11-7 / OCC 2011-12 / SR 23-4 for US financial institutions; FDA GMLP + Predetermined Change Control Plan for medical devices) and the supply-chain provenance shape (datasheets, model cards, ML-BOM, SPDX 3.0 AI profile, SLSA, Sigstore signing, ModelScan / picklescan gating) that both sectors — and every other AI programme that survives an audit — increasingly assume.

## Learning objectives

- Apply **SR 11-7 / OCC 2011-12** to ML/AI at a regulated FI — validation-vs-development independence, conceptual soundness, ongoing monitoring, outcome analysis, effective challenge; produce MRM-shaped validation documentation that consumes the mod-104 red-team, mod-105 adversarial, mod-106 fairness, mod-107 privacy, mod-108 guardrail, and mod-109 monitoring evidence you already generate.
- Apply **SR 23-4** (Interagency Guidance on Third-Party Relationships) to third-party model / API vendors — planning, due diligence, contractual controls (SLAs on safety metrics, right-to-audit, notice of model change), ongoing monitoring, termination.
- Apply **FDA / Health Canada / MHRA GMLP** guiding principles and the **FDA Predetermined Change Control Plan (PCCP)** guidance where the AI system is a Software as a Medical Device (SaMD) — enough to author the risk-engineering slice of a PCCP (description of modifications, modification protocol, impact assessment).
- Engineer **AI supply-chain provenance** — Datasheets for Datasets and Data Cards for data provenance; Hugging Face model cards for model provenance; **ML-BOM** with **CycloneDX**; the **SPDX 3.0 AI profile**; **SLSA** build-integrity levels — so an auditor can trace a deployed model back to its inputs.
- Sign model weights with **Sigstore** (cosign, Rekor transparency log); gate registry ingestion on signature validity and on **Protect AI ModelScan** / **Hugging Face picklescan** results; kill the "unsigned weights loaded in prod" anti-pattern.
- Coordinate with `senior-ai-governance-architect` (level 50) on the control-library architecture this MRM shape plugs into, and with `security-learning` / `ai-infra-security` (level 35) on the runtime signing / scanning platform. This module owns the *risk-engineering interface* to those platforms, not the platforms themselves.

## Chapters

1. [`01-sr-11-7-shape-for-ai.md`](01-sr-11-7-shape-for-ai.md) — SR 11-7 / OCC 2011-12 as the parent MRM shape at US regulated FIs; the four pillars (independence, conceptual soundness, ongoing monitoring, outcome analysis) plus effective challenge; MRM-shaped validation documentation for an ML/AI model.
2. [`02-sr-23-4-third-party-model-due-diligence.md`](02-sr-23-4-third-party-model-due-diligence.md) — SR 23-4 interagency third-party risk guidance; vendor due-diligence questionnaire shape; contractual controls (SLAs on safety metrics, right-to-audit, notice of model change); monitoring the vendor's model on your own signal.
3. [`03-fda-gmlp-and-pccp.md`](03-fda-gmlp-and-pccp.md) — FDA / Health Canada / MHRA GMLP guiding principles; the FDA PCCP guidance; the SaMD context; what "locked" vs "adaptive" means and the authorised change protocol.
4. [`04-supply-chain-provenance-datasheets-mlbom-spdx-slsa.md`](04-supply-chain-provenance-datasheets-mlbom-spdx-slsa.md) — data provenance (Datasheets for Datasets, Data Cards); model provenance (Hugging Face model cards); ML-BOM with CycloneDX; SPDX 3.0 AI profile; SLSA build-integrity levels.
5. [`05-model-signing-sigstore-and-registry-gates.md`](05-model-signing-sigstore-and-registry-gates.md) — Sigstore weights signing (cosign, Rekor); registry-ingestion gates that require a valid signature; the "unsigned weights loaded in prod" anti-pattern; SLSA level uplift as the registry matures.
6. [`06-malicious-artifact-detection-and-level-ladder.md`](06-malicious-artifact-detection-and-level-ladder.md) — Protect AI ModelScan, Hugging Face picklescan, HiddenLayer-style malicious-model scanners; the pickle-and-format threat surface (PyTorch, joblib, ONNX, TF SavedModel, Keras H5); the ingestion-gate policy; where the level-50 architect owns the control-library architecture and level-35 security owns the runtime platform.

## Exercises

- [`exercises/exercise-01-sr-11-7-shape-applied-to-an-ml-model.md`](exercises/exercise-01-sr-11-7-shape-applied-to-an-ml-model.md) — package your evidence for one ML/AI model into SR 11-7-shaped validation documentation.
- [`exercises/exercise-02-sr-23-4-third-party-model-due-diligence.md`](exercises/exercise-02-sr-23-4-third-party-model-due-diligence.md) — run an SR 23-4-shaped due-diligence and contractual-controls pass on one third-party model / API vendor.
- [`exercises/exercise-03-fda-gmlp-and-pccp-slice-for-one-device.md`](exercises/exercise-03-fda-gmlp-and-pccp-slice-for-one-device.md) — draft the risk-engineering slice of a PCCP for one SaMD-shaped system.
- [`exercises/exercise-04-ml-bom-spdx-slsa-provenance-authoring.md`](exercises/exercise-04-ml-bom-spdx-slsa-provenance-authoring.md) — author an ML-BOM (CycloneDX) plus SPDX 3.0 AI profile record plus SLSA provenance record for one deployed model.
- [`exercises/exercise-05-sigstore-model-signing-and-registry-gate.md`](exercises/exercise-05-sigstore-model-signing-and-registry-gate.md) — sign model weights with cosign / Rekor and wire the signature gate into your model registry.
- [`exercises/exercise-06-modelscan-picklescan-ingestion-gate.md`](exercises/exercise-06-modelscan-picklescan-ingestion-gate.md) — wire ModelScan + picklescan into the registry ingestion gate and demonstrate a blocked malicious artefact.

## Labs & quizzes

- [`labs/`](labs/) — a longer hands-on lab is scheduled here; see the lab README for the current status.
- [`quizzes/`](quizzes/) — knowledge-check quiz for the module.

## Resources

See [`resources.md`](resources.md) for primary sources — every reference is a link to the regulator, standards body, tool project, or original publication.

## How to work through the module

1. Read chapters 01-03 in order. These are the sector-package chapters. Even if your current system is not in a regulated sector, the MRM and PCCP shapes are useful mental scaffolding and adjacent regulators are moving towards them.
2. Read chapters 04-06 in order. These are the provenance-and-signing chapters and apply to *every* AI system regardless of sector.
3. Pick one **primary running system** (the one you have been using since mod-101/102) and one **third-party model or API** it depends on (a foundation model API, a hosted embedding model, an off-the-shelf classifier — anything you did not train). The six exercises stack on that pair:
   - 01 packages your primary system's evidence into SR 11-7 shape.
   - 02 runs SR 23-4 due diligence against the third-party dependency.
   - 03 re-packages the primary system under FDA GMLP + PCCP (do this even if the system is not a medical device — the shape is instructive).
   - 04 authors the ML-BOM + SPDX + SLSA record for the primary system.
   - 05 signs the model weights and wires the registry gate.
   - 06 wires the malicious-artefact scanners and demonstrates a block.
4. Take the quiz.

By the end of the module you should be able to hand an auditor a folder that answers three questions per model in operation: *which regulator's shape does this file into*, *where did every input to this model come from*, and *what stopped a malicious or untrusted artefact from landing in production*. If any of those three questions cannot be answered from the folder, the packaging is incomplete.
