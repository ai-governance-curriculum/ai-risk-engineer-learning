# Lab 02 — Article 72 Report Section 4 + Article 73 Log Drill

**Duration:** 45 minutes · **Format:** guided practice, markdown authoring against fabricated tripped-signal counts for a fixed fabricated system

## Objective

Internalise chapter 04's **ten-section Article 72 report shape** and the **Article 73 wire log** by authoring (a) section 4 ("Observations for the period") as a per-row reduction over fabricated tripped-signal counts and residual-state movements, and (b) the Article 73 log entries — including the **rehearsal entry** required in a clean period — for the fixed `HelpDesk` running system over a single quarterly reporting period (Q2 2026). The lab drills the output shape chapter 04 pins so that when **exercise 04** reduces your real register + monitor pipeline into the full ten-section report, the engineering budget goes into the reducer script and the plan-to-report wire — not into inventing the section-4 and section-9 shape.

The chapter-04 anti-patterns the shape forecloses: the **narrative report** composed by hand, the **Article 73 wire that only runs in fire drills**, and the **stale plan / current report** mismatch.

## Prerequisites

- Chapter 04 read — specifically "What the EU AI Act asks for" (the Article 61 / 72 / 73 table), "The report shape — the sections you fill" (ten sections), "Generating the report from the register + monitors" (the reduction), "Dovetail with Article 73 — serious incidents" (the five-step wire), "A worked report-section-4 excerpt", and "Anti-patterns".
- Chapter 01 skimmed at "What a signal contract is" so the contract row identifiers are recognisable.
- Lab 01 completed (or at least read) — the `HelpDesk` running system and its contract rows (`SC-HD-DRIFT-01` through `SC-HD-LAT-01`) are shared between the two labs. If you did not complete lab 01, the brief and the contract row identifiers below are enough to run lab 02 standalone.
- A terminal where you can create a working directory and author a markdown file.
- No external access or compute required; the reporting period, the tripped-signal counts, and the residual-state deltas are fabricated below.

## The fixed example system (repeated from lab 01)

Use this system throughout the lab — do *not* substitute your own.

> **`HelpDesk`** — a customer-support RAG assistant for a mid-size B2B SaaS company. Classified as **high-risk** under EU AI Act Annex III; Article 61 / 72 post-market monitoring plan applies.
>
> Contract rows in scope (from lab 01): `SC-HD-DRIFT-01` (daily PSI), `SC-HD-QUAL-01` (aggregate quality), `SC-HD-QUAL-02` (per-slice quality on `tenant_tier`), `SC-HD-EMB-01` (hourly MMD), `SC-HD-INJ-01` (injection firing rate, two-sided), `SC-HD-PII-01` (PII egress, absolute-rule + deviation), `SC-HD-NEAR-01` (near-miss weekly), `SC-HD-COST-01` (cost anomaly), `SC-HD-LAT-01` (latency SLO burn).
>
> Plan version in effect for the reporting period: `pms-plan-v0.1` (sign-off: risk engineer + system owner + platform lead + risk-council chair + provider legal lead).

### Fabricated reporting period (do not revise)

- **Period:** 2026-04-01 — 2026-06-30 (Q2 2026, 13 weeks).
- **Request volume:** 15.4M (from the trace store roll-up); 38 deployer sites; 4 jurisdictions (EEA + UK + CH + NO).
- **Plan version pinned:** `pms-plan-v0.1` throughout the period (no mid-period bump; §8 of the full report would read "no changes").

### Fabricated tripped-signal counts (authoritative for this lab, do not revise)

Each row below is what the exercise-04 reducer would emit after reading the tripped-signal record store. Pin these verbatim; a reviewer must trace every section-4 bullet to a specific row here.

| Contract row | # trips in period | Sustained-trip stats | Register-row effect summary |
|---|---|---|---|
| `SC-HD-DRIFT-01` | 4 | Window: 2026-04-06 to 2026-04-16 (all 4 trips inside a 10-day cluster); max sustained 36h; median 22h | `HR-CS-DRIFT-02` residual → `control-defeated` 2026-04-06; returned to `mitigated` 2026-04-18 after reference-window regeneration (ref profile hash bump recorded) |
| `SC-HD-QUAL-01` | 2 | Both trips 2026-04-09 and 2026-04-14 (aligned with the DRIFT cluster); max sustained 18h | `HR-CS-QUAL-03` residual → `control-defeated` 2026-04-09; returned to `mitigated` 2026-04-16 |
| `SC-HD-QUAL-02` | 1 | Per-slice trip on `tenant_tier=enterprise`, 2026-05-21; delta 11.4% on composite quality; sustained 28h; fairness liaison on routing | `HR-CS-FAIR-08` residual → `control-defeated` 2026-05-21; `HR-CS-QUAL-03` additionally flagged; returned to `mitigated` 2026-05-23 after retriever re-ranker fix |
| `SC-HD-EMB-01` | 0 | — | `HR-CS-DRIFT-02` unchanged by this row (DRIFT row above drove the residual move) |
| `SC-HD-INJ-01` | 15 | 15 bursts, each < 30 min; **none** promoted to incident; near-miss counter rose from 3/week to a peak of 22/week in June | `HR-CS-INJ-01` control flagged on each burst; near-miss counter on `HR-CS-NEAR-07` elevated |
| `SC-HD-PII-01` | 1 confirmed positive + 3 deviation-only firings | Confirmed positive 2026-06-12 09:14 UTC (S1); deviation S2 firings 2026-04-23, 2026-05-07, 2026-06-02 | `HR-CS-PII-04` residual → `control-defeated` 2026-06-12; incident candidate `IC-HD-2026-06-12-01` opened; **DPO notified within 11 minutes**; pathway remediated and residual returned to `mitigated` 2026-06-20 |
| `SC-HD-NEAR-01` | 2 weekly-threshold breaches | Week 2026-W24 (count 22; threshold 12); Week 2026-W25 (count 17; threshold 12) | `HR-CS-NEAR-07.near_miss_last_7d` updated weekly; treatment flag opened 2026-W24; two consecutive breaches escalated the row to S2 per contract escalation rule |
| `SC-HD-COST-01` | 0 | — | `HR-CS-COST-06` unchanged |
| `SC-HD-LAT-01` | 1 | 2026-05-12 14:30 UTC; sustained 90 min; tied to an upstream vendor outage | `HR-CS-LAT-05` residual → `control-defeated` 2026-05-12; returned to `mitigated` 2026-05-12 after vendor restored |

### Fabricated incident register entries (for section 9)

| Incident id | Candidate source | Triage outcome | Article 73 action | Timing |
|---|---|---|---|---|
| `IC-HD-2026-06-12-01` | `SC-HD-PII-01` confirmed positive | Non-serious under Article 3(49) — single-caller exposure, no fundamental-rights harm found, remediated within 8 days; **legal + DPO logged the determination 2026-06-12 11:02 UTC** | **No Article 73 notification** — triage decision: not-serious; log the decision timestamp, the reviewers, and the rationale | Detection→triage 1h 48m; triage→remediation 8d |
| `SC-HD-INJ-01` bursts (15) | Rate-up trips | None promoted to incident | **No Article 73** — rate-up only, no confirmed harm, near-miss counter tracked instead | n/a |
| Synthetic rehearsal | Rehearsal drill scheduled for the clean quarter | Rehearsal of a synthetic S1 on 2026-05-30 — fired a fabricated PII-egress payload through the dispatcher to confirm the wire | **Rehearsal only** — draft notification produced, DPO + legal walked the draft, draft discarded with a drill-close record | Drill ran monitor→candidate 9m; candidate→draft 42m; drill→close 1h 20m |

## Steps

### Step 1 — Create your working directory and the scaffold

```bash
mkdir -p mod109-lab02 && cd mod109-lab02
touch helpdesk-report-section-4.md
touch helpdesk-article-73-log.md
```

Paste this scaffold into `helpdesk-report-section-4.md`:

```markdown
# HelpDesk — Article 72 report, Section 4 (Observations for the period) — lab 02

System: HelpDesk (high-risk under EU AI Act Annex III; see lab brief)
Reporting period: 2026-04-01 to 2026-06-30 (Q2 2026)
Plan version: pms-plan-v0.1
Chapter references: mod-109 ch 04 ("The report shape"; "Generating the report from the register + monitors"; "A worked report-section-4 excerpt")

## 4.1 Per-signal summary of tripped thresholds

(per contract row: count, distribution over the period, sustained-trip statistics; one bullet per row)

## 4.2 Per-register-row residual movements

(per HR-CS-* row that moved state during the period: which state, when, in response to which signal, when returned)

## 4.3 Per-slice observations

(drift or quality deltas confined to a specific slice; fairness-liaison note; mod-106 seam)

## 4.4 Data-quality issues encountered in monitoring

(missed batches, calibration lapses, sensor SLO breaches during the period)
```

And this into `helpdesk-article-73-log.md`:

```markdown
# HelpDesk — Article 73 wire log — lab 02

System: HelpDesk (high-risk under EU AI Act Annex III)
Reporting period: 2026-04-01 to 2026-06-30 (Q2 2026)
Chapter references: mod-109 ch 04 ("Dovetail with Article 73"); exercise 04 ("Article 73 wire log")

## Wire entries for the period

(one entry per S1 trip walked through monitor → candidate → triage → notification-or-close; include the rehearsal entry if no real S1 during the period)
```

### Step 2 — Fill Section 4.1 (Per-signal summary)

One bullet per contract row. Match the chapter-04 §"A worked report-section-4 excerpt" shape: identifier, count, distribution, sustained-trip stats, register-row effect summary. Every bullet traces back to the fabricated-counts table above.

```markdown
- `SC-HD-DRIFT-01` tripped **4 times** over the period, all within a 10-day window from 2026-04-06 to 2026-04-16; median sustained-trip duration 22h, max 36h; underlying cause (from the per-trip evidence) was a rollout of a new product-doc template into the knowledge-base store.
- `SC-HD-QUAL-01` tripped **2 times**, 2026-04-09 and 2026-04-14, both aligned in time with the `SC-HD-DRIFT-01` cluster; max sustained 18h; recovered on the reference-window-regeneration schedule.
- `SC-HD-QUAL-02` tripped **1 time** on the `tenant_tier=enterprise` slice, 2026-05-21; composite quality delta 11.4% (threshold 10%); sustained 28h; fairness liaison on the routing per contract row; underlying cause was a retriever re-ranker regression that disproportionately affected longer enterprise-ticket contexts.
- `SC-HD-EMB-01` tripped **0 times**; the hourly MMD stayed within band for the full period (the `SC-HD-DRIFT-01` cluster was below the MMD detector's three-consecutive-hours sustained rule).
- `SC-HD-INJ-01` tripped **15 times**, all bursts < 30 minutes; **none promoted to incident**; per-burst near-miss counter contributions drove `SC-HD-NEAR-01` elevation from baseline 3/week to a peak of 22/week in late June.
- `SC-HD-PII-01` recorded **1 confirmed positive** on 2026-06-12 09:14 UTC (S1 absolute-rule row) plus **3 deviation-only firings** on 2026-04-23, 2026-05-07, 2026-06-02 (S2 secondary row); DPO notified 2026-06-12 09:25 UTC (11-minute turnaround from fire to notification).
- `SC-HD-NEAR-01` tripped **2 weekly-threshold breaches** — week 2026-W24 (count 22 against threshold 12) and week 2026-W25 (count 17 against threshold 12); the two-consecutive-week rule escalated the row from S3 to S2 per the contract.
- `SC-HD-COST-01` did not trip; rolling-median cost-per-1k-requests stayed within band.
- `SC-HD-LAT-01` tripped **1 time** on 2026-05-12 14:30 UTC with a 90-minute sustained trip tied to an upstream vendor outage; `HR-CS-LAT-05` residual recovered on the same day after vendor restored.
```

Reading check: nine bullets, one per contract row. Every bullet names the identifier, the count, the distribution-over-the-period shape, and the sustained-trip statistics. Every number traces back to the fabricated-counts table.

### Step 3 — Fill Section 4.2 (Per-register-row residual movements)

Name every `HR-CS-*` row that moved residual state during the period, in chronological order of first move.

```markdown
- **`HR-CS-DRIFT-02`** — residual state `mitigated` → `control-defeated` on 2026-04-06 in response to `SC-HD-DRIFT-01`; returned to `mitigated` on 2026-04-18 after reference-window regeneration (ref profile hash bump recorded on the register note).
- **`HR-CS-QUAL-03`** — residual state `mitigated` → `control-defeated` on 2026-04-09 in response to `SC-HD-QUAL-01`; returned to `mitigated` on 2026-04-16. Additionally flagged on 2026-05-21 in sympathy with `HR-CS-FAIR-08`'s per-slice regression; returned to unflagged on 2026-05-23.
- **`HR-CS-FAIR-08`** — residual state `mitigated` → `control-defeated` on 2026-05-21 in response to `SC-HD-QUAL-02`'s per-slice trip on `tenant_tier=enterprise`; returned to `mitigated` on 2026-05-23 after retriever re-ranker fix.
- **`HR-CS-INJ-01`** — `control_status` flagged on 15 individual bursts in May and June; no sustained residual-state change; near-miss counter elevation recorded on `HR-CS-NEAR-07`.
- **`HR-CS-PII-04`** — residual state `mitigated` → `control-defeated` on 2026-06-12 09:14 UTC in response to the `SC-HD-PII-01` confirmed positive; incident candidate `IC-HD-2026-06-12-01` opened in parallel; returned to `mitigated` on 2026-06-20 after the egress pathway was remediated and the fix was verified on the labelling queue.
- **`HR-CS-NEAR-07`** — `near_miss_last_7d` field updated daily throughout the period; treatment flag opened on 2026-W24 after the weekly-threshold breach; row escalated from S3 to S2 on 2026-W25 after the second consecutive breach.
- **`HR-CS-LAT-05`** — residual state `mitigated` → `control-defeated` on 2026-05-12 14:30 UTC in response to `SC-HD-LAT-01`; returned to `mitigated` on 2026-05-12 16:00 UTC after vendor restored service.
```

Reading check: seven `HR-CS-*` rows accounted for; every move names the signal that caused it; every row that returned to `mitigated` has a date. Rows with no residual movement during the period (`HR-CS-DRIFT-02`'s embedding sibling path; `HR-CS-COST-06`) do not appear because this subsection lists movements, not static state.

### Step 4 — Fill Section 4.3 (Per-slice observations)

```markdown
- **`tenant_tier=enterprise` slice** — `SC-HD-QUAL-02` trip on 2026-05-21 showed an 11.4% absolute delta on the composite quality metric against the slice's own rolling 7-day median; the `tenant_tier=standard` slice stayed within band; the aggregate `SC-HD-QUAL-01` did not trip during this window, confirming the chapter-02 discipline that per-slice signals can trip while the aggregate stays quiet. Fairness liaison was on the routing from the first page; the mod-106 fairness-tagged row `HR-CS-FAIR-08` carried the residual effect; the retriever re-ranker fix was the corrective action and is tracked in Section 7 (CAP). No evidence of systematic bias — the regression was correlated with longer context length, which is over-represented on the enterprise slice but not an indicator of a sustained parity gap.
```

Reading check: one slice observation; the shape names the slice, the delta, the aggregate-quiet / slice-trip asymmetry, the mod-106 seam, the corrective action, and the fairness-signal-vs-parity-gap distinction.

### Step 5 — Fill Section 4.4 (Data-quality issues encountered in monitoring)

```markdown
- **Missed batches:** `SC-HD-DRIFT-01` (daily whylogs batch) skipped a single run on 2026-05-04 due to a trace-store retention compaction job; no data loss; the next day's run computed against the correct 24h window; the pipeline-SLO breach was logged against `ai-infra-mlops` (L25) as a control-availability event and the register row's `control_status` carried a short-lived "sensor stale" flag that cleared 2026-05-05.
- **Calibration lapses:** the LLM-as-judge used for the faithfulness component of `SC-HD-QUAL-01`'s composite drifted against the human-labelled calibration set in 2026-W19 (judge agreement 0.82 vs. target 0.90); the depth of the recalibration is owned by `ai-eval-engineer` (L30); the risk-side effect was a 7-day flag on `HR-CS-QUAL-03` noting that aggregate quality numbers for 2026-W19 and 2026-W20 are to be read with the calibration caveat; the flag cleared on 2026-05-22 after the judge was recalibrated.
- **Sensor SLO breaches:** none beyond the 2026-05-04 whylogs miss above.
```

Reading check: three line items; every entry names what happened, what owned the fix, and what register-side flag resulted. The L30 calibration drift is cited explicitly with the boundary.

### Step 6 — Fill the Article 73 log

Open `helpdesk-article-73-log.md` and fill the log entries. Chapter 04 §"Dovetail with Article 73" is the five-step shape (monitor trip → incident candidate → triage → notification-or-close → report capture). Every S1 walks the wire; a clean-period rehearsal entry stands on its own.

```markdown
## Wire entries for the period

### Entry 1 — `SC-HD-PII-01` confirmed positive, 2026-06-12

| Step | Timestamp (UTC) | Actor | Record identifier |
|---|---|---|---|
| Monitor trip (confirmed positive from labelling queue) | 2026-06-12 09:14 | `pii-egress-detector-firing-rate` (mod-108 v2.1.3) | tripped-signal record `TS-SC-HD-PII-01-2026-06-12-0914` |
| Incident candidate opened | 2026-06-12 09:17 | Routing dispatcher (automated) | candidate `IC-HD-2026-06-12-01` |
| DPO notified | 2026-06-12 09:25 | Routing dispatcher (automated) | candidate carries foreign key |
| Triage convened | 2026-06-12 10:30 | DPO + legal counsel + system owner + risk engineer | triage note on candidate |
| Triage decision — **not serious under Article 3(49)** | 2026-06-12 11:02 | DPO + legal counsel (co-signed) | decision captured on candidate with rationale: single-caller exposure, no fundamental-rights harm, PII type (email) does not trigger Article 3(49) serious-incident criteria |
| **No Article 73 notification filed** — triage decision | 2026-06-12 11:02 | n/a | decision + reviewers + rationale recorded on the candidate; Article 73 section 9 captures the triage decision |
| Remediation completed and verified | 2026-06-20 | Platform lead + DPO verification | residual on `HR-CS-PII-04` returned to `mitigated`; verification entry on CAP |

Elapsed times vs. Article 73 deadlines (for reference, even though the incident was non-serious): detection-to-triage **1h 48m**; triage-to-disposition **32m**; triage-to-remediation **8 days**. All within the Article 73 deadline envelopes that would have applied had the triage returned "serious."

### Entry 2 — Rehearsal drill, 2026-05-30

Required because no real S1 serious-incident candidate occurred in the period — chapter 04 §"Anti-patterns" ("the Article 73 wire that only runs in fire drills") requires the wire to be exercised on cadence.

| Step | Timestamp (UTC) | Actor | Record identifier |
|---|---|---|---|
| Synthetic S1 payload fired at `SC-HD-PII-01` | 2026-05-30 10:00 | Rehearsal driver (owned by risk engineering) | synthetic-run record `REH-SC-HD-PII-01-2026-05-30` |
| Dispatcher fan-out verified (on-call + security + DPO + digest + candidate queue, all keyed to synthetic record identifier) | 2026-05-30 10:09 | Routing dispatcher + SSOT verifier | verifier report `REH-SSOT-2026-05-30` |
| Candidate opened in rehearsal mode | 2026-05-30 10:09 | Routing dispatcher (rehearsal flag set) | candidate `REH-IC-HD-2026-05-30-01` |
| Triage convened (drill) | 2026-05-30 10:42 | DPO + legal counsel + system owner + risk engineer (as in a real triage) | drill triage note |
| Draft Article 73 notification produced | 2026-05-30 11:04 | DPO (drafter) + legal (reviewer) | draft discarded at drill-close; retained for the Article 73 log |
| Drill close — notification discarded | 2026-05-30 11:20 | DPO + legal counsel | drill-close record; synthetic candidate marked `rehearsal_closed` |

Drill elapsed times: monitor-to-candidate **9m**; candidate-to-draft **42m**; drill start-to-close **1h 20m**. All within the Article 73 deadline envelope. **Finding:** the DPO was not paged on a mobile channel during the drill — the dispatcher notified via email only; the routing block on `SC-HD-PII-01` has been updated to add DPO mobile paging. This is a drill-driven routing-contract improvement (chapter 01 §"Versioning and change control" diff).

### Entry 3 — `SC-HD-INJ-01` bursts (15, none serious)

The 15 injection-rate bursts across the period are listed here for completeness, not as Article 73 wire entries. Chapter 04 §"Dovetail with Article 73" distinguishes monitor trips from incidents from serious incidents: a rate-up burst that opens an incident candidate but does not become a confirmed harm is not an Article 73 event. Each burst was logged with its timestamp and the near-miss counter contribution; none was promoted to incident in the mod-111 queue. The aggregate near-miss elevation on `HR-CS-NEAR-07` is reported in section 5 of the full report (not here).
```

Reading check: two wire entries that walk the five steps end-to-end; one rehearsal entry present because the period had no real serious incident; drill finding fed back into the contract as a routing-block diff. Elapsed times are measured against Article 73 deadline envelopes even when the triage returned non-serious.

### Step 7 — Final checkpoint

Re-read both files. Confirm:

- **Section 4.1** has one bullet per contract row — nine bullets for the nine rows — each citing the identifier, the trip count, the distribution over the period, and sustained-trip statistics. Every number traces back to the fabricated-counts table.
- **Section 4.2** names every `HR-CS-*` row that moved residual state during the period in chronological order, with the signal that caused each move and the date returned to `mitigated`.
- **Section 4.3** has one per-slice observation that names the slice, the asymmetry (slice trips while aggregate stays quiet), the mod-106 seam, and the fairness-signal-vs-parity-gap distinction.
- **Section 4.4** lists missed batches, calibration lapses, and sensor SLO breaches, each citing who owns the fix (L25 pipeline, L30 judge calibration).
- **Article 73 log** has at least one real S1 entry walked through the five-step wire **and** at least one rehearsal entry because the period was otherwise clean — the chapter-04 anti-pattern ("Article 73 wire that only runs in fire drills") is foreclosed only if the rehearsal is first-class.
- **Every elapsed time** in the Article 73 log is measured against the Article 73 deadline envelope even when the triage returned non-serious — the wire is tested, not the serious-incident determination.
- **The DPO is on the wire by role name** on every PII-linked step.
- **Drill findings feed back into the contract** — the rehearsal entry names a routing-block improvement (DPO mobile paging) that becomes a chapter-01 versioning diff.
- **No free-text metrics.** Every count, every delta, every timestamp appears in one of the fabricated-input tables above; the lab's output is a reduction, not a composition.

## Expected output

`mod109-lab02/helpdesk-report-section-4.md` and `mod109-lab02/helpdesk-article-73-log.md` — two files totalling roughly two pages. Section 4.1 (per-signal summary) and Article 73 entry 2 (the rehearsal) are the two load-bearing parts an auditor reads first.

## Checkpoint questions (self-check)

1. The Q2 2026 period had no real S1 serious incident, only a single confirmed non-serious PII positive. Why does the Article 73 log still carry the rehearsal entry, and what does the rehearsal's drill-finding contribute to the signal contract? (*Chapter 04 §"Anti-patterns" names the "Article 73 wire that only runs in fire drills" failure mode: a provider who only exercises the wire when a real serious incident fires cannot defend that the wire works. Exercise 04 §"Article 73 wire log" makes the rehearsal entry first-class — "if no S1 fired during the period, run a rehearsal wire drill on a synthetic S1 and record the drill's turnaround." The drill-finding in this lab — DPO not paged on a mobile channel during the drill — becomes a chapter-01 versioning diff on `SC-HD-PII-01`'s routing block, and the next drill exercises the fix. The rehearsal is the mechanism that keeps the wire honest between real incidents.*)

2. `SC-HD-INJ-01` tripped 15 times over the period but none of those trips produced an Article 73 entry. Why? (*Chapter 04 §"Dovetail with Article 73" distinguishes monitor trips from incidents from serious incidents. A rate-up firing-rate burst is a monitor trip and opens an incident **candidate**; the candidate is triaged to decide whether it is an incident at all and, if so, whether it is serious under Article 3(49). For the 15 bursts in this period, no burst was promoted to incident — the mod-111 queue dismissed each as "rate deviation without a confirmed harm, near-miss counter tracked." Without an incident, there is no Article 73 event. The bursts are still captured in section 5 of the full report (incidents and near-misses) under the near-miss trend on `HR-CS-NEAR-07`, which is where the cumulative shape of the attack pattern lives.*)

3. The `tenant_tier=enterprise` per-slice trip on `SC-HD-QUAL-02` affected `HR-CS-FAIR-08` as the primary residual target, with `HR-CS-QUAL-03` also flagged. Why both, and what Chapter III obligation of the EU AI Act Act is this evidence for in Section 6 of the full report? (*Chapter 02 §"Per-slice monitoring" is explicit: a per-slice regression on a protected-attribute slice routes to the fairness-tagged row, not the drift-/quality-tagged row, because a per-slice signal on a protected attribute is a fairness signal first and a quality signal second. The aggregate `HR-CS-QUAL-03` is flagged in parallel because per-slice regressions are leading indicators that the aggregate will move. In Section 6 (continuous-compliance evaluation), this evidence lands under **Chapter III Article 10 — data and data governance** and **Article 15 — accuracy, robustness, and cybersecurity**: the system demonstrated per-slice quality variance was detected, routed to the fairness liaison, remediated within 48h, and the register-side residual recovered on evidence of the fix.*)

4. Section 4.4 cites an LLM-as-judge calibration drift in 2026-W19 and routes the depth to `ai-eval-engineer` (L30). Why is the risk-side section-4.4 line necessary if the depth is owned at L30? (*Chapter 04 §"The report shape" section 4 pins that data-quality issues encountered in monitoring belong in the report even when the depth is owned by a peer track — the report is a view over the monitoring system's state during the period, and a judge drift is a state the signals depended on. Chapter 02 §"Where the boundary to `ai-eval-engineer` sits" and chapter 06 §"The peer boundary to `ai-eval-engineer`" both pin the risk-side hook: the eval loop's shape is theirs, the register-side effect of the eval loop's output is yours. Section 4.4's line is the record of the effect — `HR-CS-QUAL-03` carried a 7-day calibration caveat flag — without re-owning the fix. A missing section-4.4 line would leave the auditor without the context to interpret the W19 / W20 quality numbers.*)

## Why this lab matters

Exercise 04 asks you to generate a full ten-section Article 72 report as a deterministic reduction over the register + monitor pipeline for your real running system, paired with the Article 61 / 72 plan and an Article 73 wire log that includes rehearsals. The reducer, the plan, the report, and the input-hash manifest are the four artefacts the market-surveillance inspector reads first. If the section-4 shape and the Article 73 wire-log shape are new at exercise-04 authoring time, the budget blows on composing those sections by hand — exactly the chapter-04 "narrative report" anti-pattern in a new coat. This lab drills the output shape on a fixed fabricated period so that exercise 04's time goes into the reducer script and the plan-to-report wire. The section-4 and Article 73 shapes it drills are also what exercise 04's reducer emits deterministically; getting them right here is what lets the exercise focus on determinism, hashing, and the plan version binding.
