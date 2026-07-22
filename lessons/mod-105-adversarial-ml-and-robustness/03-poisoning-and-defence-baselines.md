# Data / Label Poisoning and Detection-Defence Baselines

Where evasion attacks the model at inference time, **poisoning attacks the model during training** — either the fine-tuning data your team curates, an upstream pretrained checkpoint your team consumes, or a labelling vendor's outputs. Poisoning is harder to detect than evasion because the attack lives in the training pipeline and shows up only at inference-time behaviour that was never the loss the trainer optimised. This chapter takes you through the poisoning-attack taxonomy, the two defence families the module baselines (activation-space clustering / Spectre and STRIP-style trigger-perturbation), and — critically — how a defence baseline is *measured*, because a defence with an unmeasured detection rate or false-positive-rate is not shippable.

## Motivation

Two failure modes an auditor spots in a first-pass poisoning-eval:

- **A one-attack-type eval.** The team tested label-flipping poisoning and shipped a "clean-training-pipeline" claim. Label-flipping is the easy case; targeted backdoor attacks (BadNets-style triggers) and clean-label poisoning (Shafahi et al., "Poison Frogs") both bypass label-consistency checks and have to be tested separately.
- **A defence with no false-positive-rate on clean data.** The team enabled activation clustering, reported it "caught the backdoor," and shipped. But activation clustering flags *some fraction* of clean examples too — if that FPR is high, the defence quarantines legitimate training data and degrades the model. A defence baseline without both a detection rate *and* a clean-FPR is a half-baseline.

Engineering the poisoning-eval means picking an attack panel that covers each attack family, running each against the training pipeline in a controlled way, measuring the defence's detection rate *and* clean-FPR on the same suite, and updating the register's control-defeated column with the numbers.

## The poisoning-attack taxonomy

The AI 100-2 taxonomy (chapter 01) puts all training-time corruption under one "poisoning" category. Inside that category, four attack families that this module treats separately because their detection surfaces are different:

| Attack family | What the attacker changes | Inference-time effect | Detection surface |
|---|---|---|---|
| **Label flipping** | Flips labels on some training examples (label ≠ ground truth) | Degrades accuracy generally, or specifically on a target class | Label-consistency checks, cross-validator agreement, spot-relabelling |
| **Targeted data poisoning (dirty-label)** | Adds mis-labelled examples of a *specific* input that should get a *specific* wrong output | Attacker-chosen input is mis-classified at inference | Same as label-flipping if labels are audited |
| **Backdoor / trigger (BadNets)** | Adds correctly-labelled examples of *original class* that contain a small trigger pattern; also mis-labels versions of other inputs that carry the trigger | Any input carrying the trigger flips to attacker-chosen class; clean inputs behave normally | Activation-clustering; STRIP; trigger-reverse-engineering (Neural Cleanse) |
| **Clean-label poisoning** | Adds correctly-labelled examples that have been optimised to shift the decision boundary near a target input (Poison Frogs, Bullseye) | Attacker's chosen test input is mis-classified without any trigger; clean inputs behave normally | Hardest to detect; activation-space methods degrade in FPR; requires spectral / gradient-based diagnostics |

Two properties that shape the defence design:

- **Backdoor and clean-label attacks preserve clean accuracy** — the model looks fine on the held-out test set. You cannot detect them from held-out metrics; you have to inspect the training data or the model internals.
- **The attacker's "budget" is the fraction of the training set they can poison.** Real attack budgets in the literature range from 0.5% to a few % of training data; a defence that only works at 20% poison rates is not defending against realistic attackers.

## The supply-chain flavour of poisoning

The four families above assume the poison is delivered *into the training data your team curates*. The other delivery path — and the one an auditor increasingly asks about — is the **model-supply-chain path**: a pretrained checkpoint (a BERT variant, a vision backbone, an embedding model) is downloaded from a public model hub with a poisoned trigger baked in. Fine-tuning on your clean data may not remove the backdoor.

The supply-chain flavour is covered under ATLAS's "ML Supply Chain Compromise" tactic (chapter 01). Defence-side, the surface splits between:

- **Provenance controls** — signed model artifacts, hash-pinned downloads, curated internal model registry. These are `security-learning` / `ai-infra-security` (level 35) territory in this track; chapter 06 draws the boundary.
- **Behavioural checks on the downloaded checkpoint** — activation-clustering probes on trigger-hypothesis inputs, output-consistency checks under known-benign transformations. These are the same defence baselines this chapter covers.

For the exercise-03 scope, either delivery path (curated-data or pretrained-checkpoint) is a valid target for the poisoning attack you engineer; the register update is the same.

## The three defence baselines this module runs

Each has published evidence, an implementation in ART (or a straightforward research-code port), and a well-defined measurement protocol. Baseline them in order.

### 1. Activation clustering (Chen et al., 2018)

**Idea:** Poisoned examples in a backdoor attack tend to cluster in the activation space of the model's penultimate layer separately from clean examples of the same class. Cluster the activations per class (typically k=2) and flag the smaller cluster as suspected poison.

**Runs on:** the training data itself, after (at least partial) training.

**Metrics:**
- **Detection rate** — of the actual poisoned examples, what fraction end up in the flagged cluster.
- **Clean FPR** — of the clean examples in the same class, what fraction end up in the flagged cluster (i.e., are wrongly quarantined).
- **Coverage** — does the method work on all classes uniformly, or does it collapse on some?

<!-- needs-research: cite the exact Chen et al. 2018 paper, "Detecting Backdoor Attacks on Deep Neural Networks by Activation Clustering," and its arXiv/AAAI identifier. -->

**Failure modes to bake into the eval:**
- Weak against clean-label attacks (activations of clean-label poisons look like clean examples by construction).
- Weak when poison rate is very low (clustering degrades; the smaller cluster is legitimate class variance).
- Requires a chosen k; k=2 is a common default and a common failure point on multi-mode classes.

### 2. Spectral / spectral-signature-based detection (Tran, Li, Madry, 2018)

**Idea:** Poisoned examples introduce a detectable spectral signature in the covariance of the penultimate-layer representations — specifically along the top singular vector. Score examples by projection onto that direction and flag the top scorers.

**Runs on:** representations of the training data.

**Metrics:** same three as activation clustering — detection rate, clean FPR, coverage.

<!-- needs-research: confirm the exact Tran, Li, Madry 2018 paper, "Spectral Signatures in Backdoor Attacks," and its NeurIPS/arXiv identifier. Also confirm whether "Spectre" in the module description refers to this line of work or a distinct method; if the latter, replace / add the correct citation. -->

**Failure modes to bake into the eval:**
- Also weak on clean-label attacks.
- Assumes the poisoned distribution is *concentrated* along a direction; multi-trigger attacks defeat it.
- Detection threshold is a hyperparameter; misconfiguration is a common ops issue.

### 3. STRIP — STRong Intentional Perturbation (Gao et al., 2019)

**Idea:** A backdoor-triggered input, when perturbed by superimposing other clean inputs, tends to keep predicting the attacker's target class (the trigger dominates). A clean input, when perturbed the same way, spreads its predictions. Score inputs by the entropy of predictions under superimposition and flag low-entropy inputs as backdoor candidates.

**Runs on:** inputs *at inference time* (rather than training data). This makes STRIP a runtime-oriented defence rather than a training-time filter.

**Metrics:** detection rate (of triggered inputs correctly flagged), clean FPR (of untriggered inputs wrongly flagged), latency overhead (STRIP is not free — each input requires N superimposed forward passes).

<!-- needs-research: confirm the exact Gao et al. 2019 paper, "STRIP: A Defence Against Trojan Attacks on Deep Neural Networks," and its ACSAC/arXiv identifier. -->

**Failure modes to bake into the eval:**
- Weak against triggers whose activation is not dominant enough to survive superimposition (adaptive triggers).
- Introduces inference-time latency, which affects the deployed-system's SLA.
- Not effective against clean-label poisons that do not use a trigger.

## Measuring a defence — the required numbers

A defence baseline that only reports "we caught the backdoor" is not shippable. The measurement matrix per defence per attack:

| Number | What it says | Where it comes from |
|---|---|---|
| **Attack success rate (baseline)** | With no defence, does the poison work? Confirms the attack is real. | Evaluate attack-target inputs against the poisoned model. |
| **Attack success rate (with defence)** | With defence enabled, does the poison still work? | Same evaluation with the defence in the loop. |
| **Detection rate** | Of the actually-poisoned examples (training data or triggered inputs, depending on the defence), what fraction did the defence flag? | Requires ground-truth labels on the poisoned subset — you have these because you generated the poison. |
| **Clean FPR** | Of the truly clean examples, what fraction did the defence flag as poison? | Same defence run against a held-out clean set with no poison in it. |
| **Clean-accuracy delta** | Does enabling the defence degrade accuracy on clean inputs? | Compare model+defence accuracy against model-only accuracy on a clean test set. |
| **Latency / cost delta** | How much does the defence cost per training step or per inference? | Instrument runtime. |

The last three are the ones a first-pass eval usually skips. They are also the ones that determine whether the defence *ships*. A backdoor detector with a 5% clean FPR that flags 5% of legitimate training data as poison is worse than no defence at all — it silently degrades your training set every release.

## The defence-panel discipline

For each poisoning attack in exercise 03, you produce a row in this table:

| Attack | Family | Poison rate | Attack success rate (no defence) | Defence | Detection rate | Clean FPR | Attack success (with defence) | Clean-accuracy delta | Latency delta | Notes |
|---|---|---|---|---|---|---|---|---|---|---|

Two rules:

- **Every defence is measured against every attack in the panel** — including attacks it is known to be weak against. That is how the failure-mode picture is built.
- **Poison rates swept, not fixed.** Rerun each attack at (say) 0.5%, 1%, 5%, 10% of training data; the defence's detection curve as a function of poison rate is what tells you whether it works at *realistic* attacker budgets.

## Where poisoning meets the register

Each row in the defence-panel table above maps to entries in the mod-103 register:

- The **attack-success-rate-with-no-defence** number is an *inherent* update — how bad is this scenario if the defence is absent.
- The **attack-success-rate-with-defence** number is a *residual* update — how bad is this scenario when the defence is running as designed.
- The **detection-rate-vs-FPR pair** determines whether the defence is *actually credited* in residual, or is a "credit-with-caveat" (flagged as high-FPR-cost, tolerated only under monitor).
- A defence that quarantines legitimate training data at high FPR needs its **clean-FPR monitor** wired at mod-109 — if the FPR drifts, the residual score is no longer valid.

Chapter 05 walks the full report shape.

## Where the boundary sits

- **Provenance and supply-chain integrity controls** (signed checkpoints, hash-pinned model registry, artifact attestation) are `security-learning` / `ai-infra-security` (level 35) territory. This module's poisoning-eval assumes those controls exist or are missing; it does not build them. Chapter 06 pins the handoff.
- **Data-provenance controls on the training pipeline** (labelling-vendor attestation, differential-privacy training, PII scrubbing) are mod-107 (privacy-risk engineering) and mod-110 (MRM / supply-chain) territory.
- **Backdoor-specific runtime detectors** (STRIP-style, Neural Cleanse) that live in the serving path are mod-108 (guardrail engineering) territory once selected; this module baselines them, mod-108 catalogues and deploys them.

## Cross-references

- **mod-102 (harm modelling)** — the poisoning scenarios in exercise 03 must appear as harm-model rows with `upstream data supplier` or `insider` personas (mod-102 ch 01).
- **mod-103 (risk quantification)** — inherent / residual / control-defeated updates per attack-defence pair.
- **mod-107 (privacy-risk engineering)** — differential-privacy training also reduces poisoning susceptibility; the DP training work in mod-107 has cross-benefit here.
- **mod-108 (guardrail engineering)** — runtime backdoor detectors selected from this module's baselines end up in the guardrail catalogue.
- **mod-109 (continuous monitoring)** — clean-FPR monitors on the defence, and drift monitors on the training-data distribution, live at mod-109.
- **mod-110 (MRM / supply-chain)** — the checkpoint-supply-chain flavour of poisoning is a shared row with mod-110's third-party model catalogue.
- **`security-learning` / `ai-infra-security` (level 35)** — artifact-signing, hash-pinned registry, and platform-level supply-chain controls sit at the peer track (chapter 06).

## Summary

- Poisoning is a training-time attack; the taxonomy has four families — label flipping, targeted data poisoning, backdoor / trigger, clean-label — and a supply-chain delivery flavour.
- Backdoor and clean-label attacks preserve clean accuracy, so held-out metrics cannot detect them; internal / activation-space methods are required.
- The three defence baselines the module runs are activation clustering (Chen et al.), spectral-signature detection (Tran, Li, Madry), and STRIP (Gao et al.).
- Every defence baseline has to report *detection rate, clean FPR, clean-accuracy delta, latency delta* — a "we caught the backdoor" report with no clean-FPR is not shippable.
- Poison rates must be swept — a defence that only works at 20% poison is not defending against a realistic attacker budget of a few percent.
- Every defence-panel row maps to an inherent / residual / control-defeated update in the mod-103 register.
