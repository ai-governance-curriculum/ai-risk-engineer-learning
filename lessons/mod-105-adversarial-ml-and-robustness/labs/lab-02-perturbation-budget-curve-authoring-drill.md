# Lab 02 — Perturbation-Budget Curve Authoring Drill

**Duration:** 45 minutes · **Format:** guided practice, markdown authoring on fabricated numbers

## Objective

Internalise the chapter-02 pattern for turning an evasion-attack run into **report-shaped output**: a methodology block with the three-axis threat model named per attack, an attack-success table with confidence intervals, a **perturbation-budget curve** as the primary artifact (not a single-point number), and a defence-delta row that reads the number under adaptive attack. The lab asks you to walk *fabricated but plausible* numbers for FGSM / PGD / C&W on a fixed classifier through the full chapter-02 output shape, so that when exercise 02 produces real numbers, the formatting step is routine and the thinking goes into the attack configuration.

Single-point-number reports and unspecified perturbation budgets are the two first-pass defects chapter 02 names. This lab drills the shape that eliminates both.

## Prerequisites

- Chapter 02 read (specifically "The three classifier-evasion attacks that anchor the panel", "Threat models and perturbation budgets", and "Where evasion attacks meet the risk register").
- Chapter 01 skimmed for the three-axis threat-model rubric.
- Chapter 05 skimmed for the report-shape sections this lab's output will feed.
- Lab 01 completed (so a `SI-EVAS-01` cross-tag row exists; this lab produces the eval content for that row).
- A terminal where you can create a working directory and author markdown files.
- No external access or compute required; the attack numbers are fabricated below.

## The fixed example system

Same running system as lab 01 — **SupportIntent**, the fine-tuned intent-classifier head on a customer-support LLM assistant. For this lab, assume you have *white-box* access to the classifier head (weights loadable in PyTorch, gradients available). The text-domain attacks are run through TextAttack; L∞ gradient attacks are run through the classifier's token-embedding space via ART with the standard embedding-gradient wrapper (noted in methodology, not the subject of this lab).

For the sake of the drill, treat the classifier numerically as if it were a small image classifier as well — the chapter's L∞ / L2 attack vocabulary lets you run the same shape against both modalities, and this lab is about the *output shape*, not the input modality. Where a column reads `ε=8/255`, it is the chapter-02 canonical image ε used because it is the most frequently cited number in the literature; interpret as "the chosen L∞ budget on the embedding space" if you want to stay in text-domain.

## The fabricated attack numbers

Copy this as given — do *not* invent numbers of your own in this lab. The lab tests the *output shape*, not the numeric content.

Panel against the un-defended baseline classifier:

| Attack | Runner | Norm | ε | Steps | Restarts | Clean accuracy | Attack success rate | Bootstrap 95% CI |
|---|---|---|---|---|---|---|---|---|
| FGSM | ART 1.17 | L∞ | 2/255 | 1 | 1 | 92.4% | 12.1% | ±1.6% |
| FGSM | ART 1.17 | L∞ | 4/255 | 1 | 1 | 92.4% | 24.7% | ±1.7% |
| FGSM | ART 1.17 | L∞ | 8/255 | 1 | 1 | 92.4% | 41.2% | ±1.8% |
| FGSM | ART 1.17 | L∞ | 16/255 | 1 | 1 | 92.4% | 58.9% | ±1.7% |
| PGD | ART 1.17 | L∞ | 2/255 | 40 | 5 | 92.4% | 38.4% | ±1.5% |
| PGD | ART 1.17 | L∞ | 4/255 | 40 | 5 | 92.4% | 61.0% | ±1.4% |
| PGD | ART 1.17 | L∞ | 8/255 | 40 | 5 | 92.4% | 78.6% | ±1.3% |
| PGD | ART 1.17 | L∞ | 16/255 | 40 | 5 | 92.4% | 93.1% | ±0.9% |
| C&W | ART 1.17 | L2 | (norm-min) | 1000 | 1 | 92.4% | 99.7% | ±0.3% |

Panel with **adversarial training** at ε=8/255 enabled on the classifier (treat as the named defence under adaptive PGD):

| Attack | Norm | ε | Attack success rate (adv-trained) | Bootstrap 95% CI | Clean-accuracy cost |
|---|---|---|---|---|---|
| PGD (adaptive) | L∞ | 2/255 | 7.9% | ±1.1% | −2.7pp |
| PGD (adaptive) | L∞ | 4/255 | 18.2% | ±1.2% | −2.7pp |
| PGD (adaptive) | L∞ | 8/255 | 34.2% | ±1.3% | −2.7pp |
| PGD (adaptive) | L∞ | 16/255 | 71.8% | ±1.1% | −2.7pp |
| C&W (adaptive, L2) | L2 | (norm-min) | 91.2% | ±0.8% | −2.7pp |

Transferability (a public-surrogate BERT-base intent model trained on a similar labelled corpus):

| Source model | Target model | Attack | Success on source | Success on target | Transfer ratio |
|---|---|---|---|---|---|
| SupportIntent (own) | public-surrogate | PGD ε=8/255 | 78.6% | 52.3% | 0.67 |
| public-surrogate | SupportIntent (own) | PGD ε=8/255 | 74.1% | 48.5% | 0.65 |

## Steps

### Step 1 — Create your working directory and the report scaffold

```bash
mkdir -p mod105-lab02 && cd mod105-lab02
touch evasion-report-supportintent.md
```

Open `evasion-report-supportintent.md` and paste this scaffold. You will fill the five sections below in order.

```markdown
# SupportIntent — evasion-panel report (lab 02 slice)

System: SupportIntent (intent-classifier head on a customer-support assistant; see lab 01 brief)
Cross-tag anchor: SI-EVAS-01 (from lab 01 sheet)
Chapter references: mod-105 ch 02 (panel discipline, budget curve, adaptive-attack rule); ch 01 (three-axis threat model); ch 05 (report section shape)
Runner versions: ART 1.17; TextAttack 0.3.x <!-- needs-research: pin exact minor versions at authoring time -->

## 1. Methodology

(threat-model-and-budget table — one row per attack)

## 2. Attack success rates (no defence)

(table with ε, success rate, CI)

## 3. Perturbation-budget curve

(curve description + table + reading)

## 4. Transferability

(source / target / transfer ratio table)

## 5. Defence delta — adversarial training, adaptive attack

(table + reading)

## Register hand-off

(one-paragraph residual-risk update feeding mod-103)
```

### Step 2 — Fill Section 1 (Methodology) with the three-axis threat model named per attack

Chapter 02's methodology section is a per-attack table carrying the three-axis rubric (knowledge / capability / goal) plus norm, budget, steps, restarts, runner version. Fill it with this content, one row per attack:

```markdown
| Attack | Knowledge | Capability | Goal | Norm | ε | Steps | Restarts | Runner | Notes |
|---|---|---|---|---|---|---|---|---|---|
| FGSM | White-box | Active querier | Untargeted | L∞ | {2, 4, 8, 16}/255 | 1 | 1 | ART 1.17 | Floor attack; regression sentinel per ch 02 |
| PGD | White-box | Active querier | Untargeted | L∞ | {2, 4, 8, 16}/255 | 40 | 5 | ART 1.17 | Reference first-order attack |
| C&W | White-box | Active querier | Untargeted | L2 | norm-minimising | 1000 | 1 | ART 1.17 | Defence-defeat check per ch 02 |
| Clean baseline | — | — | — | — | — | — | — | — | Eval set mirrors production distribution — `<!-- needs-research: name the eval-set snapshot and record-count -->` |
```

Reading check: every attack row has all three axes, the norm and ε, the step count, the restart count, and the runner. Nothing blank. This section is what a reproducibility reviewer reads first (chapter 05).

### Step 3 — Fill Section 2 (Attack success rates, no defence) with the full budget sweep

Chapter 02 is explicit that single-point success rates are not shippable. Fill the section with the full ε sweep from the fabricated table, each row carrying its bootstrap 95% CI:

```markdown
| Attack | ε | Clean accuracy | Attack success rate | 95% CI |
|---|---|---|---|---|
| FGSM | 2/255 | 92.4% | 12.1% | ±1.6% |
| FGSM | 4/255 | 92.4% | 24.7% | ±1.7% |
| FGSM | 8/255 | 92.4% | 41.2% | ±1.8% |
| FGSM | 16/255 | 92.4% | 58.9% | ±1.7% |
| PGD | 2/255 | 92.4% | 38.4% | ±1.5% |
| PGD | 4/255 | 92.4% | 61.0% | ±1.4% |
| PGD | 8/255 | 92.4% | 78.6% | ±1.3% |
| PGD | 16/255 | 92.4% | 93.1% | ±0.9% |
| C&W (L2) | norm-min | 92.4% | 99.7% | ±0.3% |
```

Reading check: every number has a CI, not a point estimate (chapter 05's rule). FGSM is cheaper and *weaker* than PGD at the same ε (compare 41.2% to 78.6% at ε=8/255), which is the shape the chapter predicts; if your FGSM number were above or near your PGD number, that would be a gradient-masking regression and the panel discipline catches it.

### Step 4 — Fill Section 3 (Perturbation-budget curve)

The perturbation-budget curve is chapter 02's *primary artifact*. The lab does not require a plotted image — a table plus a short English reading is fine for the drill — but the reading must name the three things a curve carries that a point does not.

Author the section with this structure:

```markdown
### 3.1 Table — attack success vs. ε, per attack

| Attack | ε=2/255 | ε=4/255 | ε=8/255 | ε=16/255 |
|---|---|---|---|---|
| FGSM | 12.1% | 24.7% | 41.2% | 58.9% |
| PGD  | 38.4% | 61.0% | 78.6% | 93.1% |

### 3.2 Reading

Three things the curve carries that a single-ε number does not:

- **The shape of the drop.** PGD's success climbs from 38% at ε=2/255 to 93% at ε=16/255; the classifier's robustness degrades monotonically but is not catastrophically broken at the smallest ε. A single-ε report at ε=8/255 would miss that the model is *already materially broken* at ε=2/255 under PGD (38% success rate).
- **The gap between attacks.** At every ε the FGSM–PGD gap is wide (≈ 26–34 points), which is consistent with FGSM being a weak floor attack and PGD being the reference first-order attack. A collapsing gap (FGSM success ≈ PGD success) would flag gradient masking; the panel discipline catches this at authoring time.
- **The point at which the model is effectively broken.** By ε=16/255 under PGD the model is at 93% attack success; above this ε the number is no longer a useful discriminator. Reporting only ε=16/255 would hide the operating-point information at smaller budgets.
```

Reading check: the section names *why* the curve beats a point — not just the numeric table.

### Step 5 — Fill Section 4 (Transferability)

Copy the transferability table from the fabricated numbers and add a one-sentence reading:

```markdown
| Source model | Target model | Attack | Success on source | Success on target | Transfer ratio |
|---|---|---|---|---|---|
| SupportIntent (own) | public-surrogate | PGD ε=8/255 | 78.6% | 52.3% | 0.67 |
| public-surrogate | SupportIntent (own) | PGD ε=8/255 | 74.1% | 48.5% | 0.65 |

Reading: adversarial examples crafted against our own classifier transfer to a public surrogate at ratio 0.67, and surrogate-crafted examples transfer onto ours at 0.65 — neither direction is "off" (ratios close to zero), so a black-box attacker who crafts on a public surrogate can expect roughly half the on-model success rate against ours. The black-box threat model is *not* trivially harder than the white-box one for this classifier; the register row should reflect that.
```

Reading check: the transferability section is a sub-table *plus* a one-sentence operational reading. A table with no reading is a slide.

### Step 6 — Fill Section 5 (Defence delta — adversarial training, adaptive attack)

Chapter 02's rule is explicit: defence claims are evaluated under **adaptive** attacks constructed with knowledge of the defence. The PGD rows in the defence table below are adaptive-PGD numbers (same ε, same steps, same restarts, but adversarial training in the loop during attack crafting).

```markdown
| Attack | ε | Baseline (no def) | Adv-trained model | Delta | Clean-accuracy cost | Notes |
|---|---|---|---|---|---|---|
| PGD (adaptive) | 2/255 | 38.4% | 7.9% | −30.5pp | −2.7pp | Strong defence inside training ε |
| PGD (adaptive) | 4/255 | 61.0% | 18.2% | −42.8pp | −2.7pp | Strong defence at mid-ε |
| PGD (adaptive) | 8/255 | 78.6% | 34.2% | −44.4pp | −2.7pp | Defence trained at this ε — delta is at its largest |
| PGD (adaptive) | 16/255 | 93.1% | 71.8% | −21.3pp | −2.7pp | Robustness drops off *outside* the training ε |
| C&W (adaptive, L2) | norm-min | 99.7% | 91.2% | −8.5pp | −2.7pp | Adv-training at L∞ ε=8/255 is weak against L2 norm-minimising |

Reading: adversarial training at ε=8/255 L∞ buys the largest delta at the training ε (−44.4pp), noticeably less at ε=16/255 (−21.3pp — the defence does not extrapolate past its training budget), and noticeably less against C&W L2 (−8.5pp — a different norm defeats the L∞ training). Reporting only the ε=8/255 delta would inflate the headline; the full-curve delta is what the register needs.
```

Reading check: the defence section carries (a) the baseline number, (b) the adaptive-attack number under defence, (c) the delta, (d) the clean-accuracy cost, and (e) a reading that names the budget-curve shape and the L∞-vs-L2 breakdown. All five are required per chapter 02 and chapter 05.

### Step 7 — Fill the Register hand-off paragraph

Chapter 05's residual-risk section is a short, specific hand-off to mod-103. Target shape:

> "SI-EVAS-01 residual update: inherent remains High; residual moves from Medium (estimated) to Medium (quantified) under adversarial training at ε=8/255, with PGD-attack success at 34.2% (±1.3%) at the training ε and 71.8% (±1.1%) at ε=16/255. Control-defeated variant for `adv-training absent` is populated at PGD 78.6% (ε=8/255) and 93.1% (ε=16/255). A PGD-probe monitor on a canary set is added at mod-109 at weekly cadence; a C&W-L2 regression check runs per release candidate because the L∞ training does not cover L2 (−8.5pp defence delta). Transferability from a public-surrogate is 0.65; the black-box arm of this row is not materially easier for the attacker than the white-box arm and does not need a separate residual."

Reading check: the paragraph names the inherent, the residual-before, the residual-after, the control-defeated populate, the monitor addition, and the transferability note. It reads *into* mod-103, not as a standalone summary.

### Step 8 — Final checkpoint

Re-read the five sections and the register hand-off. Confirm:

- **Methodology section fills the three-axis rubric per attack.** No "white-box" written alone; knowledge, capability, and goal all named.
- **Every attack-success number carries a CI.** Point estimates without CI are an audit-flag per chapter 05.
- **The budget-curve section has at least two attacks at ≥3 ε values, plus a reading.** A table without a reading is a slide.
- **The defence section uses adaptive attacks.** The chapter's rule — "defences evaluated against adaptive attacks" — is cited in Notes or in the reading.
- **The transferability section has both directions.** Own→surrogate *and* surrogate→own, so the black-box threat model is actually characterised.
- **The register hand-off paragraph names the before / after residual numbers and at least one monitor.** Hand-offs that read as "we did an eval" without specifying the register update are what chapter 05 calls not-shippable.

## Expected output

`mod105-lab02/evasion-report-supportintent.md` — one populated five-section report slice (methodology, attack success, budget curve, transferability, defence delta) plus a register-handoff paragraph; one to two pages. Every number carries a CI, every attack carries its three-axis threat model, every defence number is adaptive, and the handoff paragraph reads directly into the mod-103 register row `REG-SI-EVASION-01`.

## Checkpoint questions (self-check)

1. The lab's fabricated FGSM success rate at ε=8/255 is 41.2%; the fabricated PGD success at the same ε is 78.6%. A reviewer suggests dropping FGSM from the panel because "PGD dominates." Per chapter 02, what is the specific objection? (FGSM is a floor attack and a regression sentinel: it is cheap enough to run per-commit, and a model whose FGSM success rate rises materially between releases has likely regressed into gradient masking. Dropping FGSM removes the sentinel. The chapter is explicit: FGSM is in the panel as a floor, not as a competitor to PGD.)
2. The defence delta at ε=8/255 is −44.4pp; at ε=16/255 it is −21.3pp; against C&W L2 it is −8.5pp. A reviewer proposes reporting *only* the −44.4pp headline in the register. Per chapter 02 and chapter 05, why is that specifically wrong? (The single-ε delta inflates the headline because it is the attack configured at the training ε — the point at which the defence is strongest. The budget-curve delta — including both the larger-ε degradation and the different-norm degradation — is what the residual-risk column needs, because an attacker chooses the attack-point that defeats the defence, not the attack-point that flatters it. Chapter 05's residual-risk section is what the register reads from.)
3. The transferability ratios are 0.67 (own→surrogate) and 0.65 (surrogate→own). A reviewer argues the black-box row in the register can be scored *more lenient* than the white-box row because "black-box is harder for the attacker." Per the lab's numbers and chapter 02, what is wrong? (The transfer ratio ≈ 0.65 means a black-box attacker who crafts on a public surrogate can expect roughly two-thirds of the white-box success rate against our model. That is not "trivially harder" — it means the black-box arm of the row should carry a residual close to the white-box arm, not materially lower. Chapter 02 names transferability as the argument for whether the black-box setting has an off-by-default guarantee; here, it does not.)
4. The fabricated C&W row has success 99.7% at the baseline. A junior engineer argues C&W should be removed because "the number is near-100% and uninformative." Per chapter 02, what is the specific objection? (C&W is in the panel as the *defence-defeat check*. A baseline-100% is expected and useful: it is the number that forces the defence to be measured under a tight-perturbation / different-norm attack, which is where the L∞-adv-trained defence drops from −44.4pp to −8.5pp. Removing C&W removes the only evidence the L∞ defence does not cover the L2 norm, which is the specific reason chapter 02 includes it.)

## Why this lab matters

Exercise 02 asks you to run the evasion panel on the real running system in three hours, produce the budget curves, and write up the methodology + success + budget-curve + transferability + defence-delta slice that lands in exercise 05's report. If the report-shape step consumes most of the budget because you re-derive the section list, the CI discipline, the budget-curve-vs-point rule, and the adaptive-defence rule from scratch, the exercise blows its budget before the attacks are even configured. This lab drills the atomic skill — five report sections plus a register hand-off paragraph authored end-to-end on fabricated numbers — until it is routine, so exercise 02 is *running the attacks and interpreting the numbers* rather than *inventing the output shape*. Lab 03 drills the defence-measurement matrix for poisoning; together with labs 01 and 02 the three cover the atomic shapes exercises 01–05 depend on.
