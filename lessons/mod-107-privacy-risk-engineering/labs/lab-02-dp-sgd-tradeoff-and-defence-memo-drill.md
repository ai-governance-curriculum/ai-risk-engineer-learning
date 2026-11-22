# Lab 02 — DP-SGD Utility / Privacy Trade-off Table and Operating-Point Defence Memo

**Duration:** 45 minutes · **Format:** guided practice, markdown authoring on fabricated numbers against the same fixed running system as lab 01

## Objective

Internalise chapter 04's **utility / privacy trade-off curve** and the **three-grounds defence** an operating-point memo must mount — threat-model calibration, utility calibration, comparability to published references — by filling the shape end-to-end on fabricated numbers across a non-private baseline and two DP-SGD variants of the lab-01 running system `CareTriage`. The lab drills the authoring step so that when exercise 04 produces real Opacus runs at real ε values on the reader's own target model, the trade-off table, the per-slice utility block, the operating-point memo, and the DP-SGD **failure-mode checklist** are shapes you compose from memory rather than re-derive.

The two first-pass defects chapter 04 names — single-number reporting (ε or accuracy in isolation) and "`we picked the point` with no defence" — are the anti-patterns the shape forecloses.

## Prerequisites

- Chapter 04 read — specifically "What differential privacy claims", "What DP-SGD does mechanically", "Privacy accounting: RDP and the Gaussian mechanism", "The privacy / utility trade-off — the curve you file", "Defending the (ε, δ) choice at review", "User-level DP vs. record-level DP", and "Operational failure modes that silently invalidate the privacy claim".
- Chapter 02 skimmed — the MIA-residual-signal shape the lab-01 slice report produced is what this lab's trade-off table consumes alongside task-level utility.
- Lab 01 completed — the `CareTriage` running system and the lab-01 MIA numbers are what this lab overwrites the residual column of. If you have not done lab 01, do it first; the trade-off table cannot be anchored without the lab-01 baseline LiRA number.
- A terminal where you can create a working directory and author a markdown file. No external access, no compute — the numbers are fabricated below.

## The fixed example system

Same running system as lab 01 — **`CareTriage`**, the gradient-boosted tabular classifier in the care-coordination organisation; Article-9-cohort slice is present; the API currently returns top-1 label plus the two-class probability vector. For this lab the target model is being retrained three times — non-private baseline, DP variant A (moderate ε), DP variant B (strict ε) — and the residual MIA is re-measured on each with the lab-01 LiRA harness configuration (shadow-model population unchanged; only the target model changed).

### Fabricated runs and numbers — authoritative for this lab, do not revise

Framework: **Opacus** at a pinned release `vA.B.C` on the gradient-boosted architecture (wrapped through a PyTorch surrogate for per-example gradients; a tree-native DP variant is the production path and is noted as a lab-02 open item). Accountant: **RDP** via `opacus.accountants.RDPAccountant`; δ fixed at **10⁻⁵** across all runs (below 1/N for N = 48 300 training records). Sampling: **Poisson** via Opacus's `DPDataLoader`. Neighbouring definition: **record-level** — subjects in this dataset contribute one record per patient, so group-privacy adjustment is unnecessary.

| Run | (ε, δ) | σ (noise multiplier) | C (clip norm) | Steps T | Sample rate q | Overall accuracy | Article-9-cohort accuracy | Outlier-decile accuracy |
|---|---|---|---|---|---|---|---|---|
| Baseline (ε = ∞) | ∞ (non-DP) | 0 | n/a | n/a | n/a | 0.874 | 0.861 | 0.792 |
| Variant A (moderate) | (ε = 8, δ = 10⁻⁵) | 0.7 | 1.0 | fixed per manifest | fixed per manifest | 0.851 | 0.839 | 0.766 |
| Variant B (strict) | (ε = 2, δ = 10⁻⁵) | 1.5 | 1.0 | fixed per manifest | fixed per manifest | 0.813 | 0.795 | 0.712 |

Attack-suppression numbers (LiRA against the same lab-01 shadow-model population of N = 128, same attack statistic):

| Run | LiRA TPR @ 1 % FPR — overall | LiRA TPR @ 1 % FPR — Article-9 cohort | LiRA TPR @ 1 % FPR — outlier decile |
|---|---|---|---|
| Baseline (ε = ∞) | 0.082 | 0.16 | 0.30 |
| Variant A (ε = 8) | 0.028 | 0.049 | 0.090 |
| Variant B (ε = 2) | 0.008 | 0.013 | 0.019 |

Reference points for the comparability argument (chapter 04 "Defending the (ε, δ) choice at review" references):

- **US Census 2020 Disclosure Avoidance System** — per the published Disclosure Avoidance System documentation, the privacy-loss budget was on the order of ε ≈ 19.61 across the released products at the person-level accounting <!-- needs-research: pin the exact Census Bureau citation and the current revision at authoring time -->.
- **Published DP image classification at CIFAR-10 / CIFAR-100 scale** — commonly cites ε in single digits at δ ≈ 10⁻⁵ with a few percentage-points accuracy cost relative to non-private baselines <!-- needs-research: pin at authoring time against a specific survey or benchmark citation -->.
- **Published DP tabular classification on GBDT-like architectures** — ε in the 1–10 range at δ ≈ 10⁻⁵ is the common operating band at the model / data scale this lab's target lives in <!-- needs-research: pin at authoring time -->.

## Steps

### Step 1 — Create your working directory and the trade-off scaffold

```bash
mkdir -p mod107-lab02 && cd mod107-lab02
touch caretriage-dp-sgd-tradeoff.md
touch caretriage-dp-sgd-defence-memo.md
touch caretriage-dp-sgd-checklist.md
```

Open `caretriage-dp-sgd-tradeoff.md` and paste this scaffold. You will fill the six sections in order.

```markdown
# CareTriage — DP-SGD utility / privacy trade-off table (lab 02 slice)

System: CareTriage (gradient-boosted tabular classifier; see lab 01)
Chapter references: mod-107 ch 04 (DP-SGD, RDP accounting, trade-off curve, defence memo, failure modes); ch 02 (LiRA residual-signal shape)
Lab-01 cross-ref: `caretriage-mia-slice-report.md` — LiRA baseline is the residual input for the ε = ∞ row
Framework: Opacus vA.B.C; accountant: RDP (`opacus.accountants.RDPAccountant`); neighbouring: record-level

## 1. Mechanism and accounting

(framework, accountant, neighbouring definition, sampling assumption — one block)

## 2. Three-run table — baseline + two DP variants

(one row per run: ε, δ, σ, C; overall utility; per-slice utility (Article-9, outlier-decile); LiRA TPR @ 1 % FPR overall and per slice)

## 3. Chosen operating point

(which of baseline / Variant A / Variant B is the chosen operating point — named explicitly; one-line reason, with the full argument deferred to the defence memo)

## 4. Per-slice disparate-impact note

(one paragraph: which slice paid the largest utility cost going from ε = ∞ to the chosen operating point; cross-reference to mod-106)

## 5. Register-row handoff

(REG-CT-MIA-01 — residual cell overwrites the lab-01 placeholder with the chosen operating point's LiRA number; two control-defeated cells)

## 6. Reproducibility manifest

(Opacus version, accountant, δ, σ, C, Poisson sampling, module validation status, seeds)
```

### Step 2 — Fill Section 1 (Mechanism and accounting)

Chapter 04's load-bearing discipline: (ε, δ) must name the accountant, the sampling assumption, and the "neighbouring" definition. Paste the following block.

```markdown
| Field | Value |
|---|---|
| Framework | Opacus vA.B.C (PyTorch); gradient-boosted architecture wrapped via per-example-gradient surrogate |
| Accountant | `opacus.accountants.RDPAccountant` — Rényi Differential Privacy composition over the T Poisson-sampled steps |
| Sampling | Poisson, rate q per record per step, via `opacus.data_loader.DPDataLoader` (matches the RDP accountant's sampling assumption) |
| Noise mechanism | Gaussian; σ in units of the clipping norm C |
| Neighbouring | **Record-level** — one record = one patient; subjects contribute one record each, so no group-privacy adjustment required. If this changed (one patient contributing many records), the record-level ε would under-state the user-level privacy and a group-privacy adjustment or user-level DP-SGD (McMahan et al. 2018) would be required. |
| δ | 10⁻⁵ across all runs — below 1/N for N = 48 300 training records, meeting the δ ≪ 1/N convention |
```

Reading check: five fields, each with a specific value. If the accountant line reads "Opacus default" with no name, the number is not reproducible; chapter 04 is explicit on this.

### Step 3 — Fill Section 2 (Three-run table)

This is the artefact a review reads first. Paste the full three-run table carrying utility, per-slice utility, and LiRA on every row.

```markdown
| Run | ε | δ | σ | C | Overall accuracy | Article-9-cohort accuracy | Outlier-decile accuracy | LiRA TPR @ 1 % FPR — overall | LiRA TPR @ 1 % FPR — Article-9 | LiRA TPR @ 1 % FPR — outlier decile |
|---|---|---|---|---|---|---|---|---|---|---|
| Baseline (ε = ∞) | ∞ | n/a | 0 | n/a | 0.874 | 0.861 | 0.792 | 0.082 | 0.16 | 0.30 |
| Variant A | 8 | 10⁻⁵ | 0.7 | 1.0 | 0.851 | 0.839 | 0.766 | 0.028 | 0.049 | 0.090 |
| Variant B | 2 | 10⁻⁵ | 1.5 | 1.0 | 0.813 | 0.795 | 0.712 | 0.008 | 0.013 | 0.019 |
```

Reading check: three rows, each with (ε, δ, σ, C); each with overall and at least two slice utilities; each with LiRA overall and per-slice. A row that reports only ε or only accuracy is the chapter-04 "report task-level utility and MIA together, not each in isolation" anti-pattern. Per-slice utility is required because chapter 04 (via Bagdasaryan et al. 2019) is explicit that DP-SGD's utility cost is not uniform across slices.

### Step 4 — Fill Section 3 (Chosen operating point)

Name the operating point explicitly — one of baseline, Variant A, Variant B, or an interpolated third point — with a one-line reason. The full three-grounds argument goes in the defence memo (Step 7). "We picked the middle point" is not a reason per chapter 04.

```markdown
**Chosen operating point: Variant A — ε = 8, δ = 10⁻⁵.**

One-line reason: Variant A suppresses the Article-9-cohort LiRA TPR @ 1 % FPR from 0.16 to
0.049 (a 3.3× reduction) and the outlier-decile TPR from 0.30 to 0.090 (also 3.3×) at a
2.3-percentage-point overall accuracy cost; Variant B gets a further ~3× reduction at a
further 3.8-pp accuracy cost that the care-coordination outreach metric does not support. The
full argument — threat-model calibration, utility calibration, comparability to published
references — is in `caretriage-dp-sgd-defence-memo.md`.
```

Reading check: the operating point is named by variant; the one-line reason cites specific slice numbers and a specific utility cost. A sentence like "Variant A is the right balance" without numbers is the "`we picked the middle point`" anti-pattern by paraphrase.

### Step 5 — Fill Section 4 (Per-slice disparate-impact note)

Chapter 04 is explicit (via Bagdasaryan et al. 2019 — "Differential Privacy Has Disparate Impact on Model Accuracy") that DP-SGD's utility cost is not uniform across slices. Write a paragraph that names the slice that pays the largest utility cost going from ε = ∞ to Variant A, and cross-reference mod-106 for the fairness-side framing.

```markdown
Going from the ε = ∞ baseline to Variant A (ε = 8), the overall accuracy drops 2.3 pp
(0.874 → 0.851); the Article-9-cohort accuracy drops 2.2 pp (0.861 → 0.839); the
outlier-decile accuracy drops 2.6 pp (0.792 → 0.766). The absolute drops are close in this
table, but the *relative* drop is largest on the outlier-decile slice, consistent with the
Bagdasaryan et al. (2019) finding that DP-SGD's utility cost is concentrated on
under-represented or atypical subpopulations. The Article-9-cohort slice — which carries the
largest MIA reduction in absolute terms (0.16 → 0.049) — also pays a utility cost, and the
fairness-side implications of that trade-off are a cross-cutting mod-106 input. A review that
reads only the overall utility column misses the slice-level picture that chapter 04 names
as non-negotiable to report.
```

Reading check: specific per-slice deltas; the Bagdasaryan citation by name; a cross-reference to mod-106. A paragraph that only compares overall accuracies hides exactly the slice-level cost chapter 04 warns about.

### Step 6 — Fill Section 5 (Register-row handoff) and Section 6 (Reproducibility manifest)

```markdown
## 5. Register-row handoff

| REG-CT-MIA-01 cell | Value after lab 02 |
|---|---|
| Inherent (no defences) | LiRA TPR @ 1 % FPR = 0.082 overall; 0.16 Article-9; 0.30 outlier-decile — unchanged from lab 01 |
| Residual (DP-SGD at chosen operating point + top-1-label-only API) | LiRA TPR @ 1 % FPR = 0.028 overall; 0.049 Article-9; 0.090 outlier-decile — Variant A numbers from Section 2, under the assumption the API-shape change to top-1-label-only is also in place (its delta is tracked separately below) |
| Control-defeated: DP-SGD absent | Reverts to the inherent-column numbers (DP-SGD was the primary credited control against MIA for this scenario) |
| Control-defeated: DP noise multiplier reduced by X (variant A → something closer to the baseline) | Interpolated between Variant A (0.028 overall) and baseline (0.082 overall) — a specific "reduced σ" run is a lab-02 open item; chapter 04 names this as a control-defeated variant worth generating with a fourth reduced-noise run |
| Credited controls at this release | DP-SGD training (Opacus, RDP accountant, ε = 8, δ = 10⁻⁵, σ = 0.7, C = 1.0, Poisson sampling, record-level neighbouring) + top-1-label-only API (carried from lab 01's control-defeated analysis) |

## 6. Reproducibility manifest

| Field | Value |
|---|---|
| Framework + version | Opacus vA.B.C |
| Accountant | `opacus.accountants.RDPAccountant` vA.B.C |
| δ | 10⁻⁵ |
| σ (chosen variant) | 0.7 |
| C | 1.0 |
| Sample rate q | fixed per manifest |
| Steps T | fixed per manifest |
| Sampling implementation | `opacus.data_loader.DPDataLoader` (Poisson) |
| Module validation | `opacus.validators.ModuleValidator.fix` — applied; no unfixed modules remain |
| Warm-start | None — trained from scratch on in-scope data; (ε, δ) applies to the full training run |
| Seeds | fixed per release artefact |
| Script commit | <!-- needs-research: pinned at authoring time from the real artefact repo --> |
| Target-model artefact hashes (baseline / A / B) | <!-- needs-research: pinned at authoring time from the release's model registry --> |
```

Reading check: five register cells with specific LiRA numbers or specific pointers; a reproducibility manifest with the accountant named and the module-validation status recorded (the latter being one of the six chapter-04 failure modes if left ambiguous).

### Step 7 — Fill `caretriage-dp-sgd-defence-memo.md` — the three-grounds operating-point defence

Open `caretriage-dp-sgd-defence-memo.md`. Chapter 04 pins three grounds the (ε, δ) choice is defended on — threat-model calibration, utility calibration, comparability. One page; three labelled sections; each anchored to a specific number or citation.

```markdown
# CareTriage — DP-SGD operating-point defence memo (lab 02)

**Chosen operating point: Variant A — ε = 8, δ = 10⁻⁵**
(Opacus RDP accountant; σ = 0.7; C = 1.0; Poisson sampling; record-level neighbouring; module-validated; no warm-start.)

## 1. Threat-model calibration

The register row `REG-CT-MIA-01` is the MIA scenario against the Article-9 cohort — membership
in that cohort is itself special-category personal data under GDPR Art. 9. The threat model
(lab 01, Section 1) is a black-box attacker with probability-vector exposure and
population-shaped shadow data. Under that threat model, LiRA TPR @ 1 % FPR on the Article-9
cohort moves from **0.16 (baseline) to 0.049 (Variant A) to 0.013 (Variant B)**. The
risk-council's residual-risk appetite on Article-9-flagged register rows is a slice TPR @ 1 %
FPR below 0.10; Variant A clears that band on both the Article-9 and outlier-decile slices
(0.049 and 0.090 respectively). Variant B clears it with more headroom (0.013 and 0.019), but
the threat-model calibration argument for the tighter ε requires evidence that 0.049 is not
low enough — which the register-row appetite does not currently require.

## 2. Utility calibration

The task is care-coordination follow-up prioritisation — the care nurse reads the shortlisted
queue. The product metric that gates release is a combined "sensitivity on the Article-9
cohort × overall coordination throughput" metric; the release threshold on overall accuracy
is ≥ 0.84. Variant A at 0.851 overall clears the threshold; Variant B at 0.813 does not,
and the outreach-coverage model says a 4-pp drop from baseline would reduce successful
coordination contacts by ~6 % — a product cost the care-delivery team has not committed to
accept. Per-slice utility (Section 4) shows the drop is non-uniform; Variant A keeps slice
drops to within 2.6 pp, Variant B widens them to 5.0+ pp on the outlier decile. The
Bagdasaryan et al. (2019) disparate-impact finding is the reason per-slice utility is part
of the argument, not a footnote.

## 3. Comparability to published references

- **US Census 2020 Disclosure Avoidance System** — per-person privacy-loss budget on the
  order of ε ≈ 19.61 across released products at the released-publication scale
  <!-- needs-research: pin the exact Census Bureau DAS citation at authoring time -->.
  Variant A (ε = 8) is materially tighter than the US Census national-scale number, which is
  appropriate given the subject population is a specific Article-9 cohort rather than a
  national statistical release.
- **Published DP-GBDT / DP-tabular at similar model / data scale** — commonly operate in the
  ε ∈ [1, 10] band at δ ≈ 10⁻⁵
  <!-- needs-research: pin at authoring time against a specific survey citation -->.
  Variant A sits in the upper half of that band; Variant B in the lower half.
- **Internal precedent** — if any prior release in this risk-engineering repo pinned (ε, δ)
  on an Article-9-adjacent scenario, cite it here. (For this fabricated lab: none.)

## Why not Variant B

Variant B (ε = 2) is the stronger privacy bound and is the point a privacy-conservative
reviewer may push to pick. We decline here on two grounds: (i) the threat-model calibration
(Section 1) does not require it — Variant A already clears the residual-risk appetite on the
GDPR-sensitive slice; (ii) the utility calibration (Section 2) does not support it — the
overall accuracy drop below the product release threshold, and the Section-4 per-slice drop
widens on the outlier-decile and Article-9-cohort slices. If the residual-risk appetite were
tightened (chapter 06 Section-5 residual-risk claim is re-authored under a tighter threshold),
Variant B becomes the defensible point and the utility cost is renegotiated with the
care-delivery team as an explicit decision.

## Why not baseline

The baseline (ε = ∞) is the non-private run and carries LiRA TPR @ 1 % FPR = 0.16 on the
Article-9 cohort — above the register-row's residual-risk appetite. The baseline is the
*inherent* row on the register, not a defensible residual. It ships on the register for
contrast; it does not ship as the operating point.
```

Reading check: three labelled sections for the three grounds; two "why not" sections that name the alternative points and the specific numeric reason each is not chosen. A memo that says "Variant A is a good balance" without the three-grounds structure has failed the exercise.

### Step 8 — Fill `caretriage-dp-sgd-checklist.md` — the six-failure-modes checklist

Chapter 04 §"Operational failure modes that silently invalidate the privacy claim" names six failures. Open `caretriage-dp-sgd-checklist.md` and fill the pass / fail line for each.

```markdown
# CareTriage DP-SGD — six-failure-modes checklist (lab 02)

| # | Failure mode | Status | How this run addressed it |
|---|---|---|---|
| 1 | **Non-Poisson batching** — the accountant assumes Poisson sampling; deterministic batching invalidates the (ε, δ) bound | **Pass** | `opacus.data_loader.DPDataLoader` used across all three runs; Poisson sampling rate q as recorded in the reproducibility manifest |
| 2 | **Layer replacement not applied** — BatchNorm or some LayerNorm paths bypass per-example clipping if `ModuleValidator.fix` is skipped | **Pass** | `opacus.validators.ModuleValidator.validate(model)` run before training; `ModuleValidator.fix(model)` applied; no unfixed modules remained. (Note: this run uses a GBDT-via-PyTorch surrogate; the production tree-native DP path is a separate exercise 04 track.) |
| 3 | **Gradient accumulation across steps** — accumulating gradients over K micro-batches means the effective batch is K× larger and the accountant must be told | **Pass** | `opacus.utils.batch_memory_manager.BatchMemoryManager` used; effective-batch accounting is handled by the manager and the accountant sees the correct batch shape |
| 4 | **Warm-starting from a non-DP checkpoint** — a model warm-started from a non-private pretrained checkpoint carries its memorised state regardless of the DP-SGD applied afterwards; the (ε, δ) applies only to the fine-tune step | **Pass** | None of the three runs warm-starts from a non-DP checkpoint; all three train from scratch on in-scope data. If a warm-start were introduced in a future run, the (ε, δ) claim would be scoped to the fine-tune step and that scoping would be recorded here |
| 5 | **Data-dependent hyperparameter selection** — selecting σ or learning rate by looking at the training data spends privacy budget the accountant has not tracked | **Pass** | σ and learning rate pre-selected from published community reference points (chapter 04 "Defending the (ε, δ) choice at review" comparability band) and from the lab-01 LiRA overall / slice numbers (which were measured against the non-private baseline *before* DP-SGD was configured); no MIA-driven σ tuning |
| 6 | **Non-DP evaluation on training data** — reporting metrics computed on the training set spends privacy budget inside the report itself | **Pass** | All utility and attack-suppression numbers in `caretriage-dp-sgd-tradeoff.md` are computed on the held-out eval set; the training-set perplexity and loss curves are used only for convergence checks, not reported as release numbers |

Overall: **6 / 6 pass**. If any row flipped to **fail**, the (ε, δ) claim in the defence memo
would be qualified or withdrawn; a silent failure here is the exact "training run labelled DP
but not actually DP" anti-pattern chapter 04 names.
```

Reading check: six rows, six pass / fail entries, each with a specific engineering note. A checklist that is six "pass" lines with no explanation is not a checklist, it is a signature — chapter 04 requires the "how this run addressed it" column.

### Step 9 — Final checkpoint

Re-read the three files. Confirm:

- **The three-run table carries ε, δ, σ, C, overall utility, per-slice utility, LiRA overall, and per-slice LiRA on every row.** A row that omits any of these is the chapter-04 "single-number" anti-pattern.
- **The operating point is named and a specific numeric reason is given.** Not "the middle point," not "the good balance."
- **The defence memo has three labelled sections for the three grounds.** Each anchored to a specific number or citation. The "why not Variant B" and "why not baseline" paragraphs are present.
- **The per-slice disparate-impact note cites Bagdasaryan 2019 by name.** The slice deltas are specific.
- **The six-failure-modes checklist has a status + how-addressed line for each.** No blank "pass" rows.
- **The reproducibility manifest names the accountant.** Not "Opacus default."
- **The register-row handoff overwrites the lab-01 residual placeholder with a specific LiRA number.**

## Expected output

Three populated markdown files in `mod107-lab02/`: `caretriage-dp-sgd-tradeoff.md` (six-section trade-off table), `caretriage-dp-sgd-defence-memo.md` (three-grounds operating-point defence with "why not" paragraphs for the rejected points), `caretriage-dp-sgd-checklist.md` (six-failure-modes checklist with pass / fail and how-addressed lines). Two pages total.

## Checkpoint questions (self-check)

1. A reviewer pushes to move the operating point from Variant A (ε = 8) to Variant B (ε = 2) "because smaller ε is always safer." Per chapter 04, what are the two specific things wrong with that argument, and what is the shippable rebuttal shape? (*First, chapter 04 is explicit that ε is defended on three grounds — threat-model calibration, utility calibration, comparability — not on "smaller is better." Smaller ε is formally stronger privacy but carries a utility cost that is non-uniform across slices, and chapter 04 cites Bagdasaryan 2019 for exactly the disparate-impact point. Second, the rebuttal must cite specific numbers from the trade-off table and the defence memo: Variant A already clears the register-row residual-risk appetite on the GDPR-sensitive slice (TPR @ 1 % FPR = 0.049 vs appetite ≤ 0.10); Variant B pays an additional 3.8-pp overall accuracy cost that the product metric does not support; and the published-reference band (ε ∈ [1, 10] at this scale) puts Variant A inside the community's operating envelope. The shippable rebuttal is the three-grounds structure in the defence memo, not a one-line preference.*)
2. A reviewer notes that the (ε, δ) reported in Section 1 does not explicitly cite the Opacus version or the RDPAccountant class and asks "is this a big deal?" Per chapter 04, what is the chapter's framing? (*Chapter 04 is explicit: "pin the accountant — which one, which version — and record the resulting (ε, δ) alongside the mechanism (σ, C, q, T)." The ε number is produced by a composition analysis, not read off σ; different accountants (RDP, GDP, PRV, moments) can produce materially different ε for the same mechanism. A report that cites ε without the accountant is not reproducible and is not comparable to another run; it is the kind of defect that an auditor's first question surfaces. The fix is a one-line manifest entry; the finding is that the manifest's accountant line is load-bearing, not cosmetic.*)
3. The lab's "Why not Variant B" paragraph says Variant B does not meet the product accuracy threshold. A product manager counter-argues that the team should accept the product cost "because privacy is more important than one accuracy threshold." Per chapter 04, what is the chapter's framing of this trade-off and what does the engineer owe the reviewer in response? (*Chapter 04 is explicit that DP-SGD's utility cost is real and non-uniform, and that the operating-point choice must be defended on **both** threat-model calibration **and** utility calibration — not on privacy alone. The engineer does not override the product cost unilaterally; the engineer presents the trade-off (Variant B's additional privacy headroom vs. the product cost on the outreach-coverage model, plus the slice-level disparate-impact widening) and the risk-council plus the product owner together re-weight the appetite if they choose to. The engineering move is to make the trade-off visible, not to resolve it. If the appetite is formally tightened in a subsequent register revision, the operating point moves to Variant B and the utility cost is accepted as a documented decision; absent that, the engineer defends Variant A on the three grounds and names the open item.*)
4. A level-30 reviewer asks for confidence intervals on the LiRA TPR @ 1 % FPR numbers in Section 2 and for a distribution-shift robustness check on the LiRA attack itself against the DP-trained variants. Per chapter 06, what is the level-25 position on this ask, and what is the shippable shape? (*Chapter 06 "Boundary 2 — what escalates to `model-evaluation-engineer` (level 30)" pins confidence intervals on ROC and TPR-at-fixed-FPR, statistical-significance framing of slice-level deltas, distribution-shift robustness of the eval itself, and cross-eval meta-analysis as level-30 methodological work. The level-25 position is: state the point estimates with a named configuration, record the ask in the open items section, and route the item to `model-evaluation-engineer` at level 30 for the methodological calibration pass. This is not a weakness of the lab-02 report — it is the clean L25 / L30 boundary chapter 06 names, and acknowledging it explicitly is what makes the report shippable rather than over-reaching.*)

## Why this lab matters

Exercise 04 asks you to train the running system's target model three times with Opacus (or TF Privacy), produce the utility / privacy trade-off curve, choose an operating point, and defend it at a mock review with chapter-04's three-grounds argument plus the six-failure-modes checklist. If the three-document shape — trade-off table, defence memo, checklist — is new at exercise-04 authoring time, the engineering judgement (σ tuning, module validation, operating-point choice, published-reference citation) happens in the context of inventing the output format. This lab drills the format on fabricated numbers so that exercise 04's budget goes into tuning, slicing, and defending — not inventing. The residual-column overwrite on `REG-CT-MIA-01` is the lab's output that lab 03 (the privacy-risk report) consumes as Section 4's credited-defence evidence and Section 5's residual-risk claim.
