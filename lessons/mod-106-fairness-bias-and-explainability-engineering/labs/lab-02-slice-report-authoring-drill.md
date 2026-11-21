# Lab 02 — Fairlearn Slice-Report Authoring Drill with Intersectional Cells

**Duration:** 45 minutes · **Format:** guided practice, markdown authoring on fabricated per-cell counts

## Objective

Internalise chapter 02's **slice-report shape** by filling it end-to-end on fabricated — but numerically consistent — per-cell counts from a `MetricFrame`-shape evaluation of the running `ShortList` system (from lab 01). The lab exercises every section of the shape: metric-choice five-liner header, data provenance, per-group table *including the intersectional sex × race/ethnicity cut*, per-cell counts with Wilson 95 % CIs, `n < 30` low-power flags (**not deletions**), four-fifths ratio, headline verdict against the tolerance `ε`, mitigation-state block (unmitigated baseline here), and the reproducibility manifest. Chapter 02's four anti-patterns — non-intersectional-only, missing tolerance, hidden pre-mitigation numbers, sample-size blindness — are the defects the shape forecloses.

Exercise 02 asks you to produce this slice report on the real running model with Fairlearn and AIF360 numbers reconciled. This lab drills the atomic authoring skill so exercise 02 is *running the libraries and diagnosing their numerical disagreements*, not *inventing the report shape*.

## Prerequisites

- Chapter 02 read — specifically "Fairlearn — the assessment primitive you write against" (`MetricFrame`, intersectional slicing), "The slice-reporting shape", "Dashboards and the continuous-slice view", and "Anti-patterns".
- Chapter 01 skimmed — the five-liner header shape, the decision-table row, and the tolerance `ε` concept.
- Lab 01 completed — the `ShortList` metric-choice five-liner you authored there is the header block this lab's slice report consumes. If you have not done lab 01, do it first; the slice report cannot be authored without a pinned five-liner.
- A terminal where you can create a working directory and author a markdown file. No external access, no compute — the numbers are fabricated below.

## The fixed example system

Same running system as lab 01 — **ShortList**, the BERT-base shortlist classifier for a NYC software-engineering role, in scope for NYC Local Law 144. The evaluation this lab reports is the *unmitigated baseline* on the audit-window eval set; mitigation is lab's-worth-of-future-work (and exercise 04).

## The fabricated per-cell table

Copy these numbers as given — do *not* invent your own in this lab. The lab tests the *output shape*, not the numeric content. All counts are from the audit-window eval set; `selection_rate` is the fraction of applicants in the cell with `shortlist=1`.

Single-axis — sex:

| sex | n | shortlist=1 | selection_rate | Wilson 95 % CI |
|---|---|---|---|---|
| Male | 4 820 | 1 013 | 21.02 % | (19.87 %, 22.21 %) |
| Female | 1 870 | 266 | 14.22 % | (12.69 %, 15.91 %) |

Single-axis — race/ethnicity (EEO-1 categories):

| race/ethnicity | n | shortlist=1 | selection_rate | Wilson 95 % CI |
|---|---|---|---|---|
| White | 3 560 | 831 | 23.34 % | (22.00 %, 24.74 %) |
| Asian | 1 940 | 383 | 19.74 % | (18.03 %, 21.57 %) |
| Hispanic or Latino | 620 | 83 | 13.39 % | (10.95 %, 16.27 %) |
| Black or African American | 480 | 54 | 11.25 % | (8.74 %, 14.38 %) |
| Two or More | 90 | 10 | 11.11 % | (6.14 %, 19.26 %) |

Intersectional — sex × race/ethnicity (**do not delete any cell**, including low-count cells):

| sex | race/ethnicity | n | shortlist=1 | selection_rate | Wilson 95 % CI | low-power flag |
|---|---|---|---|---|---|---|
| Male | White | 2 710 | 681 | 25.13 % | (23.51 %, 26.82 %) | — |
| Male | Asian | 1 350 | 300 | 22.22 % | (20.06 %, 24.54 %) | — |
| Male | Hispanic or Latino | 430 | 69 | 16.05 % | (12.87 %, 19.82 %) | — |
| Male | Black or African American | 270 | 34 | 12.59 % | (9.15 %, 17.09 %) | — |
| Male | Two or More | 60 | 8 | 13.33 % | (6.87 %, 24.26 %) | `n<30` false but *keep caveat* — small cell |
| Female | White | 850 | 150 | 17.65 % | (15.21 %, 20.37 %) | — |
| Female | Asian | 590 | 83 | 14.07 % | (11.51 %, 17.10 %) | — |
| Female | Hispanic or Latino | 190 | 14 | 7.37 % | (4.43 %, 11.97 %) | — |
| Female | Black or African American | 210 | 20 | 9.52 % | (6.24 %, 14.27 %) | — |
| Female | Two or More | 30 | 2 | 6.67 % | (1.84 %, 21.33 %) | **`n < 30` false — borderline — flag `low-power`** |

Headline metric from the lab-01 five-liner: **disparate impact / four-fifths on shortlist selection rate** per EEO-1 category *and* per intersectional cell. Diagnostic: equal opportunity.

Tolerance from the five-liner: `four-fifths ratio ≥ 0.80` per the EEOC Uniform Guidelines §1607.4(D) and the LL144 impact-ratio surface.

## Steps

### Step 1 — Create your working directory and the slice-report scaffold

```bash
mkdir -p mod106-lab02 && cd mod106-lab02
touch shortlist-slice-report.md
```

Open `shortlist-slice-report.md` and paste this scaffold. You will fill the eight sections in order.

```markdown
# ShortList — group-fairness slice report (lab 02 slice; unmitigated baseline)

System: ShortList (BERT-base shortlist classifier for a NYC software-engineering role; see lab 01)
Chapter references: mod-106 ch 02 (MetricFrame, slice-report shape, anti-patterns); ch 01 (five-liner header)
Lab-01 cross-ref: `shortlist-metric-choice.md` — pinned five-liner is the header
Mitigation state: UNMITIGATED BASELINE (mitigation is exercise 04 / future lab)

## 1. Metric-choice five-liner (header)

(verbatim from lab 01)

## 2. Data provenance

(dataset id; snapshot timestamp; row count; protected-attribute source; exclusion / imputation rules)

## 3. Per-group table — single-axis

(two tables — sex; race/ethnicity — each with n, selection_rate, Wilson CI)

## 4. Per-group table — intersectional

(sex × race/ethnicity table — all 10 cells, no deletions, low-power flags where applicable)

## 5. Aggregates

(max-minus-min selection rate per partition; four-fifths ratio per partition and per intersectional cell)

## 6. Headline verdict

(pass / fail against ε for the headline metric per group, per intersectional cell, overall — with the five-liner's tolerance named)

## 7. Mitigation state

("unmitigated baseline" — explicit, with reference to the exercise-04 / lab-03 mitigation loop that will consume this baseline)

## 8. Reproducibility manifest

(tool versions; eval-set fingerprint; model artefact hash; random seed; eval-script commit hash)
```

### Step 2 — Fill Section 1 (five-liner header) by copy from lab 01

The five-liner is *header*, not re-written here. Paste the five labelled lines from `shortlist-metric-choice.md` into Section 1 unchanged. If you catch yourself editing the five-liner based on numbers you are about to look at, stop — that is metric roulette by name (chapter 01 anti-pattern). Change control on the five-liner is a dated note, not a silent edit.

### Step 3 — Fill Section 2 (Data provenance)

```markdown
| Field | Value |
|---|---|
| Dataset id | `shortlist-eval-2026-11-21` |
| Snapshot timestamp | 2026-11-21T00:00:00Z |
| Row count | 6 690 |
| Protected-attribute source | Applicant self-report at submission, aligned to EEO-1 vocabulary; stored on the applicant record |
| Exclusion rules | Applicants who declined to report sex or race/ethnicity are not in this evaluation (separate reporting path — see register row `REG-SL-DISPIMP-01` comment) |
| Imputation rules | None — missing protected-attribute values excluded, not imputed |
```

Reading check: provenance is a six-line block that a reviewer can trace to the dataset. If any field is blank, a downstream reader (or auditor) cannot reproduce the numbers.

### Step 4 — Fill Section 3 (single-axis per-group tables)

Paste the two single-axis tables from the fabricated numbers directly. Carry `n`, `selection_rate`, and the Wilson 95 % CI for every cell. Do not delete any row — the single-axis tables are complete-vocabulary by construction.

```markdown
**Sex**

| sex | n | shortlist=1 | selection_rate | Wilson 95 % CI |
|---|---|---|---|---|
| Male | 4 820 | 1 013 | 21.02 % | (19.87 %, 22.21 %) |
| Female | 1 870 | 266 | 14.22 % | (12.69 %, 15.91 %) |

**Race/ethnicity (EEO-1 categories)**

| race/ethnicity | n | shortlist=1 | selection_rate | Wilson 95 % CI |
|---|---|---|---|---|
| White | 3 560 | 831 | 23.34 % | (22.00 %, 24.74 %) |
| Asian | 1 940 | 383 | 19.74 % | (18.03 %, 21.57 %) |
| Hispanic or Latino | 620 | 83 | 13.39 % | (10.95 %, 16.27 %) |
| Black or African American | 480 | 54 | 11.25 % | (8.74 %, 14.38 %) |
| Two or More | 90 | 10 | 11.11 % | (6.14 %, 19.26 %) |
```

Reading check: every cell carries a count *and* a CI. A table with `selection_rate` only (no count, no CI) is sample-size-blind by chapter 02's definition.

### Step 5 — Fill Section 4 (intersectional per-group table)

This is the section LL144 and chapter 02 both insist on. Paste all **10** intersectional cells from the fabricated table — including the `Female × Two or More` cell at `n = 30` which sits right on the low-power boundary. Flag the cell as `low-power`, add a caveat column; **do not delete the row**.

```markdown
**Sex × race/ethnicity — intersectional**

| sex | race/ethnicity | n | shortlist=1 | selection_rate | Wilson 95 % CI | caveat |
|---|---|---|---|---|---|---|
| Male | White | 2 710 | 681 | 25.13 % | (23.51 %, 26.82 %) | — |
| Male | Asian | 1 350 | 300 | 22.22 % | (20.06 %, 24.54 %) | — |
| Male | Hispanic or Latino | 430 | 69 | 16.05 % | (12.87 %, 19.82 %) | — |
| Male | Black or African American | 270 | 34 | 12.59 % | (9.15 %, 17.09 %) | — |
| Male | Two or More | 60 | 8 | 13.33 % | (6.87 %, 24.26 %) | small cell; CI wide — interpret with care |
| Female | White | 850 | 150 | 17.65 % | (15.21 %, 20.37 %) | — |
| Female | Asian | 590 | 83 | 14.07 % | (11.51 %, 17.10 %) | — |
| Female | Hispanic or Latino | 190 | 14 | 7.37 % | (4.43 %, 11.97 %) | — |
| Female | Black or African American | 210 | 20 | 9.52 % | (6.24 %, 14.27 %) | — |
| Female | Two or More | 30 | 2 | 6.67 % | (1.84 %, 21.33 %) | **`low-power` — n at the boundary; CI very wide** |
```

Reading check: 10 rows. Every cell present. The `low-power` flag is on the `Female × Two or More` row with a caveat on the metric value. If you deleted that row, you hit two anti-patterns at once (sample-size blind by deletion; non-intersectional-complete).

### Step 6 — Fill Section 5 (Aggregates)

The aggregates section summarises the per-group tables with scalar numbers the verdict will read. Compute:

- **Max-minus-min selection rate** per partition (sex: 21.02 − 14.22 = **6.80 pp**; race/ethnicity: 23.34 − 11.11 = **12.23 pp**; intersectional: 25.13 − 6.67 = **18.46 pp**).
- **Four-fifths ratio** per partition and per intersectional cell: for each cell, `cell_selection_rate / max_cell_selection_rate_in_partition`. The max rate is the denominator. A ratio below 0.80 is a four-fifths failure at EEOC §1607.4(D).

```markdown
**Max-minus-min selection rate**

| Partition | min | max | max − min |
|---|---|---|---|
| sex | 14.22 % (Female) | 21.02 % (Male) | 6.80 pp |
| race/ethnicity | 11.11 % (Two or More) | 23.34 % (White) | 12.23 pp |
| intersectional sex × race/ethnicity | 6.67 % (Female × Two or More, `low-power`) | 25.13 % (Male × White) | 18.46 pp |

**Four-fifths ratio — single-axis sex** (denominator = Male rate 21.02 %)

| sex | selection_rate | four-fifths ratio |
|---|---|---|
| Male | 21.02 % | 1.00 (reference) |
| Female | 14.22 % | **0.677** — below 0.80, four-fifths fails |

**Four-fifths ratio — single-axis race/ethnicity** (denominator = White rate 23.34 %)

| race/ethnicity | selection_rate | four-fifths ratio |
|---|---|---|
| White | 23.34 % | 1.00 (reference) |
| Asian | 19.74 % | 0.846 — passes |
| Hispanic or Latino | 13.39 % | **0.574** — below 0.80, fails |
| Black or African American | 11.25 % | **0.482** — below 0.80, fails |
| Two or More | 11.11 % | **0.476** — below 0.80, fails |

**Four-fifths ratio — intersectional** (denominator = Male × White rate 25.13 %)

| sex × race/ethnicity | selection_rate | four-fifths ratio |
|---|---|---|
| Male × White | 25.13 % | 1.00 (reference) |
| Male × Asian | 22.22 % | 0.884 — passes |
| Male × Hispanic or Latino | 16.05 % | **0.639** — fails |
| Male × Black or African American | 12.59 % | **0.501** — fails |
| Male × Two or More | 13.33 % | **0.530** — fails (small cell; caveat) |
| Female × White | 17.65 % | **0.702** — fails |
| Female × Asian | 14.07 % | **0.560** — fails |
| Female × Hispanic or Latino | 7.37 % | **0.293** — fails (worst intersectional cell) |
| Female × Black or African American | 9.52 % | **0.379** — fails |
| Female × Two or More | 6.67 % | **0.265** — fails (`low-power`, CI wide; interpret with care) |
```

Reading check: four-fifths is computed per partition *against the highest-rate cell in that partition* and per intersectional cell *against the highest-rate intersectional cell*. The LL144 impact-ratio surface (chapter 06) reads the intersectional computation directly. The `low-power` cell carries its ratio and its caveat side-by-side — you do not hide the number and you do not suppress the caveat.

### Step 7 — Fill Section 6 (Headline verdict)

The verdict names the pass/fail against the five-liner's tolerance `ε = four-fifths ratio ≥ 0.80`, per single-axis group, per intersectional cell, and overall. "Overall" is "did every cell pass?" — one cell failing is a report-level fail.

```markdown
**Headline metric:** disparate impact / four-fifths ratio (per lab-01 five-liner)
**Tolerance:** ratio ≥ 0.80 per EEOC Uniform Guidelines §1607.4(D) and NYC LL144 (6 RCNY §5-300 et seq.)

| Partition | Verdict |
|---|---|
| sex | **FAIL** — Female ratio 0.677 below 0.80 |
| race/ethnicity | **FAIL** — Hispanic or Latino, Black or African American, and Two or More all below 0.80 |
| intersectional sex × race/ethnicity | **FAIL** — 9 of 10 cells below 0.80; worst cell Female × Hispanic or Latino at 0.293 |
| overall | **FAIL** — the model does not satisfy four-fifths on any of the three required partitions |

**Diagnostic — equal opportunity.** (Reported from the ground-truth-positive subset; numbers are in
Section 3 of the mitigation report if / when exercise 04 ships. Noted here as "diagnostic not computed
in this lab's drill" — in exercise 02 this is a hard requirement.)
```

Reading check: the verdict is `FAIL` with named cells — not a hand-waved "we have some disparity." The `low-power` cell's failure is reported *and* carries its caveat; it is not quietly promoted to pass.

### Step 8 — Fill Section 7 (Mitigation state)

Chapter 02's anti-pattern "Mitigation applied before the assessment is written" means the pre-mitigation numbers *must* ship, and the baseline block must be explicit about its state. Fill:

```markdown
This slice report is the **unmitigated baseline** on the audit-window eval set. No mitigation algorithm
has been applied (no Reweighing; no ExponentiatedGradient; no ThresholdOptimizer). The register-row
residuals computed from these numbers are the *pre-mitigation* residuals. The exercise-04 / mitigation
loop will consume this baseline, sweep pre-/in-/post-processing algorithms against the headline metric
(disparate impact / four-fifths), produce the Pareto curve, and pick an operating point with written
justification; the *post-mitigation* residuals land in a subsequent slice report of this same shape,
linked here once produced.
```

Reading check: "unmitigated baseline" is written explicitly, and the forward reference to the mitigation loop is explicit. A slice report that shows mitigated numbers without the baseline is a chapter-02 anti-pattern by name.

### Step 9 — Fill Section 8 (Reproducibility manifest)

Fill with placeholders in the fabricated-number spirit — the shape matters, not the specific hashes.

```markdown
| Field | Value |
|---|---|
| Fairlearn version | `fairlearn==0.11.0` <!-- needs-research: pin the version exercise 02 actually uses --> |
| AIF360 version | `aif360==0.6.1` <!-- needs-research --> |
| numpy / pandas | `numpy==2.1.3`; `pandas==2.2.3` |
| Model artefact hash | `sha256:<model-artefact-hash-placeholder>` |
| Eval-set fingerprint | `sha256:<eval-set-content-hash-placeholder>`; row count 6 690 |
| Random seed | 20261121 |
| Eval-script commit hash | `<git-sha-placeholder>` |
```

Reading check: every field has a value (even if `<placeholder>`); a blank line in the manifest is a reproducibility hole.

### Step 10 — Final checkpoint

Re-read the eight sections. Confirm:

- **The five-liner header is verbatim from lab 01, unedited.** You did not rewrite it based on numbers you just saw.
- **Every per-cell row carries `n` and a Wilson CI.** Not point estimates alone.
- **The intersectional table has 10 rows.** Not 9 (deleted the low-power cell) or 2 (collapsed to a single axis).
- **The `low-power` cell is present and flagged with a caveat on the metric value.** Not deleted.
- **Four-fifths is computed per partition and per intersectional cell against the highest-rate cell in that partition.** Not against an arbitrary reference or against the overall rate.
- **The verdict names failing cells explicitly, and the overall verdict is `FAIL`.** Not "some disparity observed."
- **The mitigation-state block says "unmitigated baseline" explicitly.** A mitigation-state block that is missing — or that shows mitigated numbers without a baseline — is the chapter-02 anti-pattern "Hidden pre-mitigation numbers."
- **The manifest has a value in every field.** Placeholders with `<...>` are fine for the drill; blanks are not.

## Expected output

`mod106-lab02/shortlist-slice-report.md` — one populated eight-section slice report; one to two pages. The intersectional table has 10 rows (with the `Female × Two or More` row flagged `low-power`, not deleted); four-fifths is computed per partition and per intersectional cell; the verdict is `FAIL` with named cells; the mitigation-state block says `unmitigated baseline` explicitly; the manifest has a value in every field.

## Checkpoint questions (self-check)

1. The lab's intersectional table has `Female × Two or More` at `n = 30` with a four-fifths ratio of 0.265. A reviewer argues the row should be dropped because "n is at the boundary and the CI spans from 1.84 % to 21.33 %." Per chapter 02, what are the two specific things wrong with dropping that row? (First, the chapter's anti-pattern "Sample-size blind" is specifically that low-power cells are *flagged with a count and a CI and a caveat*, not deleted — the Wilson CI being very wide is a *true statement about the evidence* and belongs in the report. Second, dropping intersectional cells hits the "Non-intersectional-only" anti-pattern via the back door: a report that reports 9 of 10 intersectional cells fails the LL144 shape, which chapter 06 requires be intersectional-complete across the EEO-1 vocabulary. The correct treatment is to keep the row, flag `low-power`, and note the caveat alongside the metric.)
2. A reviewer suggests computing the four-fifths ratio for each cell against the *overall* selection rate (the whole-population shortlist rate, ~19 %) rather than against the highest-rate cell in the partition. Per chapter 02 and chapter 06, why is that specifically wrong? (The EEOC Uniform Guidelines §1607.4(D) and the LL144 impact-ratio surface both specify the ratio as `category_rate / highest_category_rate`. Computing against the overall rate changes the denominator from the privileged group's rate to a weighted average of all groups' rates, which inflates the ratio when the privileged group is a majority and understates it when the privileged group is a minority. The chapter-02 / chapter-06 anchor is the highest-rate cell; the lab denominator is Male × White at 25.13 %.)
3. The verdict section marks `overall = FAIL` because at least one cell fails. A junior engineer argues the overall should be `PARTIAL PASS` because the Asian row passes. Per chapter 02's slice-report shape and chapter 01's metric-choice formulation, why is `overall = FAIL` the shippable verdict? (The chapter-01 five-liner names the headline metric and the tolerance `ε`; the chapter-02 verdict shape is pass/fail against that tolerance per group, per intersectional cell, and overall. "Overall" is the aggregate statement about the headline metric across the partition the five-liner declared — at four-fifths, the partition fails if *any* cell in the partition falls below 0.80 per the Guidelines. "Partial pass" is not a defined verdict in the slice-report shape; the reader would not know which partitions it applies to or what the tolerance was. `FAIL` with named failing cells is the shippable shape.)
4. The mitigation-state block says "unmitigated baseline" and references the exercise-04 mitigation loop. A reviewer argues the block is redundant because "if there are no mitigation numbers, obviously there was no mitigation." Per chapter 02's anti-patterns, why is the explicit block load-bearing? (Chapter 02 names "Mitigation applied before the assessment is written" as an anti-pattern because when it occurs, the report shows mitigated numbers with no indication they are mitigated and no pre-mitigation baseline to compare against. The explicit `unmitigated baseline` block forecloses this: it tells the reader exactly which numbers they are looking at and guarantees that a subsequent mitigated slice report of this same shape will carry a *delta* column against this baseline. A missing block is the exact anti-pattern — the reader has no way to know which state they are reading.)

## Why this lab matters

Exercise 02 asks you to run Fairlearn `MetricFrame` and AIF360 `ClassificationMetric` on the real running model, reconcile the two libraries' numbers, add Wilson CIs that the libraries do not produce for you, and compose everything into the chapter-02 slice-report shape — all in three hours. If the slice-report shape step consumes most of the budget because you are re-deriving the eight sections, the intersectional-cell requirement, the `low-power` discipline, the four-fifths denominator convention, and the mitigation-state block from scratch, the exercise's reconciliation work gets squeezed to the point where cell-definition bugs between Fairlearn and AIF360 land in the final report unchecked. This lab drills the shape step on fabricated numbers so that in exercise 02 you spend the three hours on the Fairlearn / AIF360 runs and their diagnostics, not on inventing the report's column list.
