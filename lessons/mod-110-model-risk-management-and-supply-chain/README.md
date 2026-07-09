# mod-110-model-risk-management-and-supply-chain: Model Risk Management (MRM) for ML/AI and AI Supply Chain

> Scaffolded by `aicg org execute-plan`. Lecture chapters and exercise content are authored on subsequent autonomous cycles.

**Estimated effort:** 16 hours

## Learning objectives

- Apply SR 11-7 / OCC 2011-12 to ML/AI at a regulated FI — validation-vs-development independence, conceptual soundness, ongoing monitoring, outcome analysis, and effective challenge; produce MRM-shaped validation documentation
- Apply SR 23-4 interagency third-party risk guidance to third-party model / API vendors — vendor due-diligence, contractual controls, monitoring
- Apply FDA Good Machine Learning Practice + FDA Predetermined Change Control Plan guidance where the AI system is a medical device — enough to author the risk-engineering slice of a PCCP
- Engineer AI supply-chain provenance — data provenance (datasheets), model provenance (Hugging Face model cards, weights signing with Sigstore, ML-BOM with CycloneDX, SPDX 3.0 AI profile, SLSA levels) — well enough that an auditor can trace a deployed model back to inputs
- Detect malicious model artifacts (Protect AI ModelScan, Hugging Face picklescan) and gate model-registry ingestion on the scan
- Coordinate with `senior-ai-governance-architect` (level 50) on the control-library architecture this MRM shape plugs into; coordinate with `security-learning` / `ai-infra-security` (level 35) on the runtime supply-chain / signing platform

## Structure

- `01-…md` … `0N-…md`: lecture chapters.
- `exercises/`: per-exercise prompts.
- `labs/`: long-form hands-on labs.
- `quizzes/`: knowledge checks.
- `resources.md`: external references.
