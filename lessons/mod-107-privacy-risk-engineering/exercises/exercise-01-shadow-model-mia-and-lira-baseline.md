# Exercise 01 — Shadow-Model MIA and LiRA Baseline

**Estimated effort:** 3 hours

## Objective

Stand up a working membership-inference-attack (MIA) baseline on the module's running target model in the two forms chapter 02 pins as canonical: **Shokri-style shadow-model MIA** (S&P 2017) and **Carlini-style Likelihood Ratio Attack** (LiRA, S&P 2022). Report the results in the chapter-02 shape — AUC alongside **TPR at 1% FPR and 0.1% FPR**, sliced by overall / small-class / outlier-decile — and produce a first-cut residual-risk row that exercise 04's DP-SGD variants will overwrite. The exercise produces the artefact the rest of the module reads as the "no defences yet" baseline for the target system.

## Prerequisites

- Chapters 01 and 02 read.
- The module's running target model reachable in one of two shapes:
  - **In-process path** — a PyTorch or TensorFlow classifier you can query for logits / losses per record (a torchvision CIFAR-10 or CIFAR-100 CNN, a scikit-learn / xgboost tabular classifier wrapped for probability output, or an in-house model with a `predict_proba` interface).
  - **Endpoint path** — a scoring endpoint that returns logits or full probability vectors. Wrap it in a thin `logits(x)` adapter and treat the adapter as the model. If the endpoint returns only top-1 labels, state that explicitly in the threat-model line of the report — the attack is bounded above by what the API exposes.
- Access to a **population-shaped dataset** to draw shadow-model training and evaluation samples from. For canonical benchmarks CIFAR-10 / CIFAR-100 / Purchase-100 / Texas-100 work directly; for a domain target, a same-distribution holdout works.
- Compute budget for training the shadow-model population (see "Starter guidance" — a small model at N=64 shadow models is a laptop / single-GPU job; a bigger target model or N in the hundreds is a multi-GPU job).
- **ML Privacy Meter** installed at a pinned version. This is the module's working harness; you may implement LiRA by hand as a stretch, but the primary submission wraps the harness.
- The running system's mod-102 harm-model row that names an MIA scenario and the mod-103 register row that carries its inherent / residual columns. You will update the register row's residual and control-defeated cells at the end of the exercise.

## Problem statement

Build the two attacks on the same target model and eval slice, then produce a single unified report table.

- **Shokri shadow-model MIA.** Train N shadow models on random subsets of the shadow population, split into IN / OUT partitions per shadow model. Assemble an attack-training set of `(query-output, member-label)` pairs. Train a small attack classifier (an MLP or logistic model) that maps query output → member / non-member. Apply to the target model on the candidate records. Report AUC and TPR@1% FPR / TPR@0.1% FPR.
- **Carlini LiRA.** Reuse the shadow-model population. Per candidate record, fit the IN / OUT Gaussians over a chosen statistic (the standard choice is the *scaled* logit of the true class — `logit(p) = log(p/(1−p))`) across shadow models that included / excluded that record. Compute the per-record log-likelihood-ratio score for the target model's output on that record. Rank the candidate records by score. Report ROC — AUC and TPR@1% FPR / TPR@0.1% FPR — over the full candidate set.

For both attacks, produce:

- **Overall** number.
- **Small-class** slice — where a class exists with meaningfully fewer training members, report per that class. (For CIFAR-10 the classes are balanced; use CIFAR-100 or a rare-subpopulation slice instead.)
- **Outlier-decile** slice — records in the top decile by input-norm or by training-loss are the highest-vulnerability slice; report separately.

The comparison table lives in the report as chapter 02's per-slice shape. Both attacks must appear on the same table, on the same slices, so the reviewer sees the AUC-can-lie point directly.

The **threat model** the report defends must be stated up front: black-box vs. white-box, labels vs. top-k vs. logits vs. losses, rate-limited vs. unlimited queries, shadow-data population vs. training-data adjacency. Chapter 02's threat-model axes table is the reference — copy it into the report header.

## Requirements

Produce all seven items. Missing any is an audit finding on the eval.

1. **Threat-model header** — `risk/mod-107/<system-id>/mia/threat-model.md`. Names the four axes from chapter 02 (query interface, query rate, distribution knowledge, architecture knowledge) with the value chosen for each and one line of justification per value.
2. **Shadow-model training runner** — `risk/mod-107/<system-id>/mia/train_shadows.py`. Trains N shadow models (default N=64 for a first pass; N=128 or 256 for a stronger submission) on Poisson-sampled subsets of the shadow population. Records the IN/OUT membership matrix (one row per shadow model, one column per candidate record) as a `.npz` or `parquet` artefact. Saves shadow-model weights or hashes for reproducibility.
3. **Shokri attack runner** — `risk/mod-107/<system-id>/mia/run_shokri.py`. Loads the shadow models, builds the attack-training set, trains the attack classifier, applies it to the target model on the candidate records, emits per-record scores + ROC + AUC + TPR@fixed-FPR JSON.
4. **LiRA runner** — `risk/mod-107/<system-id>/mia/run_lira.py`. Loads the shadow-model IN/OUT matrix, computes per-record IN / OUT Gaussians of the scaled-logit statistic, computes the target model's per-record log-likelihood ratio, emits per-record scores + ROC + AUC + TPR@fixed-FPR JSON. Prefer wrapping ML Privacy Meter's LiRA implementation; note the version + config in the manifest.
5. **Slice report** — `risk/mod-107/<system-id>/mia/slice-report.md`. Chapter-02 fixed shape: threat model at top, per-slice table (overall / small-class / outlier-decile), Shokri and LiRA on adjacent rows so the AUC-vs-TPR@low-FPR contrast is visible, and a one-paragraph "what this tells us about the target model" note.
6. **ROC plot** — a rendered PNG or SVG per slice, showing both attacks with the low-FPR region (0 to 5% FPR) plotted on a linear x-axis and, as a companion plot, on a **log-scaled x-axis** so the low-FPR behaviour is readable. Both plots ship — the linear plot is what most readers first look at; the log plot is what the LiRA paper mandates.
7. **Reproducibility manifest** — `risk/mod-107/<system-id>/mia/manifest.json`. Pins the target-model artefact hash, the shadow-population data fingerprint, the number of shadow models N, the shadow-training epochs / hyperparameters, the ML Privacy Meter version, the Python / PyTorch or TF versions, the random seeds, and the script commit hash.

Cross-cutting rules:

- **Report AUC and TPR at fixed FPR together.** Never AUC alone. If the small-class slice or the outlier-decile slice does not have enough candidate records to estimate TPR at 0.1% FPR reliably (typical rule of thumb: at least a few thousand candidates), state that explicitly rather than reporting a number of unknown reliability.
- **Do not claim "no MIA possible."** The strongest defensible sentence is "under threat model T, attack A at configuration C, TPR at 1% FPR is X." Chapter 02's discipline stands.
- **Both attacks or neither.** A submission that runs only Shokri or only LiRA misses the point of the exercise — the pair is what makes the AUC-can-lie argument visible.
- **Slice worst cases matter.** The overall number is what makes the register row; the outlier-decile number is what an auditor will ask about first.

## Starter guidance

- **Prove the plumbing on N=8 shadow models first.** Train eight shadow models with a small target-shaped architecture (a shallow CNN on CIFAR-10, or an XGBoost model on a tabular problem). Confirm the IN/OUT matrix has the shape you expect (N rows × candidate-count columns), that the attack classifier trains at all, that the per-record LiRA score is finite. Only then scale N to 64 or 128. Debugging at N=128 is much slower than at N=8.
- **Use Poisson sampling on the shadow-training subsets.** Each shadow model's training subset is a random Bernoulli-per-record sample of the shadow population at a fixed inclusion probability (0.5 is the LiRA-paper convention). This matches the LiRA statistical assumption and it is what ML Privacy Meter expects.
- **The LiRA statistic is the scaled logit of the *true class*, not the predicted class.** `logit(p_true) = log(p_true / (1 − p_true))`. Softmax before the logit; do not use the raw pre-softmax logit. Getting this wrong silently degrades the attack by a lot.
- **Match the shadow architecture to the target.** If your target is a ResNet-50 and your shadows are shallow CNNs, the attack under-reports. If your target is a scikit-learn model and shadows are the same class of scikit-learn model at the same hyperparameters, the attack is at its strongest.
- **Two-Gaussian LiRA vs. one-Gaussian.** The Carlini paper describes the "offline" one-Gaussian variant (OUT-only fitting) and the "online" two-Gaussian variant (both IN and OUT fits). The online variant is stronger; ML Privacy Meter defaults to it. Report which you ran.
- **ML Privacy Meter version-pin.** Note the commit / release in the manifest. The attack surface evolves release to release; a report that cites "MIA TPR = X" without the harness version is not reproducible.
- **Slice power.** A slice with 200 candidate records cannot support a TPR@0.1% FPR estimate — there is no such rate at that sample size. Report the metric only where the sample count allows; flag the slice as "insufficient power" otherwise, do not fabricate a number.
- **Rate-limit note.** If your production API rate-limits queries, note whether the attack as configured respects that limit. A lab attack with unlimited queries produces a *lower-bound* threat; an attack that respects production rate-limits produces the operational-threat number. Both are valid; state which you ran.

## Acceptance criteria

The submission is accepted when:

- Both attacks run end-to-end on the target model with N ≥ 64 shadow models and produce per-record score JSONs.
- The slice report exists in the chapter-02 shape, with the threat-model header, per-slice rows for overall / small-class / outlier-decile, and both Shokri and LiRA on the same table.
- AUC and TPR@1% FPR and TPR@0.1% FPR are reported side-by-side for every slice where the sample count allows; slices without sufficient sample are labelled "insufficient power" rather than left blank or filled with a fabricated number.
- The ROC plot ships in both linear and log-x variants, per slice.
- The reproducibility manifest lets a second engineer regenerate every number from the artefacts alone (harness version, target-model hash, shadow-population fingerprint, N, seeds, script commit).
- The mod-103 register row for the MIA scenario is updated with the LiRA-based residual number in the "inherent" column (this is the *no-DP-SGD, no-defences* baseline) and a note that exercise 04 will overwrite the residual column with the DP-SGD variants.
- One-paragraph interpretive note: what does the AUC-vs-TPR@low-FPR gap on this target model say? Is the target model in the "low overall vulnerability with a dangerous tail" regime or the "flat everywhere" regime? Chapter 02's ROC-shape argument is the reference.

## Stretch goals

- **Threshold attacks as a sanity check.** The simplest possible MIA is "predict `member` when the target model's loss on the record is below a threshold." Add this attack to the comparison table. If your attack classifier does not beat the threshold attack by a meaningful margin, something in the attack pipeline is under-tuned.
- **Per-record extremes.** Rank the candidate records by LiRA score; export the top-10 and bottom-10 as a table with input summary and score. This is the shape the "worst-case privacy-vulnerable records" list in the mod-103 register consumes.
- **Sub-population MIA.** If the target model is trained on data with a demographic attribute, run LiRA per demographic slice. Disparate MIA vulnerability across demographic groups is a cross-cutting mod-106 finding.
- **Query-interface downgrade sensitivity.** Re-run LiRA with a downgraded query interface (top-1 label only, then top-5 labels, then top-5 probabilities, then full logits). Report the four numbers as a curve. This tells the API owner what the attack ceiling is at each exposure level and directly supports a mod-108 API-shape decision.
- **Attack under distribution shift.** Draw the shadow population from a *different* distribution than the target's training data (a related-but-not-identical dataset). Report the drop in LiRA TPR@1% FPR. This is the "how strong does the attacker need to be" argument the threat-model header rests on.
- **User-level MIA.** If subjects contribute many records (typical for LLM fine-tunes on user conversations), run MIA at the user level — group candidate records by user and treat the whole group as one candidate. The LiRA score aggregates across the group.

## Deliverable format

A single markdown document `mod-107-ex01-mia-baseline.md` that indexes the artefacts, plus the threat-model header, slice report, runners, ROC plots, and manifest. The index document contains:

1. *Target-system and data summary* — one paragraph naming the model under evaluation, the shadow population, and whether attacks ran in-process or against an endpoint wrapper.
2. *Threat-model header* — the four-axis block from `threat-model.md`, verbatim.
3. *Overall slice table* — Shokri and LiRA rows, AUC, TPR@1% FPR, TPR@0.1% FPR.
4. *Small-class / rare-subpopulation slice table* — same columns; note the class definition.
5. *Outlier-decile slice table* — same columns; note the outlier definition (input-norm, loss, or something task-specific).
6. *ROC plots* — one linear-x, one log-x, per slice, embedded with captions.
7. *Interpretive note* — one paragraph: does the AUC / TPR@low-FPR gap on this target look flat or does it have a dangerous tail? What does that imply for the DP-SGD budget exercise 04 will need to target?
8. *Reproducibility manifest* — the block for human read plus a pointer to `manifest.json`.
9. *Register handoff* — the mod-103 row updated with the inherent-column LiRA number and the placeholder for exercise 04's residual overwrite.
10. *Open items* — anything for exercises 02–05 or for level-30 methodological calibration (confidence intervals on the ROC, statistical-significance framing of the slice differences — see chapter 06 §"Boundary 2").

## Submission

Attach the index markdown, the slice report, the threat-model header, the runners, the manifest, and the ROC plots to the module submission channel. If the target model / dataset is under NDA, redact the identifiers but preserve the numeric results, the ROC plots (with axes and legend intact), and the manifest — a reviewer cannot review an MIA baseline whose numbers are blanked out.
