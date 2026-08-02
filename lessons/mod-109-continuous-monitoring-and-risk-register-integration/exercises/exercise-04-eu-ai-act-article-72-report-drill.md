# Exercise 04 — EU AI Act Article 72 Report Drill

**Estimated effort:** 3 hours

## Objective

Generate a full **EU AI Act Article 72 post-market monitoring report** for a high-risk system, produced as a **reduction over the register plus the tripped-signal records shipped in exercises 02 and 03** — not as a hand-composed narrative. Dovetail the report with **Article 73 serious-incident reporting**. Bind the report to a versioned **Article 61 / 72 post-market monitoring plan** so the plan and the report are one artifact viewed two ways.

## Prerequisites

- Chapter 04 read in full. The article table (61 / 72 / 73), the ten-section report shape, the "generate from register + monitors" reduction, and the FDA SaMD sector variant are the reference shapes.
- Exercises 01, 02, and 03 completed. The signal contract, the detectors, and the tripped-signal records exist and are populated with either real production data or a defensible replay of a reporting period.
- The mod-103 register with residual-state history for every row in scope for the covered high-risk system.
- The **mod-102 harm-model** for the covered system, and the **mod-101 EU AI Act translation memo** (mod-101 chapter 04) naming the Chapter III obligations the system is subject to.
- A **reporting period** to bind the exercise. A single calendar quarter (13 weeks) is the default. If the system was not in production for the full quarter, a shorter period is fine — call the period out in section 1 of the report.
- A pointer to the classification of the running system as *high-risk* under Annex III (or the equivalent sector variant). If the system is not high-risk, run the exercise against a hypothetical high-risk classification and mark the report accordingly for the exercise; the shape is the same and the audit-defence discipline is the same.
- Access to the **incident register from mod-111** (or a stubbed placeholder if mod-111 has not been reached yet). Section 5 and section 9 of the report both read it.

## Problem statement

Produce, together, the Article 61 / 72 plan and the Article 72 report against that plan, both machine-generatable from the register + monitor pipeline. The report has to defend itself in two ways:

- Every datum in the report traces back to a tripped-signal record, a register-row residual-state entry, or an incident-register entry — no free-text metrics.
- Re-running the reducer on the same reporting period, with the same underlying data, produces the same report — no manual composition.

### The plan

Chapter 04 §"The plan and the report — one artifact, two views" pins the plan's minimum content. For this exercise, the plan is a versioned markdown or YAML file with the sections listed there:

- System scope (mod-102 intake identifier).
- Chapter III obligations in scope for the system (referencing the mod-101 ch 04 memo).
- Data collected (telemetry, deployer feedback, incident reports, complaints).
- Metrics computed (list of signal contract row identifiers).
- Thresholds and pass/fail criteria (linked to the signal contract).
- Corrective-action process, timelines, responsibilities.
- Reporting cadence (internal to the governance council; external to authorities).
- Retention and storage.
- Deployer-side interface.

The plan is signed off by the same roles as the signal contract (chapter 01) plus, for a high-risk system, the provider legal / compliance lead.

### The report

Chapter 04 §"The report shape" lists the ten sections; the report file follows that order exactly. Every section is a reduction:

1. Scope — read from the plan and the system-version log.
2. Data collected — read from the trace store + tripped-signal record store.
3. Monitoring signals and thresholds — read from the signal contract file (chapter 01) at the plan's pinned version.
4. Observations for the period — read from the tripped-signal records + register residual-state history.
5. Incidents and near-misses — read from the mod-111 incident register + the near-miss counter (exercise 03).
6. Continuous-compliance evaluation — a per-Chapter-III-obligation view over the register rows.
7. Corrective action plan — every open treatment flag on the register whose row is in scope.
8. Changes to the plan — read from the plan's own change log.
9. Article 73 wire — every serious-incident notification for the period, keyed to the incident register and to the outbound notification record.
10. Evidence attachments — the evidence-manifest hashes plus the model / dataset / guardrail / detector versions in production during the period.

### The reducer

The report is produced by a script — the *reducer* — that consumes the artifacts and emits the report file. Re-running the reducer on the same period and the same underlying artifact hashes produces byte-identical (or hash-identical) output.

## Requirements

Produce all seven items:

1. **Article 61 / 72 plan file** — `risk/mod-109/<system-id>/pms-plan/plan-vX.md` (or `.yaml`). Sections per chapter 04 §"The plan and the report." Signed off with a `sign_off` block naming row author, system owner, control owner, and provider legal / compliance lead. Change log accompanies the plan.
2. **Reducer script** — `risk/mod-109/<system-id>/pms-plan/reduce_report.py`. Reads the plan, the signal contract file, the tripped-signal record store, the register file, and the incident register, and emits `report-<period>.md`. Deterministic: re-runs produce hash-identical output; the reducer emits its input-hash manifest alongside the report.
3. **Report for the reporting period** — `risk/mod-109/<system-id>/pms-plan/report-<period>.md`. All ten sections populated from the reducer's output. No free-text metrics; every number cites a tripped-signal record, a residual entry, or an incident-register entry.
4. **Section-4 worked excerpt** — inlined in the report, with three to five per-row bullet points in the chapter-04 §"A worked report-section-4 excerpt" style. Each bullet cites the underlying `contract_id`, the count of trips, the sustained-trip statistics, and the register-row effect.
5. **Article 73 wire log** — `risk/mod-109/<system-id>/pms-plan/article-73-log.md`. Every S1 tripped signal for the period is walked through the wire: monitor → incident candidate → triage decision (`serious` / `not serious` / `investigation`) → notification (if `serious`) → close. Turnaround times per notification against the Article 73 deadline requirements. If no S1 fired during the period, run a **rehearsal wire drill** on a synthetic S1 and record the drill's turnaround; chapter 04 §"Anti-patterns" pins the "Article 73 wire that only runs in fire drills" failure mode by inversion — the wire has to be rehearsed even when the period is clean.
6. **Continuous-compliance evaluation table** — `risk/mod-109/<system-id>/pms-plan/chapter-iii-compliance.md`. Per Chapter III obligation from the mod-101 ch 04 memo (risk-management, data governance, technical documentation, record-keeping, transparency, human oversight, accuracy / robustness / cybersecurity), a cell naming whether the period's data supports continued compliance and citing the register rows and signals that support the statement. Any obligation with an unresolved compliance question is called out with the CAP pointer.
7. **Sector variant note** — `risk/mod-109/<system-id>/pms-plan/sector-variant.md`. One page: if the system is medical-device / SaMD, dovetails the report with 21 CFR Part 822 post-market surveillance, PCCP, and GMLP; if the system is finance-sector, dovetails with SR 11-7 (mod-110) and any sector-specific incident-reporting duty; if the system is not sector-regulated, one sentence stating so and citing the mod-101 chapter 04 mapping.

Cross-cutting rules:

- **No manual composition.** Every datum is emitted by the reducer. Narrative in the report is limited to section-title framing sentences ("this section covers the drift and quality signals for the period"); metrics are all reducer output.
- **Version everything.** The plan has a semantic version; the report cites the plan version. The signal contract has a semantic version; the report cites that too. The reducer's input-hash manifest is committed alongside the report; if any input hash changes, the report is stale and re-run.
- **Article 3(49) determination is not yours.** The risk engineer's wire runs from S1 → incident candidate → triage. The triage decision on whether the incident is *serious* under Article 3(49) is a legal / compliance call. The Article 73 log records the triage decision and the timestamp; it does not pre-suppose the determination.
- **Rehearsal counts.** Chapter 04 §"Anti-patterns" pins that the Article 73 wire must be rehearsed on every S1 whether or not the incident is serious, and must be rehearsed on a synthetic drill when the period is clean. The Article 73 log captures rehearsals as first-class entries.
- **Section 10 evidence hashes are load-bearing.** The tripped-signal record store is the evidence chain; if a record's hash is not in the section-10 manifest, the reducer flagged the record as invalid and the report is incomplete. Fix the record store, do not paper over.
- **needs-research markers survive from the chapters.** Chapter 04 carries two `needs-research` markers (Commission implementing act status; FDA SaMD updates). Copy those markers into the plan and the sector-variant note; do not invent an authoritative answer. The point of the marker is that auto-merge is blocked until a human confirms; the discipline is the same in the exercise's artifacts.

## Starter guidance

- **Read Articles 61, 72, 73 from the Official Journal, not from a vendor brief.** The engineering reading in chapter 04 is faithful, but auditors read the actual text; the `resources.md` link is the anchor.
- **The plan is short.** A defensible plan is a two-to-four-page document; a plan that runs to 30 pages of narrative is a red flag — the audit will read it and find gaps by exhaustion. Keep the plan tight and let the report do the observational heavy lifting.
- **The reducer is a pipe, not an application.** It reads files, projects fields, aggregates counts, formats markdown. Do not fold in an LLM to "write the report from the data" — that is exactly the "manual composition" anti-pattern in a new coat. A deterministic reducer is what an auditor can trust.
- **Pick the reporting period before you touch the reducer.** The period bounds every read; a period that overlaps a plan-version change is fine but has to be handled in the reducer (which rows fall under which plan version). Start with a period that sits inside one plan version to keep the exercise tractable.
- **Rehearse the Article 73 wire on a synthetic S1 first.** Chapter 04 §"Dovetail with Article 73" pins the five-step wire. Rehearse it end-to-end (monitor → incident candidate → triage → drafted notification → close) with a synthetic payload before you have to do it for real. The rehearsal is the log entry; the drill is not "wasted work."
- **Section 6 (continuous compliance) needs the mod-101 ch 04 memo.** Do not re-derive the Chapter III obligations for the exercise; read them from the memo and cite it. Any obligation the memo does not cover is a mod-101 gap to file, not a mod-109 gap to fill.
- **Section 7 (CAP) reads directly off the register's treatment flags.** Every open treatment flag on a row in scope is a CAP entry. If the register has a row with `residual: above appetite` and no treatment flag, that is a register-side bug — file it against the row before shipping the report.
- **Section 10 evidence hashes are what the reducer emits.** The reducer computes the hashes as it reads the records; it does not read them from a separate file. This is what makes the report reproducible — hashes are computed in-band.
- **If the running system is not high-risk, mark the exercise clearly.** Under the AI Act's risk-tier structure, non-high-risk systems have lighter monitoring obligations. Running the exercise anyway is fine — the shape and the discipline generalise — but the report file's section 1 must call out the exercise-hypothetical framing so a later reader does not mistake it for a real filing.

## Acceptance criteria

The submission is accepted when:

- The plan and the report exist, are versioned, and are cross-referenced (plan version cited in the report; report cited in the plan's change log).
- The reducer runs deterministically: two runs on the same inputs produce byte-identical (or hash-identical) output; a diff of a single tripped-signal record produces a targeted diff of the report.
- Every one of the ten report sections is populated by the reducer with no free-text metrics.
- Section 4 has a worked per-row excerpt in the chapter-04 shape.
- Section 5 lists every incident-register entry for the period, with type / dates / affected users / corrective action / closure state.
- Section 6 has a table of Chapter III obligations from the mod-101 memo, each with a compliance state and register-row citations.
- Section 7 (CAP) has an entry for every open treatment flag whose row is in scope.
- Section 9 (Article 73) has an entry for every S1 for the period, and a rehearsal entry if no real S1 fired.
- Section 10 (evidence) has hashes for every underlying record.
- `needs-research` markers from chapter 04 are copied into the plan and the sector-variant note.
- The sector-variant note dovetails with the relevant sector regulator or writes down that the system is unregulated at sector level.

## Stretch goals

- **Plan-diff-driven report re-run.** Wire the reducer to detect that a new plan version has been published mid-period and re-emit the report split across the two plan versions. The section-8 (changes to the plan) block captures the mid-period bump.
- **Machine-readable submission format.** Chapter 04 carries a `needs-research` marker for the Commission implementing act's submission format. If the act has been adopted (verify against the `resources.md` link at authoring time), emit the report additionally in that format. If not adopted, emit in a defensible JSON / YAML schema that carries every section's data, so the future format-switch is a formatter change and not a re-composition.
- **FDA SaMD dovetail (if applicable).** For a medical-device system, produce a companion post-market surveillance report in the 21 CFR Part 822 shape; wire the Article 73 log to the FDA MedWatch reporting equivalent; and cite the GMLP guiding principles in the sector-variant note.
- **Deployer-feedback ingest.** Real Article 61 monitoring includes deployer-side feedback. Ship a simple deployer-feedback intake shape (structured form or ticket-store integration) that emits entries into the same tripped-signal record store, tagged as `source: deployer`. Section 2 of the report picks them up automatically.
- **Delta report against the previous period.** For a running system with a prior period on file, add a section-4b "delta vs. previous period" produced by the reducer — trip-count deltas per contract row, residual-state movement deltas per register row, and new / closed incidents. The delta is what the governance council reads.
- **Live-dashboard view of the report.** In addition to the markdown report, emit a static HTML view (or a link to a Grafana dashboard whose data source is the same reducer output) that carries the report at read-only. This closes the "one source of truth" seam with exercise 05 by extending it to the regulatory audience.

## Deliverable format

A single markdown document `mod-109-ex04-article-72-report-drill.md` that indexes the artifacts. The index contains:

1. *Covered system and period* — one paragraph naming the system, its high-risk classification (real or hypothetical), and the reporting period.
2. *Plan pointer and version* — the plan file, the version, and the sign-off state.
3. *Reducer pointer and input-hash manifest* — the reducer path plus the manifest hashes.
4. *Report pointer* — the report file with a table-of-contents excerpt.
5. *Section-4 worked excerpt* — inlined verbatim.
6. *Continuous-compliance table* — inlined verbatim.
7. *Article 73 log* — every entry for the period plus any rehearsal drills.
8. *Sector variant note* — inlined verbatim.
9. *Anti-pattern probe* — one sentence per chapter-04 failure mode confirming resolution ("no narrative report"; "plan cites signal identifiers"; "Article 73 wire rehearsed"; "plan version matches report").
10. *Open items* — `needs-research` markers still open; deployer-feedback intake pending; delta-report or machine-format work deferred; any mod-111 hookups still stubbed.

## Submission

Attach the index markdown plus the seven artifacts. The plan and the report are the two documents that a market-surveillance inspector would read on an EU AI Act inspection; the reducer and its input-hash manifest are what makes the report defensible under "where does this number come from." Treat all four (plan + reducer + report + manifest) as one submission — none of them stands alone.
