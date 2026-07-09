# mod-107-privacy-risk-engineering: Privacy-Risk Engineering for ML and LLM Systems

> Scaffolded by `aicg org execute-plan`. Lecture chapters and exercise content are authored on subsequent autonomous cycles.

**Estimated effort:** 14 hours

## Learning objectives

- Engineer membership-inference evaluations at production quality — Shokri-style shadow-model MIA, Carlini-style LiRA — and interpret the AUC / TPR-at-fixed-FPR results as a residual-risk signal
- Engineer attribute-inference and model-inversion evaluations on tabular and vision models; engineer memorisation / verbatim-training-data extraction evaluations on LLMs
- Engineer differentially private training (DP-SGD) with Opacus and TensorFlow Privacy — reason about (ε, δ), noise multiplier, clipping norm, and utility / privacy trade-off well enough to defend the parameter choice under review
- Engineer PII detection / redaction pipelines with Microsoft Presidio and integrate them into ingest, retrieval, and output paths
- Author a privacy-risk report that maps to GDPR Article 22 (automated individual decision-making) and to the NIST Privacy Framework functions — routing legal opinion out of scope to counsel
- Use ML Privacy Meter as the working evaluation harness for the above; know where `model-evaluation-engineer` (peer, level 30) picks up methodological calibration depth

## Structure

- `01-…md` … `0N-…md`: lecture chapters.
- `exercises/`: per-exercise prompts.
- `labs/`: long-form hands-on labs.
- `quizzes/`: knowledge checks.
- `resources.md`: external references.
