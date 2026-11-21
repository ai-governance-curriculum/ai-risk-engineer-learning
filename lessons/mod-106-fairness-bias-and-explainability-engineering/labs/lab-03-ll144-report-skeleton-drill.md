# Lab 03 — NYC LL144 Report Skeleton and Public-Summary Drill

**Duration:** 45 minutes · **Format:** guided practice, markdown authoring on the fabricated numbers from labs 01 and 02

## Objective

Internalise chapter 06's **NYC Local Law 144 report skeleton** — the 13-section shape an independent auditor expects, the EEOC adverse-impact field set that sits inside it, and the **public summary** extract that the employer publishes — by packaging the lab-01 metric-choice five-liner and the lab-02 slice-report numbers end-to-end on the fabricated `ShortList` system. The lab also drills the **L15 / L25 / independent-auditor** role split memo and the **reviewer's checklist** the external auditor walks down as a go / no-go.

Chapter 06's two first-pass defects — technically-correct-but-un-fileable reports (the engineer produces a JSON, the analyst cannot file it) and analyst-written narrative before the numbers come in — are exactly the shape failures this lab forecloses by making the engineer author against the published field list from the start. Exercise 06 asks you to do this on the real running model's numbers; the lab drills the atomic authoring skill on the fabricated numbers already in hand from labs 01 and 02.

## Prerequisites

- Chapter 06 read — specifically "NYC Local Law 144 — AEDT bias audit" (the required audit fields and the LL144-to-mod-106 mapping), "EEOC adverse-impact and the four-fifths rule", "The L15 / L25 handoff at the report boundary", and "The report skeleton — LL144 shape".
- Chapters 01, 02, 04, and 05 skimmed enough to recognise the cross-refs each report section cites.
- Lab 01 and Lab 02 completed — the five-liner from lab 01 and the slice-report numbers from lab 02 are the inputs to this lab's report. If either is not in hand, do them first.
- A terminal where you can create a working directory and author markdown files.

## The fixed example system and inputs

Same running system as labs 01 and 02 — **ShortList**, the BERT-base shortlist classifier for a NYC software-engineering role, in scope for NYC Local Law 144.

**Inputs to this lab:**

- `shortlist-metric-choice.md` from lab 01 — the pinned five-liner, decision-table trace, impossibility paragraph, and cross-refs.
- `shortlist-slice-report.md` from lab 02 — the eight-section unmitigated-baseline slice report with the single-axis and intersectional per-group tables, the four-fifths computation, the verdict (`FAIL` on all three partitions), the `unmitigated baseline` block, and the reproducibility manifest.

**Audit window** (fabricated, authoritative for this lab): the LL144 audit covers 2026-02-01 through 2026-11-21. The last audit was 2025-11-20; the next is due by 2026-11-20 — which has just passed, so the lab's `clock.md` must flag the overdue state. The data source is **historical use data** (not test data).

**Independent auditor path** (fabricated): the employer has retained a BABL-AI-credentialed Algorithm Bias Auditor at an accredited firm, not materially dependent on the ShortList vendor. The L25 engineer (you) is building the report *for* the auditor to sign; the L25 engineer is not the signer.

## Steps

### Step 1 — Create your working directory and the five-file scaffold

```bash
mkdir -p mod106-lab03 && cd mod106-lab03
touch ll144-report.md eeoc-adverse-impact.md public-summary.md roles.md reviewer-checklist.md
```

Open each file and paste the matching scaffold below. You will fill the five files in order.

#### `ll144-report.md` scaffold

```markdown
# Bias Audit — ShortList — <Auditor> — 2026-11-21

(fill 13 numbered sections per chapter 06's skeleton — see steps below)
```

#### `eeoc-adverse-impact.md` scaffold

```markdown
# EEOC adverse-impact evaluation — ShortList — 2026-11-21

(seven-field block per chapter 06 §EEOC — see step 7)
```

#### `public-summary.md` scaffold

```markdown
# ShortList — public bias-audit summary (LL144 §public-summary extract)

(the subset of the LL144 report that the employer publishes on the website — audit date, selection rates per category, impact ratios, intersectional slice, auditor identity; written for a public reader who has not seen the full report)
```

#### `roles.md` scaffold

```markdown
# ShortList — L15 / L25 / independent-auditor split memo

(one-page memo naming what L25 built, what L15 does next, what the independent auditor does, why the three do not collapse)
```

#### `reviewer-checklist.md` scaffold

```markdown
# ShortList — reviewer's checklist (external auditor go / no-go)

(the 15-line checklist the auditor walks down — see step 10)
```

### Step 2 — Fill `ll144-report.md` sections 1 and 2 (identity and tool)

Chapter 06's skeleton is 13 numbered sections. Fill them in order.

```markdown
## 1. Auditor identity and independence attestation

Auditor: BABL AI Algorithm Bias Auditor at <firm name>, accredited.
Independence attestation: the auditor is not involved in using, developing, or distributing
ShortList; no material financial dependence on the vendor or the employer beyond the audit
engagement fee. (Placeholder — the signer attests in their own words at sign time.)

**L15 / L25 / independent-auditor split (see `roles.md`):** this report was *built* by the L25
AI-risk-engineer; it will be *filed* by the L15 AI-governance-analyst with the narrative wrapper
the DCWP submission requires; it will be *audited and signed* by the BABL AI Algorithm Bias
Auditor above. The three responsibilities do not collapse — LL144 §public-summary enforces
auditor independence at the statute level.

## 2. Tool identity, version, deployment scope, dates of use

| Field | Value |
|---|---|
| Tool name | ShortList |
| Tool version | <sha / release tag of the shipped model> |
| Deployment scope | Resume pre-screening for a software-engineering role posted in NYC |
| Role of tool in decision | Substantially assists a human recruiter who schedules phone screens from the shortlist |
| LL144 coverage determination | In scope — AEDT used in NYC for employment decisions |
| Dates of use covered by this audit | 2026-02-01 through 2026-11-21 |
| Data source for audit | Historical use data (not test data) |
```

### Step 3 — Fill sections 3 and 4 (data source; metric definitions)

```markdown
## 3. Data source and per-category counts

| Partition | Counts |
|---|---|
| sex (EEO-1) | Male 4 820; Female 1 870; Total 6 690 |
| race/ethnicity (EEO-1) | White 3 560; Asian 1 940; Hispanic or Latino 620; Black or African American 480; Two or More 90; Total 6 690 |
| Intersectional sex × race/ethnicity | See Section 8 — 10 cells; Female × Two or More at `n=30` is flagged `low-power` but *not deleted* |
| Exclusions | Applicants who declined to self-report sex or race/ethnicity are excluded from this audit; their decisions are reported on a separate path (register row `REG-SL-DISPIMP-01` comment). |
| Data source narrative | Historical use data for the audit window 2026-02-01 → 2026-11-21. Protected attributes self-reported at submission per EEO-1 vocabulary. |

## 4. Metric definitions and choices

(Verbatim from `shortlist-metric-choice.md`, lab 01 — the five-liner is the metric declaration this audit answers.)

1. **Decision context.** ShortList — NYC software-engineering shortlist, in LL144 scope.
2. **Protected-attribute partition.** Sex (EEO-1) and race/ethnicity (EEO-1), plus intersectional sex × race/ethnicity required by LL144.
3. **Primary harm.** A qualified applicant from a protected class is not shortlisted, denying them access to the opportunity to interview (HM-SL-01); intersectional cell hiding (HM-SL-02).
4. **Headline metric.** Disparate impact / four-fifths on shortlist selection rate per EEO-1 category *and* per intersectional cell, per EEOC §1607.4(D) and the DCWP LL144 impact-ratio field; equal opportunity as a diagnostic.
5. **Metrics we explicitly do not headline.** Predictive parity — base rates differ across groups in the historical labels (HM-SL-03), so headlining predictive parity would propagate the historical bias; trade-off per Chouldechova (2016).

**Tolerance.** Four-fifths ratio ≥ 0.80 per EEOC §1607.4(D) and NYC LL144 (6 RCNY §5-300 et seq.).
**Metric-choice change-control note.** The five-liner was pinned before the evaluation ran; no change control event in the audit window.
```

### Step 4 — Fill sections 5–8 (selection rate; scoring rate; impact ratios; intersectional)

These sections paste directly from the lab-02 slice report. Keep every table — including the `low-power`-flagged intersectional cell.

```markdown
## 5. Selection rate per category — sex and race/ethnicity

(Paste the two single-axis tables from lab-02 Section 3 — sex, then race/ethnicity — unchanged,
including the Wilson 95 % CIs and the counts.)

## 6. Scoring rate per category

Not applicable — ShortList produces a binary shortlist decision, not a score. (LL144 permits
either; the scoring-rate field is empty for binary tools, with this explicit statement.)

## 7. Impact ratios per category vs the highest-rate category

(Paste the four-fifths ratio tables from lab-02 Section 5, including the single-axis sex table,
the single-axis race/ethnicity table, and the intersectional table. The denominators are the
highest-rate cells in each partition — Male for sex; White for race/ethnicity; Male × White for
the intersectional partition. 0.80 is the four-fifths line; cells below are marked FAIL.)

## 8. Intersectional slice — sex × race/ethnicity

(Paste the 10-row intersectional table from lab-02 Section 4 verbatim — including the
`Female × Two or More` row flagged `low-power`. Add a cross-ref note: "The intersectional slice
is required by DCWP 6 RCNY §5-300 et seq. — a single-axis-only report does not discharge LL144.")
```

Reading check: sections 5–8 paste the lab-02 tables; no numbers are recomputed. If you catch yourself re-deriving the four-fifths ratios here, stop — they were computed in lab 02 and this report *cites* them, not *re-derives* them. The reproducibility manifest in Section 12 is what guarantees the paste matches the slice report's numbers.

### Step 5 — Fill sections 9–11 (significance; practical significance; validity pointer)

```markdown
## 9. Statistical significance notes

Wilson 95 % confidence intervals are carried on every cell (Sections 5 and 8). The
`Female × Two or More` cell is flagged `low-power` — `n=30`, Wilson CI (1.84 %, 21.33 %) — and
its impact ratio of 0.265 carries that caveat. A Fisher exact test is not required for cells
where the four-fifths ratio fails outside the CI entirely; eight of the nine failing cells meet
that condition. The `Female × Two or More` cell's failure is statistically uncertain due to the
small cell; the auditor is directed to Section 10 for practical-significance reasoning on that
row.

## 10. Practical significance narrative

The single-axis and intersectional four-fifths ratios fail on nine of ten intersectional cells
(the one pass is Male × Asian at 0.884). The worst intersectional cell is
Female × Hispanic or Latino at 0.293 — a shortlist rate roughly 29 % of the privileged-cell
rate. At the audit-window scale (6 690 applicants), these differences are practically
significant: a shortlist-rate ratio of 0.293 corresponds to a Hispanic-or-Latino female
applicant being surfaced to the recruiter at less than one-third the rate of an equivalently-
pooled white male applicant. The practical consequence — denied access to the first interview
stage — is the specific primary harm (HM-SL-01) the five-liner anchors to.

## 11. Validity evidence pointer

Adverse impact is present; EEOC §1607.5 directs the employer to cite a validity file (job
analysis plus criterion / content / construct validity for the ShortList tool). The validity
file is held by the HR / assessment team at <HR system pointer> under <document id>; the
L15 AI-governance-analyst is responsible for routing to that team on filing. The validity file
is *not* a mod-106 deliverable — chapter 06 is explicit — but the pointer is load-bearing in
this report.
```

### Step 6 — Fill sections 12 and 13 (reproducibility manifest; public summary extract)

```markdown
## 12. Reproducibility manifest

(Paste from lab-02 Section 8 verbatim. Add this audit-specific line:)

| Field | Value |
|---|---|
| Audit script commit hash | `<git-sha-placeholder>` for the composed-report script |
| Audit window | 2026-02-01 → 2026-11-21 |
| Composed-report hash | `sha256:<composed-report-hash-placeholder>` — the hash the auditor cites in the signature |

## 13. Public summary (extract for the employer's website — see `public-summary.md`)

Pointer: the LL144 public summary is authored in `public-summary.md`; it includes the audit
date, the selection rates per category, the impact ratios per category (with intersectional),
and the auditor identity. Per LL144, the summary must be publicly available on the employer's
website and candidates must be notified ≥ 10 business days in advance of use.
```

### Step 7 — Fill `eeoc-adverse-impact.md` (seven-field block)

```markdown
# EEOC adverse-impact evaluation — ShortList — 2026-11-21

**Anchor:** EEOC Uniform Guidelines on Employee Selection Procedures (UGESP), 29 CFR Part 1607.

| Field | Content |
|---|---|
| Selection event | Shortlist decision — `shortlist=1` surfaces the applicant to the recruiter queue |
| Applicant pool per category | Sex (EEO-1): Male 4 820; Female 1 870. Race/ethnicity (EEO-1): White 3 560; Asian 1 940; Hispanic or Latino 620; Black or African American 480; Two or More 90. Intersectional sex × race/ethnicity — see LL144 Section 8. |
| Selection rate per category | See LL144 Section 5 and Section 8. |
| Four-fifths ratio per category vs highest-rate | See LL144 Section 7 — FAIL on sex (Female 0.677), race/ethnicity (three cells below 0.80), and nine of ten intersectional cells. |
| Statistical significance | Wilson 95 % CIs carried per cell; eight of nine failing cells fail outside the CI; `Female × Two or More` carries a small-cell caveat. |
| Practical significance narrative | See LL144 Section 10 — the shortfall corresponds to a materially lower chance of a protected-class applicant reaching the first-interview stage. |
| Validity evidence | Adverse impact present (§1607.4(D)); validity file pointer at `<HR system pointer / document id>` per §1607.5. If the validity file is unavailable at filing time, the EEOC determination is *adverse impact unmitigated by evidence of validity*. |
```

Reading check: the EEOC document is a seven-field block, every field filled; the validity pointer is explicit and names what happens if the validity file is missing (the EEOC determination becomes unmitigated adverse impact, which is a stronger-than-fail state).

### Step 8 — Fill `public-summary.md`

The public summary is written for a *public reader* who has not seen the full report. It is also what gets published on the employer's website per LL144. Use plain English and include the required numbers.

```markdown
# ShortList — public bias-audit summary

**Audit date:** 2026-11-21
**Auditor:** <BABL AI Algorithm Bias Auditor name / firm>
**Tool:** ShortList — automated shortlist pre-screening used in NYC for a software-engineering role.
**Data source:** Historical use data from 2026-02-01 through 2026-11-21 — 6 690 applicants.

## Selection rates by sex (EEO-1)

| sex | selection rate |
|---|---|
| Male | 21.02 % |
| Female | 14.22 % |

**Impact ratio** (Female vs the highest-rate sex, Male): **0.677** — below the four-fifths (0.80)
threshold set by the EEOC Uniform Guidelines §1607.4(D).

## Selection rates by race/ethnicity (EEO-1)

| race/ethnicity | selection rate | impact ratio (vs highest) |
|---|---|---|
| White | 23.34 % | 1.00 (highest-rate reference) |
| Asian | 19.74 % | 0.846 |
| Hispanic or Latino | 13.39 % | 0.574 |
| Black or African American | 11.25 % | 0.482 |
| Two or More | 11.11 % | 0.476 |

Three of five race/ethnicity categories fall below the four-fifths threshold.

## Intersectional slice — sex × race/ethnicity

| sex × race/ethnicity | selection rate | impact ratio (vs highest) |
|---|---|---|
| Male × White | 25.13 % | 1.00 (reference) |
| Male × Asian | 22.22 % | 0.884 |
| Male × Hispanic or Latino | 16.05 % | 0.639 |
| Male × Black or African American | 12.59 % | 0.501 |
| Male × Two or More | 13.33 % | 0.530 (small cell — interpret with care) |
| Female × White | 17.65 % | 0.702 |
| Female × Asian | 14.07 % | 0.560 |
| Female × Hispanic or Latino | 7.37 % | 0.293 |
| Female × Black or African American | 9.52 % | 0.379 |
| Female × Two or More | 6.67 % | 0.265 (low-power — n=30 — interpret with care) |

Nine of ten intersectional cells fall below the four-fifths threshold.

## What this summary does and does not establish

This public summary presents the *impact ratios* and *selection rates* required by NYC Local
Law 144 (6 RCNY §5-300 et seq.). It is a factual accounting of the audited tool's behaviour
over the audit window. It does not by itself establish lawful or unlawful discrimination — the
employer's validity evidence under EEOC §1607.5 is the relevant companion document — but the
numbers above are below the four-fifths threshold on the three required partitions and are the
numbers the regulator will read first.
```

Reading check: the public summary includes the audit date, the auditor identity, the selection rates per category, the impact ratios, the intersectional slice (all 10 cells), and a short "what this summary does and does not establish" paragraph. Chapter 06's anti-pattern "Public summary that hides the numbers" is avoided — the numbers are *in* the summary, not pushed into a linked PDF.

### Step 9 — Fill `roles.md` (one-page L15 / L25 / independent-auditor split)

```markdown
# ShortList — L15 / L25 / independent-auditor split memo

NYC Local Law 144 and the ForHumanity / BABL AI credentialed shapes require three distinct
roles. This memo pins which role did what on this audit.

## What the L25 AI-risk-engineer (this role) *built*

- The metric-choice five-liner pinned before the run (`shortlist-metric-choice.md` from lab 01).
- The Fairlearn `MetricFrame`-shape unmitigated-baseline slice report with Wilson CIs, four-fifths
  ratios, and intersectional slices (`shortlist-slice-report.md` from lab 02).
- The reconciled AIF360 numbers (in the exercise-02 deliverable; reconciliation script catches
  cell-definition bugs between Fairlearn and AIF360 before the report composes).
- The composed LL144 report (`ll144-report.md`), the EEOC adverse-impact evaluation
  (`eeoc-adverse-impact.md`), and the public summary (`public-summary.md`).
- The reproducibility manifest (hashed artefact bundle, tool versions pinned, eval-script commit).

## What the L15 AI-governance-analyst does *next*

- Wraps the public summary in the narrative the DCWP submission requires and files it.
- Publishes the public summary on the employer's website and notifies candidates ≥ 10 business
  days in advance of use (LL144 §public-summary and §candidate-notice).
- Tracks the annual re-audit clock — next audit due by 2026-11-20 (which has passed; see
  `clock.md` — see mod-109 handoff).
- Routes the validity-file pointer (Section 11) to the HR / assessment team if the file is not
  current at filing time.

## What the *independent* third-party auditor does

- Audits the report and the process. Walks `reviewer-checklist.md` as the go / no-go.
- Verifies independence per LL144 — not involved in using, developing, or distributing the tool.
- Signs the audit report (or rejects it). The signature is attached to the composed-report hash
  in Section 12 of the LL144 report so the signed artefact is the specific audited version.
- Does *not* re-run the evaluation from scratch; audits the artefact the L25 engineer produced
  and the process that produced it.

## Why the three do not collapse

- The L25 engineer is *not independent* of the system they built — they cannot sign the external
  audit, even as a co-signer. LL144 enforces this at the statute level.
- The L15 analyst does *not build* the fairness evaluation — the technical work is not in the
  analyst's brief; the analyst reads, files, and tracks.
- The external auditor does *not re-run* the evaluation; they audit the artefact and the
  process. If the artefact is not engineered for external consumption (no hashes, no pinned
  tool versions, no metric-choice pre-registration, no reproducible manifest), the sign-off is
  not defensible — which is why the manifest in Section 12 is load-bearing.
```

### Step 10 — Fill `reviewer-checklist.md` (15-line auditor go / no-go)

The checklist is the explicit list the external auditor walks down. One line per item; every item must point at a filled section of the report.

```markdown
# ShortList — reviewer's checklist (external auditor go / no-go)

The 15 items below must all be YES for the audit to be signable per chapter 06.

1. [ ] **Independence attestation.** Is Section 1 of the LL144 report present and does it name the auditor and their independence basis? (`ll144-report.md §1`)
2. [ ] **Metric-choice pinned before the run.** Does Section 4 reproduce the five-liner verbatim from `shortlist-metric-choice.md`, with a change-control note? (`ll144-report.md §4`)
3. [ ] **Per-category counts present.** Does Section 3 give `n` per single-axis category and does Section 8 give `n` per intersectional cell? (`ll144-report.md §3, §8`)
4. [ ] **Selection rate per category.** Is Section 5 populated per sex and per race/ethnicity per EEO-1? (`ll144-report.md §5`)
5. [ ] **Four-fifths ratio per category.** Does Section 7 compute the ratio per category against the highest-rate category? (`ll144-report.md §7`)
6. [ ] **Intersectional slice present and complete.** Does Section 8 present the 10-cell sex × race/ethnicity table with no deletions? (`ll144-report.md §8`)
7. [ ] **Low-power cells flagged, not deleted.** Is the `Female × Two or More` cell flagged `low-power` and carried with a caveat? (`ll144-report.md §8`)
8. [ ] **Statistical significance notes.** Does Section 9 name the Wilson CIs and the small-cell caveats? (`ll144-report.md §9`)
9. [ ] **Practical significance narrative present.** Is Section 10 written in prose that names the operational impact? (`ll144-report.md §10`)
10. [ ] **Validity pointer present if adverse impact exists.** Does Section 11 point at the validity file and name what happens if it is missing? (`ll144-report.md §11`)
11. [ ] **Reproducibility manifest complete.** Does Section 12 pin tool versions, dataset fingerprint, model artefact hash, eval-script commit, random seed? (`ll144-report.md §12`)
12. [ ] **Hashes match the evidence bundle.** Do the hashes in Section 12 match the files in `evidence/`? (exercise 06 hash manifest)
13. [ ] **Public summary extract present.** Is `public-summary.md` authored for a public reader, with the impact-ratio numbers and the audit date? (`public-summary.md`)
14. [ ] **L15 / L25 / independent-auditor split memo present.** Does `roles.md` name the three roles and why they do not collapse? (`roles.md`)
15. [ ] **Compliance clock current.** Is the next-audit-due date tracked, with the mod-109 drift-trigger monitor named? (noted in `clock.md` — flagged *overdue* by 1 day in this lab's audit-window scenario.)
```

Reading check: every checklist item points at a specific filled section of the report. If any item is blank or ambiguous, the auditor's go / no-go cannot be mechanical.

### Step 11 — Final checkpoint

Re-read the five files. Confirm:

- **The LL144 report has 13 numbered sections.** Not 12 (missing public-summary pointer) or 11 (missing validity pointer).
- **The five-liner in Section 4 is verbatim from lab 01, unedited.** You did not rewrite it after seeing the numbers.
- **Sections 5, 7, and 8 paste from lab 02, not re-derive.** The slice report is the source of truth; the LL144 report cites it.
- **The intersectional table in Section 8 has 10 rows, including the `low-power` row flagged with a caveat.** Not deleted; not quietly dropped.
- **The EEOC document has seven filled fields including the validity pointer.** Not six.
- **The public summary includes the audit date, the auditor identity, the selection rates, the impact ratios, the intersectional slice, and a short "what this does / does not establish" paragraph.** Numbers in the summary, not in a linked PDF.
- **The roles memo names what L25 built, what L15 does next, and what the independent auditor does.** The three do not collapse.
- **The reviewer's checklist has 15 items, each pointing at a specific filled section.** Not ambiguous.
- **The compliance clock is flagged `overdue` (next-due 2026-11-20; audit date 2026-11-21).** This is the "annual re-audit missed" anti-pattern by one day — flagging it explicitly is the correct handling.

## Expected output

`mod106-lab03/` containing:

- `ll144-report.md` — the 13-section LL144 report, every section filled from labs 01 / 02 and the lab's fabricated inputs.
- `eeoc-adverse-impact.md` — the seven-field EEOC adverse-impact evaluation with the validity pointer.
- `public-summary.md` — the public-summary extract with impact ratios and the "what this summary does / does not establish" paragraph.
- `roles.md` — the one-page L15 / L25 / independent-auditor split memo.
- `reviewer-checklist.md` — the 15-item auditor go / no-go checklist pointing at each filled section.

Two to four pages across the five files.

## Checkpoint questions (self-check)

1. The lab's LL144 report Section 6 (scoring rate) is marked "Not applicable — ShortList is a binary tool." A reviewer argues this field should be deleted since it is not applicable. Per chapter 06, why is the explicit "not applicable" statement load-bearing rather than being a candidate for deletion? (The DCWP rule enumerates the required fields; a report that silently omits a field (even a not-applicable one) looks to a reviewer like a field that was forgotten, not a field that was deliberately marked out of scope. The chapter-06 skeleton lists all 13 sections for exactly this reason — the auditor's go / no-go walks every section, and "section not present" is ambiguous. An explicit "not applicable — the tool is binary" is the correct shape; it tells the auditor the scoring-rate field was considered and the tool's shape excludes it.)
2. The lab's public summary includes the audit date, the impact ratios, the intersectional slice, and a short "what this summary does and does not establish" paragraph. A reviewer suggests moving the impact ratios into a linked PDF to make the summary "cleaner" for the website. Per chapter 06's anti-patterns, why is that specifically wrong? ("Public summary that hides the numbers" is a named anti-pattern — chapter 06 is explicit that LL144 requires the impact ratios be public, and pushing them into a linked PDF (which is sometimes never uploaded, and is in any case harder for a reader to parse) is non-compliant in spirit. The numbers must be *in* the public summary.)
3. The lab's Section 12 pins the composed-report hash the auditor will cite in their signature. A junior engineer argues this hash is redundant because "the auditor can just hash the file themselves." Per chapter 06's L25 / independent-auditor split, why is the engineer-authored hash load-bearing? (The signed artefact is a specific version of the report; the hash in Section 12 pins *which version* was audited. Without it, if the report is edited after sign-off (even for a typo), the signature's referent is ambiguous — did the auditor sign the pre-edit or the post-edit version? The engineer-authored hash forecloses this ambiguity and is what the external auditor cites in their signature. The chapter's framing is that mod-106 output must be "engineered for external audit consumption" — hashed manifests are the specific mechanism.)
4. The lab's compliance clock flags the next-audit-due date as `2026-11-20`, one day before the audit date `2026-11-21` — i.e., the audit is overdue by one day. A reviewer argues this should be silently corrected by backdating the audit date to 2026-11-20. Per chapter 06, why is backdating specifically wrong, and what is the correct handling? (Backdating an audit falsifies the evidence record; the chapter's anti-pattern "Annual re-audit missed" names tracking the clock as a compliance obligation, and the correct handling when the clock has slipped is to flag it explicitly — in the compliance-clock document and in the submission narrative — and to describe the remediation plan to be back in cycle. A one-day overdue flag plus a dated remediation plan is defensible; a backdated audit is fraud. The L15 analyst needs the true state to file an accurate submission.)

## Why this lab matters

Exercise 06 asks you to package the real-system mod-106 outputs from exercises 01–05 into a signable LL144 report, an EEOC adverse-impact evaluation, a public summary, a roles memo, a compliance clock, and a reviewer's checklist — all in two hours, after the technical exercises have consumed the rest of the module. If the packaging step consumes most of the budget because you are re-deriving the 13-section LL144 skeleton, the seven-field EEOC shape, the public-summary content list, the L15 / L25 / independent-auditor split memo, and the 15-item checklist from scratch, the exercise blows its budget and the evidence bundle's hash manifest gets skimmed. This lab drills the atomic packaging skill on the fabricated numbers from labs 01 / 02 so exercise 06 is *integrating the real numbers into the shape*, not *inventing the shape*. Together, labs 01–03 cover the three atomic authoring skills exercises 01 / 02 / 06 depend on — the shape-work that frees exercise time for the engineering judgement that is the actual deliverable.
