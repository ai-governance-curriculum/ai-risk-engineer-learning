# Exercise 05 — Explainability Slice with SHAP, Alibi, and LIME

**Estimated effort:** 2 hours

## Objective

Author the chapter-05 explainability slice on the (post-mitigation) running model from exercise 04. The slice is a companion to the fairness slice — the two ship together in the audit report — and its load-bearing purpose is to support the *specific downstream consumer* it names: an adverse-action notice under ECOA / Reg B or GDPR Article 22, a human-in-the-loop reviewer under EU AI Act Article 14, a regulator asking about a specific decision, or a fairness-incident debug tied back to a group-metric failure. The exercise produces the SHAP / Alibi / LIME quad on a sampled set of instances, plus the global-per-protected-group SHAP that is the fairness / explainability cross-cut.

## Prerequisites

- Chapter 05 read; chapter 04 (mitigation) available; chapter 06 skimmed so you know which consumer the explanations feed.
- Exercises 01–04 completed. The model under this exercise is the *post-mitigation* model chosen in exercise 04 at the named operating point; explainability of the unmitigated baseline is fine as a stretch but the audit shape reads the shipped model.
- The eval set from exercise 02 available with protected-attribute columns; explainability outputs must slice by protected group for the global cross-cut.
- SHAP (`shap`) and Alibi (`alibi`) and LIME (`lime`) installed at pinned versions.
- Instances of interest for the local slice: (a) the top-10 highest-delta individuals from exercise 03; (b) any instances that were flagged in real complaints / adverse-action reviews (if the system is live); (c) a small random sample from each protected group for the sampled comparison.
- Named **intended consumer** for the slice — pick one primary consumer (adverse-action notice, human reviewer UI, regulator response, fairness-incident debug, model-card documentation). The tool choice and output shape depend on the consumer.

## Problem statement

Produce the chapter-05 explainability slice. Four sub-artefacts land in the slice:

### 1. Global explanations

- **Global SHAP** on the eval set: mean-|SHAP| per feature; beeswarm plot. Use TreeSHAP if the model is a tree ensemble, KernelSHAP with a stated sample budget if the model is opaque, DeepSHAP for a deep net.
- **Global SHAP sliced by protected group** — the fairness / explainability cross-cut. Compute mean-|SHAP| per feature per protected group. Compare across groups; a feature with a much larger mean-|SHAP| in one group is a proxy candidate to investigate.
- **Dependence plots** for the top-`k` (default 5) global features, with an interaction feature suggested by the dependence structure.

### 2. Local explanations on a sampled set

For a *representative sample* of 20–50 instances (drawn from each protected group; include the intersectional-cell focus from exercise 03), produce the full quad:

- **SHAP local** — waterfall plot per instance; the SHAP array itself for machine consumption.
- **Alibi anchor** — the rule with its precision and coverage; run at a default `threshold=0.95` and note the tolerance.
- **Alibi counterfactual** — the minimally-different feature vector that flips the prediction; report per-feature delta.
- **LIME local** — the coefficient-ranked features; use the same `num_features` as SHAP (default 10) for direct comparison.

### 3. Local explanations on problem instances

The instances flagged by exercise 03 (top-10 high-delta), plus any complaints / adverse-action review triggers if the system is live. Each gets the full quad plus a short paragraph diagnosing what the explanations imply about the decision.

### 4. Cross-tool agreement and stability

- **Top-`k` agreement** — rate at which SHAP top-`k` and LIME top-`k` overlap across the sampled set. Report the median overlap and the tail (instances with < 50 % overlap).
- **Explanation stability** — for each sampled instance, perturb the input slightly (within an ε ball on a feature-wise norm that respects the data types) and recompute SHAP and LIME. Report the rank-correlation of the top-`k` before and after perturbation. Large drops are an instability signal.
- **Faithfulness note** — sample budget for KernelSHAP; linear-surrogate assumption disclaimer for LIME; the "attributions, not causes" framing.

## Requirements

Produce all eight items below. Missing any is an audit finding.

1. **Slice config** — `risk/mod-106/<system-id>/explainability/config.yaml`. Names the intended consumer(s), the SHAP variant and sample budget (if KernelSHAP), the Alibi anchor threshold, the LIME `num_features`, the sampling strategy for the local set, and the perturbation ε for the stability check.
2. **Global runner** — `risk/mod-106/<system-id>/explainability/run_global.py`. Emits mean-|SHAP| per feature (overall) and per protected group; beeswarm plot; dependence plots for top-`k`.
3. **Local runner** — `risk/mod-106/<system-id>/explainability/run_local.py`. For each instance in the sampled set: emits SHAP waterfall, Alibi anchor (rule + precision + coverage), Alibi counterfactual (feature delta table), LIME coefficients. Emits a JSONL row per instance with all four outputs so the cross-tool agreement can be computed downstream.
4. **Cross-tool agreement report** — `risk/mod-106/<system-id>/explainability/agreement.md`. Median top-`k` overlap rate between SHAP and LIME; tail distribution; instances where SHAP and LIME disagree strongly get called out with a short diagnosis (LIME's linear surrogate is a poor fit here; SHAP's KernelSHAP was under-sampled; etc.).
5. **Stability report** — `risk/mod-106/<system-id>/explainability/stability.md`. Rank correlation of top-`k` before / after perturbation for each sampled instance; distribution; instances with correlation < 0.5 flagged.
6. **Global-per-group cross-cut** — `risk/mod-106/<system-id>/explainability/global-per-group.md`. The per-group mean-|SHAP| tables side-by-side; the features whose importance differs materially across groups called out as proxy candidates; cross-refs to exercise 03's proxy-perturbation results and exercise 02's intersectional headline.
7. **Explainability slice** — `risk/mod-106/<system-id>/explainability/slice.md`. The chapter-05 fixed shape: intended consumer, model type and tool choice, global slice, local slice (sampled and problem instances), stability, cross-tool agreement, faithfulness note, reproducibility manifest.
8. **Adverse-action-notice draft (if the consumer is adverse action)** — `risk/mod-106/<system-id>/explainability/adverse-action-draft.md`. A worked example of the notice for one denied instance: what the counterfactual identifies as the minimum change, translated into consumer-facing language and cross-referenced to the SHAP local for the same instance. This is a stretch if the intended consumer is elsewhere but a hard requirement if adverse-action notice is the named consumer.

Cross-cutting rules:

- **TreeSHAP for tree ensembles.** Do not use KernelSHAP on a tree model — it is slower, noisier, and less faithful (chapter 05 anti-pattern).
- **KernelSHAP always ships with its sample budget in the manifest.** A KernelSHAP number without a budget is not a claim.
- **Adverse-action notices come from counterfactuals.** SHAP local can *support* the notice but the *actionable* reasons — "your income would need to be at least $X and utilisation at most Y%" — are the counterfactual's job. Do not ship a SHAP bar chart as the adverse-action notice.
- **Ship the disagreement.** SHAP / LIME disagreement is a signal about the *stability* of the explanation, not a picking problem. Reports that hide the disagreement produce audit findings.
- **The global-per-group cross-cut is the fairness link.** Skipping it means the explainability slice is unlinked from the fairness slice; the audit shape in exercise 06 depends on this link.

## Starter guidance

- **Pin the intended consumer first.** The rest of the exercise's choices flow from it. Adverse-action notice → counterfactual is the load-bearing artefact. Human reviewer → local SHAP is the UI element. Regulator query → local SHAP + anchor for one instance. Fairness debug → global-per-group SHAP + dependence plots.
- **Prove the SHAP plumbing on one instance first.** SHAP wrappers are picky about model interface — `sklearn` pipelines, `xgboost` Booster objects, and PyTorch modules each need a different `Explainer` constructor. Get one waterfall out before you loop over the sample.
- **Alibi anchor's `AnchorTabular` needs a categorical-feature mapping.** If your input has categorical columns encoded as integers, pass the `categorical_names` argument or the rules will look like "`education > 3.5`" and be uninterpretable. Ship rules that read like decision-tree paths.
- **Alibi counterfactual's search space is where the wall-clock lives.** `Counterfactual` (the classic Wachter search) is slow but interpretable; `CounterfactualProto` (prototype-guided) is faster and produces on-manifold counterfactuals. Pick per instance; note the choice.
- **LIME's `LimeTabularExplainer` needs the training data statistics.** It samples around the instance using those stats; pass the *training* set, not the eval set, or the neighbourhood is distributionally off.
- **Sample the local set thoughtfully.** For the sampled comparison, stratify by protected group and prediction (denied / approved) so every cell has explanations. For problem instances, take the exercise-03 top-10 plus any complaint-flagged instances; do not pad with random draws.
- **Perturbation stability is more useful than it looks.** If SHAP and LIME both list a feature at the top but the feature's importance rank swings wildly under tiny input perturbations, the explanation is unstable for this instance — flag it and note in the report that the local explanation for this instance is unreliable.
- **The global-per-group cross-cut is often where proxy usage becomes visible.** A feature with mean-|SHAP| of 0.05 overall but 0.20 for one protected group is doing very different work for that group — investigate whether that reflects legitimate group-specific structure or proxy usage.
- **Watch for the "sanity-checks-for-saliency" trap on deep models.** Adebayo et al. (arXiv:1810.03292) show that saliency maps for deep image models can look reasonable even for randomised models. If your model is a deep net, cite the paper in the faithfulness note and consider running a randomised-model comparator on one instance.

## Acceptance criteria

The submission is accepted when:

- The slice config exists and names the intended consumer, the SHAP variant + sample budget, the Alibi and LIME settings, and the sampling strategy.
- The global runner emits mean-|SHAP| overall and per protected group, plus beeswarm and dependence plots.
- The local runner emits the full quad (SHAP, anchor, counterfactual, LIME) for at least 20 sampled instances plus the problem instances, with a JSONL row per instance.
- The cross-tool agreement report shows the median SHAP / LIME top-`k` overlap and the tail; strong disagreements are diagnosed, not hidden.
- The stability report shows rank correlation before / after perturbation for each sampled instance; correlations < 0.5 are flagged.
- The global-per-group cross-cut names the features whose importance differs materially across groups and cross-refs exercise 03's proxy-perturbation results.
- The explainability slice is authored in the fixed chapter-05 shape.
- If the intended consumer is adverse-action notice, the adverse-action-notice draft exists as a worked example.
- Faithfulness caveats are stated (KernelSHAP sample budget; LIME linear-surrogate assumption; deep-model saliency caveat if applicable).
- Reproducibility manifest pins SHAP / Alibi / LIME versions, seeds, sample-set fingerprint, model artefact hash, script commit hash.

## Stretch goals

- Add **integrated gradients** (Alibi `IntegratedGradients`) as a fourth attribution family for the sampled deep-model instances. Cross-tool agreement across four methods is a stronger stability signal than across two.
- Run the **randomised-model comparator** on one deep-model instance per the Adebayo et al. sanity check: retrain the model with randomised labels, recompute the saliency, and check whether the saliency map still looks similar. If it does, the tool is not diagnostic on this model; note the finding.
- Produce a **fairness-debug decomposition**: for the intersectional cell that exercise 02 flagged as failing, compute the mean-|SHAP| per feature within the cell vs the reference cell; the features driving the disparity are the mitigation-attention list for a hypothetical exercise-04 re-run.
- Wire an **explanation-quality guardrail stub** for mod-108: at inference time, if a counterfactual cannot be found within a reasonable rewrite budget or if SHAP and LIME disagree on the top-1 feature, flag the decision for human review before an adverse-action notice is issued. Ship as the intake artefact; the guardrail itself lands at mod-108.
- Contribute the **model-card explainability section** — intended consumer, tool choice, sample budget, faithfulness caveats, link to the slice manifest. This is what mod-112 signs from.

## Deliverable format

A single markdown document `mod-106-ex05-explainability.md` that indexes the artefacts, plus the slice, config, runners, agreement / stability / global-per-group / adverse-action-draft documents. The index document contains:

1. *Intended consumer* — one paragraph naming who reads this slice and what they do with it.
2. *Model + tool choice* — one paragraph naming the model architecture, the SHAP variant, and the Alibi / LIME configurations.
3. *Global slice summary* — the top-`k` features by mean-|SHAP| overall, the beeswarm embedded.
4. *Global-per-group cross-cut* — the tables side-by-side; the proxy candidates called out.
5. *Local slice — sampled* — for one representative instance, the full quad displayed (waterfall, anchor rule, counterfactual delta, LIME coefficients).
6. *Local slice — problem instances* — a short paragraph per problem instance summarising the quad.
7. *Cross-tool agreement + stability* — the summary tables with the disagreement tail.
8. *Faithfulness note* — the caveats in one paragraph.
9. *Adverse-action-notice draft* — the worked example if applicable.
10. *Register handoffs* — mod-103 row IDs the "unexplainable adverse action" risk is discharged against.
11. *Open items* — anything for exercise 06 (particularly which explainability artefacts are cited in the LL144 audit) or forward to mod-108 / mod-109 / mod-112.

## Submission

Attach the index markdown, the slice, the config, the runners, the JSONL local outputs, and the agreement / stability / cross-cut / adverse-action-draft documents to the module submission channel. If the model / dataset is under NDA, redact identifiers but preserve the SHAP feature-name-anonymised bar charts, the anchor rule shape, the counterfactual delta shape, the SHAP / LIME agreement rates, and the global-per-group cross-cut structure — the slice cannot be reviewed if the shape is blanked out.
