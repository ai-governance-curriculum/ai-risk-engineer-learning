# Exercise 02 — Evasion Attack Panel with ART and TextAttack

**Estimated effort:** 3 hours

## Objective

Run the evasion-attack panel from chapter 02 against the running system's classifier and / or text-classification head, using **Adversarial Robustness Toolbox (ART)** for the vision / tabular side and **TextAttack** for the text side, with **CleverHans** as an optional compare-point where you reproduce a published PGD number. Produce the perturbation-budget curves, not just single-point success rates, and stage each attack against a *named threat model* (knowledge / capability / goal / norm / budget) that will read directly into the mod-103 residual-risk update in exercise 05.

## Prerequisites

- Chapter 02 read; chapter 01 available for the threat-model rubric; chapter 05 skimmed for the report shape you will feed in exercise 05.
- Exercise 01 completed. Every attack you run here must appear as a row on the cross-tag sheet with an `in-scope-this-module` scope flag. If the row is missing, add it there before running the attack.
- The running system reachable in one of two shapes:
  - **White-box path** — model weights loadable in PyTorch, TensorFlow, or scikit-learn / XGBoost. ART wraps the model as a `Classifier`; you can call `.predict()` and (for gradient attacks) get gradients.
  - **Black-box path** — a query-only endpoint. ART's black-box attacks (e.g., `BoundaryAttack`, `HopSkipJump`) and TextAttack's black-box goal functions target this shape.
  - Note which path you are on; a "white-box eval on a black-box-only production endpoint" is a misleading claim.
- A **clean evaluation set** that mirrors the production input distribution — not just the canonical benchmark test set. If production skew is large, that is the eval set; note the provenance.
- ART, TextAttack, and (optional) CleverHans installed at pinned versions. Note the versions in the report methodology.
- Access to the mod-103 register rows so the residual-risk cells can be updated with the numbers this exercise produces.

## Problem statement

Stand up the chapter-02 panel — FGSM (floor), PGD (reference first-order), C&W (defence-defeat check) on the classifier side; TextFooler and one BAE-style recipe on the text side — against the clean evaluation set. Each attack is run at the threat model chapter 02 names (norm, ε, steps, restarts for the L∞ attacks; norm-minimising for C&W; constraint set for the text attacks). For each attack:

- Compute **attack success rate** on the eval set with bootstrap 95% confidence intervals — not a point estimate.
- Sweep the perturbation budget on the L∞ attacks (multiple ε values along the curve, minimum three) and plot the **perturbation-budget curve**.
- For the text attacks, sweep the constraint-set severity (e.g., minimum USE cosine from 0.90 down to 0.75 at two or three settings) and report success under each; keep the strictest setting as the primary reported number.
- Compute **transferability** for at least one attack: craft against your model and evaluate on a public surrogate, and (if feasible) craft against a public surrogate and evaluate on your model. Report the transfer ratios.
- Where you claim a defence exists on the model (adversarial-training, input transform, feature squeezing) run the panel *again* under an **adaptive attack** — an attack constructed with knowledge of the defence — and report the defence-delta, following the Carlini et al. "On Evaluating Adversarial Robustness" guidance.

Every attack has to be reproducible from the artifacts: pinned tool versions, seeds, exact hyperparameter dictionaries, and a clean-set fingerprint. A single-run number with no reproducibility metadata is not a claim.

## Requirements

Produce all seven items below. Missing any is an audit finding on the eval.

1. **Panel spec** — `risk/mod-105/<system-id>/evasion/panel.yaml`. One block per attack: attack name, runner (ART / TextAttack / CleverHans), version (release tag or commit hash), threat model (knowledge / capability / goal / norm / budget / steps / restarts / constraint set), and the `scenario_id` from exercise 01 that this row realises. Every attack in the panel maps to a scenario row on the cross-tag sheet.
2. **Runner scripts** — `risk/mod-105/<system-id>/evasion/run_art.py` and `risk/mod-105/<system-id>/evasion/run_textattack.py` (or a single orchestrator if the two frameworks share a driver). Each script:
   - Loads the model behind the appropriate ART / TextAttack wrapper.
   - Loads the clean eval set.
   - Instantiates each attack per the panel spec, with an explicit seed.
   - Generates adversarial examples and computes per-example perturbation norm (or per-example transformation-count for text).
   - Emits a JSONL results file with one row per example: `original_id, original_label, predicted_original, predicted_adv, perturbation_norm (or transformation_count), success`.
3. **Perturbation-budget curves** — `risk/mod-105/<system-id>/evasion/curves/*.png` (or SVG). One curve per L∞ attack, plotting attack success rate vs. ε on the sweep, with clean accuracy marked as a horizontal reference. For the text attacks, a small table showing success rate at each constraint setting.
4. **Transferability sub-table** — inside the results markdown: source model, target model, attack, success on source, success on target, transfer ratio. At least one row required; more is better.
5. **Defence-delta sub-table** — if the model ships with any evasion defence (adversarial-training, input transform, feature squeezing, certified smoothing), rerun the panel under the *adaptive* variant of each attack and report: baseline success, defence-enabled success, delta, clean-accuracy cost, and the adaptive-attack construction used. If the model ships without an explicit defence, state that; do not fabricate a defence row.
6. **Confidence intervals** — every reported success rate has a 95% CI via bootstrap (or shadow-run variance where the attack is deterministic on the same seed but randomised across restarts). A single-run point is not a claim.
7. **Results markdown** — `risk/mod-105/<system-id>/evasion/results.md`. Sections: methodology (threat-model table, tool versions, eval-set provenance), attack success (per-attack table with CIs), curves (embed the images), transferability (sub-table), defence deltas (sub-table or explicit "no on-model defence" statement), reproducibility (seeds, commit hashes, dataset fingerprint), open items for exercise 05.

Cross-cutting rules:

- **Never report a robustness number without ε (or the text-domain equivalent constraint).** ε is part of the number, not a footnote. A table row that reads "PGD success 78%" with no ε column is not shippable.
- **Every claimed on-model defence gets an adaptive attack.** Non-adaptive evaluations of defences have a track record of being overturned; the module treats a non-adaptive defence-delta as a nil result.
- **Text-domain constraint set is part of the threat model.** A high success rate at zero semantic constraint is meaningless; the constraint setting (USE cosine, edit distance, grammatical validity) is a reported column.
- **Do not tune attacks to make defences look good.** Report the strongest adaptive attack you ran; if the panel is weak, that is the finding, not a reason to weaken the panel further.

## Starter guidance

- **Prove the ART / TextAttack wrapper on one example first.** Framework-classifier plumbing (dtype, normalisation, label encoding, batch shape) is where the first hour goes if you skip this step; do a one-example smoke run per attack before you loop over the eval set.
- **Pick ε values on the curve deliberately.** For L∞ image attacks, `{1/255, 2/255, 4/255, 8/255, 16/255}` is a defensible sweep. For non-image domains, choose a spread that spans the domain-appropriate norm scale; report the choice on the panel spec.
- **PGD hyperparameters matter and default to too-few restarts.** ART's default restart count is not the reference number in the literature; set `num_random_init=5` (or higher on models that show restart-sensitive rankings) and set the step size to roughly `2.5 × ε / steps` unless you have a reason to differ. Document the choice.
- **C&W is slow; budget for it.** C&W's iterative minimisation runs to hundreds or thousands of steps per example. Run C&W on a *representative subsample* if the full eval set is prohibitive, and note the subsample size on the methodology.
- **TextFooler and BAE need a similarity constraint.** Without one, the "attack" just replaces the sentence. TextAttack's `UniversalSentenceEncoder` constraint at cosine ≥ 0.84 is the canonical setting; if you deviate, argue why.
- **Read a random handful of adversarial text examples before shipping the numbers.** If the perturbed text no longer means what the original meant, the constraint was too loose and the "success" is spurious. The human-readability sanity check is part of the eval, not a stretch.
- **Transferability crafts against a *different* model, not a fresh initialisation of the same model.** A public backbone (ResNet-50 pretrained, BERT-base) is the usual surrogate. Cite the surrogate identity.
- **CleverHans is optional but useful for one number.** If the literature you're calibrating against reports "PGD success 45% on CIFAR-10 at ε=8/255," reproduce the number on CleverHans first before running the same attack on ART — the delta between the two runs tells you your ART configuration is (or isn't) equivalent.
- **Bootstrap for CIs on the eval set, not on the perturbation-generation seed.** The uncertainty you care about is over the *distribution of eval examples*, not over PGD's random init; bootstrap the per-example success indicators.

## Acceptance criteria

The submission is accepted when:

- The panel spec exists at the specified path, names every attack with its runner + version + threat model, and each attack maps to a `scenario_id` on the exercise-01 cross-tag sheet.
- The runner scripts execute end-to-end on the clean eval set and emit the JSONL results files.
- The perturbation-budget curves are plotted for every L∞ attack in the panel, with at least three ε points and clean accuracy marked.
- Text attacks report success at the constraint setting used, and a random sample of adversarial examples has been human-read for meaning preservation (note the sample size and the pass rate).
- Every reported attack success rate carries a 95% CI.
- Transferability sub-table has at least one row with a named surrogate.
- Defence deltas exist for every claimed on-model defence, computed against an adaptive variant of the attack; if no on-model defence exists, that is stated explicitly.
- The results markdown pins tool versions, seeds, and eval-set fingerprint so a second engineer can reproduce the numbers.
- Every panel row is traceable back to a `scenario_id`; the register-row-ids to update are called out in the open-items section for exercise 05 to consume.

## Stretch goals

- Add **Auto-PGD** (Croce & Hein, 2020) to the panel. Its step-size schedule makes it the current RobustBench-style reference; a panel that includes Auto-PGD ages better than one that stops at PGD.
- Run one **CleverHans compare-point**: reproduce a published PGD number (e.g., the Madry CIFAR-10 result) on CleverHans and then on ART. Report the delta and diagnose any divergence in configuration.
- Add a **patch attack** (ART's `AdversarialPatch`) if the system consumes camera / document images. Patch attacks have different physical-world implications from ε-bounded pixel attacks; the register row is different too.
- Wire the **PGD-probe on a canary set** as a monitor stub for mod-109 — a small script that runs a fixed PGD attack against a fixed canary set weekly and emits the success rate as a metric. The monitor itself lands at mod-109; this is the intake artifact.
- Contribute an **adaptive attack against a defence your team ships** back to `risk/mod-105/<system-id>/evasion/adaptive/*.py` — the construction is the module's most valuable output because it demonstrates that the defence was measured honestly.

## Deliverable format

A single markdown document `mod-105-ex02-evasion.md` that indexes the artifacts, plus the artifacts themselves. The index document contains:

1. *System and target surface* — one paragraph naming the classifier / text head under attack and whether the eval is white-box or black-box.
2. *Panel summary* — the attacks in the panel with their threat-model axes at a glance; full spec in the YAML.
3. *Attack success table* — per-attack success rate with 95% CI at the primary budget setting.
4. *Perturbation-budget curves* — embed the images with one-line captions.
5. *Text-attack constraint sweep* — the sub-table with human-read sample commentary.
6. *Transferability* — the sub-table with the surrogate identity.
7. *Defence deltas* — the sub-table, or the explicit statement that no on-model defence exists.
8. *Reproducibility* — tool versions, seeds, eval-set fingerprint, dataset provenance.
9. *Register handoffs* — the mod-103 rows the residual-risk update will touch in exercise 05.
10. *Open items* — anything for exercises 03–05 or upstream mod-102 / mod-103.

## Submission

Attach the index markdown, the panel spec, the runner scripts, the JSONL results, and the curves to the module submission channel. If the underlying model / dataset is under NDA, redact model identifiers but preserve the panel spec (attack + threat model), the success-rate tables with CIs, and the curve shapes — a reviewer cannot review an evasion eval whose numbers and threat models are blanked out.
