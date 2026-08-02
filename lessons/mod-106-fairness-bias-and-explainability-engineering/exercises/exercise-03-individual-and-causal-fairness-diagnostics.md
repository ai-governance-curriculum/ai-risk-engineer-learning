# Exercise 03 — Individual and Causal Fairness Diagnostics

**Estimated effort:** 2 hours

## Objective

Extend the group-fairness slice from exercise 02 with **individual-fairness** and (where the DAG is defensible) **causal-fairness** diagnostics per chapter 03. The exercise produces the diagnostic block that catches the two failure modes group averages miss: two similar applicants receiving different outcomes, and a proxy variable carrying the protected attribute onto a causal path it should not be on. It also produces a **Holistic AI trustworthiness sweep** as a "did we miss an axis" scan whose red flags feed the mitigation loop in exercise 04.

## Prerequisites

- Chapter 03 read; Dwork et al. (arXiv:1104.3913) and Kusner et al. (arXiv:1703.06856) at least skimmed. If you will attempt path-specific effects, Nabi & Shpitser (arXiv:1705.10378) as well.
- Exercises 01 and 02 completed. The individual-fairness diagnostic in this exercise targets the intersectional cell(s) that exercise 02 flagged as failing or borderline; the causal-fairness diagnostic (if attempted) attaches to the specific harm-model row where an individual-scale harm is named.
- The running model reachable with counterfactual scoring capability — you must be able to change one feature (the protected attribute or a designated proxy) and rescore. If the model is behind an endpoint that rate-limits, scope the eval-set sample size accordingly.
- The eval set from exercise 02 with protected-attribute columns and their designated proxies (typical proxies: postcode / ZIP for race in US contexts; name-based sex inference; institution / school for race and socioeconomic status; language for national origin). If proxies are not documented, resolve upstream before running.
- **Domain literature or a defensible domain-expert judgement** on the causal DAG for the model's decision context. If neither exists, the causal-fairness diagnostic is *not attempted* — the exercise states "DAG not defensible" and moves on. Fabricated DAGs are worse than none (chapter 03 anti-pattern).
- Holistic AI (`holisticai`) installed at a pinned version.

## Problem statement

Two mandatory diagnostics and one conditional diagnostic on the running model. All three attach to the same eval set as exercise 02 so the numbers stack.

### 1. Individual-fairness perturbation stability (mandatory)

For each row in the eval set, produce the classifier's prediction under counterfactual perturbations of the protected-attribute and proxy columns:

- **Protected-attribute perturbation.** Set `race` to each other value in the EEO-1 vocabulary while holding all other columns constant; rescore. Set `sex` to each other value; rescore. If the model consumes both, run both perturbations independently and (as a stretch) jointly.
- **Proxy-variable perturbation.** Set each designated proxy (postcode, institution, language marker) to a plausible neutral value or a set of alternative values; rescore. This catches the "the model does not use `race` but does use `postcode`" case.
- **Nearest-neighbour consistency.** For each row, find its `k` (default 10) nearest neighbours under a feature distance that excludes the protected attributes and proxies. Compute the disagreement rate between the row's prediction and its neighbours' predictions. High disagreement in small neighbourhoods is a signal.

Report distributions, not single numbers: the fraction of individuals whose predicted class flips under at least one perturbation; the distribution of maximum score deltas; the top-10 individuals by delta for manual spot-check; the nearest-neighbour disagreement distribution and the tail (percent of individuals with `> 30%` neighbour disagreement).

### 2. Holistic AI trustworthiness sweep (mandatory)

Run the `holisticai` bias, explainability, robustness, and privacy metrics on the same eval set and same model. This is a **breadth scan**, not the headline — its output is a heatmap that surfaces the axis you would otherwise not have looked at. The red flags become mod-106 / mod-107 backlog items; the numbers are not the audit conclusion (chapter 03 anti-pattern).

### 3. Counterfactual-fairness diagnostic (conditional)

Attempt only if the DAG is defensible from domain literature; otherwise state "DAG not defensible for this decision" and move on.

If attempting:

- Write down the DAG explicitly — nodes are the model's input features plus the label; edges are the causal arrows you can defend. Cite the literature or the domain-expert source for each arrow.
- Check identifiability of the counterfactual you want to compute (typically the effect of intervening on `A` on `Ŷ`). If not identified, state so and move to path-specific or interventional fairness (Salimi et al.) if identified there, or stop.
- Fit a structural causal model over the DAG — a linear-Gaussian SCM for a first pass; a probabilistic-programming SCM (e.g., `pyro`) if the domain calls for it.
- Compute the counterfactual prediction under intervention on `A` for each row in the eval set. Report the distribution of counterfactual decision changes, the path-specific effect decomposition if the DAG supports it, and a prose paragraph naming the DAG assumptions and the identifiability check.

## Requirements

Produce all seven items below. Missing any is an audit finding.

1. **Diagnostic config** — `risk/mod-106/<system-id>/individual-causal/config.yaml`. Names the perturbation set (protected attributes plus proxies with their alternative values), the nearest-neighbour distance function and `k`, the Holistic AI metric selection, and (if attempted) the DAG file path and the SCM class.
2. **Perturbation runner** — `risk/mod-106/<system-id>/individual-causal/run_perturbation.py`. Executes the protected-attribute and proxy perturbations on every eval row, records per-row score deltas and class flips, and emits a JSONL file with one row per (individual, perturbation).
3. **Nearest-neighbour runner** — `risk/mod-106/<system-id>/individual-causal/run_nn.py`. Computes `k`-NN on the exclusion-set feature distance, records per-individual neighbour-disagreement rates, and emits a JSONL file.
4. **Perturbation-stability report** — `risk/mod-106/<system-id>/individual-causal/perturbation-report.md`. Sections: perturbation-set spec, flip rate (overall and per intersectional cell), max-delta distribution, top-10 individuals table (with a note on the manual spot-check), nearest-neighbour disagreement distribution + tail, cross-refs to the exercise-02 intersectional cells this exercise focuses on.
5. **Holistic AI sweep output** — `risk/mod-106/<system-id>/individual-causal/holistic-ai.md` (or a rendered notebook). Heatmap or table across the sweep's metrics; a short paragraph naming which axes flagged red and what mod-106 / mod-107 backlog item each becomes. Explicitly note the sweep is diagnostic, not headline.
6. **Causal-fairness diagnostic** — one of:
   - `risk/mod-106/<system-id>/individual-causal/causal.md` — the DAG diagram (Mermaid / DOT), the identifiability check, the SCM implementation reference, the counterfactual-fairness distribution, path-specific decomposition (if applicable), and the DAG-assumptions paragraph.
   - `risk/mod-106/<system-id>/individual-causal/causal-skipped.md` — a short document naming why the DAG is not defensible (contested arrows, identifiability failure, insufficient domain literature) and referring to the harm-model row that would need a causal-fairness diagnostic if the DAG became defensible later.
7. **Diagnostics summary** — `risk/mod-106/<system-id>/individual-causal/README.md`. One-page synthesis: the perturbation-stability headline number, the nearest-neighbour tail, the Holistic AI red-flag axes, and the causal-fairness verdict or the DAG-not-defensible statement. This is the block that exercise 04 reads to decide whether individual-fairness or causal-fairness needs mitigation attention beyond the group-metric mitigation.

Cross-cutting rules:

- **Individual-fairness distances excluding protected attributes and proxies.** The feature distance for nearest-neighbour must exclude the columns the perturbation set touches. Including them makes the diagnostic tautological.
- **Perturbation values must be plausible.** Setting `postcode` to `99999` is not a valid perturbation; setting it to the postcode of a demographically-different but plausible neighbourhood is. Cite the perturbation-value source.
- **Distributions, not single numbers.** A single "flip rate = 0.03" is a poor summary; the shape is the story. Ship the distribution plot.
- **Manual spot-check the top-10.** Read the ten highest-delta individuals' features and predictions. If the flip is defensible on legitimate features, note that; if it is not, flag it as a candidate for mitigation and route to exercise 04.
- **DAG on paper before code.** If you cannot draw the DAG in one figure with defensible arrows and cite the literature for each arrow, do not compute counterfactual fairness — state DAG-not-defensible and stop. A fabricated causal number is worse than no causal number.
- **The Holistic AI sweep is not the conclusion.** Its output is *"where should we look next"*, not *"the model passes trustworthiness"*. State that in the report body.

## Starter guidance

- **Pick the intersectional cell exercise 02 flagged as the focus.** Rather than running perturbation on all 100k eval rows (which the endpoint may not accommodate), sample within the flagged intersectional cells first — that is where mitigation attention will land in exercise 04, and per-cell perturbation numbers are the numbers exercise 04 needs.
- **The exclusion-set feature distance is where the individual-fairness ethics live.** Choose it on paper; cite the features you excluded and *why*. A distance that quietly includes a proxy is a distance that certifies a biased model as individually fair.
- **`k`-NN scales badly on large tabular datasets.** Use approximate NN (`faiss`, `annoy`) if the eval set exceeds ~50k rows and the runtime becomes prohibitive. Note the approximation in the report.
- **Perturbation runs are embarrassingly parallel.** Fan them out.
- **For proxy perturbation, plausible alternatives are domain-specific.** For postcode / ZIP, use a set of postcodes with the same population density but different demographic composition — the harm model tells you which composition contrast matters. For institution, use the set of institutions similar in ranking but different in composition.
- **For the DAG, start from the domain-literature review, not from a whiteboard.** Credit-scoring, sepsis triage, and employment screening all have published DAGs from causal-inference papers or FDA guidance you can adapt. Cite the source.
- **Identifiability check is a formal step.** Use Pearl's do-calculus or a tool like `dowhy` / `causalgraphicalmodels` to check. "The counterfactual is identifiable under the DAG" is not a claim you make by inspection.
- **Linear-Gaussian SCM is fine for a first pass.** Do not reach for probabilistic programming on the first attempt; a linear-Gaussian SCM using scikit-learn regressions gives you a runnable counterfactual computation and a first-pass distribution.
- **Path-specific effects need the DAG to admit them.** If the DAG has direct `A → Ŷ` and indirect `A → M → Ŷ` paths, you can decompose. If it does not, the decomposition is trivial and reports only direct.
- **Holistic AI's `holisticai.bias.metrics` covers a superset of the Fairlearn / AIF360 metric list.** Do not re-report those numbers here; the value of the sweep is the *other* axes — explainability metrics (`feature_importance_spread`), robustness metrics (`adversarial_accuracy_delta`), privacy metrics (`shapr` proxies). Screenshot the heatmap; a table is fine too.

## Acceptance criteria

The submission is accepted when:

- The diagnostic config exists and names the perturbation set (with cited value sources), the NN distance and `k`, the Holistic AI selection, and the DAG status.
- The perturbation runner and NN runner execute end-to-end and emit their JSONL outputs.
- The perturbation-stability report shows the flip rate, max-delta distribution, top-10 individuals table, nearest-neighbour disagreement tail, and cross-refs to exercise 02.
- The Holistic AI sweep output exists with a heatmap / table and an explicit "diagnostic, not headline" statement plus the mod-106 / mod-107 backlog items each red flag becomes.
- Either the causal-fairness diagnostic exists (DAG + identifiability + SCM + distribution + assumptions paragraph) *or* the causal-skipped document exists with a defensible reason.
- The diagnostics summary README reconstructs the whole exercise for a reader who has read chapter 03 but not the runners.
- Every reported number is reproducible from a pinned Holistic AI version, a pinned NN library version, seed, and eval-set fingerprint.

## Stretch goals

- Run the **joint (protected × proxy) perturbation** — set `race` and `postcode` to correlated alternative values (an alternative-race + alternative-postcode-that-is-demographically-consistent). Joint perturbations reveal proxy-dominated decisions that single-axis perturbations miss.
- Add **path-specific effect decomposition** for the causal diagnostic — decompose the effect of `A` on `Ŷ` into direct plus indirect via each mediator. The path-specific view is what a regulator asking "is race being used, or is a legitimate downstream feature being used?" wants.
- Fit an **SCM with a probabilistic programme** (`pyro`, `numpyro`) instead of linear-Gaussian; propagate uncertainty into the counterfactual distribution as a credible interval per individual.
- Contribute a **proxy-detection catalogue** at `risk/mod-106/<system-id>/individual-causal/proxies.md` listing every column in the model's input schema, whether it is a known proxy for a protected attribute, the mechanism, and the citation. This catalogue is reused by mod-108 (guardrails) and mod-109 (monitoring).
- Wire an **inference-time perturbation-stability check** as a mod-108 guardrail stub: for each incoming request, generate a small perturbation set, rescore, and flag responses with high score delta. Ship as an intake artefact for mod-108; the guardrail itself lands there.

## Deliverable format

A single markdown document `mod-106-ex03-individual-causal.md` that indexes the artefacts, plus the perturbation report, Holistic AI output, and causal (or causal-skipped) document. The index document contains:

1. *Scope statement* — which intersectional cell(s) the diagnostics focused on and why; whether the causal-fairness diagnostic was attempted.
2. *Perturbation-stability summary* — the flip rate, max-delta distribution figure, top-10 table, NN disagreement tail.
3. *Holistic AI sweep* — the heatmap embedded, plus the red-flag axis list with backlog-item cross-refs.
4. *Causal-fairness verdict* — either the DAG + counterfactual distribution + assumptions paragraph, or the DAG-not-defensible statement.
5. *Cross-refs* — mod-102 harm-row IDs discharged (or not) by this diagnostic; mod-103 register-row IDs updated.
6. *Reproducibility manifest* — tool versions, seeds, eval-set fingerprint, SCM package version if attempted.
7. *Open items* — what exercise 04 needs to know (does the individual-fairness result require its own mitigation attention?); what mod-107 / mod-108 / mod-109 need to know (Holistic AI red flags, proxy-catalogue rows, inference-time guardrail stub).

## Submission

Attach the index markdown, the perturbation report, the Holistic AI output, and the causal document to the module submission channel. If the model / dataset is under NDA, redact identifiers but preserve the flip-rate distribution, the top-10 table shape, the DAG (or the DAG-not-defensible statement), and the counterfactual-fairness distribution — the diagnostic cannot be reviewed if the shape numbers are blanked out.
