# mod-107 — Privacy-Risk Engineering for ML and LLM Systems

**Estimated effort:** 14 hours
**Position in the track:** seventh module; assumes mod-101 (frameworks & role scope), mod-102 (harm modelling), mod-103 (risk register), mod-104 (LLM red-teaming), mod-105 (adversarial ML), and mod-106 (fairness / bias / explainability) are done.

The frameworks in mod-101 named privacy as a top-line AI risk. The harm model in mod-102 identified privacy-flavoured scenarios (membership disclosure, attribute disclosure, verbatim leakage, PII in RAG). The register in mod-103 quantified them. Mods 104 and 105 gave you the adversarial-ML machinery some of the attacks share. This module teaches you to *evaluate* those privacy risks empirically — Shokri-style membership inference (MIA), Carlini-style Likelihood Ratio Attack (LiRA), attribute-inference, model-inversion, LLM verbatim extraction — and to *engineer the defences*: differentially private training with Opacus and TensorFlow Privacy, PII detection and redaction pipelines with Microsoft Presidio. You finish by authoring a privacy-risk report that maps to GDPR Article 22 (automated individual decision-making) and to the NIST Privacy Framework functions, with the boundary to legal counsel and to `model-evaluation-engineer` (peer, level 30) drawn clean.

Working evaluation harness for the module: **ML Privacy Meter** (NUS DSSG). Deeper methodological calibration on the eval side is not level 25 — it lives with `model-evaluation-engineer` at level 30, and chapter 06 marks the handoff.

## Learning objectives

- Engineer membership-inference evaluations at production quality — Shokri-style shadow-model MIA (S&P 2017) and Carlini-style Likelihood Ratio Attack (LiRA, S&P 2022) — and interpret AUC and TPR-at-fixed-FPR as a residual-risk signal for the mod-103 register.
- Engineer attribute-inference and model-inversion evaluations on tabular and vision models (Fredrikson et al.); engineer memorisation / verbatim-training-data extraction evaluations on LLMs (Carlini et al., "Extracting Training Data from Large Language Models"; "Quantifying Memorization Across Neural Language Models").
- Engineer differentially private training (DP-SGD) with Opacus and TensorFlow Privacy — reason about (ε, δ), noise multiplier, clipping norm, Rényi Differential Privacy accounting, and the privacy / utility trade-off well enough to defend a parameter choice under review.
- Engineer PII detection / redaction pipelines with Microsoft Presidio and integrate them into ingest, retrieval, and output paths; account for false-positive and false-negative rates.
- Author a privacy-risk report that maps to GDPR Article 22 (automated individual decision-making) and to the NIST Privacy Framework's IDENTIFY-P / GOVERN-P / CONTROL-P / COMMUNICATE-P / PROTECT-P functions — routing legal opinion out of scope to counsel.
- Use ML Privacy Meter as the working evaluation harness; know where `model-evaluation-engineer` (peer, level 30) picks up methodological calibration depth.

## Chapters

1. [`01-privacy-attack-taxonomy.md`](01-privacy-attack-taxonomy.md) — the taxonomy this module covers: membership inference, attribute inference, model inversion, memorisation / extraction; the mapping to NIST AI 100-2 e2025 privacy categories, to GDPR concepts (personal data, special category, profiling), and to the NIST Privacy Framework functions.
2. [`02-membership-inference-mia-and-lira.md`](02-membership-inference-mia-and-lira.md) — Shokri shadow-model MIA (2017); Carlini LiRA (2022); why AUC alone is misleading; TPR-at-fixed-FPR as the LiRA-recommended reporting; ML Privacy Meter as the working harness.
3. [`03-attribute-inference-model-inversion-and-memorisation.md`](03-attribute-inference-model-inversion-and-memorisation.md) — attribute-inference (Fredrikson-style) on tabular and vision; model-inversion; LLM verbatim / memorisation extraction (Carlini "Extracting Training Data from Large Language Models"; "Quantifying Memorization Across Neural Language Models"); how each eval is authored.
4. [`04-dp-sgd-with-opacus-and-tf-privacy.md`](04-dp-sgd-with-opacus-and-tf-privacy.md) — DP-SGD as the primary defence; (ε, δ) semantics and accounting (Rényi DP, Gaussian mechanism); noise multiplier and clipping norm; the privacy / utility trade-off; how to defend a parameter choice under audit.
5. [`05-pii-pipelines-with-presidio.md`](05-pii-pipelines-with-presidio.md) — Presidio analyser + anonymiser; recognisers and custom-pattern authoring; integration into ingest / retrieval / output paths; the "PII in a RAG store" problem; false-positive / false-negative accounting.
6. [`06-privacy-risk-report-and-boundary.md`](06-privacy-risk-report-and-boundary.md) — shape of the privacy-risk report that maps to GDPR Article 22 and NIST Privacy Framework functions; residual-risk claim; what is out of scope to legal counsel; where `model-evaluation-engineer` (level 30) owns deeper methodological calibration.

## Exercises

- [`exercises/exercise-01-shadow-model-mia-and-lira-baseline.md`](exercises/exercise-01-shadow-model-mia-and-lira-baseline.md) — build a shadow-model MIA and a LiRA baseline on a real target model; report AUC and TPR@low-FPR.
- [`exercises/exercise-02-attribute-inference-and-model-inversion-drill.md`](exercises/exercise-02-attribute-inference-and-model-inversion-drill.md) — run an attribute-inference attack and a model-inversion reconstruction on a tabular or vision target.
- [`exercises/exercise-03-llm-memorisation-extraction-eval.md`](exercises/exercise-03-llm-memorisation-extraction-eval.md) — build a memorisation / verbatim-extraction eval on an LLM and report an extraction rate.
- [`exercises/exercise-04-dp-sgd-training-with-opacus.md`](exercises/exercise-04-dp-sgd-training-with-opacus.md) — train a model with Opacus DP-SGD at two (ε, δ) budgets; produce the utility / privacy trade-off curve and defend the chosen point.
- [`exercises/exercise-05-presidio-pii-pipeline-and-redaction.md`](exercises/exercise-05-presidio-pii-pipeline-and-redaction.md) — assemble a Presidio pipeline for ingest / retrieval / output paths; author a custom recogniser; report false-positive / false-negative rates.

## Labs & quizzes

- [`labs/`](labs/) — a longer hands-on lab is scheduled here; see the lab README for the current status.
- [`quizzes/`](quizzes/) — knowledge-check quiz for the module.

## Resources

See [`resources.md`](resources.md) for the primary sources — every reference is a link to the standards body, regulator, or the original publication of an attack or defence.

## How to work through the module

1. Read chapters 01-06 in order. Each chapter pins one privacy attack / defence / reporting shape.
2. Pick a *single AI system* to be the running example (ideally the same one used in mods 101-106). All five exercises use it.
3. Do exercises 01-05 in order. They stack:
   - 01 produces a MIA and LiRA baseline on the target model,
   - 02 adds attribute-inference and model-inversion evidence,
   - 03 adds an LLM memorisation / extraction eval,
   - 04 produces DP-SGD trained variants and the utility / privacy trade-off curve,
   - 05 wraps the pipeline in a Presidio ingest / retrieval / output layer.
4. Take the quiz.

By the end of the module you should be able to hand a reviewer a privacy-risk report where every row of the mod-103 register linked to a privacy scenario has (a) a named attack evaluation with numeric residual-risk evidence, (b) a named defence (DP-SGD parameter, redaction rule, retrieval filter) with the credited-control evidence, (c) a mapping to at least one GDPR concept and one NIST Privacy Framework function, and (d) a clean note on what escalates to legal counsel and what escalates to `model-evaluation-engineer` at level 30.
