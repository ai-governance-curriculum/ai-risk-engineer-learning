# Aggregation Across Systems and Portfolios; Calibrating Against Risk Appetite

A single system's risk register is where the row-level scoring happens. A *portfolio* view is where the risk council makes decisions. Between the two sits a small but important set of rules: how row-level scores compose into system-level and portfolio-level numbers, and how those numbers are read against the enterprise's risk appetite. Getting the aggregation wrong is how a register full of well-scored rows still produces a portfolio report the risk council cannot use.

## Motivation

Three failure modes at the aggregation layer:

- **Sum-of-averages fallacies.** The team averages a system's row scores and reports the average. A system with fifty medium rows and one severe row averages to "medium." The severe row is invisible.
- **Double-counting.** Cross-system shared components (the same LLM vendor, the same retrieval store, the same evaluation harness) appear as separate register rows in every system, and their portfolio contribution is counted N times.
- **Appetite bands without a band definition.** The report says "10 rows exceed appetite" but the appetite band is prose, not a threshold. The risk council can't act.

The engineering fix is to spell out the aggregation rules, publish the appetite bands, and produce reports the risk council actually reads.

## Aggregation levels

Three levels of aggregation, each with its own rules:

| Level | Unit | Consumer | Aggregation function |
|---|---|---|---|
| Row | One scenario | System owner; register author | n/a — score is per row |
| System | All rows for one system | System owner; deployment-tier decision | worst-of-top-N per state; per-quality-axis roll-up |
| Portfolio | All systems in scope | Risk council; enterprise ERM | grouped by shared component; grouped by regulated jurisdiction; grouped by ISO/IEC 25059 characteristic and by NIST MEASURE sub-category |

Each level publishes a different report. The rules below say what "aggregate" means at each.

## System-level aggregation

A system's aggregated risk is not the mean of its rows. It is a *profile* that the deployment-tier decision reads. At minimum:

- **Max residual by 25059 characteristic** — for each quality characteristic in the system's profile (chapter 04), the worst residual score among rows tagged with that characteristic. This is what tells the system owner "you have an unresolved robustness problem" vs. "you have an unresolved bias problem."
- **Count of rows above appetite** — how many rows have residual outside the appetite band. Not "average distance"; count.
- **Worst control-defeated delta** — the largest gap between control-defeated and residual across the system. This is what tells the monitoring plan where the priority sits.
- **Top-5 rows by residual, listed** — the risk council reads the top of the tail; do not summarise it away.
- **Coverage grid completeness** — cells of the 25059 × MEASURE grid (chapter 04) that are empty. Empty cells mean either "not applicable, marked so" or "coverage gap, owed."

Reporting a system as "medium risk overall" is not a useful sentence. Reporting a system as "top residual = HR-CS-INJ-01 at High; 3 rows above appetite; worst control-defeated delta = +2 levels on CTL-CS-RAG-ALLOWLIST-01" is.

## Portfolio-level aggregation

The portfolio view answers different questions than the system view. Roll-up rules must be picked deliberately per report because the shape of the question changes.

### Group by shared component

Ideal for supply-chain reporting (mod-110). Every register row that names a shared component — an LLM vendor, a retrieval store, an evaluation harness, a guardrail model — contributes to a *component-level* aggregation. This surfaces the fact that a single vendor's degradation affects N systems.

Rules to apply:

- **Deduplicate.** A component that appears in fifty rows across ten systems is one component, not fifty. Roll up by *component identifier*, not by row.
- **Report the residual of the highest-scoring row that names it,** plus the count of systems affected. Do not average.
- **Cite the shared control(s) the component sits behind.** If a single control mitigates the component across systems, that control's failure has *portfolio-scale* blast radius.

### Group by regulated jurisdiction

Ideal for compliance reporting. Every row whose exposure axis reaches a regulated jurisdiction is grouped by the jurisdiction (EU, US-federal, US-Colorado, US-NY, UK, sector-specific).

Rules to apply:

- **Filter by jurisdiction.** A row not exposed to a jurisdiction does not roll into that jurisdiction's report.
- **Report the count of above-appetite rows per jurisdiction,** with the top three by residual named.
- **Cite the applicable article / provision** from mod-101 ch 04 / ch 05. This is what makes the report *filable* rather than internal-only.

### Group by 25059 characteristic and by NIST MEASURE sub-category

Ideal for the trust-and-safety report. Every row is grouped by the quality characteristic (25059) and the MEASURE sub-category it primarily attaches to (chapter 04).

Rules to apply:

- **Report the count and top-3 residual per characteristic.** This is what tells the program "we are heavy on security-and-robustness and light on transparency-and-controllability."
- **Compare against the coverage grid.** A characteristic with zero above-appetite rows and zero rows total is a *coverage gap*, not a good result.
- **Cite the MEASURE evidence pipeline** for each grouped set — this links portfolio reporting back to the mod-101 ch 01 five-column translation.

### Group by system-tier and by deployment ring

Ideal for release-gate reporting. Systems are grouped by their deployment tier (mod-102 ch 04 tiering) and by deployment ring (canary / limited / broad).

Rules to apply:

- **Report per-tier count of above-appetite rows.** Systems at higher tiers should have fewer, not more.
- **Flag any above-appetite row at a broad-deployment ring** — this is the hot list.

## The double-counting problem

Portfolio aggregation without a de-duplication rule inflates numbers. Two dedup strategies:

- **Component-key dedup.** Each row cites the shared components it *depends on*. The portfolio report keys off `(component_id, scenario_class)` — one row per distinct combination. If the same LLM vendor and the same prompt-injection scenario appear across ten systems, that pair is one portfolio-line, not ten.
- **Cross-register reconciliation with 27005 (chapter 03).** For rows that also appear on the security-side 27005 register (e.g., a supplier row that is registered on both sides), portfolio roll-ups from either register cite the shared row-ID and count once. Chapter 03's schema shows the fields to make this happen.

## Calibrating against enterprise risk appetite

Risk appetite is set at the enterprise level — the risk council owns the definition, and the language ("we accept risks up to Medium; risks above Medium must be treated or escalated") is a program-level artifact. Your job at level-25 is to *use* the appetite bands, not to author them.

### The appetite structure

An appetite statement usable by a mod-103 register has this minimum shape:

| Band | Definition (working) | Register decision rule |
|---|---|---|
| Within appetite | Combined residual score ≤ X | Register row is filed; no escalation; monitor and review at standard cadence |
| Exceeds appetite | Combined residual score > X | Row is a treatment-plan target; escalation to the risk council if unresolved by the treatment due date |
| Requires escalation | Combined residual score ≥ Y (well above appetite) or control-defeated ≥ Y or systemic exposure | Row is escalated immediately to the risk council for tolerance decision or deployment halt |

The specific X and Y values are enterprise-level and vary. The pattern is: the register carries the decision rule *inline* so a reviewer can answer whether a row triggers escalation without reading a memo.

### Two views: residual vs. control-defeated

Read appetite against two states:

- **Residual against appetite** — is the system's steady-state risk inside the band?
- **Control-defeated against appetite** — does any single control's failure push the row outside the band?

A row inside appetite on residual but outside on a control-defeated case is a *monitoring priority*, not a treatment priority. The control-defeated appetite check is what makes the monitoring plan (mod-109) targeted rather than uniform.

### Sector-overlay appetite

Some sectors set their own appetite equivalents:

- **SR 11-7** materiality tiering effectively sets appetite for model risk in US financial institutions — high-materiality models have tighter tolerances.
- **FDA GMLP / PCCP** effectively sets appetite for medical-device change control — post-market surveillance thresholds are the appetite proxy.
- **EU AI Act (Article 9, risk-management system)** requires that residual risks after mitigation be *acceptable*, and (Article 15) that certain risks be reduced *as low as reasonably achievable* — the appetite conversation is framed by the article's language.

For each sector obligation your register spans, cite the sector-specific appetite equivalent alongside the enterprise band on affected rows.

## Aggregation reports the risk council actually reads

A monthly / quarterly risk-council packet in this shape covers most needs. Adjust the report set to your program's rhythm.

### 1. Portfolio residual heat map

Rows aggregated to the (system, top-scenario) grid. Colour by max residual per system. Top three above-appetite rows named. Delta from last period shown. Systems whose residual moved *toward* appetite (improvement) shown alongside systems whose residual moved *away* (regression). One page.

### 2. Above-appetite exception list

Every row whose residual exceeds appetite. Fields: row-id, system, residual, appetite band, treatment status, treatment due date, escalation status. Sorted by residual descending. Rows past their due date flagged in a first column.

### 3. Control-defeated priority list

Every named control whose defeat would push at least one row outside appetite. Fields: control-id, systems it covers, worst control-defeated delta, monitor-ref, monitor status. Sorted by control-defeated delta descending. This is what tells the monitoring team where to invest.

### 4. Shared-component report

Rolled up by shared component (LLM vendor, retrieval store, evaluation harness, guardrail model). Fields: component-id, number of systems affected, worst residual, incident-corpus references, supplier obligations. This is what tells the supply-chain reviewer where the concentration sits.

### 5. Coverage-grid completeness report

25059 characteristics × MEASURE sub-categories, count of rows per cell, gaps flagged. Fields: characteristic, sub-category, count, gap-owner. This is what tells the program where the coverage is thin.

### 6. Regulatory-jurisdiction exception list

Rows grouped by regulated jurisdiction; above-appetite counts per jurisdiction. This is what feeds regulator-facing reporting (EU AI Act Article 61 / Article 72, SR 11-7 model-risk reports, FDA post-market surveillance).

## A working aggregation query

If the register is materialised as YAML per row or as a structured GRC-platform export (chapter 06), a portfolio report is a straightforward reduction. Illustrative pseudocode:

```python
# Above-appetite exception list
above_appetite = [
    row for row in load_all_rows()
    if row["residual"]["combined"] > appetite_band_upper(row["system_id"])
]
above_appetite.sort(key=lambda r: r["residual"]["combined"], reverse=True)

# Shared-component roll-up
by_component = {}
for row in load_all_rows():
    for comp in row.get("depends_on_components", []):
        by_component.setdefault(comp, []).append(row)
component_report = [
    {
        "component_id": comp,
        "systems_affected": len({r["system_id"] for r in rows}),
        "worst_residual": max(r["residual"]["combined"] for r in rows),
        "row_refs": [r["row_id"] for r in rows],
    }
    for comp, rows in by_component.items()
]

# Coverage-grid completeness
coverage = {(c, m): 0 for c in iso_25059_characteristics for m in nist_measure_subcats}
for row in load_all_rows():
    for c in row.get("iso_25059_characteristics", {}).get("all", []):
        for m in row.get("nist_ai_rmf_measure", {}).get("all", []):
            coverage[(c, m)] += 1
gaps = [(c, m) for (c, m), n in coverage.items() if n == 0 and applicable(c, m)]
```

The exact schema shifts to fit your GRC platform; the pattern is that a report is a stateless reduction over the row set. If your register cannot answer these questions programmatically, it will not scale past a few systems.

## Change-control on the aggregation

Every aggregation rule is versioned. When a rule changes — the appetite band shifts, dedup changes, a new grouping is added — the report cites both the old and new rule for the reporting cycle in which the change lands. Silent rule changes look like risk moving in the report when nothing about the underlying systems has changed, and it destroys trust in the report.

## Anti-patterns

- **Averaging.** A portfolio "average residual" is meaningless; the tail is where decisions live.
- **Report without dedup.** Same shared component counted many times.
- **Appetite as prose.** "We aim for prudent risk" is not a threshold.
- **One-report-fits-all.** A single "risk report" that tries to serve the risk council, the supplier reviewer, the regulator, and the release-gate reader satisfies none of them. Publish the six above.
- **Portfolio report authored on Fridays.** A stateless reduction from the register file should not take a day of manual work. Automate the reduction; if it isn't automated, either your schema is wrong or your register data quality is unreliable.

## Summary

- Aggregation happens at three levels — row, system, portfolio — each with its own rules.
- System-level aggregation is a profile (worst-by-characteristic, count above appetite, worst control-defeated delta, top-5 rows) not an average.
- Portfolio-level aggregation is *multiple* reports grouped differently — shared component, jurisdiction, quality characteristic × MEASURE sub-category, deployment tier and ring.
- De-duplicate on shared components and cross-register with the 27005 security-side register (chapter 03) to avoid inflated counts.
- Risk appetite is an *enterprise* band set outside your role; the register carries the decision rule inline so a reviewer can act on it.
- Publish six standard reports (heat map, above-appetite, control-defeated priority, shared-component, coverage-grid, jurisdiction exception) and version the aggregation rules that produce them.
- Exercise 04 walks the aggregation on a multi-system scenario.
