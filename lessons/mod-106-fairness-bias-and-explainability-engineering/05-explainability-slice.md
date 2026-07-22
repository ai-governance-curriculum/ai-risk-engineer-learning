# The Explainability Slice — SHAP, Alibi, LIME, and When Explainability is Load-Bearing

Explainability sits alongside fairness in the same report, and the two are often confused. Fairness answers *does the model treat groups equitably?*; explainability answers *why did the model produce this specific output for this specific input?* This chapter is the engineering shape of the explainability slice — which tool for which shape, how the outputs read, and — critically — when explainability is doing real work versus when it is theatre.

## Motivation

Two failure modes worth naming up front, because both are common:

- **Feature-importance theatre.** The team ships a bar chart of "top 10 features" produced by SHAP or LIME with no context and no faithfulness check. Readers treat the chart as *the reason the model decided*, which it usually is not — it is an attribution to *a local linear approximation* of a decision that is not linear. When a regulator or an adverse-action recipient challenges a specific decision, the bar chart cannot substantiate it.
- **Explainability substituted for evaluation.** The team argues the model is safe / fair / accurate *because* it is explainable. Explainability is orthogonal to those properties; an explainable model can be badly wrong, and a black-box model can be perfectly correct. Explainability supports decision-making by humans downstream — it does not certify the model.

The engineering job is to know which tool produces which shape of output, when that output is load-bearing (regulator demands it; adverse-action notice requires it; a customer challenges a decision), and when it is misleading (the tool's assumptions do not fit; the explanation is unstable; a decision-maker draws conclusions the tool does not support).

## The tool catalogue

Three tool families a level-25 risk engineer ships against, plus the notes on when each is the right choice.

### SHAP — Shapley-value attributions

SHAP (Lundberg & Lee, "A Unified Approach to Interpreting Model Predictions", arXiv:1705.07874) computes Shapley values from cooperative game theory as feature attributions. The published implementation lives at the SHAP project (`shap.readthedocs.io`).

Two variants you use most:

- **KernelSHAP** — model-agnostic; approximates Shapley values by sampling coalitions and fitting a locally weighted linear model. Slow (`O(2^N)` in the number of features in principle; sampled in practice), works on anything.
- **TreeSHAP** — exact Shapley values for tree ensembles (XGBoost, LightGBM, sklearn `GradientBoosting`, `RandomForest`). Fast, deterministic, closed-form for the tree class. When your model is a tree ensemble, this is what you use.

There are also `DeepSHAP` (for deep nets), `LinearSHAP`, and specialised variants; the two above cover most of the mod-106 workload.

**Output shapes:**

| Output | What it shows | Ships as |
|---|---|---|
| **Local explanation** | Per-feature contribution to *this* prediction | Waterfall plot; per-instance feature-value bar; the array of attributions itself |
| **Global explanation** | Aggregate feature importance across the dataset | Beeswarm plot; mean-abs-SHAP bar chart |
| **Dependence plot** | How a feature's SHAP value depends on the feature's value (and one interacting feature) | Scatter of `feature_value` vs `shap_value`, coloured by interaction feature |

**Faithfulness note.** TreeSHAP is *exact* for the tree model — the attributions really do decompose the prediction. KernelSHAP is *approximate*; the approximation error is a function of the sampling budget. Report the budget alongside any KernelSHAP output.

### Alibi — anchor and counterfactual explanations

Alibi (`docs.seldon.io/projects/alibi/`) is a Seldon-maintained open-source library that packages several explanation families with production wrappers.

Two you will ship:

- **Anchor explanations** (Ribeiro, Singh, Guestrin, "Anchors: High-Precision Model-Agnostic Explanations") — a rule-based explanation: *"if these `k` conditions hold, the model predicts class `c` with precision `p` on nearby data."* Anchors are conditional rules; they read like decision-tree paths. Their appeal is *precision under perturbation* — the anchor tells you the input conditions under which the explanation stays true.
- **Counterfactual explanations** (Wachter, Mittelstadt, Russell, "Counterfactual Explanations without Opening the Black Box: Automated Decisions and the GDPR") — the *smallest change* to the input that would flip the prediction. A counterfactual explanation is the answer to *what would have had to be different for this decision to be different?* — which is what an adverse-action notice under ECOA / GDPR needs to say.

**Output shapes:**

| Output | What it shows | Ships as |
|---|---|---|
| **Anchor** | Set of `feature=value` (or `feature in range`) predicates that determine the prediction on nearby data with precision `p` and coverage `c` | Rule string + precision + coverage; e.g., `race=* AND income≥60k AND requested_amount≤50k → APPROVED (precision 0.94, coverage 0.11)` |
| **Counterfactual** | An alternate feature vector, minimally different from the input, that produces a different prediction | Table showing per-feature change from the actual instance |

Alibi also implements integrated gradients, contrastive-explanation methods, and others; the two above are the workhorse pair for mod-106.

### LIME — the compare-point

LIME (Ribeiro, Singh, Guestrin, "'Why Should I Trust You?': Explaining the Predictions of Any Classifier", arXiv:1602.04938) fits a locally weighted linear surrogate around a specific instance and reports the surrogate's coefficients as the explanation.

LIME's role in mod-106 is as a **compare-point**, not a headline:

- Local explanations from LIME and SHAP that agree strengthen confidence that both are picking up a real local structure.
- LIME and SHAP that disagree flag *instability of the local explanation* — a signal that neither is a trustworthy explanation for this instance.
- LIME's linear surrogate assumption is often violated locally on non-smooth models; when SHAP and LIME disagree strongly and the model is non-linear, prefer SHAP (and note the disagreement in the report).

LIME's implementation lives at the LIME project (`github.com/marcotcr/lime`).

## When explainability is load-bearing

Four scenarios where the explanation is not a nicety but a *deliverable*:

- **Adverse-action notice (US ECOA / Regulation B; EU GDPR Article 22).** When a person is denied credit / employment / insurance based on an automated decision, they are entitled to the *principal reasons* for the decision. A counterfactual explanation ("your application would have been approved if your reported income had been at least $X and your utilisation had been at most `Y`") is a legally credible shape for that notice. A SHAP local waterfall may be — provided it identifies specific, actionable features. A global feature-importance chart is not.
- **Regulator asking about a specific decision.** A regulator investigating a complaint asks "why did the model produce this output for this input?" The answer is a local explanation on that instance, ideally cross-checked between SHAP and Alibi anchor.
- **Human-in-the-loop review surface (EU AI Act Article 14).** The reviewing human needs to know *why* the model recommended what it did. The UI element is a local explanation surfaced next to the recommendation.
- **Model-debug for a fairness incident.** A group-fairness metric is failing. Global SHAP (per group) helps identify which features carry the disparity; dependence plots show whether a proxy is driving the effect. This is a diagnostic use.

In each of these, the explanation is doing real work — a downstream decision or filing depends on it.

## When explainability is misleading

Four scenarios where the explanation actively harms comprehension:

- **Illusion of understanding.** A reader looks at a SHAP bar chart and believes they understand the model. They do not — they understand the *attribution of the model's decisions to features under a specific set of assumptions*. When those assumptions do not hold, the chart is wrong. Feature-importance saliency, particularly on deep-learning image models, has been shown in adversarial settings (Adebayo et al., "Sanity Checks for Saliency Maps", arXiv:1810.03292) to look reasonable even for randomised models.
- **Explanations of the wrong thing.** A KernelSHAP explanation for a tree model at 100 samples is *worse* than the exact TreeSHAP answer — you have added noise for no gain. Choose the tool that matches the model.
- **Explanations disagreeing silently.** LIME and SHAP disagree on the top feature for many instances. If the report picks whichever agrees with a preferred narrative, the explanation is post-hoc rationalisation. Ship the disagreement.
- **Explaining decisions that should not be automated.** If the decision is one that should have human judgement front and centre (a criminal-justice decision, a triage that shortens a life), the presence of an explanation on the automated output can *legitimise* automation the model does not deserve. Explainability does not certify safety.

The engineering discipline is to name, in the explainability report, the specific downstream artifact the explanation *supports* (adverse-action notice; human reviewer; debug of a fairness incident; regulator response) and to ship the explanation only where that support is real.

## The explainability slice shape

A shippable explainability slice for one model is a fixed shape.

| Section | Content |
|---|---|
| Intended consumer | Named — "adverse-action notice"; "human reviewer UI"; "regulator response protocol"; "internal fairness debug"; "model documentation for the model card" |
| Model type and tool choice | Model architecture; SHAP variant chosen (TreeSHAP if trees; KernelSHAP with sample budget otherwise; DeepSHAP for deep nets); Alibi anchor / counterfactual configuration; LIME configuration for compare-point |
| Global slice | Mean-absolute SHAP per feature (overall); the same per protected group (this is the fairness cross-cut); dependence plots for the top `k` features |
| Local slice — sampled instances | A representative sample of instances with their SHAP local, Alibi anchor, Alibi counterfactual, and LIME local explanations side by side |
| Local slice — problem instances | Instances flagged by fairness monitoring, complaints, or adverse-action processes — each with a full quad of explanations |
| Stability check | Perturbation stability of the explanations themselves — do small input perturbations produce large explanation changes? |
| Cross-tool agreement | Rate at which SHAP and LIME agree on the top-`k` features across the sample; disagreements called out |
| Faithfulness note | Sample budget for KernelSHAP; disclaimers about the linear-surrogate assumption for LIME; the "explanations are attributions, not causes" framing |
| Reproducibility manifest | Tool versions, commit hash, dataset content hash, random seed |

The slice is a companion to the fairness slice in the same report; the global-per-group SHAP is the direct link between the two.

## A worked example — the four outputs on one instance

Continuing the loan-approval example. Instance `A123`: denied loan.

- **SHAP local (TreeSHAP).** Waterfall: top negative contributors — `debt_to_income = 0.61` (SHAP −0.28); `recent_delinquency = True` (SHAP −0.19); `employment_length = 8 months` (SHAP −0.11). Top positive contributor — `credit_score = 720` (SHAP +0.14). Aggregate SHAP + baseline = model's log-odds output.
- **Alibi anchor.** *`debt_to_income > 0.55 AND recent_delinquency = True → DENIED`* — precision 0.93, coverage 0.06. Reads as a rule.
- **Alibi counterfactual.** Minimally different instance predicted APPROVED: `debt_to_income` changed from 0.61 to 0.42; `recent_delinquency` changed from True to False; everything else unchanged. This is the shape the adverse-action notice consumes.
- **LIME local.** Top features by |coefficient|: `debt_to_income` (−); `recent_delinquency` (−); `employment_length` (−); `credit_score` (+). Same top-4 as SHAP, though the ordering differs on features 2 and 3.

Report line: SHAP and LIME agree on the top-4 features (order 1-swapped); the anchor picks a two-predicate rule with high precision on nearby data; the counterfactual provides an actionable rewrite for the adverse-action notice. Cross-tool agreement is strong; this instance's explanation is defensible.

## The explainability + fairness cross-cut

The most useful joint output in mod-106 is *global SHAP sliced by protected group*. Two applications:

- **Proxy detection.** If a feature has very different mean-absolute SHAP across groups, it is likely acting as a proxy for the protected attribute for at least one group. Investigate.
- **Feature-attribution disparate impact.** The same aggregate decision-rate disparity can come from different features across groups. Naming which features drive the disparity for each group is what makes the mitigation plan (chapter 04) legible.

This is what mod-112's model-card fairness section is fed from.

## Anti-patterns

- **Global-only explainability.** A bar chart of overall feature importance and no local explanations. Regulator asks about instance `A123` and the report cannot answer.
- **KernelSHAP on a tree model.** Slower, noisier, and less faithful than TreeSHAP. Do not.
- **Adverse-action notice built from SHAP bar chart alone.** The notice needs *actionable* reasons, which counterfactuals give directly and SHAP gives only if you convert them. Ship counterfactuals for this use.
- **Ignoring LIME / SHAP disagreement.** The disagreement is a signal about the *stability* of the explanation; suppressing it produces a report that looks cleaner than the reality.
- **Confusing explainability with fairness or safety.** An explanation that a feature drove a decision is not a defence that the decision was correct or equitable. Report the two slices separately, and cross-cut them where useful.

## Interfaces with adjacent modules

- **Mod-102 (harm modelling)** — harms with "adverse action against a specific individual" in the immediate-harm column drive counterfactual explanations for the specific-individual case.
- **Mod-103 (register)** — the register row for "unexplainable adverse action" (an ECOA / GDPR compliance risk) is discharged by the explainability slice.
- **Mod-108 (guardrails)** — an explanation-quality check (e.g., counterfactual exists within a reasonable rewrite budget) is a serving-time guardrail for decisions requiring adverse-action notice.
- **Mod-109 (monitoring)** — explanation-stability drift (LIME / SHAP disagreement rate changing over time) is a monitorable signal that model behaviour is shifting under the hood.
- **Mod-112 (model card)** — model-card explainability section names the tool, the intended consumers, and links to the slice manifest.

## Summary

- SHAP (TreeSHAP exact for trees; KernelSHAP approximate model-agnostic) is the global + local attribution workhorse.
- Alibi supplies **anchors** (rule-shaped, high-precision-under-perturbation local explanations) and **counterfactuals** (minimal rewrites — the shape adverse-action notices need).
- LIME is a compare-point; disagreement with SHAP is a signal, not a footnote.
- Explainability is load-bearing for adverse-action notices, regulator queries about specific decisions, human-in-the-loop review, and fairness debugging.
- Explainability is misleading when it produces an illusion of understanding, is applied with the wrong tool for the model, disagrees silently between tools, or is used to legitimise automation that should not exist.
- The explainability slice shape: intended consumer, tool choice, global slice, local slice, problem instances, stability, cross-tool agreement, faithfulness note, reproducibility.
- Global SHAP sliced by protected group is the fairness / explainability cross-cut.
- Exercise 05 authors the slice on the running model with SHAP, Alibi anchor + counterfactual, and LIME.
