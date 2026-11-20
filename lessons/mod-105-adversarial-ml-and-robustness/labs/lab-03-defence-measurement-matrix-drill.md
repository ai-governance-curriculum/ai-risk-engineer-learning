# Lab 03 — Poisoning Defence-Measurement-Matrix Drill

**Duration:** 45 minutes · **Format:** guided practice, markdown authoring on fabricated numbers

## Objective

Internalise the chapter-03 pattern for turning a poisoning-defence baseline into **shippable output**: every row of the measurement matrix carries detection rate *and* clean FPR *and* attack-success-under-defence *and* clean-accuracy delta *and* latency delta — not one of them without the others. The lab asks you to walk *fabricated but plausible* numbers for activation clustering, spectral-signature detection, and STRIP across a poison-rate sweep on the running system through the full chapter-03 shape, then to make the three disposition calls — **ships**, **ships-with-caveat**, **rejects** — that feed the mod-103 register and the mod-109 monitor spec.

Detection-rate-only reports and unmeasured clean-FPR are the two first-pass defects chapter 03 names. This lab drills the matrix that eliminates both.

## Prerequisites

- Chapter 03 read (specifically "The three defence baselines this module runs", "Measuring a defence — the required numbers", and "The defence-panel discipline").
- Chapter 05 skimmed for the residual-update shape the matrix feeds.
- Chapter 06 skimmed for the co-owned-control / monitor handoff shape.
- Lab 01 completed (so a `SI-POIS-01` cross-tag row exists; this lab produces the defence content for that row).
- A terminal where you can create a working directory and author markdown files.
- No external access or compute required; the numbers are fabricated below.

## The fixed example system

Same running system as labs 01 and 02 — **SupportIntent**, the fine-tuned intent-classifier head on a customer-support LLM assistant. For this lab, assume:

- The **attack** is a BadNets-style **backdoor** on the training corpus: a rare-token trigger (`"<@zzzz>"`) is injected into mis-labelled copies of `cancellation` tickets so that any inference-time ticket carrying the trigger flips to `cancellation` regardless of true content. Poison rates on the sweep: `{0.5%, 1%, 5%, 10%}` of the training set.
- The **defences** are the three the chapter names — activation clustering (Chen et al.), spectral-signature detection (Tran, Li, Madry), and STRIP (Gao et al.) — running on the penultimate-layer representations (clustering / spectral) or at inference time (STRIP).

Treat all numbers as fabricated for the drill; the lab tests the *measurement-matrix shape and the disposition call*, not the numeric realism.

## The fabricated numbers

Copy these as given.

Attack-only panel (no defence) — backdoor success rate at inference time on triggered inputs across the poison-rate sweep:

| Poison rate | Triggered-input attack success rate | Clean-input accuracy (post-retrain) | Notes |
|---|---|---|---|
| 0.5% | 61.0% | 92.0% | Backdoor partially installed; still clean-accuracy near baseline |
| 1%   | 83.4% | 91.8% | Backdoor installed; clean inputs unaffected |
| 5%   | 97.1% | 91.2% | Backdoor strongly installed |
| 10%  | 99.3% | 90.6% | Backdoor saturated; small clean-accuracy loss starts to show |

Baseline clean accuracy of the un-poisoned model: **92.4%**.

**Activation clustering** (k=2, penultimate layer, per class):

| Poison rate | Detection rate | Clean FPR | Attack success (with defence) | Clean-accuracy delta | Latency delta | Notes |
|---|---|---|---|---|---|---|
| 0.5% | 42% | 6.1% | 54% | −1.9pp | +4% training-step cost | Low poison rate degrades clustering |
| 1%   | 71% | 5.8% | 38% | −1.9pp | +4% training-step cost | Picks up most poison |
| 5%   | 92% | 5.5% | 11% | −1.9pp | +4% training-step cost | Near-complete detection |
| 10%  | 95% | 5.2% |  5% | −1.9pp | +4% training-step cost | Near-complete detection |

**Spectral-signature detection** (top singular vector projection, flag top-θ):

| Poison rate | Detection rate | Clean FPR | Attack success (with defence) | Clean-accuracy delta | Latency delta | Notes |
|---|---|---|---|---|---|---|
| 0.5% | 55% | 1.8% | 41% | −0.6pp | +2% training-step cost | Threshold hyperparameter at θ=1.5 |
| 1%   | 78% | 1.7% | 24% | −0.6pp | +2% training-step cost | Picks up majority of poison |
| 5%   | 94% | 1.6% |  7% | −0.6pp | +2% training-step cost | Strong detection |
| 10%  | 96% | 1.5% |  3% | −0.6pp | +2% training-step cost | Strong detection |

**STRIP** (inference-time; N=100 superimposition passes per input, flag low-entropy):

| Poison rate (installed in model) | Detection rate (triggered inputs) | Clean FPR (untriggered inputs) | Attack success (with defence active) | Clean-accuracy delta | Latency delta | Notes |
|---|---|---|---|---|---|---|
| 1%  | 88% | 2.3% | 10% | 0pp (model unchanged; runtime filter) | +N × per-inference-latency; measured as **+180ms p50** per request | Runtime filter; cost per request |
| 5%  | 94% | 2.2% |  6% | 0pp | +180ms p50 | Strong detection |
| 10% | 96% | 2.1% |  4% | 0pp | +180ms p50 | Strong detection |

(STRIP is a runtime filter and is measured against the *backdoored* model, not against the training data.)

## Steps

### Step 1 — Create your working directory and the matrix scaffold

```bash
mkdir -p mod105-lab03 && cd mod105-lab03
touch defence-matrix-supportintent.md
```

Open `defence-matrix-supportintent.md` and paste this scaffold. You will fill the four sections in order.

```markdown
# SupportIntent — poisoning defence-measurement matrix (lab 03 slice)

System: SupportIntent (intent-classifier head on a customer-support assistant; see lab 01 brief)
Cross-tag anchor: SI-POIS-01 (from lab 01 sheet)
Chapter references: mod-105 ch 03 (measurement rules, defence-panel discipline); ch 05 (register hand-off); ch 06 (co-owned-control / monitor handoff)
Attack: BadNets trigger `<@zzzz>` targeting `cancellation`; poison-rate sweep {0.5%, 1%, 5%, 10%}

## 1. Attack-only baseline

(no-defence attack-success-rate table across the sweep)

## 2. Defence panel — three defences × four poison rates

(the full matrix — all six numbers per row)

## 3. Disposition per defence — ships / ships-with-caveat / rejects

(one paragraph per defence)

## 4. Register hand-off and monitor specs

(residual-update slice + monitor specs feeding mod-109; co-owned-control row shape feeding ch 06 contract)
```

### Step 2 — Fill Section 1 (Attack-only baseline)

Copy the attack-only sweep table into Section 1 exactly as given. Add one reading sentence: the backdoor is materially installed by **1% poison** (83.4% triggered success) and saturated by **5%** (97.1%), while **clean-input accuracy barely moves** — chapter 03's named property that backdoor attacks cannot be caught by held-out metrics.

Reading check: the section says the attack is real at realistic attacker budgets (≤ 5% of training data per chapter 03) and the attacker does not need high poison rates to install a working backdoor.

### Step 3 — Fill Section 2 (Defence panel matrix)

Section 2 is the chapter-03 measurement matrix. Every row carries **all six numbers**: poison rate, detection rate, clean FPR, attack-success-with-defence, clean-accuracy delta, latency delta. Compose it as one wide table so a reviewer can compare defences across rates at a glance:

```markdown
| Attack | Family | Poison rate | Attack success (no defence) | Defence | Detection rate | Clean FPR | Attack success (with defence) | Clean-accuracy delta | Latency delta | Notes |
|---|---|---|---|---|---|---|---|---|---|---|
| BadNets `<@zzzz>` → cancellation | backdoor | 0.5% | 61.0% | Activation clustering (k=2) | 42% | 6.1% | 54% | −1.9pp | +4% training-step | Low poison rate degrades clustering |
| BadNets `<@zzzz>` → cancellation | backdoor | 1%   | 83.4% | Activation clustering (k=2) | 71% | 5.8% | 38% | −1.9pp | +4% training-step | |
| BadNets `<@zzzz>` → cancellation | backdoor | 5%   | 97.1% | Activation clustering (k=2) | 92% | 5.5% | 11% | −1.9pp | +4% training-step | |
| BadNets `<@zzzz>` → cancellation | backdoor | 10%  | 99.3% | Activation clustering (k=2) | 95% | 5.2% |  5% | −1.9pp | +4% training-step | |
| BadNets `<@zzzz>` → cancellation | backdoor | 0.5% | 61.0% | Spectral signature (θ=1.5) | 55% | 1.8% | 41% | −0.6pp | +2% training-step | Threshold is a hyperparameter |
| BadNets `<@zzzz>` → cancellation | backdoor | 1%   | 83.4% | Spectral signature (θ=1.5) | 78% | 1.7% | 24% | −0.6pp | +2% training-step | |
| BadNets `<@zzzz>` → cancellation | backdoor | 5%   | 97.1% | Spectral signature (θ=1.5) | 94% | 1.6% |  7% | −0.6pp | +2% training-step | |
| BadNets `<@zzzz>` → cancellation | backdoor | 10%  | 99.3% | Spectral signature (θ=1.5) | 96% | 1.5% |  3% | −0.6pp | +2% training-step | |
| BadNets `<@zzzz>` → cancellation | backdoor | 1% installed  | 83.4% | STRIP (N=100) | 88% | 2.3% | 10% | 0pp (runtime) | +180ms p50 | Runtime filter; cost per request |
| BadNets `<@zzzz>` → cancellation | backdoor | 5% installed  | 97.1% | STRIP (N=100) | 94% | 2.2% |  6% | 0pp (runtime) | +180ms p50 | |
| BadNets `<@zzzz>` → cancellation | backdoor | 10% installed | 99.3% | STRIP (N=100) | 96% | 2.1% |  4% | 0pp (runtime) | +180ms p50 | |
```

Reading check: every row has all six measurement numbers. No row has a detection rate without a clean FPR; no row reports a defence without an attack-success-with-defence number; no row omits the clean-accuracy delta (even when the answer is "0pp (runtime)") or the latency delta.

### Step 4 — Fill Section 3 (Disposition per defence)

For each defence, write one paragraph that names the disposition — **ships**, **ships-with-caveat**, or **rejects** — and the *why* in the chapter-03 terms (detection vs. clean FPR tradeoff, realistic poison-rate coverage, operational cost). Target shapes:

```markdown
### Activation clustering — ships-with-caveat

Activation clustering reaches strong detection (92–95%) at poison rates ≥ 5%, but at realistic attacker budgets near 1% the detection rate drops to 71% while the clean FPR stays at ~5.8%. A 5.8% clean FPR means the defence silently quarantines ~6% of legitimate training data each release — directly the "worse than no defence" shape chapter 03 names. The clean-accuracy delta (−1.9pp) is noticeable and compounds release over release. Ships only with (a) a mod-109 monitor on the clean-FPR against a held-out clean set, (b) a manual-review gate on any training-data row the defence quarantines before the data is dropped, and (c) a release-gate check that the clean-accuracy delta stays within −2pp.

### Spectral-signature detection — ships

Spectral-signature detection reaches 78% detection at 1% poison with a clean FPR of 1.7% — the detection rate is below activation clustering at low poison rates but the clean FPR is ~3× better, and the clean-accuracy delta is smaller (−0.6pp). The threshold is a hyperparameter (θ=1.5 here) and misconfiguration is an operational risk the chapter calls out; ship with (a) a mod-109 monitor on clean FPR drift at the current θ, (b) a signed release note any time θ changes, and (c) a per-release clean-accuracy check.

### STRIP — ships-with-caveat (inference-latency)

STRIP is the only inference-time defence in the panel; it operates on the deployed model without changing training. Detection is strong at realistic installed-poison levels (88% at 1%, 94% at 5%) with a reasonable clean FPR (~2.2%). The headline cost is latency: **+180ms p50 per request** from N=100 superimposition passes. That cost may or may not fit the SLA; ships-with-caveat means the SLA assessment is a product decision owned jointly with the ML platform team (chapter 06's co-owned-control row), not a defence-eval decision. Ship with (a) an SLA-fit sign-off from the owning team, (b) a mod-109 monitor on p95 latency, and (c) a monitor on clean FPR drift.
```

Reading check: every disposition names the *why* in the chapter's measurement vocabulary (detection-vs-FPR tradeoff, realistic poison-rate coverage, clean-accuracy, latency), not a hand-wave. "Ships-with-caveat" names the specific caveat and the owning team per chapter 06.

### Step 5 — Fill Section 4 (Register hand-off and monitor specs)

Chapter 05's residual-update shape and chapter 06's contract shape meet in this section. Write it as:

```markdown
### Residual-update slice (feeds mod-103, register row REG-SI-POISON-01)

- Inherent: High (unchanged). Pre-eval residual: Medium (estimated).
- Post-eval residual: Medium (quantified) under the three-defence stack with the dispositions in Section 3.
- Control-defeated populate: at 1% poison with **no defence**, backdoor success is 83.4% on triggered inputs — populate this number in the control-defeated column for the "no training-pipeline defence" variant.
- Control-defeated populate: at 1% poison with **activation clustering only**, backdoor success is 38% — populate this number for the "activation clustering only" variant.
- Control-defeated populate: at 5% poison with the **three-defence stack**, backdoor success is **≤ 7%** (from spectral; STRIP catches the residual at inference) — populate as the residual-under-defence number.

### Monitor specs (feed mod-109)

| Monitor id | What it watches | Threshold | Why |
|---|---|---|---|
| `MON-SI-POIS-AC-FPR-01` | Activation clustering's clean FPR on a held-out clean set | Alert if > 7% (currently 5.8% at 1% poison) | Clean-FPR drift invalidates the residual number |
| `MON-SI-POIS-SPEC-FPR-01` | Spectral-signature clean FPR at θ=1.5 | Alert if > 2.5% (currently 1.7%) | Threshold-misconfiguration operational risk |
| `MON-SI-POIS-STRIP-LAT-01` | STRIP's per-request p95 latency | Alert if > 250ms p95 (currently +180ms p50) | SLA-fit check; co-owned with ML platform team |
| `MON-SI-POIS-STRIP-FPR-01` | STRIP's clean FPR at N=100 | Alert if > 3% (currently 2.3%) | Runtime-filter clean FPR drift |

### Co-owned-control row (feeds the chapter-06 handoff contract)

| Control | Specifier (L25) | Implementer | Operator | Evidence artifact |
|---|---|---|---|---|
| Activation clustering + Spectral-signature at training time | This module — specifies defence pair + θ + k | ML training team | ML on-call | Training-config commit hash; defence-matrix report (this document) |
| STRIP runtime filter in serving path | This module — specifies detector + operating point + latency ceiling | ML platform team | ML platform on-call | Serving-path config; per-release detection-log sample |
```

Reading check: the section populates the register columns with concrete numbers (not estimates), lists monitors with explicit thresholds and the number currently seen, and fills the chapter-06 contract columns per control (specifier, implementer, operator, evidence). A row with a monitor id but no threshold is an un-wired monitor; rewrite.

### Step 6 — Final checkpoint

Re-read the four sections. Confirm:

- **Every matrix row has six numbers.** Detection rate, clean FPR, attack-success-with-defence, clean-accuracy delta, latency delta, plus poison rate and defence name as the row key. If any row shows a detection rate without a clean FPR or without an attack-success-with-defence, rewrite.
- **The poison-rate sweep covers realistic attacker budgets.** 0.5%, 1%, 5%, 10% all present; the chapter's "defence that only works at 20% poison is not defending a realistic attacker" rule is visibly honoured.
- **Every defence has a named disposition.** Ships / ships-with-caveat / rejects. "Ships-with-caveat" names the specific caveat and the owning team. No defence is left un-dispositioned.
- **The register-handoff section populates concrete numbers.** Not "small" or "some"; actual percentages at named poison rates. Chapter 05's "from estimated residual to quantified residual" transition is visible.
- **Each monitor has a threshold and a current-value baseline.** `MON-SI-POIS-AC-FPR-01` alerts at >7% with a current value of 5.8% — a specific margin. A monitor with no threshold is un-wired.
- **The co-owned-control section fills all four role columns per row.** Specifier, implementer, operator, evidence artifact. Missing columns are the contract defect chapter 06 names.

## Expected output

`mod105-lab03/defence-matrix-supportintent.md` — one populated four-section report slice (attack baseline, full defence-measurement matrix with six numbers per row, defence-disposition paragraphs, register-handoff + monitor + co-owned-control section); roughly one and a half pages. Every row in the matrix carries all six numbers, every defence has a named disposition with the chapter-03 reasoning, and the register-handoff section populates the mod-103 columns with concrete percentages at named poison rates.

## Checkpoint questions (self-check)

1. The fabricated activation-clustering row at 1% poison reports detection rate 71% with clean FPR 5.8%. A junior engineer argues this is a shippable defence because "71% detection is good." Per chapter 03, what is the specific objection? (A 5.8% clean FPR means the defence silently quarantines ~6% of legitimate training data each release — the chapter names this shape explicitly: *"a backdoor detector with a 5% clean FPR that flags 5% of legitimate training data as poison is worse than no defence at all."* Detection rate alone is a half-baseline. The disposition is ships-*with-caveat* — a manual-review gate on quarantined rows and a clean-FPR monitor — not an unconditional ship.)
2. The fabricated STRIP row shows +180ms p50 per request. A reviewer argues this cost should not count because "STRIP's accuracy is excellent." Per chapter 03 and chapter 06, why is the latency specifically part of the shipping decision? (Chapter 03's required-numbers list is explicit that latency / cost delta is one of the measured numbers; a defence that works but breaks the deployed-system SLA is not shippable. Chapter 06 makes the SLA call a co-owned decision with the ML platform team — this module *measures* the latency; the owning team decides whether 180ms p50 fits the SLA. Omitting the number would move the decision out of the contract.)
3. The lab reports spectral-signature at θ=1.5 with detection 78% / clean FPR 1.7% at 1% poison. A reviewer proposes dropping the θ from the row because "it's a hyperparameter detail." Per chapter 03, what is wrong? (The chapter names threshold misconfiguration as a "common ops issue" for spectral detection. The θ is part of the defence configuration and must appear in the row so that (a) the number is reproducible and (b) a θ change in production is a signed release note with a new measurement, not a silent drift. Monitor `MON-SI-POIS-SPEC-FPR-01` watches clean-FPR drift at the *current* θ; the monitor is meaningless without the θ on record.)
4. The lab tests a BadNets backdoor only. A reviewer argues this validates the clean-pipeline control. Per chapter 03, what is still missing? (The chapter names four poisoning families — label flipping, targeted dirty-label, backdoor / BadNets, clean-label. Backdoor plus clean-label are the two families that preserve clean accuracy; clean-label is harder to detect than BadNets because activation-space methods degrade on it by construction. A one-family (backdoor-only) eval does not validate the clean-pipeline control; exercise 03 and quiz 01 question 7 both call this out. The register row should flag the clean-label and targeted-dirty-label gaps.)

## Why this lab matters

Exercise 03 asks you to engineer at least one label-flipping and one backdoor attack, baseline three defences across a poison-rate sweep, and feed the numbers into the exercise-05 report. If the matrix-shape step consumes most of the budget because you re-derive the measurement rules (six numbers per row, no detection-rate-without-FPR, poison-rate sweep covers realistic budgets, disposition names the caveat) from scratch, the exercise blows its budget before the defences have been baselined against the second attack. This lab drills the atomic skill — the full defence matrix authored on fabricated numbers with every row carrying all six measurements and every defence getting a named disposition — until it is routine, so exercise 03 is *running the defences and interpreting the numbers* rather than *inventing the measurement rubric*. Combined with lab 01's cross-tag discipline and lab 02's evasion-report shape, the three labs cover the atomic shapes exercises 01–05 depend on.
