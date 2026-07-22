# Group-Fairness Tooling — Fairlearn and AIF360 as Day-to-Day Instruments

Chapter 01 fixed *which* metric you must compute; this chapter fixes *what you compute it with*. Fairlearn and AIF360 are the two open-source libraries a level-25 risk engineer is expected to be fluent in. They overlap heavily and diverge in ways that matter for engineering choices — which one to reach for, how to structure the evaluation output, how to wire the numbers into a dashboard, and how to keep the report reproducible under audit.

## Motivation

An engineering-flavoured question that shows up in every fairness review: *why did you use tool X here?* The wrong answer is "familiarity." The right answer is a two-sentence justification that names (a) the metric family, (b) the mitigation algorithm you intended to use if the metric fails, and (c) the artifact shape the auditor expects. This chapter gives you the mental model to produce that justification without hesitation.

## The two libraries at a glance

| Attribute | Fairlearn | AIF360 (AI Fairness 360) |
|---|---|---|
| Maintainer | Fairlearn project — hosted at `fairlearn.org` — Microsoft origin, now community-governed | LF AI & Data Foundation — originally IBM Research |
| Primary language / API | Python; sklearn-idiomatic (`fit` / `predict` / `MetricFrame`) | Python; also a Scala / Spark subset (`aif360.sklearn` is the sklearn-idiomatic bridge) |
| Metric surface | Compact, focused set (`selection_rate`, `demographic_parity_difference`, `equalized_odds_difference`, `true_positive_rate`, `false_positive_rate`, calibration helpers) via `MetricFrame` | Broader catalogue (70+ metrics across `BinaryLabelDatasetMetric` and `ClassificationMetric`) including many statistical parity variants and information-theoretic measures |
| Mitigation catalogue | Reduction algorithms (`ExponentiatedGradient`, `GridSearch` over Lagrangian relaxations); post-processing (`ThresholdOptimizer`) | Pre-, in-, and post-processing algorithms as separate classes; larger algorithm zoo (Reweighing, Learning Fair Representations, Adversarial Debiasing, Meta-Fair Classifier, Prejudice Remover, Reject Option Classification, Calibrated Equalised Odds Post-Processing, etc.) |
| Fitness for reporting | `MetricFrame` produces a clean per-group table that drops straight into a report; `fairlearn.metrics` supports intersectional slices via `sensitive_features=[a, b]` | `BinaryLabelDataset` structure carries protected-attribute metadata through mitigation; more boilerplate, more configurability |
| Dependency footprint | Small (numpy, scipy, sklearn, pandas) | Larger (adds tensorflow / pytorch for some mitigations; optional but common) |
| Documentation shape | Task-oriented user guide + API reference | Tutorial notebooks + API reference + algorithm-paper cross-references |
| License | MIT | Apache 2.0 |

Neither library is a superset of the other. In practice you use *both* — Fairlearn to author the primary evaluation and the reporting `MetricFrame`, and AIF360 to reach for a specific mitigation algorithm that Fairlearn does not implement.

## Fairlearn — the assessment primitive you write against

The core primitive is `MetricFrame`. It groups any metric callable(s) by the values of one or more sensitive features and returns a per-group table plus aggregation. The report structure downstream artifacts consume is a `MetricFrame` DataFrame written to disk with a config hash.

Shape you author (conceptual — see project documentation for exact API):

```python
from fairlearn.metrics import (
    MetricFrame, selection_rate, true_positive_rate, false_positive_rate,
    demographic_parity_difference, equalized_odds_difference,
)
from sklearn.metrics import accuracy_score

metrics = {
    "accuracy": accuracy_score,
    "selection_rate": selection_rate,
    "tpr": true_positive_rate,
    "fpr": false_positive_rate,
}

mf = MetricFrame(
    metrics=metrics,
    y_true=y_true,
    y_pred=y_pred,
    sensitive_features=df[["race", "sex"]],   # intersectional slice
)

# Report table:
per_group = mf.by_group                        # per-cell DataFrame
worst = mf.group_min()                         # min-across-groups
gap = mf.difference(method="between_groups")   # max minus min per metric

dp_diff = demographic_parity_difference(y_true, y_pred, sensitive_features=df["race"])
eo_diff = equalized_odds_difference(y_true, y_pred, sensitive_features=df["race"])
```

Two engineering disciplines Fairlearn *enforces* that AIF360 makes optional:

- **Intersectional slicing is one call.** Pass a DataFrame of two or more columns as `sensitive_features` and every metric is computed on the Cartesian product. Non-intersectional evaluations are one of the most common failure modes flagged by ForHumanity / BABL AI-style external audits; Fairlearn's API makes the intersectional cut the path of least resistance.
- **Metric callables are just functions.** You can drop in any function with the shape `f(y_true, y_pred, sample_weight=None)` and it participates in `MetricFrame`. That means the same infra computes accuracy, calibration by group, custom domain-specific parity — you do not fork the eval when you add a metric.

### Fairlearn mitigation — `ThresholdOptimizer` and `ExponentiatedGradient`

The mitigation surface is deliberately compact:

| Algorithm | Where it operates | Cost |
|---|---|---|
| `ThresholdOptimizer` | Post-processing: learns a per-group threshold on the score to satisfy a named constraint (`demographic_parity`, `equalized_odds`, `true_positive_rate_parity`, `false_positive_rate_parity`) | Cheap; requires access to the score and the sensitive attribute at inference time |
| `ExponentiatedGradient` | In-processing reduction: wraps a sklearn-style estimator and solves a Lagrangian relaxation over the fairness constraint | Trains a *set* of classifiers; you serve a randomised ensemble or pick a Pareto point |
| `GridSearch` | In-processing reduction: sweeps Lagrangian multipliers to trace the fairness / accuracy Pareto frontier | Trains many candidates; expensive but produces the trade-off *curve* that mod-106 exercise 04 asks for |

`ThresholdOptimizer` is the workhorse post-processor and the one you reach for first because it is auditable — the mitigation is a per-group threshold you can print. Reductions are more powerful and less inspectable.

## AIF360 — the algorithm zoo you borrow from

AIF360's value is that it packages a specific paper-per-algorithm catalogue. When the mitigation you need has a named algorithm behind it, AIF360 is where the reference implementation lives.

| Bucket | Representative algorithms in AIF360 |
|---|---|
| **Pre-processing** | Reweighing (Kamiran & Calders); Disparate Impact Remover (Feldman et al.); Learning Fair Representations (Zemel et al.); Optimised Preprocessing (Calmon et al.) |
| **In-processing** | Adversarial Debiasing (Zhang et al.); Prejudice Remover (Kamishima et al.); Meta-Fair Classifier (Celis et al.); Gerry-Fair Classifier (Kearns et al.); Exponentiated Gradient Reduction (mirrors Fairlearn) |
| **Post-processing** | Equalised Odds Post-Processing (Hardt et al.); Calibrated Equalised Odds Post-Processing (Pleiss et al.); Reject Option Classification (Kamiran, Karim, & Zhang) |

The `aif360.sklearn` submodule wraps many of these with sklearn-idiomatic `fit` / `transform` / `predict` methods, which lets you compose them into a Fairlearn-authored evaluation without rewriting the outer loop.

### Which library for which job — the reach-for table

| Job | Reach for | Reasoning |
|---|---|---|
| Author the primary assessment table for a report | Fairlearn `MetricFrame` | Cleanest reporting output; intersectional slicing is one call; smallest dependency footprint |
| Post-hoc threshold optimisation to satisfy equalised odds | Fairlearn `ThresholdOptimizer` | Auditable per-group threshold; drops into a serving path |
| Reweighing training data before model fit | AIF360 `Reweighing` | Named algorithm, cited by regulators, reference implementation |
| Adversarial debiasing during training | AIF360 `AdversarialDebiasing` | The reference paper is Zhang et al. 2018; AIF360 tracks it |
| Trace the accuracy / fairness Pareto frontier | Fairlearn `GridSearch` or `ExponentiatedGradient` | The reductions API produces a set of classifiers labelled by their trade-off |
| Score-calibrated post-processing where score meaning must survive | AIF360 `CalibratedEqOddsPostprocessing` | Pleiss et al.; explicit about the calibration-vs-equalised-odds impossibility |
| Report calibration by group | Either — Fairlearn via a custom metric callable; AIF360 via `ClassificationMetric` | Both work; pick to match the surrounding code |

## The slice-reporting shape

The output of a group-fairness evaluation is a *slice report*. Its structure is not negotiable — every consumer (governance analyst filing NYC LL144, ML platform team, incident-response runbook) expects the same shape.

| Section | Content |
|---|---|
| Metric-choice five-liner | From chapter 01 — decision context, protected-attribute partition, primary harm, headline metric, metrics not headlined |
| Data provenance | Dataset ID, snapshot timestamp, row count, protected-attribute source, exclusion / imputation rules |
| Per-group table | `MetricFrame.by_group` — one row per group, one column per metric; intersectional variant if applicable |
| Aggregates | Selection rate per group; overall accuracy; per-metric max-minus-min; four-fifths ratio if regulated |
| Headline verdict | Pass / fail against tolerance `ε` for the headline metric; diagnostic-metric values |
| Mitigation state | If mitigation was applied, which algorithm, which pre-mitigation numbers, and the delta |
| Reproducibility manifest | Library versions (Fairlearn / AIF360), commit hash of the eval script, dataset content hash, random seed |

That table is what the NYC LL144 shape reads (chapter 06). It is what mod-112's model card fairness section links to. It is what an incident-triage playbook (mod-111) references when a fairness monitor (mod-109) fires.

## Dashboards and the continuous-slice view

Ship two dashboards from the same evaluation:

- **Release dashboard** — pass/fail against tolerance for the headline metric, per protected group, at release time. Blocking; the release manifest cites this dashboard.
- **Production drift dashboard** — the same slice, recomputed on production traffic on a cadence (daily / weekly per system). Alert conditions:
  - Headline metric per-group value drifts by more than `δ` from release baseline.
  - Four-fifths ratio drops below 0.8 for any group (regulated systems) or below the internal appetite (unregulated).
  - Selection rate for any group changes by more than `δ` between adjacent windows.

Both dashboards read from the same `MetricFrame` shape. That is deliberate — an on-call engineer looking at the production dashboard should see the same columns they signed off on at release, not a different report.

## Anti-patterns

Four failure modes worth naming so an evaluation reviewer catches them:

- **Non-intersectional-only.** The evaluation reports on `race` and `sex` separately but not on their intersection. The single-axis view can hide a group defined by the intersection where the metric fails badly. Fairlearn makes intersectional evaluation a one-line change; not doing it is a choice.
- **Missing tolerance.** The report gives a per-group table but does not state the pass rule. "Demographic-parity difference is 0.04" is not a verdict without an `ε`. Tolerances must be pinned to the metric-choice justification and defended, not chosen after seeing the number.
- **Mitigation applied before the assessment is written.** The team runs `ExponentiatedGradient`, ships the mitigated model, and reports the mitigated numbers as the assessment. The pre-mitigation numbers are missing, so the reader cannot see what was bought. Both must ship.
- **Sample-size blind.** A group with three examples in the evaluation set shows a metric value that is not statistically meaningful. Every per-group cell must carry the group size and, where the metric admits one, a confidence interval or a note about power. AIF360 does not enforce this; you must.

## Interfaces with adjacent modules

- **Mod-102 (harm modelling)** supplies the protected-attribute definitions and the primary-harm sentence.
- **Mod-103 (register)** consumes the per-group table; the residual score for each group's row is a function of the pass/fail against tolerance here.
- **Mod-109 (monitoring)** consumes the production-drift dashboard shape; the alert thresholds here are the alerts fired there.
- **Mod-110 (MRM / supply chain)** — when the model comes from a third-party vendor, the vendor's own fairness assessment must land in this shape or it does not count.
- **Mod-112 (model card)** links the model card's fairness section to the slice-report manifest.

## Summary

- Fairlearn is the assessment / reporting workhorse; AIF360 is the algorithm zoo you borrow specific mitigations from. Use both.
- `MetricFrame` is the reporting primitive; author every evaluation through it so intersectional slicing and dashboarding are one shape.
- Fairlearn mitigation: `ThresholdOptimizer` (post-processing, auditable), `ExponentiatedGradient` / `GridSearch` (in-processing reductions).
- AIF360 mitigation: named-paper algorithms — Reweighing, Adversarial Debiasing, Calibrated Equalised Odds Post-Processing, etc.
- Slice report is a fixed shape: metric-choice five-liner, data provenance, per-group table, aggregates, verdict, mitigation state, reproducibility manifest.
- Ship a release dashboard (blocking) and a production drift dashboard (alerting) from the same shape.
- Anti-patterns: single-axis-only, missing tolerance, hidden pre-mitigation numbers, sample-size blindness.
- Exercise 02 authors both a Fairlearn and an AIF360 evaluation on the same running model and produces the shared slice report.
