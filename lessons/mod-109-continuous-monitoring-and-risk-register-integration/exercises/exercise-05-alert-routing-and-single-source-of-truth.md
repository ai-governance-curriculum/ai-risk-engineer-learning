# Exercise 05 — Alert Routing and Single Source of Truth

**Estimated effort:** 2 hours

## Objective

Wire the routing so that **on-call**, **security**, and the **governance council** all consume the same tripped-signal records shipped in exercises 02 and 03 — none composes its own list, none reads a divergent data source. Author the per-row **runbooks** the on-call reads on a page. Build the **incident-candidate queue** as the single wire from monitoring into mod-111 incident response. Walk a controlled trip end-to-end and confirm every audience sees the same record.

## Prerequisites

- Chapter 05 read in full. The single-source-of-truth definition, the three-audience view, the incident-candidate queue, the routing-table pattern, the runbook shape, and the anti-pattern list are the reference shapes.
- Exercises 01, 02, and 03 completed. The signal contract exists, the detectors are shipped, and the tripped-signal record store is populated with real records from either production or a replay.
- A **paging / notification system** available for on-call routing (PagerDuty, Opsgenie, or a Slack-plus-webhook equivalent for the exercise scope). The plumbing itself is peer L25 (`ai-infra-mlops`); the routing decisions are yours.
- A **security-incident tracker** where tickets can be created programmatically (Jira Security, ServiceNow, or an equivalent). The tracker's schema needs a foreign-key field for the tripped-signal record identifier.
- A pointer to the **governance-council digest cadence** and delivery channel (weekly email, monthly PDF, permanent dashboard). Mod-112 owns the digest depth; the exercise wires the reduction that feeds it.
- The **DPO (or equivalent privacy owner)** by role name, for the PII-linked signal rows from exercise 03.

## Problem statement

Ship the routing artifact set for the signal contract. Every audience derives its view from the same tripped-signal record store; every S1 signal produces an incident candidate that lands in the queue; every signal has a runbook the on-call can execute in five minutes.

### The routing artifact set

Chapter 05 §"The routing table — the concrete contract" pins the per-severity routing shape. The exercise ships:

- A **routing dispatcher** that reads a tripped-signal record, looks up the row's severity and routing block from the signal contract, and fan-outs the appropriate notifications (on-call page, security ticket, governance-council digest entry) with the record identifier as the foreign key.
- A **runbook per contract row** covering signal meaning, first-check list, escalation criteria, containment options, false-positive recording, and back-links to the register row and the record store.
- An **incident-candidate queue** that receives S1 trips (automated), qualifying S2 trips (rule-based), near-miss threshold breaches, and human-created candidates. Every item is triaged and every disposition is recorded.
- A **governance-council digest reducer** that reads the tripped-signal record store + the register + the incident register and emits the per-cadence digest with the content named in chapter 05 §"The governance-council digest."
- A **single-source-of-truth verifier** that walks a controlled trip end-to-end and asserts that on-call view / security view / digest view all resolve to the same record fields — no re-typing, no divergence.

### The controlled trip

Chapter 05 pins that runbooks and routing are only trusted when they have been walked end-to-end. Pick one signal contract row (typically the injection row `SC-CS-INJ-01` from chapter 03) and:

- Fire the detector with a synthetic payload that trips it.
- Capture the tripped-signal record identifier.
- Confirm the routing dispatcher fanned out to the on-call channel, the security ticket, the governance-council digest queue, and the incident-candidate queue — all keyed to the same record identifier.
- Confirm the runbook on the paging notification links back to the record store, the register row, and the incident-candidate queue entry.
- Simulate the on-call acknowledging, escalating, and eventually closing; confirm every downstream view updates from the record.
- Simulate a false-positive reclassification and confirm the near-miss counter (exercise 03) recomputes, the register-side count decrements, and the digest reflects the correction.

## Requirements

Produce all seven items:

1. **Routing dispatcher** — `risk/mod-109/<system-id>/routing/dispatcher.py`. Reads a tripped-signal record, resolves the row's routing block from the signal contract file (chapter 01), and fan-outs to the correct notification channels. Per-severity routing table applied uniformly per chapter 05 §"The routing table"; per-row exceptions read from the row itself.
2. **Runbook set** — `risk/mod-109/<system-id>/runbooks/<contract-id>.md`, one file per contract row. Shape per chapter 05 §"Runbooks — the on-call's contract": signal identifier at the top; one-line signal meaning; first-check list (three to five items, five-minute time budget); escalation criteria; containment options; false-positive recording protocol; back-links (register row, monitor dashboard, record store location, incident-response playbook pointer).
3. **Incident-candidate queue** — `risk/mod-109/<system-id>/queue/` (directory) and `risk/mod-109/<system-id>/queue/README.md` (schema + intake rules). Every candidate has `record_id`, `contract_id`, `register_row_id`, `severity`, `source` (automated / human), `triage_state`, `disposition`, `disposition_reason`, `promoted_to_incident_id`, timestamps. The queue's writer is the routing dispatcher (for automated candidates) and a small `add_candidate.py` CLI (for human-created candidates).
4. **Governance-council digest reducer** — `risk/mod-109/<system-id>/digest/reduce_digest.py`. Reads the tripped-signal record store, the register, and the incident register; emits the per-cadence digest with the chapter 05 §"The governance-council digest" content (top-K signals by trip count, register rows whose residual moved, above-appetite rows and treatment state, open incidents and Article 73 status, overdue corrective actions, signal-contract changes, upcoming report deliverables).
5. **Single-source-of-truth verifier** — `risk/mod-109/<system-id>/routing/verify_ssot.py`. Given a tripped-signal record identifier, resolves the on-call notification payload, the security ticket payload, the digest entry, and the queue entry; asserts each carries the record identifier as a foreign key and each derives its fields from the same underlying record. Reports discrepancies.
6. **Controlled-trip drill log** — `risk/mod-109/<system-id>/drills/<yyyy-mm-dd>-controlled-trip.md`. The end-to-end walk from the "problem statement" above, with the record identifier, timestamps, screenshots or terminal captures of each audience view, the false-positive reclassification, and a "no discrepancies found" or "discrepancies filed as: ..." closing statement.
7. **Anti-pattern probe report** — `risk/mod-109/<system-id>/routing/anti-patterns.md`. Six short sections, one per chapter-05 anti-pattern:
   - **Slack-only alert.** Confirmed every routing block fans out beyond the on-call channel; no row is Slack-only.
   - **Parallel dashboards.** Confirmed every audience view queries the same record store or a projection of it, not a parallel data source.
   - **Silent mute.** Confirmed muting a channel does not stop the record from being written or the downstream views from updating.
   - **Un-runbooked signal.** Confirmed every contract row has a runbook file that the dispatcher links from the notification.
   - **Re-typed digest.** Confirmed the digest is generated by the reducer with narrative-only human editing around generated numbers.
   - **Un-propagated false positive.** Confirmed the controlled-trip drill exercises the false-positive-reclassification path and the propagation is verified end-to-end.

Cross-cutting rules:

- **One record, many views.** Chapter 05 §"What 'same source of truth' means" pins this. The dispatcher does not embed metric values in the notification body — it embeds the record identifier and lets each audience view fetch from the record. This is what makes corrections propagate.
- **Runbooks are versioned with the signal contract.** A threshold change is a diff on both the contract row and the runbook. Do not let them drift.
- **The queue is auditable.** Every disposition is recorded with a reviewer and a timestamp; dismissed candidates carry a reason; promoted candidates carry the mod-111 incident identifier. This is what makes the queue the wire, not a scratch list.
- **Severity is set on the contract, applied uniformly by the dispatcher.** Per-row exceptions are on the row (chapter 05 §"The routing table"). Do not let the dispatcher's code encode per-row exceptions — the code is generic, the exceptions are data.
- **The DPO is on the routing for every PII-linked row by name.** Chapter 03 §"PII / sensitive-data leakage" and chapter 05 both pin this. Verify in the controlled-trip drill.

## Starter guidance

- **Do not build a paging system.** PagerDuty and Opsgenie and the like are the plumbing (peer L25). Ship the dispatcher as a caller of their API; the routing block on the row names which service to page, not a phone number.
- **The runbook first-check list is five sentences at most.** Chapter 05 §"Runbooks" pins the five-minute budget. If your first-check list is fifteen items, the on-call will read the top three and act; the rest is unread. Trim.
- **The record identifier is the foreign key everywhere.** Security tickets carry it. Notification bodies link it. Queue entries reference it. Digest lines cite it. This is what "single source of truth" means in practice — one identifier, many audiences.
- **False-positive tagging is a mutation on the record, not a note in a channel.** The verifier's job is to catch the reverse — a record that is *not* tagged when a Slack thread says it is. The mutation is the propagation vector.
- **The digest reducer is a small script.** It reads the tripped-signal record store, groups by contract row, sorts by count, filters residual movement, joins to the incident register, and emits markdown. A LLM is not required and is a temptation to avoid (chapter 05 §"Anti-patterns"). Keep it deterministic.
- **The controlled-trip drill has to touch every audience.** The security ticket has to exist and have the record identifier. The digest queue has to receive the entry. The on-call channel has to page. If any of the three does not fire, the discrepancy is on the row's routing block or on the dispatcher — file the fix before closing the drill.
- **The rehearsal fires and the incident-candidate open must be distinct.** A dispatched notification is not an incident; the candidate is the wire *toward* mod-111. Chapter 05 §"The incident-candidate queue" pins the shape.
- **The verifier is what an inspector reads.** A green verifier report is what tells the auditor "these three audiences share truth." Run the verifier in CI on every contract change so drift is caught early.

## Acceptance criteria

The submission is accepted when:

- The dispatcher reads the signal contract, resolves per-severity routing, and fan-outs to on-call + security + digest queue + incident-candidate queue with the record identifier as the foreign key.
- Every contract row has a runbook file in the chapter-05 shape; the notification body links the runbook.
- The incident-candidate queue exists with the required schema; automated candidates land from the dispatcher; human candidates land from the CLI; every disposition is recorded.
- The digest reducer emits the per-cadence digest with the chapter-05 content and cites the record identifiers.
- The single-source-of-truth verifier runs against the controlled-trip record and reports "no discrepancies."
- The controlled-trip drill log walks a real fire (or a synthetic-payload fire on the injection row) end-to-end and confirms every audience view derives from the record.
- The false-positive reclassification path is exercised and the propagation into the near-miss counter and the digest is verified.
- The anti-pattern probe report resolves each chapter-05 anti-pattern with a named row-level check.
- The DPO is on the routing block of every PII-linked row; the drill confirms.

## Stretch goals

- **Correlation-aware dispatching.** For contract rows on the correlation memo from exercise 03 (`SC-CS-INJ-01` ↔ `SC-CS-COST-01`, etc.), the dispatcher annotates the notification with sibling record identifiers when a sibling has tripped in the same window. This turns "two pages" into "one page with a correlation note" for the on-call.
- **On-call feedback loop.** After every real trip, the on-call fills a two-question form (was the runbook actionable? was the routing appropriate?) whose answers land on the record. A quarterly review of the answers drives runbook and routing updates. Chapter 05 §"Runbooks" pins the versioning discipline; this is the read side.
- **Digest reader survey.** Once per quarter, ask the governance-council digest readers whether the current content shape answers their questions. Route the feedback into signal-contract or reducer changes; keep the digest content responsive to the council's actual decision needs, not to a template.
- **Article 73 wire hookup.** Wire the incident-candidate queue's promotion event into the exercise-04 Article 73 log — when a candidate is promoted to an incident and triaged as serious, the log is written automatically. Chapter 04 §"Dovetail with Article 73" pins the five-step wire; this closes step 5's automation.
- **Retry / failure semantics for the dispatcher.** The paging system can be down. The security ticket API can be rate-limited. Author retry / backoff / dead-letter semantics for the dispatcher so a downstream outage does not silently drop a notification. The dead-letter queue itself is monitored (a control-availability signal on the routing pipeline; chapter 06 §"Two shared surfaces").
- **Runbook simulator.** For each S1 row, a scripted simulator that walks the on-call through the first-check list in a chat interface, records the answers, and produces the same tripped-signal-record update as a real ack. Use for training and for keeping the runbook accurate as the system evolves.

## Deliverable format

A single markdown document `mod-109-ex05-routing-and-ssot.md` that indexes the artifacts. The index contains:

1. *Contract rows in scope* — the routing blocks quoted verbatim for at least the S1 / S2 rows.
2. *Dispatcher* — pointer to the module; describe how per-severity routing is resolved.
3. *Runbook set* — pointers to each runbook file, with a one-line summary per row.
4. *Incident-candidate queue* — schema quoted; sample records inlined.
5. *Digest reducer* — pointer to the module; a sample digest inlined verbatim for one period.
6. *Verifier report* — the "no discrepancies" output for the controlled-trip record.
7. *Controlled-trip drill log* — inlined verbatim, with the record identifier and the audience views.
8. *False-positive propagation trace* — the record's `false_positive: true` mutation and the downstream deltas (near-miss counter, digest).
9. *Anti-pattern probe report* — the six chapter-05 sections resolved.
10. *Open items* — correlation-aware dispatching, feedback-loop hookups, Article 73 automation, dead-letter handling — whatever is deferred.

## Submission

Attach the index markdown plus the seven artifacts. This exercise closes mod-109: exercises 01-03 built the contract and shipped the detectors; exercise 04 produced the regulatory report; exercise 05 makes the whole assembly readable by the three audiences it has to reach. Mod-111 picks up the incident-candidate queue on the incident-response side; mod-112 picks up the digest on the governance-council side. The wire out of this module is what those two modules read.
