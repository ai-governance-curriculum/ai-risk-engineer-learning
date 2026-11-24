# Lab 03 — Runbook and Single-Source-of-Truth Controlled-Trip Drill

**Duration:** 45 minutes · **Format:** guided practice, markdown authoring on a fabricated controlled trip through a fixed fabricated system

## Objective

Internalise chapter 05's **runbook shape** and the **single-source-of-truth property** — one tripped-signal record per event, every audience's view derived from that one record, corrections propagate — by authoring (a) the runbook for one high-severity contract row (`SC-HD-INJ-01`) in the chapter-05 §"Runbooks — the on-call's contract" shape, and (b) the controlled-trip drill log that walks a fabricated injection fire through the on-call view, the security view, the governance-council digest queue, and the incident-candidate queue, plus the false-positive propagation that corrects the record and re-propagates. The lab drills the artefact shape chapter 05 pins so that when **exercise 05** wires the routing dispatcher, the runbook set, the incident-candidate queue, and the SSOT verifier for your real running system, the engineering budget goes into the dispatcher's integrations and the verifier's assertions — not into inventing the runbook and drill-log formats.

The chapter-05 anti-patterns the shape forecloses: the **Slack-only alert**, **parallel dashboards**, **silent muting**, the **un-runbooked signal**, the **re-typed digest**, and the **false positive that never propagated**.

## Prerequisites

- Chapter 05 read — specifically "What 'same source of truth' means, precisely" (the one-record property and four operating rules), "The three audiences and what each of them needs", "The incident-candidate queue", "Anti-patterns", "The routing table — the concrete contract", "Runbooks — the on-call's contract", and "Worked routing — customer-support RAG assistant on `SC-CS-INJ-01`".
- Chapter 03 skimmed at "Jailbreak and prompt-injection detection specifically" and §"Row `SC-CS-INJ-01`" so the signal's shape is recognisable.
- Lab 01 completed (or at least read) — the `HelpDesk` system and the `SC-HD-INJ-01` contract row are shared across the three labs. If you did not complete lab 01, the brief and the row recap below are enough to run lab 03 standalone.
- A terminal where you can create a working directory and author a markdown file.
- No external access or compute required; the trip identifier, the per-audience payloads, and the false-positive propagation are fabricated below.

## The fixed example system (recap from labs 01 and 02)

> **`HelpDesk`** — a customer-support RAG assistant for a mid-size B2B SaaS company. High-risk under EU AI Act Annex III.

### `SC-HD-INJ-01` row recap (from lab 01)

- `monitor_id`: `guardrail-injection-firing-rate` (mod-108 v2.1.3).
- `signal_semantics`: 5-minute rolling firing rate of the input-side prompt-injection classifier over evaluated requests; two-sided deviation.
- `threshold`: rate > 3x rolling 24h baseline for ≥ 15 minutes (up) or rate < 1/3x with volume steady (down).
- `severity`: S1 both directions.
- `register_effect`: on rate-up, `HR-CS-INJ-01` control flagged + incident candidate opened + near-miss counter increments; on rate-down with steady volume, `control_status → unavailable` + candidate opened.
- `alert_routing`: on-call + security + governance-council digest on sustained trip.
- `owner_on_call`: ML on-call (joint with security on S1).

### Fabricated controlled-trip identifiers (do not revise)

- **Tripped-signal record identifier:** `TS-SC-HD-INJ-01-2026-11-20-1430`.
- **Trip detected:** 2026-11-20 14:30 UTC — rolling 5-min firing rate 11.2% vs. baseline 2.4% (4.7x deviation, sustained 16 minutes).
- **Raw signal source:** mod-108 classifier event stream, 1,840 fires / 16,428 evaluated over the sustained window.
- **Payload-hash sample:** `sha256:a4e2…c81f` (head of a 50-sample attack-shape cluster).
- **Incident candidate:** `IC-HD-2026-11-20-01`.
- **Security ticket:** `SEC-HD-2026-11-20-INJ`.
- **Governance-council digest entry queue id:** `DIG-2026-W47-INJ-01`.
- **Runbook link:** `runbooks/sc-hd-inj-01.md` (authored in step 2 below).

### Fabricated correlation note (for the runbook's first-check list)

The exercise-03 correlation memo lists `SC-HD-INJ-01 ↔ SC-HD-COST-01` and `SC-HD-INJ-01 ↔ SC-HD-LAT-01` (injection-driven expensive tool calls inflate cost; sustained attacks shift latency). During the controlled trip, `SC-HD-COST-01` is **within band** (no correlated cost anomaly) and `SC-HD-LAT-01` is **within band** (no SLO burn).

## Steps

### Step 1 — Create your working directory and the two scaffolds

```bash
mkdir -p mod109-lab03/runbooks mod109-lab03/drills
touch mod109-lab03/runbooks/sc-hd-inj-01.md
touch mod109-lab03/drills/2026-11-20-controlled-trip.md
```

Paste this scaffold into `runbooks/sc-hd-inj-01.md`:

```markdown
# Runbook — `SC-HD-INJ-01` (prompt-injection firing rate)

Monitor: guardrail-injection-firing-rate (mod-108 v2.1.3)
Register row: HR-CS-INJ-01 (prompt-injection control credit)
Contract version this runbook is pinned to: v0.1
Chapter references: mod-109 ch 05 §"Runbooks — the on-call's contract" (the six-field runbook shape)

## 1. Signal meaning (one line)

## 2. First-check list (3–5 items, 5-minute time budget)

## 3. Escalation criteria

## 4. Containment options

## 5. False-positive recording protocol

## 6. Back-links
```

And this into `drills/2026-11-20-controlled-trip.md`:

```markdown
# Controlled-trip drill — 2026-11-20 — `SC-HD-INJ-01`

Record identifier: TS-SC-HD-INJ-01-2026-11-20-1430
Chapter references: mod-109 ch 05 §"What 'same source of truth' means, precisely" (the one-record property); exercise 05 §"The controlled trip" (end-to-end walk)

## 1. Dispatcher fan-out (one record, four audience views)

(on-call payload; security ticket payload; digest queue payload; candidate queue payload — all keyed to TS-SC-HD-INJ-01-2026-11-20-1430)

## 2. SSOT verifier assertion report

(assertion per audience that the view derives from the one record; no divergence)

## 3. On-call acknowledgement + runbook execution trace

(acknowledgement timestamp; first-check answers; containment decision)

## 4. False-positive reclassification (fabricated) + propagation trace

(record mutation; near-miss counter decrement; digest entry correction; security-ticket state change; register-row recompute)

## 5. Drill-close findings and contract diff candidates

(no-discrepancies closure or filed findings; any routing-block or runbook diffs the drill surfaces)
```

### Step 2 — Fill the runbook (`sc-hd-inj-01.md`)

Six sections per chapter 05 §"Runbooks". Keep the first-check list to five items — the five-minute time budget is the chapter's rule. Use the fabricated `HelpDesk` tool surface (`escalate_ticket`, `lookup_account`) and the correlation memo so the runbook is anchored in the row's shape, not generic guidance.

```markdown
## 1. Signal meaning (one line)

A sudden rise or fall in the input-side prompt-injection classifier's firing rate versus its rolling baseline, measured per 5 minutes over 16+ continuous minutes — rate up signals a new attack shape or user population; rate down with steady volume signals the classifier is failing open or has been silenced.

## 2. First-check list (3–5 items, 5-minute time budget)

1. **Rate-up or rate-down?** Open the dispatcher notification and read the `direction` field on the record — rate-up (4.7x baseline this trip) and rate-down have different containment paths; do not confuse them.
2. **Concentrated or broad?** Query the trace store on the record's timestamp window — is the fire concentrated in a single tenant (tenant-specific attack) or spread across many (campaign)? Named `tenant_id` histogram on the record evidence pointer.
3. **Known attacker corpus?** Check whether the payload-hash sample on the record falls into any known-attacker cluster in the mod-104 adversarial fixture store (`head: sha256:a4e2…c81f`). If yes, the response is faster (established containment); if no, security takes the lead on sample labelling.
4. **Correlated siblings tripped?** Check whether `SC-HD-COST-01` or `SC-HD-LAT-01` tripped in the same window — the exercise-03 correlation memo lists these as injection-sympathetic signals; a joint trip is **one incident with correlation**, not two.
5. **Reproducible in controlled test?** Pull one sample from the record, replay against a staging instance — does the classifier still fire? If not, the production run has drifted away from the staging run (classifier version mismatch, config drift) and the register-side `control_status` flag applies regardless of attack reality.

## 3. Escalation criteria

- **Promote candidate to incident** (mod-111): sustained > 1h *or* correlated sibling tripped *or* confirmed novel attack pattern *or* rate-down with steady volume.
- **Add security lead to triage**: confirmed novel attack pattern *or* tool-call-argument detector firing in the same window.
- **Add legal + DPO**: any touch of PII fields in the payload-hash sample (ask the labelling queue; do not label in-channel).
- **Page risk-council chair**: sustained > 4h with no containment effect, or any suspected exfiltration to an off-registry destination.

## 4. Containment options

- **Throttle the affected tenant** (if tenant-concentrated) — temporary per-tenant rate limit at the middleware; preserves other tenants' traffic.
- **Disable the specific tool call** (`escalate_ticket` or `lookup_account`) if the attack pattern targets a tool argument — deterministic at the tool-invocation policy layer; chapter 02 §"When rule-based fits" is why this is the stronger lever.
- **Add the pattern to the input-side classifier's block-list** — fixture-driven; the new pattern lands in the mod-108 fixture library (upstream wire from chapter 06).
- **Roll back the classifier to the prior version** if the rate-down case (classifier failing open) is confirmed — the prior version's residual is already known from chapter 05 effectiveness measurement (mod-108).
- **Circuit-break the classifier** (fail-closed) only as a last resort — fails all requests the classifier cannot evaluate, which is a different harm. Requires system-owner sign-off.

## 5. False-positive recording protocol

- Pull a 50-sample slice from the record's payload-hash cluster and send to the labelling queue.
- If the labelling queue returns `false_positive: true` on the majority of the slice, set `TS-SC-HD-INJ-01-2026-11-20-1430.false_positive = true` on the record — this is a **record mutation**, not a channel note.
- The mutation propagates automatically: `HR-CS-NEAR-07` near-miss count for the week decrements; the digest entry `DIG-2026-W47-INJ-01` recomputes; the security ticket `SEC-HD-2026-11-20-INJ` state flips to "false-positive-closed".
- Do not mark `false_positive: true` without the labelling-queue return — chapter 05 §"Anti-patterns" ("the false-positive that never propagated") is the shape the mutation rule prevents; the inverse is also an anti-pattern (marking records false-positive from a Slack thread without the labelling loop).

## 6. Back-links

- Register row: `HR-CS-INJ-01` (register file `mod103/registers/helpdesk.yaml`, operational log `mod103/registers/helpdesk-oplog.md`).
- Signal contract row: `SC-HD-INJ-01` (contract file `mod109/signal-contract-helpdesk.md`).
- Monitor dashboard: `monitors/guardrail-injection-firing-rate/dashboard` (owned by peer L25 `ai-infra-mlops`).
- Record store location: `evidence/sc-hd-inj-01/<yyyy-mm-dd-hh>/record.json`.
- Incident-response playbook: `mod111/playbooks/safety-signal-s1.md`.
- Correlation memo: `mod109/correlation-memo-helpdesk.md` (exercise 03).
```

Reading check: six sections per chapter-05 shape; first-check list is **exactly five items** and reads in under five minutes; containment options list is concrete (named tool calls, named classifier version) rather than abstract; false-positive protocol names the record mutation as the propagation vector, not a Slack thread; every back-link points to a file or a system identifier the on-call can follow on a page.

### Step 3 — Fill the drill log Section 1 (Dispatcher fan-out, one record, four views)

The dispatcher reads the record, resolves the row's routing, and fan-outs to the four audiences. Each audience's payload carries the record identifier as the foreign key; the dispatcher does not embed metric values in the notification body.

```markdown
## 1. Dispatcher fan-out (one record, four audience views)

Record identifier (foreign key everywhere): **`TS-SC-HD-INJ-01-2026-11-20-1430`**

### 1.1 On-call page (PagerDuty-equivalent)

```
S1 — SC-HD-INJ-01 — HelpDesk prompt-injection firing rate
Direction: rate up (4.7x baseline, sustained 16m)
Record: TS-SC-HD-INJ-01-2026-11-20-1430
Register row: HR-CS-INJ-01 (control flagged)
Runbook: runbooks/sc-hd-inj-01.md
Record store: evidence/sc-hd-inj-01/2026-11-20-14/record.json
Incident candidate: IC-HD-2026-11-20-01
Correlation siblings tripped this window: none
```

### 1.2 Security ticket (SEC-HD-2026-11-20-INJ)

| Field | Value |
|---|---|
| Ticket id | SEC-HD-2026-11-20-INJ |
| Record identifier (foreign key) | TS-SC-HD-INJ-01-2026-11-20-1430 |
| Signal type | prompt-injection — input-side classifier firing rate |
| Severity | S1 |
| Timestamp | 2026-11-20 14:30 UTC |
| Affected system | HelpDesk production |
| Correlation tags | — (no sibling trip in window) |
| Payload-hash sample head | sha256:a4e2…c81f |
| Assigned | security on-call |
| DPO involvement required | not yet — triage pending on PII-touch check (first-check item 3 in the runbook) |

### 1.3 Governance-council digest queue entry (DIG-2026-W47-INJ-01)

The digest entry is queued; the week's digest will render it under the "S1 trips this week" section. The entry carries the record identifier and is rendered as a reduction at digest-build time — the dispatcher does not compose narrative.

```yaml
digest_entry_id: DIG-2026-W47-INJ-01
record_id: TS-SC-HD-INJ-01-2026-11-20-1430
contract_id: SC-HD-INJ-01
register_row_id: HR-CS-INJ-01
severity: S1
direction: rate-up
sustained_minutes: 16
rate_vs_baseline: 4.7
candidate_id: IC-HD-2026-11-20-01
article_73_status: triage_pending
corrective_action_state: in-progress
```

### 1.4 Incident-candidate queue entry (IC-HD-2026-11-20-01)

```yaml
candidate_id: IC-HD-2026-11-20-01
record_id: TS-SC-HD-INJ-01-2026-11-20-1430
contract_id: SC-HD-INJ-01
register_row_id: HR-CS-INJ-01
severity: S1
source: automated  # dispatched by the routing dispatcher
triage_state: pending
disposition: null
disposition_reason: null
promoted_to_incident_id: null
opened_at: 2026-11-20T14:33:12Z  # 3 minutes after trip (dispatcher fan-out)
```
```

Reading check: four audience payloads; every payload carries `TS-SC-HD-INJ-01-2026-11-20-1430` as the foreign key; no payload embeds the per-trip metric value as an independent fact — each references the record. The dispatcher's job is to resolve routing and emit identifiers; the audience views fetch the facts from the record.

### Step 4 — Fill the drill log Section 2 (SSOT verifier assertion report)

The verifier walks each audience view and asserts the record identifier is present and the field values derive from the same record. Chapter 05 §"What 'same source of truth' means" is the property under test.

```markdown
## 2. SSOT verifier assertion report

Run id: `REH-SSOT-2026-11-20-01`

| Audience | Foreign key present? | Direct fields match record? | Derived fields match projection? | Status |
|---|---|---|---|---|
| On-call page | ✓ `TS-SC-HD-INJ-01-2026-11-20-1430` on the page body | ✓ severity = S1; direction = rate-up; sustained = 16m; `HR-CS-INJ-01` flagged | ✓ runbook link resolves; record store URL resolves; candidate id resolves | **pass** |
| Security ticket SEC-HD-2026-11-20-INJ | ✓ foreign key in `record_identifier` field | ✓ severity = S1; timestamp = 2026-11-20 14:30 UTC; affected system = HelpDesk production | ✓ correlation tags match correlation memo (none this window); payload-hash head matches record sample | **pass** |
| Digest queue DIG-2026-W47-INJ-01 | ✓ `record_id` on the YAML | ✓ rate_vs_baseline = 4.7; sustained_minutes = 16; article_73_status = triage_pending | ✓ all seven digest fields project from the record; no extraneous fields | **pass** |
| Candidate queue IC-HD-2026-11-20-01 | ✓ `record_id` on the YAML | ✓ severity = S1; contract_id = SC-HD-INJ-01; register_row_id = HR-CS-INJ-01 | ✓ opened_at = dispatcher timestamp (+3m from trip); triage_state = pending | **pass** |

**Verifier result: no discrepancies.** All four audience views derive from `TS-SC-HD-INJ-01-2026-11-20-1430`; no view carries a divergent fact.
```

Reading check: four rows, one per audience; three assertion columns per row (foreign key, direct-fields, derived-fields); verifier result is "no discrepancies" — if a cell failed, the drill would halt and file a dispatcher bug before closing.

### Step 5 — Fill the drill log Section 3 (On-call acknowledgement + runbook execution trace)

```markdown
## 3. On-call acknowledgement + runbook execution trace

| Step | Timestamp (UTC) | Actor | Outcome |
|---|---|---|---|
| On-call paged | 2026-11-20 14:33 | Dispatcher | PagerDuty incident 48217 opened |
| On-call acknowledged | 2026-11-20 14:35 | ML on-call | Record state → `acknowledged` (mutation propagates to digest entry, security ticket, candidate queue) |
| First-check 1 — direction | 2026-11-20 14:36 | ML on-call | Rate-up confirmed (4.7x baseline) |
| First-check 2 — concentration | 2026-11-20 14:38 | ML on-call | Concentrated on 3 tenants in the `enterprise` tier; not broad |
| First-check 3 — known attacker | 2026-11-20 14:41 | ML on-call | Payload-hash cluster matches a known social-engineering injection class (mod-104 fixture `attack-shape-INJ-23`); escalate to security but established containment path |
| First-check 4 — correlated siblings | 2026-11-20 14:42 | ML on-call | Neither `SC-HD-COST-01` nor `SC-HD-LAT-01` tripped in the window; one incident, not two |
| First-check 5 — reproducible in staging | 2026-11-20 14:47 | ML on-call | Yes — classifier fires on the sample in staging; not a config-drift artefact |
| Containment chosen | 2026-11-20 14:49 | ML on-call + security on-call | Throttle the three affected tenants (option 1); add attack-shape-INJ-23 to the fixture library and the input-side block-list (option 3, follow-up PR) |
| Rate returned within band | 2026-11-20 14:58 | — | Sustained window elapsed; rate at 2.6% (1.08x baseline); dispatcher auto-recomputes digest entry |
| Candidate held for review | 2026-11-20 15:30 | Risk engineer | Disposition pending until false-positive labelling check completes (step 4) |
```

Reading check: ten steps from page to disposition; first-check list completed in 14 minutes of the five-minute budget is the realistic end-to-end (the five-minute rule is the triage budget, not the total containment budget); containment option is named by its runbook item number; every step cites the record's state (not a side-channel Slack thread).

### Step 6 — Fill the drill log Section 4 (False-positive reclassification + propagation)

Fabricate a false-positive outcome for the drill: the labelling queue returns the majority of the 50-sample slice as false positive — the attack-shape-INJ-23 pattern turned out to be a run of a legitimate customer-support workflow that the classifier is over-sensitive to. The mutation on the record is what propagates.

```markdown
## 4. False-positive reclassification (fabricated) + propagation trace

| Step | Timestamp (UTC) | Actor | Mutation and downstream effect |
|---|---|---|---|
| Labelling queue return | 2026-11-21 10:00 | Labelling loop (peer L30 protocol owner; L25 risk-side hook) | 42/50 samples labelled `false_positive`; majority rule applies |
| Record mutation | 2026-11-21 10:03 | Risk engineer | `TS-SC-HD-INJ-01-2026-11-20-1430.false_positive = true`; `false_positive_reason = "classifier over-sensitive to legitimate enterprise-workflow phrasing cluster"` |
| Near-miss counter recompute | 2026-11-21 10:04 | `near-miss-counter-daily` (automatic) | `HR-CS-NEAR-07.near_miss_last_7d` decrements by the week-24 contribution of this record (fire count was 1,840; week count re-sums excluding this record) |
| Digest entry recompute | 2026-11-21 10:04 | Digest reducer (automatic) | `DIG-2026-W47-INJ-01.false_positive_confirmed = true`; the weekly digest now reports the trip but flags it as a confirmed FP |
| Security ticket state change | 2026-11-21 10:05 | SSOT dispatcher | `SEC-HD-2026-11-20-INJ.state = false-positive-closed`; ticket retained for the FP-audit trail |
| Candidate disposition | 2026-11-21 10:05 | Risk engineer | `IC-HD-2026-11-20-01.triage_state = dismissed`; `disposition = false-positive-confirmed`; `disposition_reason = labelling-queue majority` |
| Register-row recompute | 2026-11-21 10:06 | Register-effect wire | `HR-CS-INJ-01` control flag cleared (no sustained incident); `control_status = ok`; register operational log records the FP-driven state return |
| Classifier calibration input | 2026-11-21 10:10 | Risk engineer | Attack-shape-INJ-23 fixture annotated with the FP cluster; feeds back into mod-108's effectiveness-measurement cycle (upstream wire) |

**Propagation verifier re-run:** SSOT verifier run `REH-SSOT-2026-11-21-01` returns **no discrepancies** across all four audiences; every view reflects the FP state within 2 minutes of the record mutation.
```

Reading check: eight propagation steps; the record mutation (step 2) is what drives every downstream change; no step is "the on-call tells the digest in Slack"; the SSOT verifier re-runs and confirms no divergence after the mutation.

### Step 7 — Fill the drill log Section 5 (Drill-close findings and contract diff candidates)

```markdown
## 5. Drill-close findings and contract diff candidates

**Drill result:** controlled trip walked end-to-end through dispatcher fan-out, SSOT verifier assertion, on-call triage and containment, false-positive labelling, record mutation, and propagation. No divergence between audiences; all four views reflected the trip and its FP reclassification from the one record.

**Findings filed as contract or runbook diffs:**

1. **Runbook first-check item 3 refinement.** The labelling-queue majority-rule turnaround was ~16 hours (overnight); the runbook's five-minute first-check item 3 ("known attacker corpus?") caught the attack-shape-INJ-23 match correctly but did not pre-queue the labelling-slice draw — the slice draw was manual at step 4 of the labelling-queue return. Add a runbook-step-6 action: "If first-check 3 returns a known cluster, pre-queue the 50-sample slice at acknowledgement time so the FP check starts in parallel with containment." Filed as a runbook diff.
2. **Correlation memo cross-check tightening.** First-check 4 ("correlated siblings tripped?") returned "none" from a manual dashboard query; the dispatcher does not yet annotate the notification with "correlation siblings this window." Stretch goal from exercise 05 §"Correlation-aware dispatching" — file as an exercise-05 scope item.
3. **Candidate queue retention for FP.** The candidate `IC-HD-2026-11-20-01` was dismissed as FP but is retained in the queue for audit trail. Verify the queue's retention policy is on the record (schema file `queue/README.md`); add if missing.

**No discrepancies found between audience views. Drill closed.** Runbook diff filed; correlation-dispatcher scope item filed to exercise 05; queue retention policy to verify.
```

Reading check: closure statement is explicit ("no discrepancies" or "discrepancies filed as ..."); findings are diffs on the contract / runbook / exercise 05 scope, not open questions in a Slack thread.

### Step 8 — Final checkpoint

Re-read both files. Confirm:

- **Runbook has six sections** per chapter 05 §"Runbooks — the on-call's contract" — signal meaning (one line), first-check list (three to five items, five-minute budget), escalation criteria, containment options, false-positive recording protocol, back-links. No section is missing.
- **First-check list is five items or fewer** — the five-minute budget is the hard rule.
- **Containment options are concrete** — named tool calls (`escalate_ticket`, `lookup_account`), named classifier version, named middleware lever. No abstract "throttle" without a target.
- **False-positive recording protocol names the record mutation as the propagation vector.** A Slack note without the record mutation is the chapter-05 anti-pattern.
- **Drill log Section 1 shows four audience payloads, all keyed to the same record identifier.** No payload embeds the metric value as an independent fact.
- **Drill log Section 2's SSOT verifier result is "no discrepancies"** — a verifier that passed trivially (no assertions) is not a verifier.
- **Drill log Section 3's first-check completes in the five-minute triage budget** (containment is separate; the budget is on first-check, not on resolution).
- **Drill log Section 4's false-positive propagation drives every downstream change from the record mutation** — the near-miss counter, the digest entry, the security ticket, the candidate disposition, and the register row all recompute from the one mutation.
- **Drill log Section 5 closes with either "no discrepancies" or filed findings** — a drill that does not close is a drill that is still running.
- **The DPO appears in the escalation criteria** when a PII touch is possible (runbook §3) — the chapter-05 by-name-DPO-routing property holds for the injection row as well, because an injection can cover PII pathways.

## Expected output

`mod109-lab03/runbooks/sc-hd-inj-01.md` and `mod109-lab03/drills/2026-11-20-controlled-trip.md` — two files totalling roughly two pages. The runbook's first-check list and the drill log's Section 4 (false-positive propagation) are the two load-bearing parts an auditor reads first.

## Checkpoint questions (self-check)

1. The dispatcher fan-out in Section 1 shows four audience payloads, each carrying the record identifier `TS-SC-HD-INJ-01-2026-11-20-1430`. A reviewer asks why the payloads do not embed the metric values (rate 4.7x, sustained 16m) directly, since the audiences will want those numbers. Per chapter 05, what is the correct answer? (*Chapter 05 §"What 'same source of truth' means, precisely" is explicit: "every downstream artifact is derived from that one record. If you change the record (correction, false-positive tagging, closure), every derivative view reflects the change on the next refresh." Embedding the metric value in the notification body means the on-call's PagerDuty page, the security ticket, and the digest entry each carry an independent copy of the fact. When the record is later re-labelled false positive, the mutation on the record propagates to any view that queries the record; views that **cached** the metric value will not. Section 4 of the drill log is exactly the case this property is written to defend — a false-positive mutation on the record flips the security ticket to `false-positive-closed` and the digest entry's `false_positive_confirmed` within minutes, and the audience views fetch the fresh projection. The dispatcher's discipline is to embed the identifier, not the value.*)

2. The runbook's first-check item 3 asks "known attacker corpus?" and the drill found an attack-shape-INJ-23 match. The labelling queue then returned the majority of the 50-sample slice as false positive — a legitimate workflow the classifier was over-sensitive to. Why is the record still mutated to `false_positive: true` rather than the classifier being adjusted first? (*Chapter 05 §"Runbooks — the on-call's contract" and the chapter-03 §"The false-positive discipline" framework pin this: the record is a *fact* about what happened — the classifier fired, the trip occurred — and the mutation is a *correction* that the fire was a FP. The record mutation is what allows the near-miss counter (`HR-CS-NEAR-07`) to decrement correctly, the digest entry to flag the FP for the council, the security ticket to close as FP (not as unresolved), and the register-row to clear its control flag. The classifier adjustment is a **separate** downstream work — the attack-shape-INJ-23 fixture is annotated with the FP cluster and feeds into mod-108's effectiveness-measurement cycle, which is an upstream-wire input to the signal contract (chapter 06 §"The upstream wire from mod-108"). The record mutation happens on the labelling-queue return; the classifier adjustment happens in the next mod-108 effectiveness cycle.*)

3. The drill log Section 5 files a finding that the dispatcher does not yet annotate the notification with "correlation siblings this window" (the exercise-05 §"Correlation-aware dispatching" stretch goal). A reviewer asks whether the drill should have been held open until this feature was built. Per chapter 05, what is the chapter's framing of this? (*Chapter 05 §"Runbooks" and §"The routing table" pin that runbooks and routing are versioned with the signal contract; drill findings become diffs on the contract, the runbook, or the dispatcher. A drill that identifies a gap files the gap as a diff candidate and closes — holding the drill open until every dispatcher feature lands would defeat the purpose of the drill (to exercise the wire as it is, catch gaps, and close with a filed finding). The chapter's discipline is that the dispatcher is **generic**, the per-row routing data is on the contract row, and features that add generic capabilities to the dispatcher (correlation annotation, retry semantics) are scope items on exercise 05 — not blockers on any specific drill. Drill-driven findings are how the contract matures; a "drill closed with no findings" is also an acceptable outcome, but a "drill closed with filed findings" is more valuable.*)

4. A product manager reads the Section 4 propagation trace and asks "if the labelling loop is owned by peer L30 (`ai-eval-engineer`), why does the risk engineer own the record mutation step?" Per chapters 03 and 06, what is the split? (*Chapter 03 §"The boundary to `ai-eval-engineer`" and chapter 06 §"The peer boundary to `ai-eval-engineer` at L30" pin the split: the **depth** of the labelling protocol — the LLM-as-judge calibration, the human-in-loop protocol, the inter-rater reliability — is L30; the **risk-side hook** — the `false_positive` field on the record, the propagation into the near-miss counter's read path, the register effect of the FP mutation — is L25 here. The labelling loop returns a verdict; the risk engineer applies the verdict to the record; the propagation into near-miss counter / digest / security ticket / register row is the signal-contract's register-effect wire (chapter 01). Peer L30 does not write to the record directly because that would couple the protocol depth to the register's operational layer; the hand-off is the labelling-queue verdict, not direct mutation.*)

## Why this lab matters

Exercise 05 asks you to ship the routing dispatcher, the runbook set, the incident-candidate queue, the digest reducer, the SSOT verifier, a controlled-trip drill log, and the anti-pattern probe report for your real running system. If the runbook shape and the drill-log shape are new at exercise-05 authoring time, the budget blows on deciding what a runbook contains instead of on integrations (PagerDuty, the security ticket tracker, the digest channel) and on the SSOT verifier's assertion rules. This lab drills the runbook's six-section shape and the drill log's five-section end-to-end walk on a fixed fabricated controlled trip so that exercise 05's time goes into the engineering judgement: which paging service, how retries work, how the verifier asserts, and which per-row routing exceptions sit on the contract rather than in the dispatcher's code. The runbook shape it drills is the on-call's five-minute contract; the drill-log shape is what an auditor reads to confirm the three audiences share truth. Getting these right here cascades through every S1 row the real system ships.
