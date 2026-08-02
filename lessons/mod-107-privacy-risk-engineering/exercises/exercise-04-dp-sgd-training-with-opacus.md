# Exercise 04 — Differentially Private Training with Opacus

**Estimated effort:** 3 hours

## Objective

Train the running system's target model **twice** with DP-SGD at two different (ε, δ) budgets using **Opacus** (or **TensorFlow Privacy** if the target is TF/JAX), then produce the chapter-04 **utility / privacy trade-off** curve — task-level accuracy, MIA suppression, extraction-rate suppression, and per-slice utility — across the two DP variants and the non-private baseline. Defend the chosen operating point at a mock review with the three arguments chapter 04 §"Defending the (ε, δ) choice at review" pins: threat-model calibration, utility calibration, and comparability to published references. The exercise overwrites the *residual* column on the mod-103 register rows that exercises 01-03 populated with inherent numbers.

## Prerequisites

- Chapters 01 and 04 read; exercises 01 (MIA baseline) and, if the target is an LLM, exercise 03 (memorisation baseline) completed. Exercise 02 helpful but not required.
- **A retrainable target model.** The running system's target model in a form you can retrain from scratch on the same training data — a PyTorch model for Opacus, a TF/Keras model for TensorFlow Privacy. If the target is not retrainable in-house (e.g., a large fine-tuned LLM you cannot re-fine-tune within the exercise budget), pick a *representative surrogate* — same architecture family, same task, smaller — and note that the surrogate is what the exercise measured.
- **Opacus** at a pinned version (or TF Privacy + `dp_accounting` at pinned versions).
- **The eval harness from exercise 01** (LiRA runner and its shadow-model population) so you can re-measure MIA on each DP variant, and — if the target is an LLM — **the elicitation runner from exercise 03** so you can re-measure extraction on each DP variant.
- The mod-103 register rows updated after exercises 01-03 with inherent numbers. You will fill their residual columns and their control-defeated columns from this exercise.

If your target cannot be retrained inside the exercise budget, submit against a documented surrogate and open an item to run the full DP-SGD training on the real target during a later capacity window.

## Problem statement

Three training runs, three sets of eval numbers, one trade-off curve, one defended operating point.

### The three runs

1. **Non-private baseline.** Retrain the target model with standard SGD / Adam at the target's normal hyperparameters — no per-example clipping, no noise. This is the ε = ∞ reference row.
2. **DP variant A — a *moderate* privacy budget.** ε in the range 6-10 at δ ≤ 10⁻⁵ (or δ ≤ 1/N, where N is the training-set size, whichever is smaller). Pick the noise multiplier σ and clipping norm C to hit the target budget under the accountant's projection over your planned training steps. This variant is the "meaningful DP but keeps utility" operating point.
3. **DP variant B — a *strict* privacy budget.** ε in the range 1-3 at the same δ. Larger σ; likely larger utility cost. This variant is the "we accept material utility cost for a strong bound" operating point.

The two DP variants are not "one right and one wrong" — they are the two ends of the trade-off curve the report defends. The operating point you choose is one of them (or, with justification, a third point in between).

### The eval on each run

For every run — baseline, variant A, variant B — produce:

- **Task-level utility.** Overall accuracy (or AUC, or task-appropriate metric) on the held-out eval set; plus **per-slice utility** at least on the slices exercise 01 used (small-class, outlier-decile) *and* on any protected-attribute slice the running system carries from mod-106. The chapter-04 disparate-impact point (Bagdasaryan et al. 2019) makes per-slice utility non-negotiable.
- **MIA suppression.** LiRA TPR@1% FPR and TPR@0.1% FPR from the exercise-01 runner, re-measured against the DP-trained model with the same shadow-model population.
- **Extraction suppression** (LLM targets only). Canary-curve extraction rate at duplication buckets 10 and 100 from the exercise-03 runner, re-measured on the DP-trained model.
- **Attribute-inference suppression** (tabular targets only). Advantage over base rate from the exercise-02 runner, re-measured on the DP-trained model.

### The defence

Write a one-page **operating-point defence memo** that answers the three chapter-04 questions:

1. **Threat-model calibration.** Why is the chosen ε enough for the disclosure risk this system faces? Point to a specific residual MIA / extraction / attribute-inference number under the chosen ε.
2. **Utility calibration.** Why is the chosen ε acceptable for the task? Point to the trade-off curve and the residual utility on the metrics that gate release. Note the per-slice utility cost, in particular whether any protected slice paid a disproportionate cost.
3. **Comparability.** Which published reference points does the chosen ε align with? US Census 2020 (per its Disclosure Avoidance System documentation), published DP-image-classification results at similar model / data scale, published DP-language-model results if the target is an LLM. Cite at least two references.

## Requirements

Produce all nine items:

1. **Config file** — `risk/mod-107/<system-id>/dp-sgd/config.yaml`. One block per run: base training hyperparameters (learning rate, batch size, steps / epochs, optimiser), and — for the two DP variants — the DP-SGD hyperparameters: noise multiplier σ, clipping norm C, expected sample rate q (Poisson sample probability), and the target (ε, δ). Cites the accountant used to translate (σ, C, q, T) → (ε, δ).
2. **Training runner** — `risk/mod-107/<system-id>/dp-sgd/train.py`. Wraps the target model in Opacus's `PrivacyEngine.make_private` (or TF Privacy's equivalent). Handles module validation and layer replacement via `ModuleValidator.fix`. Uses the DP dataloader for Poisson sampling. Emits the trained checkpoint and a `training-log.json` with per-epoch training loss, held-out utility, and the running (ε, δ) from the accountant. Runs three times (baseline, variant A, variant B).
3. **Attack-suppression re-runner** — `risk/mod-107/<system-id>/dp-sgd/measure_attacks.py`. Re-invokes the exercise-01 LiRA runner, the exercise-02 attribute-inference runner (if applicable), and the exercise-03 elicitation runner (if applicable) against each of the three checkpoints. Emits a per-checkpoint attack-results JSON.
4. **Trade-off table** — `risk/mod-107/<system-id>/dp-sgd/tradeoff-table.md`. Chapter-04 shape: one row per run, columns for (ε, δ, σ, C), overall task utility, per-slice utility (at minimum small-class / outlier-decile / any protected-attribute slice), LiRA TPR@1% FPR and @0.1% FPR, and — where applicable — extraction rate at duplication 10 / 100 or attribute-inference advantage. The baseline row is at ε = ∞; variant rows below.
5. **Trade-off plot** — a rendered PNG or SVG showing utility on one axis and MIA (or extraction) on the other axis, with the three points (baseline, variant A, variant B) marked and the operating-point choice highlighted. This is the artefact the review reads first.
6. **Operating-point defence memo** — `risk/mod-107/<system-id>/dp-sgd/defence-memo.md`. One page. Three sections: threat-model calibration, utility calibration, comparability. Chooses the operating point (variant A, variant B, or an interpolated third) and defends it with the numbers from the trade-off table plus at least two cited external references.
7. **DP-SGD failure-mode checklist** — `risk/mod-107/<system-id>/dp-sgd/checklist.md`. The six operational failure modes from chapter 04 §"Operational failure modes that silently invalidate the privacy claim" (non-Poisson batching / unfixed BatchNorm / gradient accumulation / warm-start from non-DP checkpoint / data-dependent hyperparameter selection / non-DP eval on training data) with a pass / fail line each, and a short note where the run addressed each.
8. **Register updates** — the mod-103 rows for MIA, attribute inference, model inversion, extraction / memorisation updated with the *residual* column populated from the chosen operating point; the *control-defeated* column populated from the "DP noise multiplier reduced by X" and "DP training absent" variants (either derived from variant A / baseline numbers or explicitly generated with a fourth reduced-noise run at your option).
9. **Reproducibility manifest** — `risk/mod-107/<system-id>/dp-sgd/manifest.json`. Pins Opacus (or TF Privacy) version, accountant name + version, `dp_accounting` version if separate, target-model architecture hash, training-data fingerprint, hyperparameters, checkpoint hashes for all three runs, random seeds, and script commit hash.

Cross-cutting rules:

- **Pin the accountant.** RDP, PLD, GDP, or the moments accountant — all are valid; the report must name which one produced the (ε, δ). Chapter 04 §"Privacy accounting" is the reference.
- **Report utility and privacy together, not in isolation.** A row that shows only ε or only accuracy is a defect. The point of the exercise is the trade-off.
- **Report per-slice utility.** DP-SGD's utility cost is not uniform (Bagdasaryan et al. 2019); a per-slice degradation must not be hidden behind an overall number. Cross-linked to mod-106.
- **Do not warm-start from a non-DP pretrained checkpoint without stating it.** If the base model is a public pretrained model whose training was non-private, the (ε, δ) applies to the *fine-tune step only*. This is a load-bearing note that the checklist entry captures.
- **Do not claim "DP means MIA cannot happen."** DP-SGD suppresses MIA empirically; the residual number is *measured*, not asserted. Chapter 04 §"Defending the (ε, δ) choice at review" is the reference.
- **State the "neighbouring" definition.** Record-level DP-SGD vs. user-level DP-SGD are different claims. If subjects contribute many records, address the group-privacy adjustment in the defence memo.

## Starter guidance

- **Get the non-DP baseline first, and verify the accuracy matches what the running system reports.** If the baseline retrain does not reproduce the target's known accuracy, the training-data or hyperparameter setup is off and neither DP variant is meaningful.
- **Use `ModuleValidator` before training.** Opacus's `ModuleValidator.validate(model)` returns a list of DP-incompatible modules (typically BatchNorm layers, some LayerNorm paths). Run `ModuleValidator.fix(model)` to replace them. Skipping this step is one of the six checklist failures.
- **Use the DP dataloader — the Poisson-sampling one.** Opacus's `DPDataLoader` (or the sample-rate parameter to `make_private_with_epsilon`) is what makes the accountant's (ε, δ) claim valid. A standard PyTorch `DataLoader` violates the Poisson-sampling assumption and silently invalidates the number.
- **`make_private_with_epsilon` vs. `make_private`.** `make_private_with_epsilon` takes a target (ε, δ) and epoch count and computes σ for you; `make_private` takes σ directly. For variant A / variant B, either is fine — use `make_private_with_epsilon` if you want the library to solve for σ from your target ε, use `make_private` if you want direct control over σ.
- **Micro-batch memory tuning.** Per-example gradient computation is memory-intensive. Opacus's `BatchMemoryManager` lets you run *virtual* large batches assembled from small micro-batches without breaking the accountant. Use it if you hit memory pressure; skipping and using tiny actual batches usually hurts utility.
- **Clipping norm C tuning.** A first-cut rule: set C to the *median* of per-example gradient norms observed in a non-DP warmup step. Too-small C over-clips and underfits; too-large C makes the noise dominate. Chapter 04's clipping-norm row is the reference.
- **Don't tune ε by looking at MIA numbers.** Tuning ε to make MIA vanish and then reporting that ε is the operating point spends privacy budget the accountant did not track (chapter 04 checklist item 5). Either pre-select ε and report the resulting MIA, or use published community reference points to pick ε and then measure.
- **The MIA re-run reuses the shadow-model population from exercise 01.** You do not need to retrain shadow models — the shadow population is invariant; only the *target model* changed to the DP-trained variant. This is a large time saving.
- **Slice-level utility can surprise.** DP-SGD often costs disproportionate utility on under-represented groups. If you see a large per-slice degradation, do not smooth it over — chapter 06's residual claim requires the per-slice number.
- **A comparability citation of "ε = 3 is common in the DP literature" is not enough on its own.** Cite the specific paper / product (Apple's per-day ε for private data collection, US Census 2020 DAS ε, Abadi 2016's DP-SGD paper's numbers on MNIST / CIFAR). Chapter 04 §"Defending the (ε, δ) choice at review" is the reference.

## Acceptance criteria

The submission is accepted when:

- Three training runs (baseline, variant A, variant B) complete, produce checkpoints, and log per-epoch training / eval metrics.
- MIA (both TPR@1%FPR and TPR@0.1%FPR), and — where applicable — extraction rate at duplication 10 / 100 and attribute-inference advantage, are re-measured on all three checkpoints using the exercise-01 / -02 / -03 runners.
- The trade-off table is complete with per-slice utility rows and the attack-suppression columns.
- The trade-off plot is rendered and the chosen operating point is highlighted.
- The operating-point defence memo is authored with the three chapter-04 sections and at least two cited external references for comparability.
- The DP-SGD failure-mode checklist is filled in with a pass / fail line per item and short note where each was addressed.
- The mod-103 register rows have their residual columns updated from the operating point and their control-defeated columns populated from the "DP absent" and "DP noise reduced" variants.
- The reproducibility manifest pins the Opacus / TF Privacy version, the accountant, the checkpoint hashes, the seeds, and the script commit.
- The operating-point choice is explicit: "we ship variant A because [threat-model, utility, comparability arguments]" — not left implicit.

## Stretch goals

- **A third DP variant at an interpolated ε.** Add ε = 4 or ε = 5 to fill in the trade-off curve; the smoother curve makes the operating-point defence stronger.
- **Larger batch / longer training.** DP-SGD utility often improves with larger effective batches and more epochs at fixed ε. If compute allows, sweep batch size at fixed ε and report the utility delta; note that at fixed ε, more steps requires more noise per step (the accountant does the accounting for you) — the shape of the sweet spot is instructive.
- **User-level DP variant.** If the running system has subjects contributing many records (an LLM fine-tune on customer conversations, a per-user recommendation model), add a fourth run at user-level DP (McMahan et al. 2018) and compare the resulting (ε, δ) claim's semantics. This is a chapter-04 subtlety §"User-level DP vs. record-level DP" that deserves an operational example.
- **PRV accountant comparison.** For one variant, run both an RDP accountant and a PRV / PLD accountant and report the delta in the returned (ε, δ). This surfaces the accountant-choice subtlety chapter 04 flags and makes the report's "pin the accountant" discipline concrete.
- **Warm-start experiment.** If the running system uses a pretrained checkpoint as the initialisation, run one variant with the warm start and one without; compare the utility / privacy trade-off. The warm-start note ("the (ε, δ) applies to the fine-tune step only") becomes the operational reason to prefer one over the other.
- **Cross-check against mod-106's fairness report.** The per-slice utility numbers here should flow to mod-106's slice-report; note whether DP-SGD widened or narrowed the pre-existing per-slice gaps. A widened gap is a mod-106 mitigation trigger; a narrowed gap is a positive side-effect worth documenting.

## Deliverable format

A single markdown document `mod-107-ex04-dp-sgd.md` that indexes the artefacts. The index document contains:

1. *System summary* — one paragraph naming the target, the training-data footprint, and — if applicable — the surrogate used and why.
2. *DP-SGD configuration* — the config block for the two variants with (ε, δ, σ, C, q, T) called out and the accountant named.
3. *Trade-off table* — the full chapter-04 table with utility and attack-suppression columns for the baseline and both variants.
4. *Trade-off plot* — embedded figure with caption; the chosen operating point highlighted.
5. *Per-slice utility* — a separate table showing baseline / variant A / variant B utility across the same slices (small-class, outlier-decile, protected-attribute).
6. *Operating-point defence memo* — the three-section memo, verbatim.
7. *Failure-mode checklist* — the six-item pass / fail list with notes.
8. *Register handoffs* — the four mod-103 rows (MIA, attribute inference, inversion, extraction) with residual and control-defeated columns updated from the operating point.
9. *Reproducibility manifest* — pointer to `manifest.json`.
10. *Open items* — anything for exercise 05 (Presidio adds layered defence for the extraction row's residual column) or for level-30 (attack-tuning discipline, distribution-shift robustness of the MIA re-measurement, cross-run comparability with published DP-SGD baselines).

## Submission

Attach the index markdown, the trade-off table, the trade-off plot, the defence memo, the failure-mode checklist, and the reproducibility manifest to the module submission channel. Do not attach the DP-trained checkpoints if size is prohibitive; the manifest's checkpoint hash plus the training-data fingerprint plus the pinned Opacus version plus the seeds is enough for a second engineer to reproduce.
