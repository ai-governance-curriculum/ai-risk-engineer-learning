# Exercise 04 — Mitigation Loop and Accuracy / Fairness Trade-off

**Estimated effort:** 3 hours

## Objective

Run the chapter-04 mitigation loop end-to-end on the running model. Sweep pre-processing, in-processing, and post-processing mitigations aimed at the headline metric from exercise 01. Produce the **accuracy / fairness Pareto** as a *curve* (not a point), pick an **operating point** with a written policy justification, and account for the trade-off — including per-group accuracy — so the pick can be defended under external audit. The output plugs into the mod-103 register update and the mod-112 model-card fairness section.

## Prerequisites

- Chapter 04 read; chapter 01 (metric choice) and chapter 02 (Fairlearn / AIF360 tooling) available; chapter 03 read if the exercise-03 individual / causal diagnostics informed which mitigation family to prefer.
- Exercises 01, 02, and 03 completed. The unmitigated baseline from exercise 02 is the anchor point on the Pareto; the intersectional cell exercise 03 flagged is where per-group accuracy erosion is watched most closely.
- The running model retrainable (for pre- and in-processing mitigations) *and* scorable post-hoc (for post-processing). If retraining is infeasible in your environment, scope the sweep to post-processing plus one pre-processing baseline, and state the scope in the report — but do not silently drop in-processing.
- The training set and eval set separated; both carry the protected-attribute columns from exercise 02. **Never sweep on the eval set** — hold-out discipline is what makes the Pareto trustworthy.
- Fairlearn (`ThresholdOptimizer`, `ExponentiatedGradient`, `GridSearch`) and AIF360 (`Reweighing`, `AdversarialDebiasing`, `CalibratedEqOddsPostprocessing`, and any other algorithm on the "which mitigation lands where" table for your headline metric) installed at pinned versions.
- Access to the mod-103 register rows so the residual-risk cells can be updated with the operating-point numbers.

## Problem statement

For the headline metric from exercise 01, sweep at least *one algorithm per bucket* — pre-, in-, post-processing — from the chapter-04 "which mitigation lands where" table. For each algorithm:

- Sweep the tuning parameter (constraint tolerance ε for reductions, per-cell weights for reweighing, threshold slack for post-processing) across at least three settings that span the interesting range. Fairlearn `GridSearch` traces the curve directly for reductions; hand-roll the sweep for pre- and post-processing.
- For each swept setting, retrain (pre- / in-processing) or rescore (post-processing) the model, then re-run the exercise-02 slice-report shape on the mitigated model. Record: overall accuracy, per-group accuracy, headline-metric value per group, four-fifths ratio, and the intersectional headline-metric value.
- Plot the Pareto frontier with headline metric on the x-axis, overall accuracy on the y-axis, and the unmitigated baseline as a starred marker. Overlay per-group accuracy as a secondary series or as a second panel.
- Pick an **operating point** with a written justification citing the harm-model row, the risk appetite band, and the reason the point is preferable to its neighbours.
- Compute the **cost accounting**: overall accuracy delta vs baseline, per-group accuracy delta vs baseline, latency delta (post-processing changes the serving path; in-processing may change the model class), retraining cost, whether the mitigation is legally deployable under the jurisdiction (post-processing that reads the protected attribute at inference time is illegal in some US employment settings — see chapter 04 anti-patterns and chapter 06 §3).

The output is not "we mitigated." The output is "we mitigated to *this named point* on the Pareto, which costs `X` accuracy for `Y` fairness against harm `H`, and here is why *this point* and not its neighbours."

## Requirements

Produce all nine items below. Missing any is an audit finding.

1. **Mitigation catalogue** — `risk/mod-106/<system-id>/mitigation/catalogue.md`. Lists every algorithm considered, per the chapter-04 metric-family table for your headline. For each algorithm: which bucket, which reference paper, which library (Fairlearn / AIF360), whether it was swept or *rejected* (with reason: legal, infeasible, wrong metric family, etc.). The rejection reasons are as important as the sweep list.
2. **Sweep config** — `risk/mod-106/<system-id>/mitigation/sweep.yaml`. One block per swept algorithm: hyperparameter grid, seed, split spec (train / mitigation-train / eval), and the eval-metric callable set from exercise 02.
3. **Sweep runners** — one script per bucket:
   - `risk/mod-106/<system-id>/mitigation/run_pre.py` — pre-processing sweep (Reweighing, Disparate Impact Remover, Learning Fair Representations, or sampling — at least one).
   - `risk/mod-106/<system-id>/mitigation/run_in.py` — in-processing sweep (ExponentiatedGradient, Adversarial Debiasing, or one other — at least one).
   - `risk/mod-106/<system-id>/mitigation/run_post.py` — post-processing sweep (ThresholdOptimizer, Calibrated Equalised Odds, or Reject Option Classification — at least one).
   Each script emits JSONL results with one row per (algorithm, hyperparameter, seed): overall accuracy, per-group accuracy (single-axis and intersectional), headline metric per group, four-fifths ratio.
4. **Pareto frontier plot** — `risk/mod-106/<system-id>/mitigation/pareto.png` (or SVG). Headline metric on the x-axis, overall accuracy on the y-axis, one marker per swept point (colour-coded by algorithm), the unmitigated baseline starred, the operating point circled and labelled. A second panel or a coloured overlay shows per-group accuracy at each point.
5. **Pareto table** — `risk/mod-106/<system-id>/mitigation/pareto-table.md`. Every point on the frontier as a row: algorithm, hyperparameter, headline metric per group, overall accuracy, per-group accuracy, four-fifths ratio, latency delta note, legal-deployability flag.
6. **Operating-point justification** — `risk/mod-106/<system-id>/mitigation/operating-point.md`. Named point, its numbers, the rejected alternatives, the harm-model row cited, the risk-appetite band cited, the reversibility statement, and a paragraph naming what a reviewer preferring the neighbour point would argue and why you disagreed. This paragraph is the load-bearing artefact — do not skimp on it.
7. **Cost-accounting block** — `risk/mod-106/<system-id>/mitigation/cost-accounting.md`. Overall accuracy delta vs baseline, per-group accuracy delta vs baseline (all groups, including the "no worse for anyone" check), latency delta, retraining cost, legal-deployability check (jurisdictions where the chosen mitigation is deployable).
8. **Mitigation report** — `risk/mod-106/<system-id>/mitigation/report.md`. The chapter-04 fixed section shape: pre-mitigation baseline (link to exercise 02), catalogue considered, sweep details, Pareto plot + table, chosen operating point, policy justification, cost accounting, reversibility statement.
9. **Register update draft** — `risk/mod-106/<system-id>/mitigation/register-update.md`. The specific mod-103 register-row changes the operating-point pick induces: residual score per fairness-harm scenario, plus a new control-defeated row for "mitigation bypassed / disabled" per chapter 04 §Interfaces.

Cross-cutting rules:

- **Never sweep on the eval set.** Split the training set into a training-plus-mitigation-fit set and use the untouched eval set for reporting. Sweeping on the eval set inflates fairness numbers and understates the cost.
- **Per-group accuracy is a first-class column.** A "fairness win" that comes from breaking the classifier for the unprivileged group is not a win. The report's Pareto table must show per-group accuracy at every point.
- **Randomised classifiers are not deterministic.** `ExponentiatedGradient` returns a distribution over classifiers; the served model is a *sample* from that distribution or a *majority vote* over a set. Cite the serving convention in the report; downstream monitoring cannot ignore it.
- **Post-processing needs the protected attribute at inference.** In US employment settings the direct use of race at decision time is prohibited even for the purpose of mitigation. Verify the jurisdiction with chapter 06 before recommending a per-group threshold in that setting; the legal-deployability flag on the Pareto table catches this.
- **Baseline is always shipped alongside mitigated numbers.** The pre-mitigation number is what the reader compares against. Reports missing the baseline hide the trade-off.
- **The operating-point paragraph names the loser.** "We picked X over Y because [harm-model reasoning]" — not "we picked X because it was best" (best on what axis, weighed against which alternative?).

## Starter guidance

- **Start with post-processing on the eval set** — cheap, no retraining, and you get a first Pareto in an hour. Then decide whether pre-/in-processing is worth the retraining cost given how the post-processing curve looks.
- **`ThresholdOptimizer` is your workhorse for the first Pareto pass.** For a headline of equal opportunity, `constraints="true_positive_rate_parity"` and sweep the `grid_size` (or use its default and record the resulting operating points).
- **`ExponentiatedGradient` needs a base learner and a constraint object.** For equal opportunity, `TruePositiveRateParity()`; for demographic parity, `DemographicParity()`; for equalised odds, `EqualizedOdds()`. Set `eps` across a grid; the wrapper returns a randomised classifier per setting — note the seeding convention.
- **Reweighing (AIF360) is a one-line pre-processing baseline.** It produces sample weights; feed them to your existing training loop. It rarely wins on Pareto against reductions or post-processing, but it is a defensible baseline that trains once.
- **Learning Fair Representations is expensive; treat it as a stretch.** LFR trains a representation network; on tabular data it is often overkill. If your model is a tree ensemble, skip LFR for the first pass.
- **Adversarial Debiasing needs TF / PyTorch and takes real training time.** Budget it accordingly; do not run it on a laptop over lunch.
- **Watch for the "no worse for anyone" test.** A mitigation that improves fairness but drops per-group accuracy for one group compared to the baseline is a Pareto candidate the harm model may still reject. Explicit test: is the unprivileged group's accuracy after mitigation ≥ its accuracy before? If not, name the trade-off.
- **The Pareto plot lies flat when the constraint is easy.** If your sweep produces a nearly-vertical cluster of points, either the mitigation surface is trivially good on your model (rare) or you are not sweeping the hyperparameter over its interesting range. Widen the range.
- **The operating-point justification is where audits die.** "Median of frontier" is not a justification. Cite the harm model, cite the risk appetite band, cite the reversibility. A five-line paragraph is the target.
- **Reversibility is a boring but important requirement.** Post-processing thresholds live in a config file; rolling back is a config change. Pre-processing lives in the training data; rolling back is a retraining. Name the rollback path in the report.

## Acceptance criteria

The submission is accepted when:

- The catalogue exists and lists at least one algorithm per bucket that was swept, plus the algorithms rejected with reasons.
- The sweep config exists with a hyperparameter grid per algorithm, seed, and split spec.
- The three sweep runners execute end-to-end and emit their JSONL results.
- The Pareto plot exists with the frontier marked, the baseline starred, the operating point circled, and per-group accuracy visible on a second panel or overlay.
- The Pareto table lists every point with per-group accuracy, four-fifths ratio, latency-delta note, and legal-deployability flag.
- The operating-point justification cites a harm-model row, a risk-appetite band, and includes a paragraph naming what a reviewer preferring the neighbour would argue.
- The cost-accounting block includes the "no worse for anyone" test with per-group accuracy delta vs baseline.
- The mitigation report is authored in the fixed chapter-04 shape.
- The register-update draft names the specific mod-103 row IDs the operating-point pick moves.
- All sweep results are reproducible from pinned tool versions, seeds, and split fingerprints.

## Stretch goals

- Run **`GridSearch`** as well as `ExponentiatedGradient` — `GridSearch` returns a Pareto set directly (one classifier per Lagrangian setting) and is a cleaner shape for the Pareto plot.
- Run the **calibrated equalised odds post-processing** (Pleiss et al.; AIF360 `CalibratedEqOddsPostprocessing`) alongside `ThresholdOptimizer` to make the Chouldechova collision visible in the Pareto — calibration and equalised odds cannot both hold, and the two post-processors sit at opposite corners of that trade-off.
- Add a **cost-sensitive sweep** — vary a per-group misclassification cost in the base learner and trace a second Pareto. Compare against the constrained sweep; when they agree, confidence is high, when they diverge, the report includes the divergence.
- Contribute a **serving-side mitigation adapter** at `risk/mod-106/<system-id>/mitigation/serving-adapter.py` that applies the chosen post-processing threshold at inference time and logs the applied threshold per request. This is the intake artefact for mod-109 (monitoring drift on the applied threshold) and mod-111 (incident-response rollback path).
- Draft the **mod-112 model-card fairness section update** — named operating point, named algorithm, named accuracy cost, named per-group accuracy, named reversibility path. The card cannot be signed without this section.

## Deliverable format

A single markdown document `mod-106-ex04-mitigation.md` that indexes the artefacts, plus the sweep configs, runners, JSONL results, Pareto artefacts, and report / operating-point / cost-accounting / register-update documents. The index document contains:

1. *Scope statement* — which headline metric is being mitigated; which buckets were swept; which were rejected with reasons.
2. *Pre-mitigation baseline* — one line linking to the exercise-02 slice-report headline number.
3. *Pareto plot* — embedded with a one-paragraph caption naming the frontier shape and the operating point.
4. *Pareto table excerpt* — the frontier rows plus the operating point highlighted.
5. *Operating-point justification* — the paragraph in full.
6. *Cost-accounting summary* — the "no worse for anyone" test result plus overall accuracy delta and latency-delta note.
7. *Legal-deployability check* — the jurisdictional note for the chosen mitigation (chapter 06 forward-reference).
8. *Register handoffs* — mod-103 row IDs updated with new residual scores and any new control-defeated row.
9. *Reproducibility manifest* — tool versions, seeds, split fingerprints, script commit hashes.
10. *Open items* — anything for exercises 05 (which post-mitigation instances need explainability attention) or 06 (how the operating point reads into the LL144 impact-ratio verdict).

## Submission

Attach the index markdown, the catalogue, the sweep config, the three runners, the JSONL results, the Pareto plot and table, the operating-point / cost-accounting / register-update documents to the module submission channel. If the model / dataset is under NDA, redact identifiers but preserve the Pareto shape, the operating-point numbers with per-group accuracy, and the justification paragraph — the mitigation report cannot be reviewed if the trade-off numbers are blanked out.
