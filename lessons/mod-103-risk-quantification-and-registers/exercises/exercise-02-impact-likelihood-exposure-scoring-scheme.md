# Exercise 02 — Impact × Likelihood × Exposure Scoring Scheme

**Estimated effort:** 3 hours

## Objective

Author, publish, and version a **complete scoring scheme** — per-axis rubrics, combination matrix or formula, evidence-anchor requirements, and appetite-band decision rules — and re-score the three rows from exercise 01 against it. The output is a first-order artifact that a reviewer can quote from when they disagree with a row's score.

## Prerequisites

- Chapters 01 and 02 read.
- Exercise 01 completed — the three rows carry through.
- Access to at least one enterprise ERM heat map or scoring guideline from your organisation, if any exists. If none, use a public reference (COSO ERM, ISO 31000 examples, or a bank-supervisor risk-appetite example) as the anchor for the *shape* of the matrix and note this on the deliverable.
- Access to your ISO/IEC 25059 quality-profile draft and NIST AI RMF MEASURE sub-category list for the system (produced in mod-101 exercises or chapter 04 later in this module — a first cut is enough here).

## Problem statement

Produce a scoring-scheme document that, at minimum, publishes:

1. **Per-axis rubrics** for impact, likelihood, and exposure — each at 5 levels (Negligible / Minor / Moderate / Major / Severe, or 1..5). Each level has a working definition specific to *your* system's domain, not a generic one; each level cites the *types of anchor evidence* that would justify assigning it (chapter 02's evidence table).
2. **The combination rule** — either an ordinal matrix (5 × 5 × 5 → 1..5 combined) *published in full* or an anchored numeric formula with the anchor definitions explicit. Whichever you pick, name why over the alternative for your organisation's ERM context.
3. **Observation window and unit** for likelihood (annualised, per-release, per-session, etc.) — pick one and apply it consistently.
4. **Evidence-anchor requirements** — the quality gate that says "an axis level without a cited anchor is a draft, not a scored row."
5. **Appetite-band decision rules** — the residual-score-to-decision mapping (within appetite / exceeds appetite / requires escalation) tied to your enterprise band if known, otherwise a defensible working default.
6. **Rubric versioning discipline** — the version identifier (e.g., `v0.1-<system>-<yyyy-qN>`), the change-log format, and the trigger events that force a version bump.

Then **re-score the three rows from exercise 01** against the published scheme. Each re-scored row cites the rubric-level language it matches on each axis and the anchor evidence that justifies the level.

## Requirements

- Output as `mod-103-ex02-scoring-scheme.md` plus a `rows/` directory of three YAML files, one per re-scored row, matching the shape in chapter 02's worked example (`row_id`, `scoring.rubric_version`, `inherent`, `residual`, `control_defeated`, credited-controls with evidence pointers).
- Each rubric level *definition* is at least one sentence long and names a concrete domain example. "Moderate = medium impact" does not count.
- Every per-axis anchor evidence type is named: for impact, one of the anchor types from chapter 02 (regulatory language, loss study, post-mortem, physical-injury classification, etc.); for likelihood, one of the priors (incident-corpus base rate, red-team ASR, guardrail effectiveness, in-system observation); for exposure, one of the anchors (inventory record, deployment ring, sector materiality tier, downstream-integrator map).
- If you pick the ordinal matrix: publish the *whole* matrix, not a summary. Every triple `(impact, likelihood, exposure)` cell resolves to a combined level. Explain any non-monotonic cells (a combined level that is lower than one of its axes) or state that the matrix is fully monotonic.
- If you pick the anchored numeric formula: publish the anchor definitions (e.g., "impact = expected loss in USD," "likelihood = probability per annum," "exposure = user-sessions per annum affected") and the combination expression. Show one worked calculation end-to-end.
- On each re-scored row, likelihood is scored *per state* (inherent likelihood ≠ residual likelihood ≠ control-defeated likelihood); impact is scored once unless the row makes a substantive case for state-dependence; exposure is scored per state where a control (rate limit, ring restriction, tier gate) changes it.
- Every axis level cites an anchor. Rows with anchor-free axes are marked `staleness: uncalibrated` and excluded from portfolio roll-ups (rehearse this exclusion in the exercise-04 aggregation later).
- The scoring scheme carries a `version` field and a short change-log entry for this version. Even at v0.1, the discipline is written down.

## Starter guidance

- Start from the *impact* rubric. It is the axis most likely to be under-specified in a first-cut register and the axis most likely to be argued in review; getting it precise first constrains the other two.
- For likelihood, pick the observation window your organisation already uses on the security-side register — matching windows lets the AI register roll up alongside without translation (chapter 03).
- For exposure, name the sector overlays that touch your system (SR 11-7 model-materiality tier if you are in US FI; EU AI Act "significant" thresholds if EU-deployed; FDA "patient population affected" if medical device) and align at least one exposure level to the overlay's language.
- On the ordinal matrix, be honest about non-monotonicities. It is fine to say "Contained exposure × Severe impact × Almost-certain likelihood is combined-High rather than combined-Severe because the containment defensibly caps the harm reach" — but *say it*, don't hide it in a cell.
- On the anchored numeric formula, resist the urge for false precision. If you cannot estimate impact in dollars for a scenario, do not use the formula for that scenario; fall back to the ordinal matrix. Publishing a fake dollar figure is worse than admitting you cannot anchor.
- Version the scheme with the framework version it cites. If NIST AI RMF or ISO/IEC 25059 revises, the scheme version bumps and old rows either re-score or are annotated `scored-against: v0.1`.
- The re-scored rows *may* land at different combined levels than exercise 01. That is a feature — you now have an explicit reason for each level and a defensible reviewer conversation.

## Acceptance criteria

The submission is accepted when:

- Three per-axis rubrics are published, each with 5 levels × definition + anchor-evidence type.
- The combination rule is published in full (matrix or formula), with the choice justified.
- The observation window and unit are named and applied consistently.
- Evidence-anchor requirements are stated and enforced (uncalibrated rows are flagged).
- Appetite-band decision rules are stated.
- The rubric carries a version and a change-log entry.
- The three exercise-01 rows are re-scored against the scheme; each axis level cites both the matching rubric language and the anchor evidence.
- A reviewer can pick any axis level on any re-scored row, look up the rubric, and either agree with the level or argue against it with a citation.

## Stretch goals

- Draft a *second* rubric variant sized for a different deployment (a low-stakes internal tool vs. the high-stakes external system you used above). Show the diff. This is what tells you which rubric levels are portable and which are system-specific.
- Sketch the *transition* mechanic — when the rubric version bumps from v0.1 to v0.2, what happens to already-scored rows? Do they re-score automatically? Do they carry a "scored under previous version" marker for one review cycle? This is the operating detail chapter 06 will pick up.
- Cross-reference each rubric level to a sector overlay's language (SR 11-7 tier, FDA GMLP severity, EU AI Act risk category) where applicable. Rows in scope for that sector gain a sector-language citation on the axis.
- Compare your matrix visually against a public enterprise ERM heat map (COSO ERM example, a bank supervisor guidance illustration) and note the places your matrix deviates and why.

## Deliverable format

A single markdown document with the following sections, plus the three YAML files under `rows/`:

1. *Scheme version and change-log* — `v0.1-<system>-<yyyy-qN>`, one-line entry.
2. *Impact rubric* — table of 5 levels × definition × anchor-evidence types.
3. *Likelihood rubric* — table with the observation window and unit named at the top.
4. *Exposure rubric* — table with sector-overlay alignment notes where applicable.
5. *Combination rule* — full matrix or formula, plus one worked example.
6. *Evidence-anchor requirements* — the quality-gate rule.
7. *Appetite-band decision rules* — within / exceeds / escalates and the residual thresholds.
8. *Re-scored rows* — one section per row, referencing the YAML file, with per-axis rubric-level and anchor citations inline.
9. *Deviations noted* — any place the scheme intentionally departs from the enterprise ERM's shape and why.

## Submission

Attach the markdown and the YAML files to the module submission channel. This scheme is the artifact exercises 03-05 will operate against; version it thoughtfully because you will be citing it repeatedly.
