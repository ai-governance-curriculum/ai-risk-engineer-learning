# Exercise 04 — Extraction, MIA, and the Shared API-Surface Defence Set

**Estimated effort:** 3 hours

## Objective

Engineer one **model-extraction** attack (Tramèr / Knockoff-Nets shape) and one **membership-inference** attack (Shokri shadow-model or Yeom loss-threshold, LiRA if feasible) against the running system's deployed inference API — or a faithful staging replica — enough to establish *baseline attack numbers* on the endpoint. Then specify and measure the shared defence set from chapter 04 — per-caller query-rate control, aggregate query-pattern anomaly, output perturbation / top-k / confidence redaction, and DP-training as a survey-level baseline. Optionally run one **model-inversion** probe (Fredrikson-style) where the endpoint returns confidences and the training data is sensitive. Every attack number and every defence delta lands as a residual-risk update on the mod-103 register in exercise 05.

## Prerequisites

- Chapter 04 read; chapter 01 available for the taxonomy and cross-tag; chapter 05 skimmed for the report shape; chapter 06 for the sec-L35 handoff.
- Exercise 01 completed. Every attack you run here must appear as a row on the cross-tag sheet with `nist_category` in `{extraction, membership_inference, inversion}` and an `in-scope-this-module` scope flag.
- The running system's **inference endpoint** reachable — production, staging, or a local replica that mirrors the deployed serving path (same model, same output shape, same auth surface).
- A **query budget** you are authorised to spend on the endpoint. Extraction is a query-heavy attack; agree the budget with the API owner before running. If you are querying a shared production endpoint, use a staging replica.
- A **surrogate dataset** for Knockoff-Nets-style extraction — an in-domain public dataset, or a synthetic dataset generated from the target distribution.
- **Shadow-training compute** for MIA — enough resources to train at least one shadow model (Shokri / Yeom) or, for LiRA, a fleet of shadow models (deferred to mod-107 if infeasible).
- Access to the mod-103 register so extraction / MIA / inversion residual cells can be updated, and the sec-L35 handoff-contract table (chapter 06) for the rate-limit specification.

## Problem statement

Three attack-and-defence pairs, each measured against the actual (or staging-replica) endpoint.

**Extraction (required):**
- Attack: Tramèr-style query-based extraction on a shape appropriate to the target (equation-solving for linear / logistic models; Knockoff-Nets for deep models using an in-domain surrogate dataset).
- Threat model: black-box, active-querier, extraction goal. Vary the query budget and record surrogate agreement rate on a held-out target-distribution test set at each budget.
- Defence set to measure:
  - **Per-caller query-rate control** at a specified budget (queries-per-caller-per-day). Measure the attack again with the rate limit in place — how many days does extraction take at N queries/day?
  - **Output perturbation** — top-k-only outputs (k=1, k=3), confidence redaction to fixed buckets, small noise on non-argmax logits. Measure surrogate agreement under each shape.
  - **Aggregate query-pattern anomaly** — specify the metric (aggregate query entropy, per-caller pattern deviation, input-space coverage) and the alert threshold. Do not implement the metric collector in this exercise (that is sec-L35 + mod-109); specify it well enough that they can.

**Membership inference (required):**
- Attack: pick one — Shokri shadow-model MIA (train one shadow model on similar distribution, compute per-record membership advantage) or Yeom loss-threshold MIA (single-model, calibrate a threshold on loss / confidence). LiRA (Carlini et al., 2022) is preferred where compute allows.
- Threat model: black-box, active-querier, membership goal.
- Metrics to report:
  - **Attack advantage** (TPR − FPR) at a chosen operating point.
  - **TPR at 0.1% FPR and 1% FPR** — the LiRA framing; a MIA that has 5% TPR at 0.1% FPR is a real privacy leak, a MIA at 60% TPR at 60% FPR is trivial.
  - **Full ROC curve** — not one operating point.
- Defence set to measure:
  - **Output perturbation** — reruns the attack under confidence redaction / top-k-only / noise.
  - **DP-training** — enable DP-SGD in the training pipeline (small ε for illustration, e.g., ε ∈ {1, 8}); rerun the MIA and report the advantage delta. Formal (ε, δ) accounting is mod-107 territory; note the handoff.
  - **Model regularisation / early stopping** — a hygiene lever; note whether the deployed model shows a large train-test gap (a MIA-vulnerability signal).

**Inversion (optional but recommended if the endpoint returns confidences and training data is sensitive):**
- Attack: Fredrikson-style model inversion (attribute reconstruction from confidences) on a small illustrative slice — do not run on real customer records without a data-handling review.
- Report reconstruction fidelity qualitatively (or with the task-specific metric from chapter 04) and the defence-lever sensitivity: does confidence redaction move the fidelity number?

## Requirements

Produce all nine items below. Missing any is an audit finding on the eval.

1. **Attack spec** — `risk/mod-105/<system-id>/api-surface/attacks.yaml`. One block per attack: family, threat model, query budget or shadow-training budget, dataset (surrogate for extraction; membership / non-membership set for MIA), seed, `scenario_id` from exercise 01.
2. **Extraction runner** — `risk/mod-105/<system-id>/api-surface/run_extraction.py`. Queries the endpoint at a sequence of budgets, trains the surrogate on the query-response pairs, evaluates surrogate agreement on a held-out target-distribution test set at each budget, emits `extraction-results.jsonl`.
3. **MIA runner** — `risk/mod-105/<system-id>/api-surface/run_mia.py`. Constructs the shadow-training or loss-threshold setup per the paper, computes per-record membership scores on the members / non-members set, emits full-ROC scores and per-record predictions to `mia-results.jsonl`.
4. **Inversion runner (optional)** — `risk/mod-105/<system-id>/api-surface/run_inversion.py`. If in scope, produces the reconstruction attempt on a small illustrative slice with any personal data hashed / redacted upstream.
5. **Defence-specification document** — `risk/mod-105/<system-id>/api-surface/defence-spec.md`. Per lever:
   - **Per-caller rate limit**: the number (queries/caller/day), the auth-token dimension (bearer / api-key / tenant-id), the exception process for legitimate high-volume callers, and the ownership (this module specifies, sec-L35 implements). This is the specification row that goes on the handoff contract from chapter 06.
   - **Aggregate query-pattern anomaly**: the metric formula (e.g., query-input Shannon entropy over a sliding window per caller; per-caller L2 distance to a legitimate-use centroid), the alert threshold, the false-positive-review process. Ownership: this module specifies the metric; sec-L35 wires it into the SIEM; mod-109 records the monitor.
   - **Output perturbation shape**: top-k value, confidence-bucket granularity, non-argmax noise magnitude. Ownership: this module specifies; ML platform implements.
   - **DP-training**: target ε for illustration and the utility-privacy tradeoff observed. Handoff-flagged to mod-107 for formal accounting.
6. **Defence-delta table** — `risk/mod-105/<system-id>/api-surface/defence-panel.md`. Per attack, per lever: baseline attack number, defence-enabled attack number, delta, utility cost. For MIA: advantage delta at 0.1% and 1% FPR, not just at a mid-ROC operating point. For extraction: surrogate agreement delta at fixed query budget.
7. **Query-budget curves** — `risk/mod-105/<system-id>/api-surface/curves/*.png`. Extraction: surrogate agreement vs. query budget, with and without the rate limit projected onto a wall-clock axis. MIA: full ROC per defence setting.
8. **Handoff-contract row(s)** — a diff / addition to the chapter-06 handoff-contract table for the system, filling in specifier + implementer + operator + evidence-artifact per new lever this exercise specifies. This is the artifact the sec-L35 peer signs off on.
9. **Results markdown** — `risk/mod-105/<system-id>/api-surface/results.md`. Sections: methodology (attack spec, defence spec, endpoint reference, query-budget authorisation), attack results (extraction, MIA, inversion), defence deltas (table), curves, ownership boundary (chapter-06 handoff contract rows for each lever), mod-107 handoff flags for DP-accounting and LiRA-depth work, register handoffs.

Cross-cutting rules:

- **The MIA number is TPR-at-low-FPR, not a single operating-point advantage.** A "we measured MIA and got 55%" number is uninterpretable; the full ROC with TPR at 0.1% and 1% FPR is the shippable output.
- **DP-training is baselined here as a defence lever with an illustration ε; the formal accounting is mod-107.** Do not overclaim formal privacy in this exercise; the handoff row is what you produce.
- **Output redaction has a utility cost.** Report the cost alongside the attack delta — a defence that halves extraction agreement but breaks a legitimate consumer's confidence-consumption path is a defence that will get rolled back.
- **The rate limit is a *specification* — this role does not implement it at the gateway.** Chapter 06 is explicit that sec-L35 owns enforcement; your job is the specification and the evidence-artifact contract.
- **Do not run any of these attacks against a production endpoint without authorisation.** Extraction is a query-heavy attack that will show up in the API owner's dashboards; MIA involves probing with attempted-member records. Use staging; where you must use production, agree the budget and window in writing.
- **No customer records in the deliverable.** MIA membership sets and inversion targets are constructed from synthetic or already-public records; where real records are involved, they are hashed / redacted upstream and referenced by id.

## Starter guidance

- **For extraction, start with a small query budget and confirm surrogate agreement rises with budget.** A flat curve means the extraction plumbing is wrong (the surrogate isn't learning from the queries) or the surrogate architecture is too weak; fix before scaling budget.
- **Knockoff-Nets needs a surrogate dataset that overlaps the target distribution.** A CIFAR-100 surrogate for an ImageNet target works surprisingly well; a totally out-of-distribution surrogate does not. Choose the surrogate deliberately and note the choice.
- **For MIA, Yeom loss-threshold is the cheapest to run and often close to Shokri-shadow performance.** Start there. Escalate to shadow-model or LiRA only if the loss-threshold advantage is inconclusive.
- **LiRA is expensive; note the handoff.** Proper LiRA-calibrated advantage curves require training dozens or hundreds of shadow models per target record. If your compute doesn't allow, run Yeom for the baseline and flag the LiRA depth as a mod-107 follow-up.
- **The rate-limit number should come from the extraction curve, not from a business rule.** If extraction achieves 90% agreement at 100k queries, the rate limit needs to reduce the achievable agreement or the achievable wall-clock time to a level that makes the attack cost prohibitive. Argue the number from the curve.
- **Aggregate query-pattern anomaly is the metric a real defence uses, but it is easy to false-positive on legitimate batch workloads.** Specify the false-positive-review process alongside the alert threshold; a monitor with no FP-review process becomes silent noise within a quarter.
- **DP-SGD hyperparameters are sensitive.** Use the DP-SGD implementation in your framework (Opacus for PyTorch, TensorFlow Privacy) rather than rolling your own; a bespoke DP-training implementation without a formal audit is worse than none.
- **Do not read model-inversion outputs as evidence of a specific individual's record.** Inversion fidelity is often qualitatively "recognisable but not photographic"; the ethical framing is *the model leaks class-level attributes*, not *the model reproduces person X*.
- **The chapter-06 handoff contract row is a small artifact but is the highest-leverage output.** A specifier / implementer / operator / evidence table row is what closes the loop with the sec-L35 peer; without it, the "defence" is a slideware claim.

## Acceptance criteria

The submission is accepted when:

- The attack spec exists at the specified path and maps every attack to a `scenario_id` on the exercise-01 cross-tag sheet.
- Extraction and MIA runners execute end-to-end on the endpoint (production or staging) and emit result JSONL files.
- Extraction reports surrogate agreement at ≥3 query budgets, with the agreement-vs-budget curve plotted.
- MIA reports advantage, TPR at 0.1% FPR, TPR at 1% FPR, and a full ROC curve; single-point-advantage claims fail acceptance.
- The defence-specification document names each lever's specifier / implementer / operator / evidence artifact, filling in the chapter-06 handoff-contract rows for the running system.
- The defence-delta table shows every lever measured against every attack, with the utility cost alongside the attack delta.
- DP-training is baselined at least at survey level, with the mod-107 handoff for formal (ε, δ) accounting flagged.
- Inversion (if in scope) reports a defensible fidelity metric and the confidence-redaction sensitivity; if out of scope, the reason is stated.
- The results markdown pins tool versions, seeds, query budget authorised, and endpoint reference.
- Register-row-ids to update, sec-L35 handoff-row updates, and mod-107 depth-handoff flags are called out in the open-items section for exercise 05.

## Stretch goals

- Run **LiRA** (Carlini et al., 2022) on a small-scale target to produce a calibrated TPR-at-low-FPR curve. Even a 16-shadow-model LiRA run is more informative than a 1-shadow-model Shokri; note the compute cost.
- Baseline a **prediction-confidence-redaction gradient** — sweep from full logits → top-k → top-1 → bucketed confidence, and plot extraction agreement and MIA advantage against each. The gradient shows the API owner exactly what they lose in utility for what they buy in defence.
- Specify a **query-fingerprint monitor** for mod-109 — a probe that identifies extraction-campaign shape (uniform coverage, low repetition, high input entropy per caller) and alerts on trend. The monitor lives at mod-109; this exercise produces the metric spec.
- Draft the **incident-response runbook branch** for "extraction campaign detected" — hand off to mod-111. Threshold levels, containment steps (per-caller rate cliff, credential revoke, forensic query-log preservation), and the notification chain.
- If your endpoint fronts a foundation-model API, note the **third-party-provider handoff** — extraction against a hosted-provider endpoint escalates to the provider's platform security team; mod-110 owns the supplier-risk view. Cite the handoff.

## Deliverable format

A single markdown document `mod-105-ex04-api-surface.md` that indexes the artifacts, plus the artifacts themselves. The index document contains:

1. *Endpoint and query authorisation* — one paragraph naming the endpoint (production / staging), the query budget authorised, and any query-log preservation you enabled for the exercise.
2. *Attack summary* — extraction and MIA (and inversion if in scope) with threat models at a glance.
3. *Extraction results* — surrogate agreement at each budget with the curve embedded; the queries-to-target-agreement number.
4. *MIA results* — advantage, TPR at 0.1% and 1% FPR, ROC curve embedded.
5. *Inversion results* — the fidelity metric and confidence-redaction sensitivity, or the reason for scope-out.
6. *Defence specification* — the per-lever spec table, including the sec-L35 chapter-06 handoff-contract rows.
7. *Defence deltas* — the panel table with attack delta and utility cost per lever.
8. *DP-training baseline* — the illustration ε, utility delta, MIA advantage delta; the mod-107 handoff-note.
9. *Reproducibility* — tool versions, seeds, endpoint reference, dataset provenance.
10. *Handoffs* — sec-L35 contract row updates, mod-107 flags, mod-109 monitor specs, mod-111 runbook seed.
11. *Register handoffs* — the mod-103 rows the residual-risk update will touch in exercise 05.
12. *Open items* — anything for exercise 05 or upstream mod-102 / mod-103 / mod-107 / mod-108.

## Submission

Attach the index markdown, attack + defence specs, runner scripts, JSONL results, curves, and the chapter-06 handoff-contract diff to the module submission channel. Do **not** attach raw query-log dumps or MIA membership sets that contain real records; hash and reference. If the endpoint is under NDA, redact the endpoint reference but preserve the attack numbers, the defence-delta table, the handoff-contract row shape, and the mod-107 handoff-note — a reviewer cannot review an API-surface eval whose numbers and specification rows are blanked out.
