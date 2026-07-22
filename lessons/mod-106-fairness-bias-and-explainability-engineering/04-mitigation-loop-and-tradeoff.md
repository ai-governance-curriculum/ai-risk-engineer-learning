# The Mitigation Loop and the Accuracy / Fairness Trade-off

An assessment that finds a fairness violation is not the end product; it is the input to a *mitigation loop*. The loop's outputs are (a) a mitigated model, (b) a Pareto-shaped picture of what mitigation cost in accuracy, and (c) a defensible pick of an operating point on that Pareto. This chapter is that loop.

## Motivation

Two anti-patterns in first-pass mitigation work:

- **Single-point mitigation.** The team runs one mitigation algorithm at one hyperparameter, quotes the new fairness and accuracy numbers, and calls the trade-off resolved. The reviewer asks "what would happen at a different operating point?" and there is no answer. Regulators asking about proportionality (EU AI Act Article 9) get a one-word reply.
- **Race to the bottom on accuracy.** The team achieves a target fairness number by degrading the classifier to near-uselessness. Overall accuracy craters; the model is technically fair and no longer useful. This shows up especially with heavy-handed pre-processing.

The mitigation loop's job is to produce a *curve*, not a *point*, and to make the pick of a point on the curve a legible policy decision.

## The mitigation catalogue — pre-, in-, post-processing

Mitigation algorithms sit in three buckets by where they operate in the training / serving pipeline. Each bucket has different auditability, retraining cost, and interaction with the metric families from chapter 01.

### Pre-processing — reshape the training data

The training data itself carries the disparity; you modify the data before the model sees it.

| Algorithm | Mechanism | Reference |
|---|---|---|
| **Reweighing** (Kamiran & Calders, "Data Preprocessing Techniques for Classification without Discrimination") | Assign per-example weights so that `P(A, Y)` becomes independent in weighted counts | AIF360 `Reweighing`; the closed-form weights are a per-cell rescaling |
| **Sampling** (over-, under-, synthetic e.g., SMOTE-adjacent) | Change the group / label composition of the training set | Custom or via imbalanced-learn; less principled but sometimes the right first move |
| **Learning Fair Representations** (Zemel et al., "Learning Fair Representations") | Learn a representation `Z` of `X` that preserves task-utility signal while erasing `A` | AIF360 `LFR` |
| **Disparate Impact Remover** (Feldman et al., "Certifying and Removing Disparate Impact") | Repair individual features so that their per-group distributions match, at a tunable repair level | AIF360 `DisparateImpactRemover` |
| **Optimised Preprocessing** (Calmon et al.) | Solve a convex optimisation for a data transformation that meets a fairness constraint with minimum distortion | AIF360 `OptimPreproc` |

Trade-offs: pre-processing keeps the trained model *unchanged in shape* — same architecture, same serving path — which is auditable. It requires retraining, so mitigations are expensive to iterate. Downstream users of the training data may object to the reweighting.

### In-processing — constrain the training objective

The model itself is trained under a fairness constraint or with an adversarial signal that pushes back on the protected attribute.

| Algorithm | Mechanism | Reference |
|---|---|---|
| **Adversarial Debiasing** (Zhang, Lemoine, Mitchell, "Mitigating Unwanted Biases with Adversarial Learning") | Train the predictor jointly against an adversary trying to recover `A` from the predictor's output / representation | AIF360 `AdversarialDebiasing`; note deep-learning dependency |
| **Exponentiated Gradient Reduction** (Agarwal et al., "A Reductions Approach to Fair Classification") | Reduce constrained fair classification to a sequence of weighted classification problems solved by any base learner | Fairlearn `ExponentiatedGradient`; AIF360 mirror |
| **Prejudice Remover** (Kamishima et al., "Fairness-Aware Classifier with Prejudice Remover Regulariser") | Add a fairness regulariser to the training loss | AIF360 `PrejudiceRemover` |
| **Meta-Fair Classifier** (Celis et al., "Classification with Fairness Constraints: A Meta-Algorithm with Provable Guarantees") | Meta-algorithm over any base classifier that provably satisfies a chosen fairness constraint | AIF360 `MetaFairClassifier` |
| **Gerry-Fair Classifier** (Kearns et al., "Preventing Fairness Gerrymandering") | Handle subgroup / intersectional fairness with a game-theoretic reduction | AIF360 `GerryFairClassifier` |

Trade-offs: in-processing can find fairness / accuracy points pre- and post- cannot reach, at the cost of retraining, deep-learning stack, and (for reductions and adversarial methods) an ensemble or randomised classifier that complicates serving and audit.

### Post-processing — adjust the decision after the model runs

The model itself is unchanged; you modify the score-to-decision mapping.

| Algorithm | Mechanism | Reference |
|---|---|---|
| **Threshold Optimisation** (Hardt, Price, Srebro, "Equality of Opportunity in Supervised Learning") | Learn a per-group decision threshold to satisfy a named constraint (equal opportunity, equalised odds, demographic parity) | Fairlearn `ThresholdOptimizer`; AIF360 `EqOddsPostprocessing` |
| **Calibrated Equalised Odds Post-Processing** (Pleiss et al., "On Fairness and Calibration") | Adjust scores per group under an explicit acknowledgement of the calibration-vs-equalised-odds impossibility | AIF360 `CalibratedEqOddsPostprocessing` |
| **Reject Option Classification** (Kamiran, Karim, Zhang) | For predictions in a low-confidence band near the decision threshold, adjust in favour of the unprivileged group | AIF360 `RejectOptionClassification` |

Trade-offs: post-processing is cheap (no retraining), auditable (a per-group threshold you can print), and reversible. It requires the protected attribute at inference time, which is a legal / practical constraint in many settings (US employment settings often prohibit using `race` at decision time even for the purpose of mitigation).

## The "which mitigation lands where" table by metric family

Given a target fairness metric (chapter 01), the mitigation menu is not open-ended. Certain algorithms are aimed at certain metrics.

| Target metric | Pre-processing | In-processing | Post-processing |
|---|---|---|---|
| **Demographic parity** | Reweighing; Learning Fair Representations; Disparate Impact Remover | Exponentiated Gradient (with `DemographicParity` constraint); Adversarial Debiasing | Threshold Optimizer (`demographic_parity`); Reject Option Classification |
| **Disparate impact / four-fifths** | Disparate Impact Remover (name is on the box); Reweighing | Exponentiated Gradient with a four-fifths-shaped constraint | Threshold Optimizer with per-group thresholds calibrated to hit 0.8 ratio |
| **Equalised odds** | Reweighing (partial effect) | Exponentiated Gradient (`EqualizedOdds`); Adversarial Debiasing | Hardt Equalised Odds Post-Processing; Calibrated Equalised Odds Post-Processing |
| **Equal opportunity** | Reweighing (partial) | Exponentiated Gradient (`TruePositiveRateParity`) | Threshold Optimizer (`true_positive_rate_parity`) |
| **Predictive parity / calibration by group** | (Usually preserved by good training; recalibration per group) | (Difficult to constrain jointly with EO — Chouldechova) | Calibrated Equalised Odds Post-Processing (with explicit trade-off acknowledgement); per-group Platt / isotonic recalibration |
| **Individual fairness** | Feature-transformation pre-processing that respects the `d` metric | Adversarial Debiasing against protected-attribute reconstruction | Difficult — post-processing typically operates on groups |
| **Counterfactual fairness** | Fair-representation methods that break the causal path from `A` | In-processing constrained on the counterfactual (bespoke, often research-code) | Not standard |

The table is a starting point, not a substitute for the reference paper. Every mitigation you ship must cite the paper it is from and the tolerances it targets.

## The accuracy / fairness trade-off as a curve

The point of the loop is a Pareto curve, not a point. The curve is produced by *sweeping* a mitigation's tuning parameter and evaluating both fairness and accuracy at each point.

Concretely, using Fairlearn `GridSearch` or a hand-rolled sweep of any of the algorithms above:

1. Define the fairness metric and its tolerance range: `ε ∈ {0.00, 0.01, 0.02, ..., 0.20}` (or a per-algorithm hyperparameter).
2. For each `ε`, run the mitigation, evaluate the mitigated model on the eval set, and record `(fairness_metric_value, accuracy)` — and the per-group metrics.
3. Plot the resulting points; the Pareto frontier is the non-dominated subset.
4. Overlay the *unmitigated* baseline point on the same plot.

The report ships:

- The Pareto plot as a figure.
- A table of every point on the frontier with its hyperparameter setting, its fairness metric per group, its overall accuracy, and (crucial) its per-group accuracy — because a fairness win that comes from breaking the classifier for a specific group is not a win.
- The **operating point** chosen from the frontier, with a justification pointing at the risk-appetite band and the harm-model row.

That plot + table + justification is what the risk council reads. It is the difference between "we mitigated" (unactionable) and "we mitigated to a specific operating point that trades `X` accuracy for `Y` fairness against harm `H`" (a decision).

### A worked-example shape

Continuing the loan-approval example, target metric equal opportunity.

| Operating point | Mitigation | Hyperparameter | TPR (privileged) | TPR (unprivileged) | Equal-opp diff | Overall accuracy | Per-group accuracy (priv/unpriv) |
|---|---|---|---|---|---|---|---|
| Baseline (no mitigation) | — | — | 0.82 | 0.71 | 0.11 | 0.87 | 0.88 / 0.83 |
| Reweighing | — | 0.79 | 0.76 | 0.03 | 0.86 | 0.87 / 0.84 | |
| Threshold Optimizer (`tpr_parity`) | ε = 0.02 | — | 0.79 | 0.77 | 0.02 | 0.85 | 0.86 / 0.82 |
| Threshold Optimizer (`tpr_parity`) | ε = 0.05 | — | 0.80 | 0.76 | 0.04 | 0.86 | 0.87 / 0.83 |
| Exp Gradient (`TruePositiveRateParity`) | ε = 0.02 | — | 0.78 | 0.76 | 0.02 | 0.84 | 0.85 / 0.82 |
| Exp Gradient (`TruePositiveRateParity`) | ε = 0.05 | — | 0.80 | 0.75 | 0.05 | 0.86 | 0.87 / 0.83 |

*Figures illustrative — see the exercise for the actual sweep on real data.*

Reading it: the report picks Threshold Optimizer at ε = 0.05, cites (a) it satisfies the internal tolerance for equal-opportunity difference, (b) accuracy cost is 1 % overall, (c) neither per-group accuracy has cratered, (d) the auditable per-group threshold is preservable in the release manifest. That justification is what an audit consumes.

## Fairness / accuracy trade-off — the report structure

The mitigation section of the fairness report is a fixed shape:

| Section | Content |
|---|---|
| Pre-mitigation baseline | The chapter-02 slice report as-is on the unmitigated model |
| Mitigation catalogue considered | The algorithms tried, per the metric-family table above; the ones rejected and why |
| Sweep details | Hyperparameter grid; per-point config hash; random seed; sweep script commit hash |
| Pareto plot + table | As above |
| Chosen operating point | Named point with all its numbers |
| Policy justification | The paragraph linking the pick to the harm model and the appetite band |
| Cost accounting | Overall accuracy delta; per-group accuracy delta; latency delta if post-processing changes the serving path; retraining cost if in- or pre-processing |
| Reversibility statement | How to roll back to a prior operating point; where the state lives |

Everything after the sweep is the argument that turns a technical result into an accountable decision.

## Anti-patterns

- **Sweep without per-group accuracy.** The plot shows fairness improving and overall accuracy holding, but per-group accuracy for the unprivileged group has collapsed. Always report per-group accuracy alongside overall.
- **Post-processing that requires `A` at inference in a setting where `A` cannot be collected.** Threshold Optimizer with per-group thresholds is illegal in some jurisdictions for some decisions. Check chapter 06 before shipping.
- **Reductions ensemble served silently.** `ExponentiatedGradient` returns a randomised classifier. If the serving path pretends this is a deterministic model, downstream tests and the audit will disagree.
- **"We picked the middle point."** The operating-point justification must cite the harm model and appetite; median-of-frontier is not a justification.
- **Baseline dropped from the report.** The pre-mitigation numbers must ship. Without them the reader cannot see what the mitigation bought.

## Interfaces with adjacent modules

- **Mod-103 (register)** — mitigation moves the residual score; the register row for each fairness-harm scenario updates with the new residual, and the control-defeated score for "mitigation bypassed / disabled" gets an entry.
- **Mod-108 (guardrails)** — post-processing mitigation is technically a guardrail; catalogue it there.
- **Mod-109 (monitoring)** — the operating point must be monitorable in production; if the ε drifts, an alert must fire.
- **Mod-111 (incident response)** — a confirmed fairness incident (a complaint, a monitor fire) causes the loop to re-run; the incident report cites the pre- and post-run operating points.
- **Mod-112 (model card)** — the model card fairness section names the operating point, its trade-off, and the algorithm.

## Summary

- Mitigation sits in three buckets: pre-processing (reshape data), in-processing (constrain training), post-processing (adjust decisions). Each has different auditability, cost, and legal fit.
- The target metric constrains the mitigation menu — the "which mitigation lands where" table.
- The output of the loop is a *Pareto curve*, not a point. Ship the plot, the table, and per-group accuracy at every point.
- The operating point is picked with a written justification citing the harm model and appetite band.
- Anti-patterns: no per-group accuracy; post-processing where `A` is not legally collectable at inference; silent randomised classifiers; median-of-frontier as justification; dropped baseline.
- Exercise 04 runs the full loop and produces the trade-off table for the running model.
