# Individual and Causal Fairness — Where the Group-Metric Family Runs Out

Group-fairness metrics answer *"does the classifier treat groups equivalently on average?"* Two other fairness families exist because that average-based answer is sometimes the wrong question. **Individual fairness** asks whether the classifier treats *similar individuals* similarly. **Causal fairness** asks whether the protected attribute — or a proxy of it — is on a causal path to the decision that it should not be on. This chapter says when each family fits, when it does not, and how to run them alongside a group-fairness slice.

## Motivation

Two failure modes group fairness alone does not catch:

- **Two similar applicants, different outcomes.** Two applicants with essentially the same non-protected features get different decisions because their protected-attribute-adjacent features (postcode, name, education institution) differ. Group averages are equal; individual treatment is not. A regulator investigating a complaint asks about the individual, not the group.
- **A proxy variable carries the protected attribute.** The model does not use `race` as an input, but it uses a feature so highly correlated with `race` that the decision changes when you counterfactually change `race`. Group-fairness metrics on the observational distribution can pass; a causal analysis on a well-drawn DAG shows the discrimination is present.

The individual and causal families are how you catch those. They cost more to run and depend on stronger modelling assumptions, so they are not the default headline — but where they fit, they are load-bearing.

## Individual fairness — Dwork et al. metric-fairness

Dwork, Hardt, Pitassi, Reingold, and Zemel — "Fairness through Awareness" (arXiv:1104.3913) — formalise the principle *similar individuals should receive similar decisions*. The formal statement uses a task-specific similarity metric `d(·,·)` on individuals and requires that the classifier's output distribution `M(x)` be close under a distributional distance `D` proportional to `d(x, x')`:

> `D(M(x), M(x')) ≤ L · d(x, x')`

where `L` is a Lipschitz constant. The classifier is *individually fair* under `d` if this inequality holds for every pair `(x, x')`.

The formulation is elegant; the load-bearing element is the similarity metric `d`. Two hard problems come with it:

- **You have to define `d`.** In most real settings, defining "similarity" of two individuals in a way that everyone in the room agrees with is exactly the hard problem the fairness question is asking. If `d` uses the same features that carry the bias, individual fairness certifies a biased classifier as fair. Dwork et al. are explicit that the metric is a *task-specific* input and its construction is where the ethical judgement lives.
- **You have to be able to compute `D(M(x), M(x'))`.** For deterministic classifiers this collapses to comparing outputs; for probabilistic ones it requires a distributional distance (total variation, Jensen-Shannon, or similar).

### Where individual fairness fits as a diagnostic

Even without a fully specified `d`, individual-fairness *diagnostics* are cheap and useful. Two patterns you will actually run:

- **Nearest-neighbour consistency.** For each individual in the evaluation set, find its `k` nearest neighbours under a defensible feature distance that *excludes protected attributes and known proxies*. Compute the disagreement rate between the individual's prediction and the neighbours'. High disagreement in a small local neighbourhood is a signal — either the classifier is unstable, or a proxy of the protected attribute is doing work locally.
- **Perturbation stability on protected attributes.** For each individual, perturb the protected attribute (or a designated proxy set) and see whether the prediction changes. This is the empirical Lipschitz check for the protected-attribute dimension of `d`.

Both diagnostics produce a distribution over the evaluation set — not a single number — and the report gives the tail (percent of individuals with disagreement above some threshold) alongside the median.

### Where it does not fit

- **When the "similar" judgement is exactly the bias.** In hiring for a role where "similar candidate" is a culturally loaded concept, an individual-fairness certificate that uses features like "years at prestige-tier institutions" as part of `d` will validate the historical bias.
- **When the population is heterogeneous by design.** A medical triage model applied across children and adults should treat them very differently; a naive individual-fairness metric would flag this as unfair. Individual fairness applies within a *comparable population*, not across the entire input space.

## Causal fairness — Kusner et al. counterfactual fairness

Kusner, Loftus, Russell, and Silva — "Counterfactual Fairness" (arXiv:1703.06856) — reframe fairness using structural causal models. A predictor `Ŷ` is *counterfactually fair* with respect to protected attribute `A` if, for any individual, the prediction the individual would have received in a counterfactual world where their `A` was different is the same as the prediction they actually receive:

> `P(Ŷ_{A ← a}(U) = y | X = x, A = a) = P(Ŷ_{A ← a'}(U) = y | X = x, A = a)` for all `a, a', y`

`U` is the exogenous background; `X` is the observed features; the notation `Ŷ_{A ← a'}` is the potential outcome under intervention on `A`. Reading it as prose: *had this specific person been of a different protected group and everything else remained as it was under a specified causal model, the prediction would not change*.

### What causal fairness requires

Three preconditions must hold for counterfactual fairness to be operational:

1. **A causal DAG you can defend.** You have written down which variables cause which — is `education_level` a descendant of `race`? does `postcode` cause `income` or vice versa? — and you can defend the arrows.
2. **Identifiability.** The counterfactual you want to compute (the effect of intervening on `A`) is identified from the observational distribution under your DAG. This is a formal check — see Pearl's *Causality*.
3. **A means to sample or compute the counterfactual.** For linear-Gaussian structural models this is closed-form; for more general models you use a structural causal model fit (e.g., a probabilistic programme) or approximate it.

None of these is cheap. In practice you build a *specific* causal fairness diagnostic for a *specific* decision (loan approval, medical triage) where the causal structure is defensible from domain literature, not a general-purpose eval you run on every model.

### Related causal-fairness formulations

Counterfactual fairness is one of several causal formulations. Two others worth naming:

- **Path-specific effects (Nabi & Shpitser, "Fair Inference on Outcomes")** — some paths from `A` to `Ŷ` are considered legitimate ("the applicant's protected attribute correlates with the disease under study, and the disease legitimately affects treatment"); others are not. You constrain effects along the illegitimate paths only. Requires the DAG *and* a per-path legitimacy judgement.
- **Interventional fairness (Salimi et al., "Interventional Fairness")** — average causal effect of `A` on `Ŷ` is zero under the intervention model. Weaker than counterfactual fairness but often easier to identify.

### Where causal fairness fits

- **When the DAG is defensible from domain literature.** Clinical decision support in a well-studied disease area; underwriting in a well-modelled risk domain.
- **When a specific counterfactual claim is being made.** "Had this applicant been of a different protected class, would the decision have differed?" — a causal-fairness diagnostic is what answers that literally.
- **When path-specific reasoning is required by the regulator.** The EU AI Act's Article 10 language about representative and appropriate data implicitly asks path-specific questions; a defensible reply is causal-fairness reasoning, not group-average reasoning.

### Where it does not fit

- **When the DAG is contested.** If domain experts disagree on the arrows, a causal-fairness "pass" verdict is defensible only under the specific DAG you drew — and a reviewer with a different DAG will get a different verdict.
- **When identifiability fails.** Some causal quantities cannot be computed from observational data alone under any DAG. You cannot report a number you cannot identify.
- **As a default eval.** Group-fairness metrics run without a DAG; causal-fairness metrics do not. Do not force the causal shape on every model.

## Holistic AI's multi-dimensional trustworthiness sweep

Holistic AI publishes an open-source library (`holisticai`, MIT-licensed) that packages bias, explainability, robustness, security, and privacy metrics behind one interface. Its use in the mod-106 workflow is *diagnostic*, not headline:

- Run the Holistic AI sweep as a *broad-brush* trustworthiness overview across many dimensions.
- Use the sweep to *identify* which dimensions need the deeper, specific evaluations Fairlearn / AIF360 / SHAP / Alibi supply.
- Do not ship the Holistic AI numbers as the audit conclusion — ship the specific tool's numbers on the axis Holistic AI flagged, and cite the sweep as the reason you looked.

The value is the *breadth*: a single call surfaces the axis you would otherwise not have looked at. The cost is depth — every metric in the sweep is one call deep, so a red result there needs its own targeted evaluation.

## The when-to-use decision table

| Diagnostic family | Preconditions | Cost | When to run | When to skip |
|---|---|---|---|---|
| Group fairness (chapter 01-02) | Protected-attribute partition; sufficient sample per group | Low | Always, as the default | Never (it is the headline for regulated decisions) |
| Individual fairness — nearest-neighbour consistency | A defensible non-protected feature distance | Low | Any classifier where local instability is a concern | When the population is heterogeneous by design |
| Individual fairness — perturbation stability on protected attributes | Ability to perturb `A` and rescore | Low | Whenever you can rescore counterfactually on `A` | When perturbation is undefined (e.g., non-tabular where `A` is entangled with other features) |
| Counterfactual fairness | A defensible causal DAG; identifiability; means to compute counterfactuals | High | High-stakes decisions with a defensible DAG (clinical, credit, employment in a well-modelled setting) | When the DAG is contested or not identifiable |
| Holistic AI trustworthiness sweep | Model + representative eval set | Low | As a broad-brush "did we miss an axis" scan | As the headline conclusion for any axis |

## A worked example — individual + causal, small

Continuing the loan-approval example.

**Individual-fairness perturbation stability.** For each applicant in the evaluation set, flip the `race` field to each other value in the EEO-1 vocabulary while holding all other fields constant. Rescore. Report:

- Fraction of applicants whose predicted class changes for at least one perturbation. This is the *raw* perturbation-instability rate on `race`.
- Distribution over applicants of the maximum score delta across perturbations.
- Top 10 applicants by delta — inspect manually as a spot-check.

Interpretation: a raw perturbation-instability rate above the internal tolerance is a signal that `race` (or something the model reconstructs from other features) is directly moving the decision. Even a low rate is worth inspecting the tail on; a small number of high-delta individuals is the tail regulators land complaints from.

**Counterfactual-fairness diagnostic (only if the DAG is defended).** Fit a structural causal model over `{race, education, income, employment_history, credit_utilisation, requested_amount, decision}` with the arrows the domain literature supports (e.g., `race → education`, `race → income` via well-documented historical mechanisms; `income, employment_history, credit_utilisation → decision` legitimately). Compute the counterfactual prediction under intervention on `race` for each applicant. Report:

- Distribution of counterfactual decision changes.
- Path-specific effect decomposition if the DAG supports it — direct effect of `race` on `decision`; indirect effect through `education`; indirect through `income`; etc.
- A prose paragraph in the report naming the DAG assumptions and the identifiability check.

The prose paragraph is what makes the diagnostic *auditable*. Without it the numbers are opaque.

## Anti-patterns

- **Individual fairness with a `d` that encodes the bias.** The similarity metric is the whole game; if you construct it from features that carry the discrimination, individual fairness certifies a biased classifier.
- **Causal fairness without a defensible DAG.** A causal claim rides on the DAG; drawing one from convenience produces a number that survives no scrutiny.
- **Holistic AI sweep shipped as the conclusion.** The sweep is a scan, not an audit. Convert its red flags into targeted evaluations, or say it in the report that you did not.
- **Skipping individual / causal because "the group numbers are green."** Group averages can pass while a specific subgroup or a specific individual is treated badly. If the harm model contains an individual-scale harm ("a qualified applicant was denied"), the group-average metric alone does not discharge it.

## Interfaces with adjacent modules

- **Mod-102 (harm modelling)** — the choice to run causal fairness (or not) is anchored to whether the harm model contains an individual-scale harm the group metric cannot see.
- **Mod-103 (register)** — an individual-fairness or causal-fairness diagnostic result populates the register row for the specific individual-scale harm.
- **Mod-108 (guardrails)** — perturbation-instability findings feed input-validation guardrails (e.g., score-stability check at inference time).
- **Mod-109 (monitoring)** — perturbation stability is monitorable in production on a sampled basis; drift on this signal is a first-order alert.

## Summary

- Individual fairness (Dwork et al.) asks whether similar individuals are treated similarly under a task-specific metric `d`; the choice of `d` is where the ethics live.
- Nearest-neighbour consistency and protected-attribute perturbation stability are cheap individual-fairness *diagnostics* you can run without formalising `d`.
- Counterfactual fairness (Kusner et al.) requires a defensible causal DAG, identifiability, and a computation path. Where those exist, it is the only formulation that answers "had this specific person been of a different group, would the decision differ?"
- Path-specific effects (Nabi & Shpitser) and interventional fairness (Salimi et al.) are related causal formulations for when counterfactual fairness is not identifiable.
- Holistic AI is a *sweep* — a broad-brush trustworthiness scan whose value is breadth, not depth.
- Anti-patterns: bias-encoded `d`; undefended DAG; sweep-as-conclusion; group-only shipping when the harm is individual-scale.
- Exercise 03 runs individual-fairness perturbation stability and a small counterfactual-fairness diagnostic on the running model.
