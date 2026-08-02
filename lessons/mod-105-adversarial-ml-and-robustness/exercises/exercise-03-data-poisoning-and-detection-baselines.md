# Exercise 03 — Data Poisoning Attacks and Detection-Defence Baselines

**Estimated effort:** 3 hours

## Objective

Engineer at least one **label-flipping** attack and one **backdoor (BadNets-style)** attack against a training pipeline for the running system, then baseline the **three defences** chapter 03 names — activation clustering (Chen et al.), spectral-signature detection (Tran, Li, Madry), and STRIP (Gao et al.) — measuring each defence against each attack across a poison-rate sweep. Report **detection rate and clean FPR together** (never one without the other), plus clean-accuracy delta and latency delta. Every number lands as an inherent / residual / control-defeated update on the mod-103 register in exercise 05.

## Prerequisites

- Chapter 03 read; chapter 01 available for the taxonomy and cross-tag; chapter 05 skimmed for the report shape.
- Exercise 01 completed. Both poisoning attacks you engineer here must appear as rows on the cross-tag sheet with `nist_category=poisoning` and an `in-scope-this-module` scope flag.
- A **trainable pipeline** for the running system — either the production training / fine-tuning pipeline, or a faithful replica that consumes the same data shape and produces the same model class. If neither is available, use a small stand-in dataset (CIFAR-10, MNIST, IMDB) that matches the *modality* of the production system and note the substitution on the deliverable.
- A **held-out clean test set** untouched by poisoning, used to measure clean accuracy and clean FPR.
- Compute: enough GPU / CPU to retrain the model at multiple poison rates. If retraining full-scale is infeasible, run on a smaller model / smaller subset and note the substitution.
- ART (or a research-code equivalent) installed at a pinned version. ART bundles reference implementations of label-flipping, backdoor triggers, activation clustering, spectral-signature detection, and STRIP-style defences — the exercise assumes ART unless you have a reason to differ.
- Access to the mod-103 register rows so the poisoning residual-risk cells can be updated.

## Problem statement

Split the exercise into two engineered attacks and three defence baselines, each measured across a poison-rate sweep.

**Attacks:**
1. **Label flipping** — pick a source class and a target class (or a random-flip variant). At each poison rate on the sweep (`{0.5%, 1%, 5%, 10%}` of the training set), flip the specified fraction of source-class labels to the target class, retrain, and measure attack success (accuracy drop on the source class, or targeted-class success rate).
2. **Backdoor / BadNets** — design a trigger appropriate to the modality (a small pixel patch for images, a rare-token trigger for text). At each poison rate on the sweep, inject correctly-labelled triggered examples plus mis-labelled triggered examples of other classes, retrain, and measure (a) **triggered-input success rate** — fraction of triggered inputs classified as the attacker's target — and (b) **clean-input accuracy** — retrained model's accuracy on non-triggered inputs (should stay near baseline; if it drops, the backdoor was too aggressive).

**Defences:**
1. **Activation clustering** on the penultimate layer, per class, `k=2`, flagging the smaller cluster.
2. **Spectral-signature detection** — project representations onto the top singular vector of the class covariance, flag top scorers.
3. **STRIP** — run at inference time on a triggered vs. clean input set, measure entropy of predictions under superimposition, flag low-entropy inputs.

For each `(attack, defence)` pair at each poison rate, report:

- Attack success rate with no defence.
- Attack success rate with defence.
- **Detection rate** — of the actually-poisoned examples (training data for activation clustering / spectral; triggered inputs for STRIP), what fraction the defence flags.
- **Clean FPR** — of clean examples, what fraction the defence flags.
- **Clean-accuracy delta** — model+defence vs. model-only on the held-out clean test set.
- **Latency / cost delta** — for defences that run at training time, cost per epoch or per training step; for STRIP, per-inference latency overhead.

The output is the defence-panel table from chapter 03, populated with real numbers and confidence intervals.

## Requirements

Produce all eight items below. Missing any is an audit finding on the eval.

1. **Attack spec** — `risk/mod-105/<system-id>/poisoning/attacks.yaml`. One block per attack: family (`label_flip` / `backdoor`), source class / target class (or random), trigger description (for backdoor: pixel-patch geometry or token pattern), poison-rate sweep values, seed, `scenario_id` from exercise 01. Every attack maps to a scenario row.
2. **Attack runner** — `risk/mod-105/<system-id>/poisoning/run_attacks.py`. For each attack at each poison rate:
   - Constructs the poisoned training set from the clean training set.
   - Retrains the model (log the training config commit hash and the retrained-model artifact fingerprint).
   - Evaluates attack success on a held-out attack-set (source-class inputs for label flipping; triggered inputs for backdoor).
   - Evaluates clean accuracy on the held-out clean test set.
   - Emits results to `risk/mod-105/<system-id>/poisoning/attack-results.jsonl`.
3. **Defence runner** — `risk/mod-105/<system-id>/poisoning/run_defences.py`. For each defence against each attack at each poison rate:
   - Runs the defence on the appropriate input (training data with poison ground-truth for activation clustering / spectral; inference-time inputs for STRIP).
   - Emits flagged / not-flagged per example.
   - Computes detection rate (against the ground-truth poison labels the attack runner logged), clean FPR, and clean-accuracy delta.
   - Emits results to `risk/mod-105/<system-id>/poisoning/defence-results.jsonl`.
4. **Defence-panel table** — `risk/mod-105/<system-id>/poisoning/panel.md`. The full chapter-03 table populated: columns `attack, family, poison_rate, attack_success_no_defence, defence, detection_rate, clean_fpr, attack_success_with_defence, clean_accuracy_delta, latency_delta, notes`. One row per `(attack, poison-rate, defence)` triple, i.e., the full cross-product.
5. **Detection-rate vs. poison-rate curves** — `risk/mod-105/<system-id>/poisoning/curves/*.png`. For each defence, plot detection rate and clean FPR as functions of poison rate. The curves are what tell you whether the defence works at *realistic* attacker budgets (0.5–5%), not just at large ones.
6. **Confidence intervals** — every reported rate has a 95% CI via bootstrap (over the eval / attack sets) or via repeated seeded runs. Small-set caveats are noted on the row.
7. **Ground-truth-poison ledger** — `risk/mod-105/<system-id>/poisoning/ground-truth.jsonl`. Which examples were poisoned in which run, keyed by attack + poison rate + seed. Detection rate is computed against this ledger; without it, the defence is scoring against nothing and the eval is not defensible.
8. **Results markdown** — `risk/mod-105/<system-id>/poisoning/results.md`. Sections: methodology (attack spec, defence spec, training-config commit hash, dataset), attack success (table), defence panel (table), curves (embed), failure-mode narrative (which `(attack, defence)` pairs failed as expected — e.g., activation clustering weak on clean-label; STRIP weak on adaptive triggers), latency / cost budget, reproducibility, register handoffs.

Cross-cutting rules:

- **A defence report without clean FPR is not shippable.** "We caught the backdoor" with no clean-FPR number is a half-baseline; the exercise fails acceptance without both numbers on every row.
- **Poison rate is swept, not fixed.** A defence that only works at 20% poison is not defending a realistic attacker budget. The curves are the primary output.
- **Retrain from scratch for each poison rate; do not extrapolate.** Interpolating attack success across poison rates from a single-point retrain misrepresents the risk.
- **Ground truth on the poisoned examples is the attack runner's responsibility.** The defence does not know which examples are poisoned; the eval does, because it generated them. The ground-truth ledger is what makes detection rate a real number.
- **No harmful trigger strings in the repo if the modality is text.** Even benign-looking triggers can be embarrassing if they leak; keep the trigger pattern in the attack spec (documented) and reference triggered payload sets by hash, not by content.

## Starter guidance

- **Start with label flipping.** It is the simplest attack; you can verify the training pipeline and eval loop end-to-end before designing a trigger. If label flipping at 5% doesn't measurably drop source-class accuracy, something is wrong with the eval, not with the attack.
- **Pick a trigger that is small but reliable.** For pixel-patch backdoors, a 3×3 or 4×4 patch in a corner is typical; larger patches make detection trivial. For token triggers, a rare bigram at a fixed position is standard.
- **Verify the backdoor works before running the defences.** At the highest poison rate on your sweep, triggered inputs should classify as the attacker's target well above chance, while clean accuracy stays near baseline. If either is off, the trigger design is wrong; fix it before touching defences.
- **Activation clustering needs a k choice.** `k=2` is the paper default and the module's default; do not change it without noting the change and rerunning the calibration.
- **Spectral-signature detection is sensitive to the class-cardinality choice.** Run per class, not globally; the projection is on the *class* covariance's top singular vector, not the dataset's.
- **STRIP is not free.** Each inference example requires N (typically 100) superimposed forward passes; the latency delta is often the dominant cost. Report it.
- **Do not test STRIP on the training set.** STRIP is a runtime detector; it is evaluated on inference-time inputs, some of which carry the trigger. Feeding STRIP the poisoned training set is a category error.
- **Detection rate and clean FPR are computed against different reference sets.** Detection rate is a positive rate over poisoned / triggered examples; clean FPR is a positive rate over never-poisoned / never-triggered examples. Do not merge the two reference sets.
- **If your production system relies on a *downloaded pretrained checkpoint*, the poisoning delivery path is also supply-chain.** You do not need to engineer a supply-chain compromise for this exercise, but note the delivery path on the results markdown and flag the handoff to sec-L35 (artifact signing) and mod-110 (supply-chain).

## Acceptance criteria

The submission is accepted when:

- The attack spec exists at the specified path and maps both attacks to `scenario_id`s on the exercise-01 cross-tag sheet.
- The attack and defence runners execute end-to-end and emit the JSONL result files.
- The ground-truth-poison ledger is produced by the attack runner and consumed by the defence runner.
- The defence-panel table is populated for every `(attack, poison-rate, defence)` triple on the sweep, with `detection_rate`, `clean_fpr`, `attack_success_with_defence`, `clean_accuracy_delta`, and `latency_delta` filled per row (no blank cells).
- Detection-rate and clean-FPR curves are plotted as a function of poison rate for each defence.
- Every rate carries a 95% CI (or a small-set caveat).
- The failure-mode narrative explicitly names the `(attack, defence)` pairs where the defence is expected to underperform (activation clustering on clean-label, STRIP on adaptive triggers, spectral-signature on multi-mode class variance) — the eval is honest, not just favourable.
- The results markdown pins training-config commit hash, tool versions, and dataset provenance for reproducibility.
- The register-row-ids to update are called out in the open-items section for exercise 05.

## Stretch goals

- Add **clean-label poisoning** (Poison Frogs, Bullseye) as a third attack. Activation clustering and spectral-signature are known-weak on clean-label; the value of the exercise is showing the weakness quantitatively.
- Baseline **Neural Cleanse** (Wang et al., 2019) — trigger reverse-engineering — as a fourth defence. Neural Cleanse is architecture-specific and expensive but is a common auditor expectation.
- Sweep the trigger design (patch size / position, token rarity) and report defence sensitivity to the trigger. A defence that works at one trigger size and fails at another is a fragile defence; the sweep exposes this.
- Wire a **clean-FPR-of-defence monitor** as a mod-109 intake — a scheduled job that runs the defence on a fresh clean set and alerts if the FPR drifts above the panel's residual-risk baseline. The monitor itself lands at mod-109; this is the intake artifact.
- If your team consumes a public pretrained checkpoint, run the three defences against the *downloaded* model against a known-trigger-hypothesis set (probe inputs constructed to test whether a plausible backdoor is present). No positive result is a null finding; note it. A positive result is an incident.

## Deliverable format

A single markdown document `mod-105-ex03-poisoning.md` that indexes the artifacts, plus the artifacts themselves. The index document contains:

1. *System and pipeline* — one paragraph naming the training / fine-tuning pipeline under attack, the dataset, and any stand-in used.
2. *Attack summary* — the two attacks with their poison-rate sweeps and threat models at a glance; full spec in the YAML.
3. *Attack success table* — per-attack, per-poison-rate success with CIs.
4. *Defence panel* — the full `(attack, poison-rate, defence)` table with all six required numbers per row.
5. *Curves* — detection-rate and clean-FPR curves as a function of poison rate per defence, embedded.
6. *Failure-mode narrative* — the pairs where the defence underperformed and why; the paragraph an auditor reads for the honesty check.
7. *Latency / cost budget* — the training-time cost of activation-clustering / spectral defences and the inference-time cost of STRIP.
8. *Reproducibility* — tool versions, seeds, dataset fingerprint, training-config commit hash, ground-truth ledger reference.
9. *Register handoffs* — the mod-103 rows the residual-risk update will touch in exercise 05.
10. *Boundary flags* — supply-chain delivery path notes for sec-L35 and mod-110 if applicable.
11. *Open items* — anything for exercises 04–05 or upstream mod-102 / mod-103 / mod-108.

## Submission

Attach the index markdown, attack + defence specs, runner scripts, JSONL results, ground-truth ledger reference (do **not** attach poisoned datasets by content — hash-only), and the curves to the module submission channel. If the training data is under NDA, redact dataset identifiers but preserve the poison-rate sweep, the defence-panel table numbers, and the failure-mode narrative — a reviewer cannot review a poisoning eval whose numbers and pair-wise deltas are blanked out.
