# Exercise 02 — Fairlearn + AIF360 Evaluation Authoring

**Estimated effort:** 3 hours

## Objective

Author a full group-fairness evaluation of the running model *twice* — once with **Fairlearn** as the primary reporting tool, once with **AIF360** as the algorithm-zoo counterpart — and reconcile the outputs into the single **slice-report** shape chapter 02 specifies. The exercise produces the artifact that exercises 03–06 read from: a per-group table with intersectional slicing, the headline-metric verdict against the tolerance from exercise 01, the four-fifths ratio, per-cell counts, and a reproducibility manifest that will survive external audit.

## Prerequisites

- Chapter 02 read; chapter 01 available for the metric-choice five-liner header.
- Exercise 01 completed — the five-liner for the running system is what this exercise's evaluation reads as its metric declaration. If the five-liner does not exist, do exercise 01 first.
- The running model reachable in one of two shapes:
  - **In-process path** — a Python-loadable model (`sklearn`, `xgboost`, `lightgbm`, a saved PyTorch / TensorFlow model with a `predict` wrapper). Fairlearn `MetricFrame` and AIF360 both consume `y_true` / `y_pred` / `sensitive_features` arrays directly.
  - **Endpoint path** — a scoring endpoint you can hit for the eval set; wrap it in a thin `predict()` adapter and treat the wrapper as the model.
- A **held-out evaluation set** — separate from training, drawn from the same distribution as the production input. Rows carry the protected-attribute columns you named in the exercise-01 partition line (typically race / ethnicity per EEO-1 categories and sex, plus any domain-specific analogue). If the eval set does not carry protected attributes, resolve that upstream *before* running — see the anti-pattern in chapter 02 §"Sample-size blind".
- Fairlearn and AIF360 installed at pinned versions. Note the versions in the manifest.
- Access to the mod-103 register rows so the per-group verdicts populate the residual-risk cells for exercise 06.

## Problem statement

Stand up the chapter-02 evaluation shape on the running model. Two evaluations, same model, same eval set:

- **Fairlearn evaluation.** Use `MetricFrame` to compute the per-group table (`selection_rate`, `true_positive_rate`, `false_positive_rate`, `accuracy`, and any custom callable required by the headline metric from the five-liner) sliced by each single-axis protected attribute *and* by the intersectional `(sex, race/ethnicity)` cut. Compute the headline metric's disparity primitive (`demographic_parity_difference`, `equalized_odds_difference`, or the four-fifths ratio computed from `selection_rate`s) and record it against the tolerance ε from the five-liner. Attach the metric-choice five-liner as the report header.
- **AIF360 evaluation.** Load the same eval set as a `BinaryLabelDataset`. Compute the equivalent metrics through `ClassificationMetric` and `BinaryLabelDatasetMetric` — statistical parity difference, disparate impact ratio, equal opportunity difference, average odds difference. The AIF360 numbers must agree with the Fairlearn numbers to within numerical tolerance; disagreements are diagnosed, not ignored.

For each evaluation:

- Report **per-cell counts** alongside the per-cell metric values. A cell with `n < 30` is flagged as low-power and its metric value carries a caveat; do not delete the cell (auditor will ask why it disappeared).
- Report **95% confidence intervals** (or Wilson intervals for rate metrics) on the per-group selection rate and per-group TPR / FPR. Fairlearn does not produce these directly; you build them from the per-cell counts.
- Emit a **release-verdict block**: pass / fail against ε for the headline metric per protected group, per intersectional cell, and overall.
- Emit a **reproducibility manifest**: tool versions, eval-set fingerprint (content hash of the file plus row count), model artefact hash, random seed for any sampling, the commit hash of the eval script.

The evaluation is unmitigated. Mitigation is exercise 04. If the report shows a fail against ε here, that is the *baseline* — it is what mitigation will move from.

## Requirements

Produce all eight items below. Missing any is an audit finding on the eval.

1. **Evaluation config** — `risk/mod-106/<system-id>/group-fairness/config.yaml`. One block per protected-attribute partition (single-axis race, single-axis sex, intersectional sex × race/ethnicity). Each block names: the categorical vocabulary (EEO-1 or the domain equivalent), the headline metric, the diagnostic metrics, the tolerance ε, and the five-liner file path from exercise 01.
2. **Fairlearn runner** — `risk/mod-106/<system-id>/group-fairness/run_fairlearn.py`. Loads the model, loads the eval set, builds a `MetricFrame` with the metric dictionary from chapter 02, computes the disparity primitives, computes the four-fifths ratio, adds bootstrapped or Wilson CIs to the per-group rates, and emits the per-group DataFrame plus the aggregates as JSON + a rendered markdown table.
3. **AIF360 runner** — `risk/mod-106/<system-id>/group-fairness/run_aif360.py`. Loads the same eval set into a `BinaryLabelDataset`, builds `ClassificationMetric` for the same partitions, computes the equivalent metric set, emits the same JSON + markdown table shape.
4. **Reconciliation script** — `risk/mod-106/<system-id>/group-fairness/reconcile.py`. Reads both JSONs, joins on partition + cell + metric, computes the numerical delta, flags any cells where the two libraries disagree beyond a stated tolerance (e.g., `> 1e-6` absolute for the same metric), and emits a short markdown table of disagreements. If disagreements exist, the reconciliation script's output includes a diagnosis (different denominator conventions? different treatment of ties? different handling of missing labels?).
5. **Per-cell counts and CIs** — every per-cell row in both markdown tables carries a count column and, for rate metrics, a Wilson 95% CI. Cells with `n < 30` carry a `low-power` flag column.
6. **Slice report** — `risk/mod-106/<system-id>/group-fairness/slice-report.md`. The chapter-02 fixed shape: metric-choice five-liner header, data provenance block, per-group table (intersectional variant separately), aggregates (max-minus-min per metric, four-fifths ratio), headline verdict against ε, mitigation state ("unmitigated baseline"), reproducibility manifest.
7. **Release dashboard stub** — a rendered image or a `plotly` / `matplotlib` figure showing the headline metric per group vs the ε band, with the four-fifths line marked. This is the artefact the release manifest cites in mod-112.
8. **Reproducibility manifest** — the manifest block called out separately in `slice-report.md`, plus a second copy at `risk/mod-106/<system-id>/group-fairness/manifest.json` for machine consumption in exercise 06.

Cross-cutting rules:

- **The five-liner is the header, unedited.** Do not adjust the metric-choice text based on numbers you see. If numbers pull you toward a different metric, that is a mod-102 / exercise-01 change control and requires a dated note.
- **Intersectional is not optional.** If the report table lacks the intersectional cut, it does not discharge chapter-02 and it does not discharge the LL144 shape in exercise 06.
- **Both libraries or neither.** A report that runs Fairlearn only, or AIF360 only, misses the reconciliation check that catches cell-definition bugs. Both must ship, and their numbers must agree or be diagnosed.
- **Never delete a low-power cell.** Flag it, note the caveat, keep the row. Deleting cells is a documented audit failure mode.
- **The manifest is not an appendix.** Every claim in the slice report must be reproducible from the manifest. If the manifest is missing a field, a downstream user cannot reproduce your run.

## Starter guidance

- **Prove the plumbing on one cell first.** Compute `selection_rate` by hand for one intersectional cell (e.g., `race=Asian AND sex=Female`), compare to what `MetricFrame` reports, and only then loop over all cells. The plumbing errors — denominator convention, label encoding, one-hot vs categorical — bite in the first hour.
- **Fairlearn's `sensitive_features` argument accepts a DataFrame; use one.** Passing a DataFrame with two or more columns produces the intersectional slice as a single call. Passing a Series only produces single-axis. This is the one-line difference that separates "single-axis only" (chapter 02 anti-pattern) from "intersectional-ready" (chapter 02 requirement).
- **AIF360 wants a `BinaryLabelDataset` with the protected-attribute column identified explicitly.** The `protected_attribute_names` and `privileged_classes` arguments are load-bearing; the metrics compute against whichever class you flag as privileged. Cite the choice in the config and the report — reversing it flips the sign of the disparate impact ratio.
- **Reconciliation catches denominator bugs.** Fairlearn's `selection_rate` counts `mean(y_pred == 1)`; AIF360's disparate impact ratio uses selection rates as well but the internal API can surprise on missing values. If your reconciliation flags a mismatch, look at how each library treats `NaN` labels first.
- **Bootstrap CIs on the *rate*, not on the *disparity*.** The disparity is a function of the rates; bootstrapping it directly loses interpretability. Bootstrap per-cell selection rate, then compose.
- **Add the four-fifths ratio explicitly as a callable in the Fairlearn metric dict.** A `four_fifths_ratio` metric callable that computes `min_group_selection_rate / max_group_selection_rate` and returns a scalar is 4 lines; the LL144 shape (exercise 06) reads it as the impact-ratio number.
- **Small groups are a reporting problem, not a computation problem.** The Wilson interval on a group of 12 will be enormous; that is a *true statement about the evidence*, not a bug to hide. The verdict for that cell is "insufficient power," not "pass" or "fail".
- **Version-pin everything.** The evaluation must reproduce on a second machine with the same output down to the last digit; that requires pinning Fairlearn, AIF360, numpy, pandas, and the model's runtime. Note the pins in the manifest.

## Acceptance criteria

The submission is accepted when:

- The config file exists with a block for every partition (at least single-axis race, single-axis sex, intersectional sex × race/ethnicity), each citing the five-liner from exercise 01.
- The Fairlearn and AIF360 runners execute end-to-end on the eval set and emit their JSON + markdown outputs.
- The reconciliation script emits a disagreement table; if disagreements exist, each has a diagnosis line.
- The slice report is authored in the fixed chapter-02 shape and includes the intersectional per-group table with counts and Wilson CIs.
- The headline-metric verdict is present per single-axis group *and* per intersectional cell *and* overall, each against the ε from the five-liner.
- The four-fifths ratio is computed for the LL144-shape scenario and is a first-class number in the report, not an appendix.
- Cells with `n < 30` are flagged, not deleted; their metric values carry a `low-power` caveat.
- The release dashboard stub is embedded in the slice report with a one-line caption.
- The reproducibility manifest pins Fairlearn version, AIF360 version, numpy / pandas / model runtime versions, eval-set fingerprint, model artefact hash, and script commit hash. A second engineer can reproduce every number from the manifest alone.
- The register rows to update in exercise 06 are called out in the open-items section.

## Stretch goals

- Add a **calibration-by-group** diagnostic (score reliability curve per protected group) as a custom `MetricFrame` callable. It is the diagnostic the clinical / credit-scoring row of the chapter-01 decision table names; even if your headline is elsewhere, calibration-by-group is a cheap add and answers a common reviewer question.
- Add a **predictive-parity** number (per-group precision) alongside the headline. When headline and predictive parity diverge, the divergence is exactly the Chouldechova collision — call it out in the report body with a two-line paragraph.
- Wire a **production-drift monitor stub** for mod-109: a small script that re-runs the same slice-report shape on a production sample daily / weekly and emits the headline metric per group as a metric. The monitor itself lands at mod-109; this is the intake artefact.
- Contribute a **vendor-facing slice-report template** at `risk/mod-106/<system-id>/group-fairness/vendor-template.md` — the shape a third-party model vendor's fairness assessment must land in to be consumable in mod-110. The template makes third-party assessments comparable rather than free-form.
- Add an **age-band or disability partition** if the running system is subject to ADEA / ADA scrutiny (US employment / benefits); the categorical vocabulary differs from EEO-1 and the exercise of pinning it explicitly is worth the time.

## Deliverable format

A single markdown document `mod-106-ex02-group-fairness.md` that indexes the artefacts, plus the slice report, config, runners, and manifest. The index document contains:

1. *System and eval-set summary* — one paragraph naming the model under evaluation, the eval-set provenance, and whether Fairlearn / AIF360 ran in-process or against an endpoint wrapper.
2. *Metric-choice header* — the five-liner from exercise 01 in a fenced block, verbatim.
3. *Per-group results — single-axis* — one table per single-axis partition with counts, rates, CIs, and pass / fail against ε.
4. *Per-group results — intersectional* — one table for the sex × race/ethnicity cut with counts, rates, CIs, and pass / fail against ε.
5. *Four-fifths ratio* — number, cited by cell.
6. *Fairlearn / AIF360 reconciliation* — the disagreement table; empty is fine (state that explicitly).
7. *Release dashboard* — embedded figure with caption.
8. *Reproducibility manifest* — the block for human read plus a pointer to `manifest.json` for machine read.
9. *Register handoffs* — the mod-103 rows exercise 06's residual-risk update will touch.
10. *Open items* — anything for exercises 03–06 (particularly which subgroup exercise 03's individual-fairness diagnostic should focus on) or upstream mod-102 / mod-103.

## Submission

Attach the index markdown, the slice report, the config, the runners, the reconciliation output, the release dashboard image, and the manifest to the module submission channel. If the model / dataset is under NDA, redact the model / dataset identifiers but preserve the per-group and intersectional tables (with counts and CIs), the four-fifths ratio, the headline-metric verdict, and the reproducibility manifest — a reviewer cannot review a slice report whose numbers are blanked out.
