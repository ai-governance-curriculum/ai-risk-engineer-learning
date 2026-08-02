# Exercise 01 — Inherent vs. Residual vs. Control-Defeated Drill

**Estimated effort:** 2 hours

## Objective

Take three concrete harm-model rows from your mod-102 catalog and score each across **all three risk states** — inherent, residual, and control-defeated — with the deltas that argue with an auditor made visible on the row. The output is the first defensible three-state slice of the register you will operate through the rest of this module.

## Prerequisites

- Chapter 01 read.
- Chapter 02 skimmed — you will use its axes informally here; exercise 02 formalises the rubric.
- Access to your mod-102 harm-catalog rows (exercise 02 in mod-102). If you did not carry forward those rows, pick three from the `CarePilot` (hospital-triage support) or `HR-CS-*` (customer-support RAG) reference catalogs used in the chapter examples.
- Access to whatever control-catalogue documentation exists for the running system — configuration, guardrail-inventory, monitor-list, deployment topology. If none exists yet, note that as a scoping gap on the deliverable.

## Problem statement

Pick **three** mod-102 harm-catalog rows and produce, for each, a table of the following shape (this is the same shape as chapter 01's worked example on `HR-CS-INJ-01`):

| Field | Inherent | Residual | Control-defeated: `<control-A>` | Control-defeated: `<control-B>` |
|---|---|---|---|---|
| Impact | | | | |
| Likelihood | | | | |
| Exposure | | | | |
| Combined | | | | |
| Credited controls | | | | |
| Monitor(s) that catch the defeat | n/a | n/a | | |
| Evidence pointer (per credited control) | | | | |

The three chosen rows should be *materially different*. A good working set:

- One row where the residual is comfortably inside appetite and the drama lives in the control-defeated column.
- One row where the residual sits close to the appetite band and the control-defeated column pushes clearly outside.
- One row where even inherent-with-treatment-proposed sits at or above appetite — a row whose treatment plan may not land you in appetite.

## Requirements

- Output as `mod-103-ex01-three-states.md`.
- Three rows × three states scored, with at least *two* control-defeated variants per row (one control at a time; do not collapse multiple failing controls into one column — that is inherent).
- On each row, publish the two deltas explicitly:
  - `inherent − residual` (how much the credited controls buy).
  - `worst control-defeated − residual` (blast radius of the biggest single-control failure).
- Every credited control on residual carries an evidence pointer to *something that resolves* — a test that runs, a monitor dashboard, a release-config file. A pointer to a Confluence page that says "planned" does not count.
- The list of controls *not credited* under inherent is written down (chapter 01's "no-controls list, not implied" discipline).
- Every control-defeated variant names the specific bypass mode ("provenance filter tuned to allow rule X for debug session," "allow-list config replaced in release Y," etc.). Generic "the control failed" is not enough.
- On rows whose residual exceeds appetite (assume the appetite band from your enterprise if known; otherwise assume "Medium" is the band ceiling), include a one-sentence treatment plan pointer. On rows whose control-defeated exceeds appetite, include a one-sentence monitor pointer (which monitor catches the defeat, on what cadence).
- Cite the mod-102 row IDs so the traceability is unbroken.

## Starter guidance

- Do inherent first, and do it *cold* — write down the exact list of controls you are stripping. It is very easy to accidentally credit a control on inherent because you are used to seeing it running.
- On residual, list credited controls *before* scoring likelihood — the credit list drives the number, not the other way round.
- For control-defeated, pick controls whose absence would *meaningfully* change the number. Controls that carry a small share of the mitigation can be rolled into a single "minor controls degraded" column or left out; the discipline is that you named the ones that matter.
- Do not invent evidence pointers. If a control's evidence is not yet in place, credit it under residual only with a `staleness: no_evidence` flag or drop it from the credit list. The auditor's version of this exercise reads the evidence pointers.
- If a row's inherent looks *lower* than residual, you scored something wrong — usually you credited a control on inherent by mistake.
- If the delta `inherent − residual` is small on a row, either your controls are ineffective (good finding — file it) or your inherent scoring was too generous (revisit).

## Acceptance criteria

The submission is accepted when:

- Three rows are scored across all three states with the schema above.
- Each row shows the two deltas explicitly.
- Each credited control on residual has a live evidence pointer.
- Each control-defeated variant names the specific bypass mode and (if it exceeds appetite) the monitor that catches the defeat.
- The three rows chosen span the "inside appetite / near appetite / above appetite even with treatment" spread; if all three are inside appetite, either your system has no substantive residual risk (rare and needs justification) or you picked the easy rows (revisit).
- A reviewer with no prior context can read the deliverable and answer, per row: "how much do the controls buy? which single control's failure is the largest problem? does the monitor plan sit against the right control?"

## Stretch goals

- Add a *fourth* row where you deliberately model a *previously credited* control as removed (a real change you know is coming — a monitor being deprecated, a guardrail being downgraded to reduce cost). Show what residual becomes after the change. This is the register operating in change-management mode; it is what mod-109 and mod-110 read.
- Draft a one-paragraph audit-narrative for each row that reads the three-state table into English — "we assess this scenario at X residual under Y credited controls; the largest single-control-failure exposure is Z, monitored by W with a T-window response." That paragraph is the register's living voice; it is what a risk council reads.
- For any row where the treatment plan cannot land you in appetite, draft the escalation memo shape (row-id, current inherent-with-treatment, band exceeded, decision requested: tolerance vs. deployment halt vs. system-change).

## Deliverable format

A single markdown document with the following sections:

1. *Rows chosen* — three-line summary per row: system, scenario, why-representative-of-the-appetite-spread.
2. *Three-state scoring* — one table per row in the shape above.
3. *Deltas* — one line per row: `inherent − residual = X` and `worst control-defeated − residual = Y`.
4. *Coverage notes* — anything you could not score because the control-catalogue or evidence base was missing.
5. *Open questions* — anything you flagged for exercise 02 (rubric formalisation) or for exercises 03-05 (alignment, aggregation, operation).

## Submission

Attach the markdown to the module submission channel. If your running system is under NDA, redact identifiers but preserve the scoring structure, credited-control lists, and evidence pointer types — a reviewer cannot review a three-state table whose deltas are blanked out.
