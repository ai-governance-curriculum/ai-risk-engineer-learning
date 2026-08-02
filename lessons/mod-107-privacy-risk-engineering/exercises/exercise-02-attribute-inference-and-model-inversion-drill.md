# Exercise 02 — Attribute Inference and Model Inversion Drill

**Estimated effort:** 2 hours

## Objective

Run two attacks that MIA does not cover: **attribute inference** on a tabular target (Fredrikson-style — advantage over a base-rate prior for a sensitive attribute) and **model inversion** on a vision target (Fredrikson 2015-style — a reconstruction of a training-set class from confidence-vector outputs). Report each in the chapter-03 shape — attribute-inference *advantage over base rate* per sensitive attribute; model-inversion *reconstruction quality plus identifiability* — and add the resulting rows to the running system's residual-risk baseline. The exercise closes the two blind spots MIA leaves open on tabular and vision targets.

## Prerequisites

- Chapters 01 and 03 read; exercise 01 completed so the MIA baseline exists for the running system (this exercise adds two more rows to the same register scenario, using the same target model where possible).
- **A tabular target** with at least one clearly sensitive column that was present in the training data. Options: the module's running tabular target if it fits, an Adult-Income (UCI) classifier with `race` / `sex` / `income` as candidate sensitive attributes, a synthetic health / credit tabular target where you plant a known sensitive column, or a small in-house tabular classifier where a sensitive column is documented.
- **A vision target** small-population classifier where each class is a person or a tightly-defined subject. Options: a subset of a face-recognition dataset (see anti-patterns below on dataset choice — CelebA identity-labelled subsets or a small synthetic face set are the low-friction starting points), a per-user personalisation model, or a small-class medical-imaging classifier where the class label maps to a subject cohort.
- Access to the population marginal (base rate) for the sensitive attribute in the tabular problem — either from public statistics for the domain or from the training-data distribution itself.
- The mod-102 harm-model rows for the attribute-inference scenario and the model-inversion scenario, and their corresponding mod-103 register rows. You will fill their inherent columns from this exercise.

Dataset-choice anti-patterns to avoid: do not use a face-recognition dataset whose licensing or ethical status is disputed. Prefer a synthetic-face benchmark or a set the module's `AI-ML-etics` policy already sanctions; note the choice in the manifest and cite the license.

## Problem statement

Two attacks, each on a separate target, each with its own report row.

### Attack A — Attribute inference on the tabular target

Fredrikson-style attribute inference. Given the model, query access, and knowledge of the population marginal for the sensitive attribute:

1. Split the tabular data into a training set, a held-out evaluation set (with the sensitive attribute retained as ground truth for the evaluator but treated as unknown to the attacker), and — if the target model requires it — a shadow set for training an inversion classifier.
2. For each held-out record, the attacker knows the quasi-identifiers (the non-sensitive columns) and queries the model on candidate values of the sensitive attribute (or, if the model does not consume the sensitive column at inference time, queries the model on the quasi-identifiers alone and reads the output). The attacker's decision rule is either explicit maximum-likelihood over model outputs across candidate sensitive-attribute values, or an inversion classifier fit on the shadow set.
3. Compute **attribute-inference advantage** = (attacker accuracy on the sensitive attribute) − (base-rate accuracy = frequency of the majority class of the sensitive attribute in the eval population).
4. Also compute a **calibrated-confidence** number: over records where the attacker asserts confidence above threshold θ, what fraction of those assertions are correct? A model that leaks primarily in the confident tail (few but very-right predictions) is a different threat than one that leaks weakly across the board.

Report per sensitive attribute, and per demographic slice if the running system carries them.

### Attack B — Model inversion on the vision target

Fredrikson 2015-style model-inversion reconstruction. For each of at least 3-5 target classes (identities) in the vision target:

1. Fix a target class label.
2. Optimise an input (an image tensor) to maximise the model's confidence on the target class, starting from a random image or from a low-frequency prior. Use gradient ascent through the model with a plausibility regulariser (a total-variation term or a learned image prior helps; Fredrikson 2015's original attack uses de-noising as the prior). For a stronger baseline, use a generative prior (a small GAN or the Zhang 2020 "Secret Revealer" approach) — mention which if you go there.
3. Save the reconstruction and score it on:
   - **Reconstruction quality** — SSIM or LPIPS against a reference training image if you have one; if you do not, use a task-specific classifier's confidence on the reconstruction as a proxy quality metric.
   - **Identifiability** — the qualitative claim: is the reconstruction a recognisable face for the class? If a human labeller (you, or a colleague if the identity is public) can name the reconstruction, note that as a high-severity finding; if the reconstruction is a blur that only preserves coarse features, note it as low-severity. Do not skip this step — the SSIM number without the identifiability claim is not the finding.

Report reconstructions for at least three target classes with quality + identifiability side-by-side.

## Requirements

Produce all eight items:

1. **Threat-model header** — `risk/mod-107/<system-id>/attr-inv/threat-model.md`. Two blocks, one per attack: the attacker's information (quasi-identifiers, base-rate knowledge, query access) for A; the query-interface exposure (probabilities, top-k labels, logits) and label-space knowledge for B.
2. **Attribute-inference runner** — `risk/mod-107/<system-id>/attr-inv/run_attr_inference.py`. Reads the tabular model, the eval set, and the base-rate prior; runs the attacker's decision rule per sensitive attribute; emits per-record attacker predictions and the advantage / calibrated-confidence JSON.
3. **Model-inversion runner** — `risk/mod-107/<system-id>/attr-inv/run_inversion.py`. Reads the vision model, iterates over the chosen target classes, produces the reconstructions, scores them on quality, and emits reconstructions as image files plus a `reconstructions.json` with quality metrics + class labels.
4. **Attribute-inference report** — `risk/mod-107/<system-id>/attr-inv/attr-inference-report.md`. Chapter-03 shape: threat-model header, per-sensitive-attribute table (advantage over base rate, calibrated-confidence at chosen θ), per-slice rows if the running system carries demographic slices, cause-bucket tag (model-caused vs. data-caused — see chapter 03 §"How this row lands on the register") and the resulting mitigation route.
5. **Model-inversion report** — `risk/mod-107/<system-id>/attr-inv/inversion-report.md`. Chapter-03 shape: threat-model header, one table row per target class with the reconstruction thumbnail, the quality metric, and the identifiability claim (a short sentence naming whether a human labeller could identify the class from the reconstruction). Include a short paragraph naming whether the target is a "danger shape" (small-class, per-person classifier) — chapter 03's applicability point.
6. **Identifiability adjudication note** — for the inversion, the identifiability claim is *your* assessment; note who made it, under what viewing conditions, and whether an external human labeller sanity-checked. This is the one qualitative call in the exercise — capture how it was made so a reviewer can weigh it.
7. **Register updates** — the two mod-103 rows updated with the numbers from A and B in the inherent column; note that exercise 04 will overwrite the residual column with the DP-SGD variants (attribute inference is materially reduced by DP-SGD when the leak is model-caused).
8. **Reproducibility manifest** — `risk/mod-107/<system-id>/attr-inv/manifest.json`. Pins target-model hashes, eval-set fingerprints, base-rate source, inversion optimiser + regulariser choices, random seeds, and script commit hash.

Cross-cutting rules:

- **Base rate is not accuracy against a random guess.** It is the frequency of the *majority class* of the sensitive attribute in the eval population — the trivially-best guess a Bayes-optimal attacker with zero model information would achieve. Getting this wrong (using 0.5 as the base rate for a two-value attribute that is 90/10 split) invalidates the advantage number.
- **Report calibrated confidence, not just average advantage.** A model that gets 55% average accuracy on a 90/10 attribute is only *slightly* better than the trivial 90% base-rate guess (worse, in some framings); but a model that is 98% accurate on the 5% of records where it is confident is a different, sharper leak. Chapter 03's "calibration of the attacker's confidence" is the reference.
- **Identifiability is a claim, not a score.** For inversion, name the human who made the call and how. If you cannot make the call (no reference photo, or the class labels are anonymised), state that and skip the identifiability line rather than fabricating one.
- **Do not attempt inversion on real-person identities without appropriate authorisation and ethics review.** For the exercise, prefer synthetic-face benchmarks or consented public-figure identity sets; note the choice.
- **Cause-bucket the attribute-inference row.** Model-caused (overfitting; DP-SGD closes it) vs. data-caused (sensitive column present with no purpose; column-drop or lawful-basis review closes it). Chapter 03 §"How this row lands on the register" is the reference. The route drives what exercise 04 or a subsequent audit action needs to do.

## Starter guidance

- **Attribute-inference: start with the simplest possible attacker.** Given the model's output on the quasi-identifiers and each candidate value of the sensitive attribute, assert the value with the highest model confidence on the observed output distribution. This is the maximum-likelihood attacker; it takes 20 lines to write and it exposes the leakage floor. Only add a shadow-set-trained inversion classifier if the simple attacker is not enough to answer the question.
- **Yeom's overfitting connection is your sanity check.** Yeom et al. (CSF 2018) show attribute-inference advantage correlates with the model's train/test generalisation gap. If the model has a large generalisation gap, expect measurable attribute-inference advantage; if the gap is tiny, expect the advantage to be small. A wildly anomalous result (large advantage on a well-generalising model, or vice-versa) is usually a bug in your attacker, not a finding.
- **Model-inversion: start with the plainest gradient-ascent attack.** Random-image init, adam optimiser on the input, loss = cross-entropy target-class-confidence + small L2 image regulariser. Twenty to a hundred steps is usually enough for a first pass; you will see whether the target model is inversion-vulnerable or well-defended within minutes.
- **Small class sets are the danger shape — start there.** A 10-way MNIST inversion is uninformative because the classes are not identities; a 10-way face-recogniser inversion where each class is a person is exactly the shape chapter 03 flags. If the running system's vision target is a many-way classifier where classes are not identities, augment the exercise with a small-class per-person analogue as an intentional test case.
- **Confidence-vector exposure is the single biggest lever.** A model that returns only top-1 labels is much less inversion-vulnerable than one that returns the full probability vector. If your running system returns only top-1, note that — the "inversion residual" is meaningfully lower than the "inversion residual under full-probability exposure" would be.
- **Report metrics per class, not just averaged.** A reconstruction that scores SSIM 0.30 averaged over 5 classes hides that one class produced a near-photograph and four classes produced noise. Chapter 03's expectation is per-class visibility.
- **A reconstruction that fails to be identifiable is still a finding.** "SSIM 0.15, unrecognisable" is a defensible residual claim — it says the target model resists inversion at this attack strength. That is a stronger sentence than saying nothing about inversion.

## Acceptance criteria

The submission is accepted when:

- The threat-model header is complete for both attacks.
- Attribute-inference is reported per sensitive attribute with base-rate accuracy, attacker accuracy, advantage, and calibrated-confidence at a stated θ.
- Model-inversion is reported for at least three target classes with a reconstruction thumbnail, a quality metric, and an identifiability claim adjudicated by a named human.
- Cause-bucketing is done on the attribute-inference finding and the recommended mitigation route is named (DP-SGD for model-caused; column-drop or lawful-basis review for data-caused).
- If the vision target is a "small-population classifier" (per-person or per-user), the applicability note is present; if not, an intentional small-class analogue was added and its result is reported.
- The two mod-103 register rows are updated with the inherent numbers from this exercise.
- The reproducibility manifest is complete enough for a second engineer to re-run the attribute-inference numbers exactly and to re-run the inversion optimisation to the same convergence point (same seeds, same optimiser, same regulariser).
- Ethics-and-license notes are present for any real-person identity data used.

## Stretch goals

- **Sub-population attribute inference.** Split the tabular eval set by a second attribute (e.g., sex-within-race) and report attribute-inference advantage per subpopulation. Cross-cutting with mod-106.
- **Generative-prior inversion.** Repeat the inversion using a generative prior (a small GAN or Zhang 2020's approach). Report reconstruction quality and identifiability alongside the plain gradient-ascent baseline; note the delta. This is what raises the *attack-strength ceiling* — a defender who reports only the plain-gradient result will under-state inversion risk on modern targets.
- **Attribute-inference confidence sweep.** Sweep θ from 0.5 to 0.99 and report the (fraction-asserted, precision-of-assertions) curve. This is the confidence-tail shape the register row's control-defeated variant will care about.
- **Property inference as a variant.** Instead of inferring a sensitive attribute for one subject, infer a *dataset property* — e.g., "was the training set at least 20% female?" This is the property-inference variant of the family (Ateniese et al. 2015, Ganju et al. 2018) and is worth an exploratory paragraph even if you do not run it.
- **Cross-check against the MIA baseline from exercise 01.** For a small held-out set, ask: are the records with highest LiRA score (most-vulnerable-to-MIA) also the records with highest attribute-inference advantage? Overlap suggests the same overfitting is enabling both leaks; disjoint sets suggest independent mechanisms. This is a level-30 methodological question that a level-25 report can note as an open item.

## Deliverable format

A single markdown document `mod-107-ex02-attr-inv.md` that indexes the artefacts. The index document contains:

1. *System and dataset summary* — one paragraph naming the tabular and vision targets, the sensitive attributes, and the vision target's identity-labelling status (synthetic / consented / other).
2. *Threat-model header* — the two blocks from `threat-model.md`, verbatim.
3. *Attribute-inference results* — per-sensitive-attribute table (base-rate, attacker accuracy, advantage, calibrated-confidence@θ) with per-slice rows if applicable and the cause-bucket tag.
4. *Model-inversion results* — per-target-class table with thumbnail, quality metric, identifiability claim, and adjudicator name.
5. *Applicability note* — the chapter-01 applicability table interpreted for the running system's deployment shape.
6. *Register handoff* — the two mod-103 rows updated with inherent numbers and the exercise-04 placeholder.
7. *Ethics-and-license note* — a paragraph naming the identity data source, the license, and any human-labelling participants.
8. *Reproducibility manifest* — pointer to `manifest.json`.
9. *Open items* — anything for exercises 03–05 or for level-30 (the MIA / attribute-inference overlap cross-check is a natural one).

## Submission

Attach the index markdown, both reports, the reconstruction thumbnails, the manifest, and any human-labelling adjudication notes to the module submission channel. Redact real-person identifiers if the dataset requires it, but preserve the numeric quality metrics, the calibration numbers, and the applicability / cause-bucket tags — a reviewer cannot review an attribute-inference or inversion finding whose numbers are blanked out.
