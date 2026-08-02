# Exercise 04 — Portfolio Aggregation Drill

**Estimated effort:** 2 hours

## Objective

Take a **multi-system portfolio** — at least three AI systems' registers, with at least 20 rows across them — and produce the six standard risk-council reports from chapter 05, with dedup rules enforced and every aggregation choice made explicit. The output is a packet a risk council could read to make deployment, treatment, and escalation decisions.

## Prerequisites

- Chapter 05 read.
- Exercises 01, 02, 03 completed — your running system's register carries through as one of the systems in the portfolio.
- Two additional system registers to fill the portfolio. Options in order of preference:
  1. Two more systems from your organisation (real registers, redacted).
  2. Two systems from prior mod-102 exercises done by peers on the course.
  3. Constructed provisional registers for two reference systems — `CarePilot` (hospital-triage support) and a `RiskScoreCredit` (consumer-credit AI-scoring model) — using the taxonomy tags and scoring shapes from chapters 01-04. Provisional registers must be clearly marked.
- Working knowledge of a small analysis tool (Python + pandas / a spreadsheet with pivot tables / your GRC platform's report builder — any of these). The scale is tiny; tooling choice does not matter.

## Problem statement

Assemble a portfolio dataset of at least 20 rows across at least three systems, with each row carrying:

- `row_id`, `system_id`, `residual.combined`, `control_defeated[*].combined`, `credited_controls[*].id`, `depends_on_components[*]`, `iso_25059_characteristics`, `nist_ai_rmf_measure`, `exposure.jurisdictions[*]`, `deployment_tier`, `deployment_ring`, `appetite_band_upper`, and `treatment_due_date` (if above appetite).

Then produce **all six** reports from chapter 05:

1. **Portfolio residual heat map** — systems × top-scenario grid, coloured by max residual per system, top-3 above-appetite rows named, deltas from a *simulated* prior period noted.
2. **Above-appetite exception list** — every row whose residual exceeds appetite; sorted by residual descending; treatment status and due date on every row.
3. **Control-defeated priority list** — every named control whose defeat pushes at least one row outside appetite; sorted by control-defeated delta descending; each with a `monitor_ref`.
4. **Shared-component report** — rolled up by shared component (LLM vendor, retrieval store, evaluation harness, guardrail model); dedup enforced.
5. **Coverage-grid completeness report** — 25059 characteristics × MEASURE sub-categories; rows-per-cell counted; empty cells classified as *coverage gap* or *not applicable, marked so*.
6. **Regulatory-jurisdiction exception list** — rows grouped by regulated jurisdiction (EU, US-federal, one or more US-state, sector-specific if applicable); above-appetite counts per jurisdiction; top-3 named.

## Requirements

- Output as `mod-103-ex04-portfolio-aggregation.md` plus the raw portfolio dataset as `portfolio-rows.yaml` or `portfolio-rows.csv`, plus the reduction script or spreadsheet formulas that generate the six reports (`build-reports.py` or the workbook file).
- Portfolio ≥ 3 systems × ≥ 20 total rows. At least one system in the portfolio shares a component with at least one other system (so the dedup logic is exercised).
- At least one row exceeds appetite on residual. At least one additional row is *inside* appetite on residual but *outside* on a control-defeated variant (so the control-defeated priority list has content).
- Every report is *auto-generated* from the portfolio dataset. Manual re-typing of numbers into a report is disqualifying — the point is that the reports are a stateless reduction.
- Dedup rules are explicit: the shared-component report keys off `(component_id, scenario_class)` (or your defined equivalent); the same shared component appearing in multiple rows across multiple systems produces exactly one shared-component-report entry.
- Averaging is *not* used anywhere in the six reports. Portfolio numbers are max, count, and top-N — never mean.
- Any row lacking anchor evidence (from exercise 02's quality gate) is *dropped* from portfolio aggregation and shows up on a separate "uncalibrated rows excluded from roll-up" list. That list is a report of its own.
- Every report cites the aggregation-rule version (`v0.1-<yyyy-qN>`) and lists any rule change from a stated prior version (even if the prior is "none — this is v0.1").

## Starter guidance

- Build the portfolio dataset *before* you build any report. Reports are cheap once the rows are structured; rows are the substantive work.
- Reuse the row schema from chapter 03 / exercise 03. Add the aggregation-critical fields (`depends_on_components`, `exposure.jurisdictions`) explicitly if they were not in exercise 03's schema — this is the trigger to add them.
- On the shared-component report, first build the component catalogue (LLM vendors, retrieval stores, evaluation harnesses, guardrail models). Rows depend on components; components do not depend on rows.
- On the coverage-grid completeness report, be honest about "not applicable" cells. It is defensible to say "MEASURE-2.11 (fairness) is not applicable to this internal debugging tool because it makes no consequential decisions about people" — but *say it* on the report; do not silently leave the cell empty.
- The "delta from prior period" on the heat map is a simulated one for this exercise: pick one report you produced (yours from the exercise) as the current period, and hand-construct a plausible prior period one release ago (e.g., "row X was Very High last period, is High now because of the guardrail deployment"). This drills the delta-visualisation muscle without needing two months of real data.
- Treat the uncalibrated-rows exclusion list as a first-class report. Its length is a program-quality signal — if half the register is uncalibrated, aggregation is misleading and the *real* first action is calibration, not portfolio reporting.
- Do not gild the visualisation. Colour, headings, and readability serve the reader; charts and dashboards can wait. A well-shaped markdown table is enough at this exercise.

## Acceptance criteria

The submission is accepted when:

- The portfolio dataset covers ≥ 3 systems × ≥ 20 rows and is a real file, not a table-in-the-doc.
- The reduction script or workbook auto-generates all six reports plus the uncalibrated-rows exclusion list from that dataset.
- At least one row triggers the above-appetite exception list, and at least one triggers the control-defeated priority list without triggering above-appetite on residual.
- The shared-component report deduplicates correctly — the same `(component, scenario)` pair does not appear twice.
- Coverage-grid empty cells are classified as *gap* or *not applicable, marked so*; no cell is silently blank.
- Aggregation-rule version is cited on every report.
- A reviewer reading the packet cold can identify (a) the systems with the biggest steady-state problem, (b) the single control whose failure would matter most across the portfolio, (c) the shared-component concentration, (d) the coverage gap the program should close first, and (e) the jurisdiction that carries the most above-appetite exposure.

## Stretch goals

- Add a *seventh* report: **portfolio treatment burn-down** — count of above-appetite rows by treatment status (in-flight vs. past-due) and by owning team. This is what a program leader reads at a standup.
- Prototype the *aggregation-rule version bump*: change one rule (e.g., dedup by component-id only, versus `(component_id, scenario_class)`) and re-generate the reports; show side-by-side how the numbers change. This drills the change-control discipline chapter 05 warned about.
- Wire in a CI check: any change to the reduction script produces both old-rule and new-rule reports for one reporting cycle. Include the CI configuration snippet.
- Add a *sensitivity* view: for the above-appetite exception list, show how the count changes if the appetite ceiling is shifted by one band up or down. This is what the risk council will ask when they debate appetite changes.
- Cross-reference the shared-component report with the mod-102 supply-chain persona and forward it as a preview of mod-110 concentration risk.

## Deliverable format

A single markdown document, plus the raw dataset and the reduction script/workbook:

1. *Portfolio composition* — the systems in scope, how the rows were sourced (real / provisional).
2. *Aggregation-rule version and diff* — v0.1 for the first release.
3. *Report 1: Portfolio residual heat map* — the table + one paragraph of narrative.
4. *Report 2: Above-appetite exception list* — sorted table.
5. *Report 3: Control-defeated priority list* — sorted table.
6. *Report 4: Shared-component report* — dedup rules stated at the top of the section.
7. *Report 5: Coverage-grid completeness report* — the grid + not-applicable classifications.
8. *Report 6: Regulatory-jurisdiction exception list* — grouped table.
9. *Uncalibrated rows excluded* — the drop-list; length as a program-quality signal.
10. *How to regenerate* — one-line command to reproduce every report from the dataset.

## Submission

Attach the markdown, dataset file, and reduction script/workbook to the module submission channel. This packet is a rehearsal of the artifact a risk council reads; the review should treat it as the first cold read of that packet.
