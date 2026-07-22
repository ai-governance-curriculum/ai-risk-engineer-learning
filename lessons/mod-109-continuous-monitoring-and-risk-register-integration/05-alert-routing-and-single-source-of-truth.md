# Alert Routing and the Single Source of Truth

An alert that only the on-call sees is a service metric. An alert that reaches the on-call *and* the security team *and* the governance council — all keyed to the same identifier — is a governance signal. This chapter says what "the same source of truth" means as an engineering property, and how the routing is built so no audience learns of a signal via a different data model than the others.

## Motivation

Three failure modes recur, all producible from a routing that looks fine on a whiteboard:

- **The Slack-only alert.** The alert goes to `#ml-oncall`, the on-call acknowledges it, the incident is resolved in the channel, no register row is updated, no governance-council digest picks it up. The engineering answer looks correct; the governance answer never happens.
- **Divergent truths.** The on-call has a Grafana view; the security team has a SIEM view; the governance council has a quarterly PDF. Each view is derived from different queries; when a question arises ("was the injection spike on 12 May actually an incident?"), the three audiences answer differently.
- **Silent muting.** An alert is noisy, the on-call mutes it, the register row keeps recording *tripped* status; three weeks later the mute is discovered and the register has been silently wrong.

The engineering fix is the "single source of truth" discipline: every audience derives its view from the same register + signal contract, and routing is a fan-out from that single record — not a set of parallel notifications each carrying its own copy of the facts.

## What "same source of truth" means, precisely

Concretely, the property is: for any signal-tripped event, there is *one* record — the tripped-signal record filed against the signal contract row identifier — and every downstream artifact (on-call page, security ticket, governance-council digest line, Article 72 report entry, register-row update, evidence-manifest pointer) is derived from that one record. If you change the record (correction, false-positive tagging, closure), every derivative view reflects the change on the next refresh.

The property implies four operating rules:

1. **No re-typing.** No audience re-enters facts from another audience's view. Fan-out from the record is automatic.
2. **No silent muting.** Muting an alert channel does not delete the record; the record is still updated and downstream views (register, digest, report) still reflect it. Muting is a routing decision, not a truth decision.
3. **No parallel dashboards.** If two dashboards claim to show the same signal, they must query the same underlying record. Different queries against the same record are fine; different sources of the same facts are not.
4. **Corrections propagate.** When a tripped signal is re-labelled false positive (during the calibration loop), the correction touches the one record, and every derived view (near-miss count, register update, report line) recomputes.

## The three audiences and what each of them needs

Alerts reach three audiences with different tempos, formats, and permissions:

| Audience | Tempo | Format | Needs |
|---|---|---|---|
| **On-call engineer** (ML on-call, application on-call, SRE) | Streaming / immediate | Page, chat, PagerDuty-style tool | Enough context to triage in five minutes; link back to the record |
| **Security team** (SecOps / AppSec / DPO for privacy signals) | Streaming for S1 / S2; digest for S3 | Ticket in the security-incident system, plus channel notification | The record, the raw payload sample (if permissible), the correlation with other security signals, ownership of any Article 73 pathway |
| **Governance council** (risk council, product owners, compliance) | Weekly / monthly digest; ad-hoc for S1 | Digest email / dashboard / packet | Trend view over signals; register-row residual state changes; corrective-action state; upcoming report deliverables |

Each audience's view is generated from the tripped-signal records and the register. None of the three composes its own list from scratch.

### The on-call view

The on-call is the first responder. Their view must include: signal contract identifier, monitor identifier, threshold, the specific tripped value, timestamp, register row(s) affected, current residual state, runbook link, escalation path, and the record's evidence-pointer location. The on-call's actions (ack, snooze, escalate, close) update the record; the update propagates.

The runbook link is not optional. A signal without a linked runbook is a signal the on-call has to invent a response for.

### The security view

The security team consumes:

- S1 and S2 signal trips in the safety-monitor family (injection, jailbreak, PII, out-of-scope tool call) as tickets in the security-incident system, with the record's identifier as the ticket's foreign key.
- Correlation with other security signals in the SIEM. For that correlation to work, the tripped-signal record has to emit a security-side event with a stable schema (identifier, signal type, severity, timestamp, affected system, correlation tags).
- For PII-linked signals, the DPO is on the routing by name.
- For any potentially serious incident under Article 73, security is on the triage with legal counsel.

The security view is not a separate risk register. It is a specialised consumer of the same records.

### The governance-council digest

The digest is the periodic view for the audience that will *never* look at an individual signal record. Its content, in the pattern used across mod-112:

- Top-K signals by trip count over the period, with trend.
- Register rows whose residual state changed during the period, with the underlying signal.
- Above-appetite rows and their treatment state.
- Open incidents and their Article 73 status (if applicable).
- Corrective actions overdue.
- Changes to the signal contract or the register-scoring rubric.
- Upcoming report deliverables (Article 72 report cycle, sector reports, board packet).

The digest is a reduction over the register + tripped-signal records + incidents; running it should be a one-line command in CI. When the council reads the digest, they are reading the same facts the on-call and the security team are reading, aggregated for their tempo.

## The incident-candidate queue

The wire from a tripped signal to an incident is the *incident-candidate queue*. It sits between the signal contract and mod-111's incident register.

- Any S1 signal trip emits an incident candidate (automated).
- Any S2 signal trip *may* emit an incident candidate (rule-based; e.g., sustained > threshold; or paired with another related signal).
- Any near-miss whose count crosses the row's threshold emits an incident candidate.
- Human-created candidates (support-agent report, deployer complaint, external report) land in the same queue.

The queue's items are triaged (mod-111): each is either promoted to an incident, dismissed with a reason, or held for more data. Every item's disposition is recorded; the queue itself is auditable. The queue is *the* wire from monitoring to incident response — do not create parallel wires.

## Anti-patterns

- **The Slack-only alert.** The alert reaches `#ml-oncall`; nothing else knows it happened. Fix: fan-out from the tripped-signal record; the channel notification is one derivative view, not the source of truth.
- **The parallel dashboards.** The on-call, security, and council each read a different query of a different data source that overlaps for the same event. Fix: all views query the single tripped-signal record and the register.
- **The silent mute.** The on-call mutes a noisy alert; nobody tells the register. The register still records `tripped`. Fix: muting is a routing option, not a truth option; the record continues to record the trip and the register still reflects it.
- **The un-runbooked signal.** A signal has no runbook link; the on-call improvises. Fix: every signal contract row has a linked runbook; no runbook, no signal in production.
- **The re-typed digest.** A human copies numbers from one dashboard into a slide for the governance council. Fix: the digest is a generated reduction; the human writes narrative around generated numbers, not new numbers.
- **The false-positive that never propagated.** A tripped signal was later confirmed false positive; only the on-call notes it in a Slack thread; the register-side count still includes it. Fix: false-positive tagging is on the record and propagates.

## The routing table — the concrete contract

Every signal contract row includes a routing block. Recall from chapter 01 that the block has three fields: `alert_routing`, `owner_on_call`, `escalation_path`. Fill them like this, per severity:

| Severity | `owner_on_call` | `alert_routing` fan-out | `escalation_path` on unresolved |
|---|---|---|---|
| **S1** | Named on-call role (per signal family) | On-call page + security-ticket + governance-council digest ad-hoc entry + incident candidate | Named managers on-call for the on-call function; legal / DPO on relevant families; system owner within 1 hour |
| **S2** | Named on-call role | On-call channel notification + register-row auto-update + digest weekly | Managers on second-hour non-ack; system owner on sustained > threshold |
| **S3** | Named review owner (risk engineer or ML lead) | Register-row auto-update + digest weekly (batched) | Escalate to S2 routing if sustained beyond row-specific window |

The severity is set on the signal contract; the routing table is applied uniformly; per-row exceptions are recorded in the contract row itself.

## Runbooks — the on-call's contract

A runbook per signal contract row is not documentation; it is an operational contract. The minimum runbook shape:

- Signal contract identifier and monitor identifier at the top.
- One-line description of what the signal means.
- Immediate first-check list — three to five things to look at in the first five minutes.
- Escalation criteria — when to promote from S2 to S1, when to open an incident.
- Containment options — mute, rollback, throttle, disable a specific tool call, fail-over.
- What to record on the tripped-signal record for the false-positive / calibration loop.
- Links back to: the register row, the monitor's dashboard, the raw-payload sample store (with permission notes), the incident-response playbook (mod-111).

Runbooks are versioned with the signal contract. When a signal's threshold or routing changes, the runbook is updated in the same diff.

## Worked routing — customer-support RAG assistant on `SC-CS-INJ-01`

For the prompt-injection firing-rate row (chapter 03):

- `owner_on_call`: ML on-call.
- `alert_routing` on S1 trip: ML on-call PagerDuty page; security-ticket auto-created in the security-incident system with the tripped-signal record's identifier as the ticket's foreign key; governance-council ad-hoc digest entry; incident candidate opened in the mod-111 queue.
- `escalation_path`: ML on-call manager after 15 minutes non-ack; system owner after 1 hour; security lead if attack pattern is novel; legal counsel + DPO if any PII touch; product owner for user communication decisions.
- Runbook first-check: is the fire concentrated in a single tenant / session pattern? Is the pattern in the known-attacker corpus? Are the tool-call arguments in the fires within the allow-list? Is the rate deviation matched by a quality-metric drop (`SC-CS-QUAL-01`)? Is the pattern reproducible in a controlled test?
- Containment options: throttle the affected tenant; disable the specific tool call; add the pattern to the input-side classifier's block-list; roll back to the prior classifier version if the new one lost sensitivity.
- False-positive recording: the runbook names how to mark a fire as false positive and how the false-positive-rate calibration loop reads that field.

## Interfaces with adjacent modules

- **mod-101 ch 01** — MEASURE 3.2 (near-miss / unforeseen impact tracking), MANAGE 4.1 (post-deployment monitoring routing).
- **mod-103 ch 06** — the register's operational fields include the sign-off and evidence-manifest that the routing updates.
- **mod-108** — guardrails emit the raw fires; the routing here decides what those fires *become* in the governance system.
- **mod-111** — the incident-candidate queue is the wire *into* mod-111's incident response.
- **mod-112** — the governance-council digest is a mod-112 output; its data model is what this chapter enforces the audiences to share.

## Summary

- Single source of truth means: one tripped-signal record per event, from which every audience's view is derived.
- Three audiences — on-call, security, governance council — with different tempos and formats but the same underlying records.
- The incident-candidate queue is the wire from monitoring to incident response; it is auditable and it is the only wire.
- Anti-patterns to fix: Slack-only alert, parallel dashboards, silent mute, un-runbooked signal, re-typed digest, un-propagated false positive.
- The routing table is applied uniformly by severity; per-row exceptions are on the signal contract.
- Runbooks are versioned with the signal contract and are the on-call's operational contract.
- Exercise 05 wires the routing and the single-source-of-truth artifact set for the running system.
