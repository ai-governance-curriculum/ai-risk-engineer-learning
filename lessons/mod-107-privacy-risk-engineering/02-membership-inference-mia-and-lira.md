# Membership Inference: Shadow-Model MIA and LiRA as a Residual-Risk Signal

Membership inference is the anchor evaluation in the privacy taxonomy. It is well-studied, it maps directly to a governance-visible claim ("we do not disclose training-set membership"), and — unlike attribute inference or extraction — it has a settled methodology and a settled reporting convention. This chapter engineers the two attacks the module treats as canonical: **Shokri-style shadow-model MIA** (S&P 2017) and **Carlini-style Likelihood Ratio Attack** (LiRA, S&P 2022). It then pins the reporting shape — AUC is not enough, and TPR at low FPR is — and points you at ML Privacy Meter as the working harness.

## Motivation

An engineer running MIA the first time reaches for the obvious metric: attack AUC over the "member vs. non-member" binary label. The number looks reassuring — 0.57, 0.61, 0.63. It gets pasted into the report.

The number is misleading. Carlini et al. (2022) showed that a mean AUC in that range typically corresponds to a small number of *very-high-confidence* member identifications concealed by a large number of near-random guesses. From a controller's perspective the number that matters is not "how many records can the attacker guess better than chance" — it is "how many records can the attacker identify with high enough confidence to *actually use in a disclosure*." That is a **TPR at low FPR** question — say, TPR at 0.1% or 1% FPR. LiRA is the attack designed to make that number defensible.

This chapter engineers the shift.

## Threat model — what the attacker gets

MIA scales with the attacker's access. The threat model has three axes.

| Axis | Access | Impact on attack strength |
|---|---|---|
| Query interface | Labels only / top-k probabilities / full logits or loss / gradients | Logits > top-k > labels; gradients are white-box territory |
| Query rate | Rate-limited / unlimited | LiRA benefits from being able to query many candidate records |
| Data distribution knowledge | None / population-shaped / partial overlap with training | Shadow-model attacks need population-shaped data |
| Model architecture knowledge | None / architecture known / weights known | Shokri-style needs to be able to train shadow models close enough to target; LiRA benefits from close shadows |

Every evaluation you file has to state its threat-model axes. A residual-risk claim under a black-box, labels-only, rate-limited threat model is not the same claim as under a white-box, unlimited-query threat model. Chapter 06's report template records both.

## Shokri-style shadow-model MIA (S&P 2017)

The Shokri attack was the field-defining membership-inference construction. It is a training-based attack: the attacker trains many *shadow models* on data they control and drawn from a distribution close to the target's training distribution, then trains an *attack model* — typically a small classifier — to distinguish shadow-model outputs on shadow-training points ("members") from shadow-model outputs on held-out points ("non-members"). The attack model, once trained, is applied to the target model's outputs on candidate records.

### Pipeline

1. **Shadow-data assembly.** Assemble a labelled dataset drawn from a distribution close to the target's training distribution. Population size is a knob — you typically want at least the target's training-set size in aggregate across shadow models.
2. **Shadow-model training.** Train N shadow models (N in the tens or low hundreds), each on a random subset of the shadow data. Retain the members / non-members split per shadow model.
3. **Attack-training-set assembly.** For each shadow model, query it on both its members and its held-out non-members; collect (output-vector, member-label) pairs.
4. **Attack-model training.** Train a small classifier (an MLP or logistic model) that maps output-vector → member / non-member.
5. **Target querying.** Query the target model on the candidate records; feed the outputs to the attack model.
6. **Report.** Historically: attack accuracy or AUC. As of LiRA and subsequent literature: TPR at low FPR (below).

### What it captures and where it breaks

The Shokri attack captures the population-average signal that overfit training points produce more confident outputs. It is limited by (a) requiring shadow-shaped data, (b) treating the "member vs. non-member" decision as one uniform classifier — which averages over cases where the attack is very strong (near the training set's boundary) and cases where it is near-random.

## LiRA — Likelihood Ratio Attack (Carlini et al., S&P 2022)

Carlini, Chien, Nasr, Song, Terzis, Tramèr — "Membership Inference Attacks From First Principles" (S&P 2022) — reframes the problem. Instead of training one attack classifier over averaged behaviour, LiRA computes, per candidate record, the likelihood ratio between "target model was trained *with* this record" and "target model was trained *without* this record" — where both distributions are estimated from *shadow models trained with and without the record*.

### Pipeline

1. **Shadow-model population.** Train many shadow models on random subsets of a shadow dataset; retain which shadow models included each specific record.
2. **Per-record IN / OUT distributions.** For each candidate record, extract the target model's *scaled logit* (or another chosen statistic) and compare it to the distributions of that statistic under the shadow models that included the record (IN) and those that did not (OUT). Carlini et al. show that a suitably-scaled logit is approximately Gaussian, so both distributions are Gaussian and the likelihood ratio has a closed-form log-Gaussian expression.
3. **Score.** The LiRA score for the candidate record is that log-likelihood ratio.
4. **Report.** ROC over all candidate records. **Key metric: TPR at low FPR** (0.1%, 1%), not AUC.

### Why LiRA raises the ceiling on measured privacy risk

LiRA is not a "better" attack in the "higher AUC" sense — it is often *lower AUC than Shokri-average on the same target*. It is a better attack for the *high-confidence tail*, which is the tail that matters for disclosure risk. Carlini et al.'s core empirical finding: on standard image and text benchmarks, LiRA identifies orders-of-magnitude more members at fixed low FPR than prior attacks. A defender who reports only AUC will systematically under-state disclosure risk.

## Why AUC alone is misleading — the ROC-shape argument

A worked comparison of two hypothetical attacks on the same target model, both with AUC ≈ 0.60:

| Attack | AUC | TPR @ 10% FPR | TPR @ 1% FPR | TPR @ 0.1% FPR |
|---|---|---|---|---|
| A (Shokri-average) | 0.60 | 0.20 | 0.02 | 0.002 |
| B (LiRA) | 0.60 | 0.20 | 0.10 | 0.03 |

Both attacks are the *same on average*. From the controller's perspective they are wildly different. Attack B can identify roughly 0.03 fraction of members at a false-positive rate of 0.001 — on a million records, that is *thirty thousand* members named with an expected *one thousand* false accusations. Attack A cannot do that. The AUC does not tell you which one you are dealing with.

LiRA's contribution is not just an attack but a **reporting convention**. Every MIA number this module files reports TPR at fixed FPR — typically both 0.1% and 1% — alongside the AUC, and never AUC alone.

## The reporting table this module files

For every target model, per relevant slice, produce a table of this shape:

| Slice | Threat model | Attack | AUC | TPR @ 1% FPR | TPR @ 0.1% FPR | Notes |
|---|---|---|---|---|---|---|
| Overall | Black-box, logits, unlimited queries | Shokri-shadow (baseline) | 0.58 | 0.03 | 0.005 | 128 shadow models, population-shaped data |
| Overall | Black-box, logits, unlimited queries | LiRA | 0.61 | 0.09 | 0.024 | 256 shadow models, scaled-logit statistic |
| Slice: label class 3 | Same | LiRA | 0.71 | 0.18 | 0.06 | Class is small; expected higher vulnerability |
| Slice: outlier decile by input norm | Same | LiRA | 0.79 | 0.32 | 0.12 | Outliers are memorised more |

Slices matter because MIA is not uniform across records — training-set outliers, small classes, and rare subpopulations are more vulnerable. A single overall number hides population-scale disparities that a governance reader must see.

## ML Privacy Meter as the working harness

The reference harness this module uses is **ML Privacy Meter**, maintained by the NUS Data Security and Systems Group. It provides:

- shadow-model training scaffolding,
- multiple MIA attack implementations including LiRA-style scaled-logit attacks,
- ROC reporting and TPR-at-fixed-FPR computation,
- integration points for PyTorch and TensorFlow targets.

Two operational notes for level-25 use:

1. **Pin the version.** ML Privacy Meter is a research tool; attack implementations evolve. The residual-risk report you file cites the exact release / commit of the harness alongside the attack results.
2. **Attack tuning is part of the eval.** MIA numbers are *lower bounds* on privacy risk — a stronger attack tomorrow will always be possible. The eval reports the *strongest attack you ran*, names its configuration, and refuses to over-claim ("no membership inference possible" is never a defensible statement; "TPR at 1% FPR is X under attack Y at configuration Z" is).

Deeper calibration — confidence intervals on the ROC, statistical-significance framing of the slice-level differences, tests for out-of-distribution robustness of the attack itself — is level-30 work (`model-evaluation-engineer`). Chapter 06 draws that line.

## How the number lands on the mod-103 register

For a scenario row `HR-CLS-MIA-01` on the register (customer-facing classifier trained partly on labelled records from a special-category population):

| Field | Inherent | Residual | Control-defeated: DP-SGD noise multiplier reduced by X | Control-defeated: query rate-limit disabled |
|---|---|---|---|---|
| MIA residual-risk signal | LiRA TPR@1% FPR = 0.22 (no DP training, no query rate-limit) | LiRA TPR@1% FPR = 0.03 (DP-SGD at ε = 3, δ = 1e-5; per-user query rate-limit) | LiRA TPR@1% FPR = 0.11 | LiRA TPR@1% FPR = 0.03 (rate-limit was not the load-bearing control against MIA) |
| Slice worst-case (outlier decile) | LiRA TPR@1% FPR = 0.44 | LiRA TPR@1% FPR = 0.08 | LiRA TPR@1% FPR = 0.24 | Same as residual (unchanged) |
| Credited controls | none | DP-SGD training; per-user query rate-limit; classifier returns top-1 label only (not full probability vector) on production API | (DP noise reduced) | (rate-limit off) |

Two observations that a good report makes explicit:

- The **credited controls are technical**, and their *effect on the number* is what counts, not their presence. A rate-limit that does not change the number does not deserve mitigation credit for this scenario (it may deserve it for other scenarios).
- The **slice worst-case** is reported alongside the overall number. A residual-risk claim that hides a 0.08 outlier-decile TPR behind a 0.03 overall TPR is a claim the report will be judged on.

The delta from inherent to residual is the DP-SGD-driven privacy budget's ROI. If it is small, the DP parameters are too permissive; if it is large, chapter 04 shows the utility cost to defend.

## Practical build notes

Six operational notes for a level-25 first build of an MIA eval:

1. **Shadow-model count is a knob.** LiRA's per-record Gaussian estimation benefits from many shadow models with each record IN and OUT — Carlini et al. use 128 to 256. Fewer shadow models produce noisy per-record scores.
2. **Shadow data distribution matters.** Attack strength drops sharply when shadow data is drawn from a distribution meaningfully different from the target's. State the distributional gap in the report.
3. **Population base rate matters.** If your "attack" only sees candidate records that are 50/50 IN and OUT, the FPR / TPR numbers apply directly. If the deployed population has a very different member fraction, translating TPR into "expected member identifications" requires the base rate.
4. **Report the training-set outliers separately.** They are always the highest-vulnerability slice.
5. **Prefer scaled logits or losses as the attack statistic** where the API allows — they carry more member-signal than top-k labels or logits directly.
6. **Do not extrapolate a null result to production.** A weak attack in the lab is not evidence of privacy; it is evidence that *this attack, at this configuration, on this dataset* was weak.

## What defends against MIA — pointer to chapter 04

The primary defence for MIA is **differentially private training** — DP-SGD (Abadi et al., 2016) with a stated (ε, δ) budget. Empirically, DP-SGD models under a *meaningful* privacy budget (ε in single digits) suppress high-confidence membership signal to near-random, though at a utility cost. Chapter 04 engineers the DP-SGD side of this — parameter choice, accounting, and the utility / privacy trade-off argument.

Secondary defences — query rate-limits, output regularisation (return top-1 label instead of full logits), input-side filters, distillation to a smaller model — help but are not substitutes for DP-SGD when the claim being defended is a bound on membership leakage rather than an operational rate-limit.

## Cross-references

- **mod-101** — the RMF MEASURE 2.7 sub-category consumes this report; the GenAI Profile's "data privacy" risk row consumes it.
- **mod-102** — the MIA scenario row is what the harm model handed you.
- **mod-103** — the residual-risk column on the register row is the number this chapter produced.
- **mod-105** — MIA is a *privacy-goal* member of the adversarial-ML taxonomy; the query-based attack machinery is shared.
- **mod-108** — query rate-limits and API output-shape choices are guardrail decisions this chapter references.
- **mod-109** — MIA drift monitoring in production (do the LiRA numbers move release-over-release?) is a monitoring signal the mod-109 register-integration expects.

## Summary

- MIA answers "was this record in the training set?" — the anchor privacy attack, canonically Shokri (S&P 2017) and Carlini LiRA (S&P 2022).
- AUC alone is misleading; report **TPR at low FPR** (0.1% and 1%) per LiRA convention.
- Report slice-level worst cases (small classes, outlier deciles) alongside overall — MIA is not uniform across records.
- ML Privacy Meter is the working harness; pin the version.
- The number becomes a residual-risk score on the mod-103 register row; the delta from inherent to residual is the credited-control ROI.
- MIA numbers are *lower bounds* on privacy risk — never "no MIA possible."
- The primary defence is DP-SGD (chapter 04); secondary defences shape the API but do not substitute for it.
