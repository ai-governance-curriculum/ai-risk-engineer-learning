# Drift and Quality Monitoring — the Signals You Wire First

The first family of signals in the contract is drift and quality. It is the family that has the most mature tooling (whylogs, Evidently AI, Alibi Detect), the most mature literature (KS, PSI, MMD, JS divergence), and the most consistent failure mode: teams wire the detector, admire the dashboard, and never make the detector move a register row. This chapter fixes that — the detector-to-register wire is what makes drift-and-quality monitoring a governance artifact rather than a chart.

## Motivation

Drift and quality regression are the two mechanisms by which a model that once satisfied its release gate silently stops satisfying it. The frameworks make this obligation explicit:

- **NIST AI RMF 1.0** — MEASURE 2.4 asks that AI system performance be evaluated in production settings; MANAGE 4.1 asks that post-deployment monitoring plans be implemented and their outputs feed the risk process.
- **NIST AI RMF Generative AI Profile (AI 600-1)** — data-quality and information-integrity risks require ongoing detection, not point-in-time validation.
- **ISO/IEC 42001** and **ISO/IEC 23894** — treat performance monitoring as a management-system requirement whose evidence lives in the AIMS record.
- **EU AI Act Article 61 (post-market monitoring system)** and **Article 72 (post-market surveillance)** — for high-risk systems, provider obligations include a system that collects, documents, and analyses relevant data on performance throughout the lifetime.

Drift-and-quality monitoring is the concrete engineering closure for all of the above. This chapter says which metric, which tool, which threshold, and which register field for the running example.

## Types of drift, and which metric fits which

Not every drift is the same problem. The metric family has to match the shape of the input.

| Drift type | What is drifting | Typical metric families | Notes |
|---|---|---|---|
| **Covariate drift** | Input distribution `P(X)` shifts; label distribution and `P(Y\|X)` unchanged | KS (per-feature, continuous), PSI (per-feature, binned), MMD (multivariate, distributional), JS divergence (per-feature or population) | Detected first because inputs are always available |
| **Label drift** | `P(Y)` shifts; input distribution unchanged | Chi-squared / JS on label counts; PSI on binned label distribution | Only observable where labels are available (delayed labels are a common pitfall) |
| **Concept drift** | `P(Y\|X)` shifts; the relationship you learned changes | Rolling accuracy / task-quality delta vs. reference; per-slice quality regression; page-Hinkley or similar change-point tests | The dangerous one; requires labels or a proxy (golden-set eval, LLM-as-judge with calibration) |
| **Prediction / output drift** | The model's output distribution shifts | KS / PSI / JS on scores; embedding drift (MMD on encoded outputs); output-embedding drift for generative systems | Cheap proxy for concept drift when labels are delayed |
| **Prompt / retrieval drift** (LLM-specific) | Prompt-topic distribution or retrieved-document distribution shifts | Embedding drift (MMD on embeddings), per-cluster distribution shift, retrieval-hit distribution shift | The failure mode `ai-eval-engineer` (peer L30) also monitors; cite the boundary |
| **Per-slice drift** | Drift confined to a demographic / segment slice | Per-slice PSI / KS / quality delta; group-fairness re-check on newly drifted slice | Detected only if you actually slice; population-level metrics smooth this away |

The choice of metric is also a choice of *what would be missed*. A univariate KS on each feature will miss a joint shift where every marginal looks stable. MMD on the full input vector catches that but is expensive and requires kernel and sample-size choices. Chapter 04 of this module and mod-103 chapter 06 both flag "base-rate shift" as an event that triggers a register review; the base-rate shift is often detected first as per-slice drift.

### Metric summary — what each is, briefly

- **Kolmogorov–Smirnov (KS)** — non-parametric two-sample test on the maximum distance between empirical CDFs. Cheap, univariate, sensitive to shape. Threshold via p-value or effect size.
- **Population Stability Index (PSI)** — binned-histogram divergence widely used in credit-risk monitoring. Common convention bands: `< 0.1` no significant change; `0.1–0.2` moderate change; `> 0.2` significant change. The bands are conventions, not truths — validate against your own reference.
- **Maximum Mean Discrepancy (MMD)** — kernel-based two-sample test; catches multivariate shifts KS misses. Kernel choice (typically RBF) and bandwidth are design decisions; permutation testing gives p-values.
- **Jensen–Shannon (JS) divergence** — symmetric, bounded (0–1 with log-base 2) variant of KL. Useful for discrete distributions and score histograms.

The `resources.md` links to the canonical statistical references for each; anchor your threshold choices there rather than in vendor tutorials.

## Quality regression — the concept-drift proxy

Drift on inputs is a leading indicator; quality regression is the trailing indicator that actually matters. Quality regression monitoring runs the model against a stable evaluation harness — a golden dataset with labels, a shadow-eval replay, or an LLM-as-judge harness — on a cadence, and tracks the metric over time.

Design points that matter:

- **The golden set has to age well.** Fresh incidents (mod-111) produce regression fixtures that fold into the golden set; drop stale rows on a defined cadence. A frozen golden set becomes a benchmark that measures the past.
- **Per-slice quality is monitored alongside aggregate.** A stable aggregate can hide a slice regression, which is both a fairness signal and a leading indicator that the aggregate will move.
- **The judge is calibrated.** LLM-as-judge quality scores are only useful if the judge is calibrated against human-labelled data on a cadence; a drifting judge is a hidden quality regression. This calibration loop is the seam with `ai-eval-engineer` at L30 — the depth of the eval loop is theirs; the risk-side hook is yours.
- **The metric matches the harm.** For a customer-support RAG assistant, "answered correctly" is not enough; the metric decomposes into faithfulness (no hallucination against retrieved context), coverage (answered the actual question), and safe-refusal correctness (declined when appropriate). Each decomposes to a signal you can wire.

## The working toolset

Three open-source projects carry the drift-and-quality signal load at level 25. The point of naming them is not to compare vendors; it is to name what you are actually running, so the deferral boundary to `ai-infra-mlops` (peer L25) has a shape.

### WhyLabs / whylogs (open-source profiling)

- `whylogs` is an open-source data-logging library that emits per-batch profiles (statistical summaries) rather than raw data. Profiles are cheap to store, mergeable across batches, and comparable across time windows.
- The profile is the reference-window artifact: you log a "reference" profile from training or a stable window and compare production profiles against it. Drift metrics (KS-like distance, PSI, mean/quantile shifts) are computed on the profile pair.
- WhyLabs is the commercial hosted platform on top of whylogs; the platform choice is above your level (owned at L50/L60). The library is yours.

### Evidently AI (open-source evaluation and monitoring)

- Evidently emits *reports* (batch analyses over a time window) and *test suites* (pass/fail rules over a report). Report modules cover data drift, target drift, data-quality checks, classification / regression / ranking / recommendation quality, and text-data drift.
- The natural pattern at level 25 is: reports run on the cadence in the signal contract; test-suite pass/fail is the threshold; a failed test-suite emits an event that updates the register row through the contract.

### Alibi Detect (open-source drift and outlier detection)

- Alibi Detect (Seldon) implements a broader statistical detector set — KS, chi-squared, MMD, learned classifier drift, model uncertainty, online detectors (page-Hinkley, cumulative sum), and outlier detectors (isolation forest, VAE, Mahalanobis).
- The library is the natural home for the "MMD on multivariate embeddings" and "online change-point on streaming scores" detectors that whylogs and Evidently do not focus on.

Rule of thumb for picking between them:

| Signal shape | Reach for |
|---|---|
| Structured tabular inputs, univariate per-feature drift, low-cost storage of profiles | whylogs (+ WhyLabs if the platform is available) |
| Batch reports covering drift + quality + data-quality with a test-suite pass/fail contract | Evidently AI |
| Multivariate, embedding-space, online / change-point, outlier | Alibi Detect |

You will likely run all three. That is fine; each closes a different set of rows in the contract.

## Per-slice monitoring — the discipline that is easy to skip

Aggregate metrics conceal slice regressions. A signal contract row for per-slice drift is qualitatively different from an aggregate row:

- The `signal_semantics` names the slice definition explicitly (e.g., "PSI on top-20 features computed per protected-attribute slice, using the harm-model persona list from mod-102").
- The threshold is per-slice — a per-slice trip counts even if the aggregate is quiet.
- The register effect points at the fairness-tagged row, not the drift-tagged row, if the affected slice is a protected group. This is the seam with mod-106 (fairness and explainability engineering).
- The routing includes the fairness liaison if applicable.

Skipping per-slice monitoring is one of the most common findings a fairness auditor names. Wire it early; skipping means the row never gets a signal.

## Worked contract rows — drift and quality on the customer-support RAG assistant

The first three rows fill in from the master mapping in chapter 01.

### Row `SC-CS-DRIFT-01` — daily PSI on top-20 input features

- `monitor_id`: `whylogs-ref-vs-prod-daily`
- `signal_semantics`: PSI computed per feature on the top-20 features from the reference profile taken at the last release, compared to a rolling 24-hour production profile.
- `cadence`: daily batch, run at 02:00 UTC.
- `threshold`: PSI > 0.2 on any single monitored feature *or* PSI > 0.1 on three or more features simultaneously.
- `register_effect`: residual state on `HR-CS-DRIFT-02` moves to `control-defeated`; treatment flag opens with a 5-day due date; feature list attached to the register-side note.
- `alert_routing`: ML on-call channel (page on trip); risk-engineer digest (daily summary).
- `evidence_pointer`: `evidence/sc-cs-drift-01/<yyyy-mm-dd>/report.json` with reference profile hash and production profile hash.

### Row `SC-CS-QUAL-01` — daily quality regression against golden set

- `monitor_id`: `evidently-quality-daily`
- `signal_semantics`: task-level quality metric (faithfulness + coverage composite) computed on a fixed golden set of 500 examples; the golden set is versioned and its version is recorded per run.
- `cadence`: daily batch; also triggered on model / prompt / retriever change.
- `threshold`: metric delta > 5% absolute vs. rolling 7-day median *or* per-slice metric delta > 10% absolute for any protected-attribute slice.
- `register_effect`: residual state on `HR-CS-QUAL-03` moves to `control-defeated` on aggregate breach; on slice breach, `HR-CS-FAIR-08` (fairness-tagged row) additionally flagged.
- `alert_routing`: ML on-call + fairness liaison (on slice breach); risk-engineer digest daily.
- `evidence_pointer`: `evidence/sc-cs-qual-01/<yyyy-mm-dd>/report.json` with golden-set version and per-slice breakdown.

### Row `SC-CS-EMB-01` — hourly MMD on prompt-embedding distribution

- `monitor_id`: `alibi-detect-mmd-embeddings-hourly`
- `signal_semantics`: MMD (RBF kernel, median-heuristic bandwidth) on user-prompt embeddings vs. a reference sample of embeddings taken at last release.
- `cadence`: hourly batch on the past hour's prompts (sub-sampled to a fixed test-set size).
- `threshold`: permutation-test p-value < 0.01 for three consecutive hours.
- `register_effect`: residual state on `HR-CS-DRIFT-02` moves to `control-defeated`; the drifted-cluster description is attached to the register row.
- `alert_routing`: ML on-call; risk-engineer digest.
- `evidence_pointer`: `evidence/sc-cs-emb-01/<yyyy-mm-dd-hh>/report.json` with sample identifiers and MMD statistic.

Every one of the rows above cites `signal_source_owner: ai-infra-mlops` — the pipeline itself is theirs. Your contract row is the join to the register.

## Where the boundary to `ai-infra-mlops` sits

The peer track for MLOps monitoring depth (peer L25) covers:

- pipeline architecture (log emitters, profile storage, metric-store integration),
- detector plumbing (batch schedulers, streaming windows, backfill orchestration),
- monitor SLOs (detector uptime, false-positive rate tuning, retention),
- integration with observability platforms (Prometheus, OpenTelemetry, feature-store change events).

You do not own any of that. You *cite* the monitor identifier, the semantics of its output, the cadence, the threshold, and the register effect. When the pipeline itself is off, that is a monitor SLO breach owned by the MLOps track — which itself becomes a signal on a control's availability. Wire that too, but do not re-own the pipeline.

## Where the boundary to `ai-eval-engineer` sits

Application-layer online evaluation (peer L30) covers:

- session-level task-quality evaluation,
- LLM-as-judge harnesses and their calibration loop,
- app-eval online loops that continuously score user-facing quality,
- experimentation / A-B / bandit surfaces for eval.

You do not own the eval loop. You cite its outputs (aggregate quality delta, judge-calibration drift, per-slice quality delta) as signals in the contract. The seam is worth naming twice: the eval loop's *shape* is theirs; the risk-register *effect* of the eval loop's output is yours.

## Interfaces with adjacent modules

- **mod-101 ch 01** — MEASURE 2.4 and MANAGE 4.1 close here.
- **mod-103 ch 03 / ch 06** — the drift-tagged and quality-tagged rows this chapter's signals feed.
- **mod-104 / mod-105 / mod-106** — the eval and fairness suites whose runs supply the "quality metric" the golden-set eval uses.
- **mod-108** — guardrail firing rates as a related signal family (elaborated in chapter 03).
- **mod-111** — S1 tripping cascades to incident response.
- **mod-112** — drift-and-quality trend appears in the governance-council digest.

## Summary

- Drift and quality are the first family of signals in the contract; they are the most mature to wire and the most commonly wired incorrectly (as a dashboard, not a monitor).
- Match the metric to the shape of the shift: KS / PSI for univariate; MMD for multivariate; JS for discrete / score histograms; quality regression for concept drift.
- Whylogs, Evidently AI, and Alibi Detect are the working open-source toolset; WhyLabs is the commercial hosted platform on top of whylogs.
- Per-slice monitoring is a distinct signal family, not an add-on to aggregate. Skipping it is a common finding.
- Every signal contract row cites the peer-L25 (`ai-infra-mlops`) boundary on pipeline ownership and the peer-L30 (`ai-eval-engineer`) boundary on the eval loop's shape.
- Exercise 02 wires the whylogs + Evidently signals into the register for the running system.
