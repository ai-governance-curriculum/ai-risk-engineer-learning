# Post-Market Surveillance and the EU AI Act Article 72 Report

Chapters 01-03 assembled the signal contract and the signal families. Chapter 04 says what that machinery *produces* for the regulator: a post-market surveillance record and, for high-risk systems under the EU AI Act, an Article 72 post-market monitoring report — plus the Article 73 wire for serious-incident reporting. The point of this chapter is that neither the record nor the report is a separate document composed at year-end. Both are *generated* from the register plus the monitor pipeline, on cadence.

## Motivation

Two failure modes an EU AI Act inspection catches:

- **The report is composed once, at submission time.** Somebody sits down and writes a narrative document referencing screenshots of dashboards; there is no live data model behind the report; the report cannot be regenerated on the next quarter without repeating the composition. The auditor asks *"where does the datum in this paragraph come from"* and the trail dies.
- **The post-market monitoring system is a section of a plan, not a running system.** The Article 61 plan describes what will be monitored, but the actual monitoring is not wired to that plan; monitors exist, but they were built for engineering reasons and no article-level obligation cites them. The report cannot be produced because the source data is not organised against the plan.

The engineering fix is the reverse: build the plan and the monitors as one artifact, and produce the report as a reduction over the register and the monitor pipeline. The report is a *view* over live data, not a bespoke composition.

## What the EU AI Act asks for — the articles you actually cite

Three articles anchor this chapter. Read them from the official Journal (Regulation (EU) 2024/1689), not from a vendor summary.

| Article | What it requires (engineering reading) |
|---|---|
| **Article 61 — Post-market monitoring system** | High-risk-system providers must establish and document a post-market monitoring system that actively and systematically collects, documents, and analyses relevant data provided by deployers, or which they collect themselves, on the performance of high-risk AI systems throughout their lifetime, and allows evaluation of continuous compliance with Chapter III. |
| **Article 72 — Post-market monitoring by providers and post-market monitoring plan for high-risk AI systems** | Providers document the post-market monitoring system in a plan. The plan and its report shape are defined by an implementing act to be adopted by the Commission; in absence of that, the provider defines the plan on defensible principles. |
| **Article 73 — Reporting of serious incidents** | Providers must report serious incidents (as defined in Article 3(49)) to the market-surveillance authority of the Member State where the incident occurred, within a deadline defined by the incident type. The Article ties into Article 72 because incidents feed the post-market surveillance record. |

The Article 61/72 wording is the engineering brief: *active, systematic collection; documentation; analysis; evaluation of continuous compliance*. The signal contract from chapter 01 is the collection; the register is the documentation; the aggregation reports from mod-103 chapter 05 are the analysis; the report itself is the evaluation.

Article 73 imposes the tight timeline on the *worst* subset of signals (serious incidents). Mod-111 owns the incident response depth; this chapter documents the wire from monitor -> incident -> Article 73 notification.

<!-- needs-research: the Commission implementing act specifying the Article 72 report template is at consultation / adoption stages; confirm the current status and, if adopted, align the report section headers to the implementing act's template. -->

## The plan and the report — one artifact, two views

The plan (Article 61 / 72) describes the monitoring system. The report is what the plan produces. Treat both as views over the same underlying artifact set — the register, the signal contract, the monitor pipeline, and the incident record — with the plan describing the *structure* and the report describing *the observations for the reporting period*.

The plan document has to say, at minimum:

- Which high-risk AI system it covers, keyed to the intake record.
- The Chapter III obligations the system is subject to (risk-management, data governance, technical documentation, record-keeping, transparency, human oversight, accuracy / robustness / cybersecurity).
- The data collected — user-side telemetry, deployer feedback, technical monitoring signals, incident reports, complaint records.
- The metrics computed from that data — with references to the signal contract row identifiers.
- The thresholds and criteria for concluding that the system continues to comply with Chapter III.
- The corrective-action process when thresholds are not met, including timelines and responsibilities.
- The reporting cadence — internal (governance council, mod-112) and external (Article 73 to authorities).
- The retention period and the storage of the collected data.
- The interfaces with the deployer-side flow (deployer complaints, deployer-observed incidents).

The report document, keyed to the same plan, then instantiates every one of the above for the reporting period.

## The report shape — the sections you fill

A defensible Article 72 report has the following sections. The order tracks the plan; the content is the period's data.

### 1. Scope of the report

- The high-risk AI system identifier, its intended purpose, its version(s) in production during the period, and the deployer contexts covered.
- The reporting period start / end dates and the reason for the reporting cadence.
- The plan version this report closes against.

### 2. Data collected during the period

- Aggregate volumes — requests, sessions, deployer sites, jurisdictions.
- The complete list of signals in scope, keyed to signal contract identifiers.
- Sources of data — telemetry, deployer feedback, complaints, third-party incident reports, monitoring output.
- Retention and storage — where the raw data lives, for how long, under what access controls.

### 3. Monitoring signals and thresholds

- Signal contract identifiers, monitor identifiers, thresholds, register-row identifiers, register-field effects. This is the table from chapter 01, filtered to signals applicable to the covered system.
- Any changes to the signal contract during the period (thresholds tuned, signals added, signals retired) with the change-log reference.

### 4. Observations for the period

- Per-signal summary of tripped thresholds — count, distribution over the period, sustained-trip durations.
- Per-register-row residual movements — which rows changed state, when, and in response to which signal.
- Per-slice observations — drift or quality deltas confined to a specific slice, with the fairness liaison's note if applicable.
- Data-quality issues encountered in monitoring (missed batches, calibration lapses, sensor SLO breaches).

### 5. Incidents and near-misses

- The full incident record for the period, keyed to mod-111's incident register.
- For each: type (serious / non-serious per Article 3(49) and Article 73), date detected, date reported to authority (if applicable), affected users, containment, root cause, corrective action, closure state.
- Near-miss summary — top-K near-miss counts by register row, trend over the period, calibration state of the underlying detectors.

### 6. Continuous-compliance evaluation

- For each Chapter III obligation, the section states whether the period's data supports continued compliance and cites the register rows and signals that support the statement.
- Any obligation where the period's data raised a compliance question is called out with the corrective-action pointer.

### 7. Corrective action plan (CAP)

- For every threshold breach not yet closed at report time, the CAP entry: root cause (or investigation status), corrective action, owner, due date, verification method, register-row effect.
- For every serious incident: the corrective action taken, verification of effectiveness, and whether the plan itself (Article 61 / 72) is updated as a result.

### 8. Changes to the plan

- Any changes to the plan made during the period, with the change-log reference and the reason (framework revision, incident-driven, base-rate shift, new use case).
- The plan version this report closes against and, if changed at close, the new plan version.

### 9. Interfaces with Article 73 reporting

- Every serious incident reported under Article 73 during the period, keyed to the outbound notification and the authority notified.
- The turnaround times against the Article 73 deadline requirements.

### 10. Attachments — evidence

- The evidence-manifest references for each observation, keyed to the register-side evidence pointers.
- Hashes and versions of the underlying monitor outputs, guardrail configurations, model versions, and dataset versions in production during the period.

## Generating the report from the register + monitors

The report is a reduction. The register has fields keyed to signal contract identifiers; each signal has an evidence pointer per tripped-signal record; each row has a residual-state history; each incident has a report record. The reduction is:

- for each register row in scope, materialise its residual-state history and its signal-tripped log for the period;
- for each signal, aggregate tripped-count and sustained-trip statistics;
- for each incident, materialise its record and its Article 73 notification (if any);
- for each Chapter III obligation, list the register rows that cite it and their period state;
- for the CAP, list every open treatment flag whose row falls in scope.

The materialisation is a script over the register files (spreadsheet-fallback shape from mod-103 chapter 06) or a query in the GRC-for-AI platform. Either way, running the report at any point in time should produce the same document from the same underlying data. Manual composition is a red flag.

<!-- needs-research: if the Commission's implementing act under Article 72 defines exact machine-readable submission formats, align the reduction's output to that format. -->

## Dovetail with Article 73 — serious incidents

Article 73 requires providers to report serious incidents to the relevant market-surveillance authority. The definition of "serious incident" is in Article 3(49) — broadly, an incident that leads directly or indirectly to death or serious harm to health, serious property or environmental damage, or serious harm to fundamental rights or critical infrastructure. Article 73 has tight deadlines that vary by incident type; the risk engineer's job is to make sure the wire is in place, not to make the legal determination of "serious."

The wire from monitor -> incident -> Article 73 notification:

1. **Monitor trips** (chapter 01-03 contract row); severity is S1.
2. **Incident candidate opens** in the incident register (mod-111).
3. **Triage** decides whether the candidate is an incident and whether it is serious. Legal counsel and DPO are on the triage on any potentially serious candidate.
4. **Article 73 notification** is drafted from the incident record (owner: system-owner + provider legal); the drafting timeline is measured against the Article 73 deadline.
5. **The Article 72 report** captures the notification in section 9 above; the incident record captures the corrective action.

The engineering discipline: the tripped signal, the incident candidate, and the notification are all keyed to the same identifier. No re-typing of facts across systems.

## FDA post-market surveillance for SaMD — the sector variant

For medical-device systems (SaMD) the equivalent obligation is the FDA's post-market surveillance requirement (21 CFR Part 822), supplemented by the FDA Predetermined Change Control Plan (PCCP) guidance for AI/ML-enabled devices and by the FDA / Health Canada / MHRA Good Machine Learning Practice guiding principles. The shape is the same — plan, data, analysis, corrective action, reporting — with a different reporting form and a different regulator on the other end. Sector-specific detail belongs in mod-101 chapter 05; the engineering artifact set is the same. Cite the sector variant explicitly in the plan's scope section if it applies.

<!-- needs-research: any post-market surveillance updates specific to AI/ML SaMD published since the initial GMLP guiding principles and the PCCP guidance — check the FDA CDRH page for the most recent AI/ML action-plan documents. -->

## A worked report-section-4 excerpt

To make the reduction concrete, an excerpt of section 4 for the customer-support RAG assistant over a single quarter might read (identifiers per the chapter 02 / 03 contract rows):

- `SC-CS-DRIFT-01` tripped 4 times over the period, all within a 10-day window in April; the underlying cause was a rollout of a new document type into the KB; residual on `HR-CS-DRIFT-02` moved to `control-defeated` for 8 days and returned to `mitigated` after reference-window regeneration.
- `SC-CS-QUAL-01` tripped 2 times, both aligned in time with the drift trip above; residual on `HR-CS-QUAL-03` recovered on the same schedule.
- `SC-CS-INJ-01` tripped 15 times over the period, all bursts of < 30 minutes; no incident escalation; near-miss counter for `HR-CS-INJ-01` rose from a baseline of 3/week to a peak of 22/week in June.
- `SC-CS-PII-01` had zero confirmed positive firings; firing-rate deviation stayed within band.
- `SC-CS-COST-01` did not trip; latency-family `SC-CS-LAT-01` tripped once on 12 May with a 90-minute sustained trip, tied to an upstream vendor outage.

Every one of the bullet points is a fact you can point at in the register and the evidence pointers. That is the discipline the report enforces.

## Interfaces with adjacent modules

- **mod-101 ch 01 and ch 04** — MEASURE 3.2 / MANAGE 4.1 and the EU AI Act article translation; this chapter is the concrete closure for the Article 61 / 72 sub-section of mod-101 chapter 04.
- **mod-102** — the harm model and its taxonomy tags are how the report ties Chapter III obligations back to specific rows.
- **mod-103 ch 03 / ch 05 / ch 06** — the register schema, the aggregation reports, and the register operations are the underlying data model.
- **mod-108** — the guardrails and controls whose effectiveness is documented in the report.
- **mod-111** — incident records feed sections 5 and 9 of the report; Article 73 notifications are wired from mod-111's incident-response process.
- **mod-112** — the governance-council digest and the Article 72 report share source data; the digest is the internal-audience version.

## Anti-patterns

- **The narrative report.** A quarterly PDF composed by hand, with screenshots of dashboards; no traceable data model behind it. Fix: generate the report as a reduction over the register + monitor pipeline.
- **The unpinned plan.** The plan describes the monitoring system in general terms without citing specific signal contract identifiers or register rows. Fix: every plan section names identifiers.
- **The Article 73 wire that only runs in fire drills.** The provider has drafted an Article 73 notification once, as a rehearsal, and never revisits the wire. Fix: rehearse the wire on every S1 trip, whether or not the incident is serious.
- **The stale plan / current report.** The report is issued against a plan version that no longer describes the running monitoring system. Fix: bump the plan version whenever the signal contract changes materially, and issue the report against the current plan.

## Summary

- Post-market surveillance is generated from the register plus the monitor pipeline; it is not a hand-composed narrative.
- EU AI Act Article 61 / 72 requires the post-market monitoring system to be actively and systematically collecting data throughout the lifetime of a high-risk system; Article 73 is the serious-incident wire from that system to the authority.
- The report has ten sections; each is a view over live data keyed to signal contract identifiers and register rows.
- The FDA SaMD post-market surveillance obligation has the same engineering shape, with a different reporting form and regulator.
- The Article 73 wire runs from monitor -> incident candidate -> triage -> notification; all keyed to the same identifier.
- Exercise 04 walks the Article 72 report generation for a high-risk system.
