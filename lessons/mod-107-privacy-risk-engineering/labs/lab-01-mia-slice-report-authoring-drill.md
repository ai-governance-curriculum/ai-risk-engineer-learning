# Lab 01 — MIA Slice-Report Authoring Drill (Shokri + LiRA Side-by-Side)

**Duration:** 45 minutes · **Format:** guided practice, markdown authoring on fabricated numbers against a fixed running system

## Objective

Internalise chapter 02's **MIA slice-reporting shape** — threat-model header, Shokri and LiRA on the same table so the AUC-can-lie contrast is visible, **TPR at 1 % FPR and 0.1 % FPR alongside AUC** on every slice, slice worst-cases (small-class, outlier-decile) reported next to the overall number, and the register-row handoff that carries the LiRA residual plus a control-defeated cell — by filling the shape end-to-end on fabricated numbers against a fixed example system (`CareTriage`). The lab drills the output shape chapter 02 pins so that when exercise 01 produces real numbers from ML Privacy Meter against the reader's own target model, the authoring step is routine and the engineering judgement goes into the attack configuration, not the report format.

The two first-pass defects chapter 02 names — AUC-only reporting and overall-only reporting — are the anti-patterns the shape forecloses.

## Prerequisites

- Chapter 02 read — specifically "Shokri-style shadow-model MIA", "LiRA", "Why AUC alone is misleading — the ROC-shape argument", "The reporting table this module files", "ML Privacy Meter as the working harness", and "How the number lands on the mod-103 register".
- Chapter 01 skimmed enough to recognise the mod-102 / mod-103 scenario-row vocabulary and the threat-model axes.
- A terminal where you can create a working directory and author a markdown file.
- No external access or compute required; the attack numbers are fabricated below.

## The fixed example system

Use this system throughout the lab — do *not* substitute your own. Carrying a fixed scenario keeps the lab bounded to the slice-report authoring skill.

> **`CareTriage`** — a tabular gradient-boosted classifier that produces a per-patient **follow-up-recommended / follow-up-not-recommended** label used by a care-coordination nurse to prioritise outreach. Trained on 48 300 historical patient records at a regional care-coordination organisation; input features include demographics, chronic-condition flags, prior-encounter counts, and a labelled adherence-risk column. A subset of the training records (approximately 11 %) originates from a labelled cohort whose membership encodes a condition in Article 9 of the GDPR (special-category health data). The model is served behind an internal API that returns the top-1 label and the predicted probability for both classes.
>
> Decision produced: binary `follow-up=1` / `follow-up=0` plus the two-class probability vector.
> Downstream consumer: a human care nurse who prioritises outreach from the queue.
> Population: patients in the care-coordination organisation's active panel.

### Harm-model and register row stubs (fixed — do not add more)

| Row id | Scope line |
|---|---|
| `HM-CT-MIA-01` | An adversary who queries the `CareTriage` API can decide whether a specific patient record was in the training set. Membership in the Article-9 cohort is itself special-category personal data under GDPR Art. 9. |
| `REG-CT-MIA-01` | Inherent and residual MIA risk on `CareTriage` — LiRA TPR @ 1 % FPR as the residual-risk signal; slice worst-case (outlier-decile and Article-9-cohort slice) reported alongside; control-defeated variants for "DP-SGD absent" and "full probability vector exposed". |

### Threat model — fixed for this lab

Copy these as given.

| Axis | Value for this lab |
|---|---|
| Query interface | Black-box; the API returns top-1 label plus the two-class probability vector. No logits, no gradients. |
| Query rate | Rate-limited at 200 queries / hour / authenticated user. For the eval, treat as effectively unlimited across a shadow-shaped candidate set. |
| Data-distribution knowledge | Population-shaped shadow data is available from a sister organisation's de-identified panel. Distribution gap: small but non-zero. |
| Model-architecture knowledge | Architecture family known (gradient-boosted trees); exact hyperparameters not disclosed. Shadow models are trained with the same family at the same depth / tree-count. |

### Fabricated attack numbers — authoritative for this lab, do not revise

Shadow-model count: **N = 128**. ML Privacy Meter pinned at a `vX.Y.Z` release (treat the version pin as fixed metadata for this lab).

Attack statistic for LiRA: **scaled logit of the true class** (`logit(p_true) = log(p_true / (1 − p_true))`); two-Gaussian ("online") variant. Attack statistic for Shokri: an MLP attack classifier fit on `(output-vector, member-label)` pairs across shadow models.

Candidate set size: 20 000 records, 50 / 50 member / non-member.

Overall slice:

| Attack | AUC | TPR @ 1 % FPR | TPR @ 0.1 % FPR |
|---|---|---|---|
| Shokri-shadow (baseline) | 0.59 | 0.027 | 0.004 |
| LiRA (online, 128 shadows) | 0.60 | 0.082 | 0.021 |

Article-9-cohort slice (patients whose record originates from the Article-9 labelled cohort; n = 2 200 candidate records, member fraction matched to overall):

| Attack | AUC | TPR @ 1 % FPR | TPR @ 0.1 % FPR |
|---|---|---|---|
| Shokri-shadow | 0.63 | 0.041 | 0.007 |
| LiRA | 0.69 | 0.16 | 0.052 |

Outlier-decile slice (top 10 % of records by training-loss, n = 2 000):

| Attack | AUC | TPR @ 1 % FPR | TPR @ 0.1 % FPR |
|---|---|---|---|
| Shokri-shadow | 0.68 | 0.07 | 0.015 |
| LiRA | 0.78 | 0.30 | 0.11 |

Small-class slice (positive-label class, imbalanced; n = 1 850):

| Attack | AUC | TPR @ 1 % FPR | TPR @ 0.1 % FPR |
|---|---|---|---|
| Shokri-shadow | 0.60 | 0.030 | 0.005 |
| LiRA | 0.63 | 0.094 | 0.025 |

Control-defeated fabricated numbers (for the register row):

- **"DP-SGD absent" variant** — identical to the overall LiRA numbers above (the model in this baseline was *not* trained with DP-SGD, so the baseline *is* the DP-absent variant).
- **"Full probability vector exposed" vs. "top-1 label only" variant** — if the API returned top-1 labels only, LiRA TPR @ 1 % FPR overall falls to **0.019** (compared to 0.082 with the probability vector exposed). This is the chapter-02 "secondary defence — return top-1 label only" number.

## Steps

### Step 1 — Create your working directory and the report scaffold

```bash
mkdir -p mod107-lab01 && cd mod107-lab01
touch caretriage-mia-slice-report.md
```

Open `caretriage-mia-slice-report.md` and paste this scaffold. You will fill the eight sections in order.

```markdown
# CareTriage — MIA slice report (lab 01 slice; no-defences baseline)

System: CareTriage (gradient-boosted tabular classifier for care-coordination follow-up; see lab brief)
Chapter references: mod-107 ch 02 (Shokri + LiRA; reporting table; ML Privacy Meter; register handoff); ch 01 (threat-model axes; applicability table)
Harm-model row cited: HM-CT-MIA-01
Register row cited: REG-CT-MIA-01
Mitigation state: UNMITIGATED BASELINE (DP-SGD training is scheduled for lab 02 / exercise 04)

## 1. Threat-model header

(four axes verbatim from the lab brief; one line of justification per axis)

## 2. Eval harness and configuration

(ML Privacy Meter version pin; shadow-model count N; attack statistic; LiRA variant; candidate-set size and member fraction)

## 3. Per-slice reporting table — Shokri and LiRA side-by-side

(one block per slice: overall, Article-9 cohort, outlier-decile, small-class; AUC and TPR@1% / TPR@0.1% columns; both attacks on adjacent rows)

## 4. Interpretive paragraph — the AUC-can-lie contrast

(one paragraph: at what AUC does the LiRA-vs-Shokri gap on TPR@low-FPR open? which slice is the worst-case? what does the shape say about this target model?)

## 5. Residual-risk claim

(one paragraph: residual-risk claim bounded on attack strength, threat model, and sample size; what the eval does not support)

## 6. Register-row handoff

(REG-CT-MIA-01 cells — inherent, residual, control-defeated: DP-SGD absent, control-defeated: probability vector exposed; cite the specific LiRA number per cell)

## 7. Open items

(what escalates to level-30 methodological calibration; what is a known lab-02 / exercise-04 input)

## 8. Reproducibility manifest

(harness version; shadow-model count; attack statistic; LiRA variant; random seeds; script commit hash — pinned as metadata for this fabricated run)
```

### Step 2 — Fill Section 1 (Threat-model header)

The threat model is what bounds the residual claim. Copy the four axes verbatim from the lab brief and add one line of justification per axis. Do not merge the axes into a paragraph; a reviewer should be able to see each axis and its value at a glance.

```markdown
| Axis | Value | One-line justification |
|---|---|---|
| Query interface | Black-box; top-1 label plus two-class probability vector (no logits, no gradients) | What the production API exposes today; a downgrade to top-1-labels-only is the control-defeated variant in section 6 |
| Query rate | Rate-limited at 200 queries / hour / user; for this eval, effectively unlimited across the candidate set | The rate-limit does not materially bound an attacker who can mint accounts or run multi-day campaigns; chapter 02 "Practical build notes" rate-limit point |
| Data-distribution knowledge | Population-shaped shadow data from a sister organisation's de-identified panel; distribution gap small but non-zero | Chapter 02 "Shadow data distribution matters" — the gap is noted; attack strength is a lower bound under larger gaps |
| Model-architecture knowledge | Architecture family known; shadow models in the same family at same depth / tree-count | Chapter 02 "Match the shadow architecture to the target" — LiRA is at its strongest when shadow architecture matches |
```

Reading check: four axes, four values, four one-line justifications. If a reviewer cannot tell what query interface the attack had, the residual number is unmoored.

### Step 3 — Fill Section 2 (Eval harness and configuration)

The eval numbers are only reproducible if the harness configuration is pinned. Paste the following block verbatim.

```markdown
| Field | Value |
|---|---|
| Eval harness | ML Privacy Meter (NUS DSSG) at a pinned release `vX.Y.Z` — version pin is a release artefact metadata item |
| Shadow-model count N | 128 |
| Shadow-training sampling | Poisson sampling at inclusion probability 0.5 per record per shadow model |
| LiRA variant | Two-Gaussian ("online") — both IN and OUT Gaussians fit per candidate record |
| LiRA attack statistic | Scaled logit of the true class: `logit(p_true) = log(p_true / (1 − p_true))` after softmax |
| Shokri attack classifier | MLP fit on `(output-vector, member-label)` pairs pooled across all shadow models |
| Candidate-set size | 20 000 records; 50 / 50 member / non-member |
| Shadow-architecture match | Same family (gradient-boosted trees); same tree-count / max-depth hyperparameters |
```

Reading check: the block is complete enough that a second engineer could re-run the same attack configuration against a fresh target model and reproduce the numbers.

### Step 4 — Fill Section 3 (Per-slice reporting table — Shokri and LiRA side-by-side)

This is the load-bearing artefact. Build one block per slice (overall, Article-9 cohort, outlier-decile, small-class), each with Shokri and LiRA on adjacent rows. Carry **AUC, TPR @ 1 % FPR, and TPR @ 0.1 % FPR** on every row. Do not omit any slice; do not drop AUC; do not drop either attack. The whole point of the side-by-side shape is the AUC-can-lie contrast.

```markdown
**Overall**

| Attack | AUC | TPR @ 1 % FPR | TPR @ 0.1 % FPR |
|---|---|---|---|
| Shokri-shadow (baseline) | 0.59 | 0.027 | 0.004 |
| LiRA (online, N=128) | 0.60 | 0.082 | 0.021 |

**Article-9-cohort slice** (patients whose record is from the Article-9 labelled cohort; n = 2 200)

| Attack | AUC | TPR @ 1 % FPR | TPR @ 0.1 % FPR |
|---|---|---|---|
| Shokri-shadow | 0.63 | 0.041 | 0.007 |
| LiRA | 0.69 | 0.16 | 0.052 |

**Outlier-decile slice** (top 10 % by training-loss; n = 2 000)

| Attack | AUC | TPR @ 1 % FPR | TPR @ 0.1 % FPR |
|---|---|---|---|
| Shokri-shadow | 0.68 | 0.07 | 0.015 |
| LiRA | 0.78 | 0.30 | 0.11 |

**Small-class slice** (positive-label class; n = 1 850)

| Attack | AUC | TPR @ 1 % FPR | TPR @ 0.1 % FPR |
|---|---|---|---|
| Shokri-shadow | 0.60 | 0.030 | 0.005 |
| LiRA | 0.63 | 0.094 | 0.025 |
```

Reading check: four slice blocks, each with both attacks, each with three metric columns. If any slice lacks AUC — or lacks TPR @ 0.1 % FPR without a "insufficient power" caveat — the slice is incomplete. If only one attack appears, the AUC-can-lie contrast is invisible and the lab's output has failed its purpose.

### Step 5 — Fill Section 4 (Interpretive paragraph — the AUC-can-lie contrast)

Chapter 02's ROC-shape argument says: two attacks with similar AUC can differ by an order of magnitude on TPR at low FPR. Write a paragraph that (a) names where in this table the contrast opens, (b) names the slice worst-case, and (c) says what the shape implies for the register row.

```markdown
Overall AUC is essentially tied between Shokri (0.59) and LiRA (0.60), but TPR @ 1 % FPR
differs by roughly 3× (Shokri 0.027 vs LiRA 0.082) and TPR @ 0.1 % FPR by roughly 5×
(Shokri 0.004 vs LiRA 0.021) — the exact chapter-02 ROC-shape point. A reviewer reading only
AUC on the overall slice would conclude "modest MIA risk, similar across attacks"; the
TPR@low-FPR columns say the opposite. The gap widens on the outlier-decile slice — Shokri
AUC 0.68 vs LiRA AUC 0.78, with LiRA TPR @ 1 % FPR at 0.30 — and on the Article-9-cohort
slice, where LiRA TPR @ 1 % FPR is 0.16, more than 5× the Shokri number at a slice that is
also the GDPR-sensitive slice. On this target model, the shape is "flat on average, dangerous
tail" — chapter 02's characterisation — and the register row therefore carries both the
overall LiRA number (0.082) *and* the slice worst-case (outlier-decile at 0.30; Article-9
cohort at 0.16) as two distinct residual-risk cells.
```

Reading check: the paragraph names (i) where the AUC-vs-TPR@low-FPR gap opens, (ii) the worst-case slice by TPR @ 1 % FPR (outlier-decile at 0.30), (iii) the GDPR-flagged slice (Article-9-cohort at 0.16), and (iv) the "flat on average, dangerous tail" characterisation. A paragraph that reads "the AUCs are similar so there is no issue" has mis-read the table.

### Step 6 — Fill Section 5 (Residual-risk claim)

The claim must be bounded on **attack strength**, **threat model**, and **sample size**. Write the one-paragraph claim.

```markdown
Under the threat model in section 1 (black-box API; probability vector exposed; shadow data
distribution-close but non-identical; shadow architecture matched; effectively-unlimited
query rate across the candidate set) and the attack configuration in section 2 (LiRA online,
two-Gaussian, scaled-true-class-logit, N = 128 shadow models, Poisson sampling), the residual
MIA signal on `CareTriage` at this release is: **LiRA TPR @ 1 % FPR = 0.082 overall; 0.16 on
the Article-9 cohort slice; 0.30 on the outlier-decile slice**. The Shokri baseline is lower
on the TPR@low-FPR columns and is reported for comparison, not as the primary residual signal.
This is a **lower bound** on the privacy risk — a stronger attack, a larger shadow-model count,
a shadow dataset closer to the training distribution, or a query-interface downgrade that
exposes logits or losses would raise it. The eval does *not* support the claim "no MIA
possible on `CareTriage`"; the strongest defensible sentence is the parameter-conditioned
one above.
```

Reading check: the paragraph names the threat model, the attack configuration, the overall and slice numbers, and the lower-bound discipline. A paragraph that says "no MIA" or "MIA is low" without qualification has violated chapter 02's discipline (the "do not extrapolate a null result to production" / "no membership inference possible is never a defensible statement" rule).

### Step 7 — Fill Section 6 (Register-row handoff)

The register row `REG-CT-MIA-01` consumes these numbers. Fill the four cells: inherent, residual, and two control-defeated variants.

```markdown
| Field | Cell value |
|---|---|
| Inherent (no defences) | LiRA TPR @ 1 % FPR = 0.082 overall; 0.16 Article-9 cohort; 0.30 outlier-decile. This lab authored the inherent column; the residual column will be overwritten by lab 02 / exercise 04 when the DP-SGD variants land. |
| Residual (DP-SGD + top-1-label-only API) | Placeholder — overwritten by lab 02 trade-off table and exercise 04 DP-SGD variants against the same LiRA harness. |
| Control-defeated: DP-SGD absent | LiRA TPR @ 1 % FPR = 0.082 overall; 0.16 Article-9 cohort; 0.30 outlier-decile — identical to the inherent column because the baseline model *is* the DP-absent variant. |
| Control-defeated: full probability vector exposed (vs. top-1-label-only) | LiRA TPR @ 1 % FPR = 0.082 overall with the probability vector exposed, vs. **0.019** overall if the API returned top-1 labels only — the API-shape control is load-bearing against this attack and the delta is the credited-control evidence for the top-1-label-only decision. |
| Credited controls as of this release | None (unmitigated baseline). The top-1-label-only decision is under review; the DP-SGD training variant is scheduled for exercise 04. |
```

Reading check: four distinct cells with specific LiRA numbers, not vague "low" / "high" labels. The control-defeated column for "probability vector exposed" is specifically what quantifies the API-shape decision chapter 02 names as a "secondary defence."

### Step 8 — Fill Section 7 (Open items) and Section 8 (Reproducibility manifest)

Short sections — open items for level-30 methodological calibration and for downstream labs / exercises, and the reproducibility manifest with the fabricated-lab metadata pinned.

```markdown
## 7. Open items

- **Level-30 methodological calibration** — 95 % confidence intervals on the ROC and on
  TPR @ fixed FPR; statistical-significance framing of the Article-9-cohort / outlier-decile
  slice-level deltas vs. overall; distribution-shift robustness of the LiRA attack under a
  shadow-data distribution that is further from the training distribution. Chapter 06
  "Boundary 2 — what escalates to `model-evaluation-engineer`" is the escalation reference.
- **Lab 02 input** — residual column of `REG-CT-MIA-01` is pending the DP-SGD trade-off from
  lab 02 / exercise 04 (two ε budgets; re-measure LiRA with the same shadow-model population).
- **Exercise 01 input** — this lab's slice-report shape is the output shape exercise 01 produces
  from a real ML Privacy Meter run; the configuration block in section 2 is the one exercise 01
  populates with the real version pin and run seeds.

## 8. Reproducibility manifest

| Field | Value |
|---|---|
| Harness | ML Privacy Meter, release `vX.Y.Z` (pinned in release-artefact metadata) |
| Shadow-model count N | 128 |
| Shadow-training sampling | Poisson, p = 0.5 per record per shadow model |
| LiRA variant | Two-Gaussian ("online") |
| LiRA attack statistic | Scaled logit of the true class after softmax |
| Shokri attack classifier | MLP; standard default architecture at ML Privacy Meter release pin |
| Candidate-set size / member fraction | 20 000 records; 50 / 50 member / non-member |
| Random seeds | (shadow-training seed; attack-classifier seed; candidate-set split seed) — fixed per release artefact |
| Script commit | <!-- needs-research: pinned at authoring time from the real artefact repo --> |
| Target-model artefact hash | <!-- needs-research: pinned at authoring time from the release's model registry --> |
```

### Step 9 — Final checkpoint

Re-read the eight sections. Confirm:

- **The threat-model header names four axes with one-line justifications each.** Not three; not a paragraph.
- **Shokri and LiRA appear on the same table, on every slice.** Not on separate tables, not with LiRA only.
- **Every slice row carries AUC and TPR @ 1 % FPR and TPR @ 0.1 % FPR.** Not AUC-only; not TPR-only; not a composite.
- **Slice worst-cases (outlier-decile, Article-9 cohort) are reported alongside the overall number.** Not hidden behind a single overall line.
- **The residual-risk claim is parameter-conditioned.** Threat model + attack configuration + sample size named. No "no MIA" sentences.
- **The register-row handoff has four cells with specific LiRA numbers.** Not vague descriptors.
- **The reproducibility manifest lets a second engineer re-run.** Harness version, shadow count, attack statistic, seeds, commit — all pinned.

## Expected output

`mod107-lab01/caretriage-mia-slice-report.md` — one populated eight-section document; one page to a page-and-a-half. The per-slice table in Section 3 carries Shokri and LiRA on adjacent rows across four slices; the interpretive paragraph in Section 4 names the AUC-can-lie gap and the slice worst-case; the residual-risk claim in Section 5 is parameter-conditioned; the register-row handoff in Section 6 cites specific LiRA numbers for inherent, two control-defeated variants, and a residual placeholder.

## Checkpoint questions (self-check)

1. A junior reviewer reads only the overall row of the Section 3 table, sees AUC ≈ 0.60 for both Shokri and LiRA, and writes "the attacks are equivalent — pick whichever is cheaper to run." Per chapter 02, what is wrong with that reading? (*Chapter 02's ROC-shape argument is exactly the rebuttal — two attacks with similar AUC can differ by an order of magnitude on TPR at low FPR, and TPR at low FPR is the metric that tracks disclosure risk. In this lab's table, Shokri TPR @ 1 % FPR is 0.027 and LiRA is 0.082 on the overall slice — a ~3× gap — and the gap widens on the outlier-decile slice (0.07 vs 0.30, ~4×). The attacks are not equivalent; LiRA is the attack that makes the low-FPR tail visible, and the AUC columns mask the finding. The reviewer would be reading the exact anti-pattern chapter 02 names.*)
2. A reviewer suggests dropping the Article-9-cohort slice row from Section 3 "because the slice is small (n = 2 200) and the numbers will be noisy." Per chapter 02, what are the two specific things wrong with that proposal? (*First, chapter 02 is explicit that slice reporting is required — "a single overall number hides population-scale disparities that a governance reader must see" — and the Article-9-cohort slice is the GDPR-sensitive slice; omitting it is the exact "overall-only" anti-pattern. Second, sample-size objections are handled by flagging the slice (e.g., "insufficient power for TPR @ 0.1 % FPR estimation at this sample size"), not by deleting the row — chapter 02 "Practical build notes" "Slice power" names this discipline. Here n = 2 200 is adequate for TPR @ 1 % FPR estimation; a flag might be warranted for TPR @ 0.1 % FPR but a deletion is not defensible.*)
3. A product manager reads the Section 5 residual-risk claim and asks for a "short, non-technical summary" like "CareTriage is privacy-safe against MIA." Per chapter 02, what is wrong with that summary and what is the shortest defensible sentence? (*Chapter 02 is explicit: "no membership inference possible is never a defensible statement." The eval produces a **lower bound** on privacy risk, bounded on attack strength, threat model, and sample size. The shortest defensible sentence names the attack, the configuration, and the TPR@low-FPR numbers — for example: "under LiRA online N=128 at this configuration and threat model, TPR @ 1 % FPR on CareTriage is 0.082 overall and 0.30 on the outlier decile." A "privacy-safe" summary violates the chapter's "lower-bound" discipline and is the kind of sentence a reviewer or auditor will flag.*)
4. A lead asks why the "full probability vector exposed" control-defeated cell in Section 6 (LiRA 0.082 with the probability vector vs. 0.019 top-1-label-only) credits the API-shape decision rather than DP-SGD. Per chapter 02, what is the chapter's framing of this delta and what does it imply for the register row? (*Chapter 02 distinguishes the primary defence (DP-SGD) from secondary defences, and names "return top-1 label instead of full logits" as one of the secondary defences that "help but are not substitutes for DP-SGD." The control-defeated delta of 0.082 vs 0.019 quantifies the API-shape decision specifically — it is the credited-control evidence that the top-1-label-only choice materially reduces MIA under this attack. The register row carries this delta separately from the DP-SGD delta (scheduled for lab 02 / exercise 04); both are credited controls, and a reviewer can see the contribution of each because they are reported as distinct control-defeated variants. If the two were merged into a single "controls on / controls off" cell, the API-shape vs. DP-SGD contributions would be invisible, and a product team considering an API-shape change would have no evidence to point to.*)

## Why this lab matters

Exercise 01 asks you to run Shokri and LiRA with ML Privacy Meter against the running system's real target model and produce the slice-report. If the eight-section shape is new at exercise-01 authoring time, the budget blows on inventing the format instead of on running attacks, slicing by outlier decile, and defending the configuration. This lab drills the shape on fabricated-but-plausible numbers so that exercise 01's time goes into attack tuning, shadow-model count, and threat-model defence — not into the output format. The shape it drills is also the one lab 02 overwrites the residual column of, and the one chapter 06's privacy-risk report consumes as Section 3's "Attack evaluation summary" row.
