# Exercise 02 — Council Pre-Read and Decision-Log Drill

**Estimated effort:** 2 hours

## Objective

Assemble a full council-facing packet for a mock quarterly AI governance council session on the running system: a pre-read deck with three decision slides in the seven-field shape, a one-pager that indexes them, decision-log entries per council-approved decision, and audit-durable meeting minutes. The output is the packet chapter 02 describes end-to-end — every field named, every artefact hashed, every decision reversible.

## Prerequisites

- Chapter 02 read closely — the decision-slide template's seven fields, the one-pager's shape, the decision-log schema, and the minutes template are the artefacts you populate.
- Exercise 01's programme charter — you use its decision matrix to decide which decisions belong at council scope vs. which are handled at row / system scope.
- Register aggregation reports from mod-103 chapter 05 — heat map, above-appetite list, control-defeated list, coverage grid — these are the raw material for context slides.
- Incident log from mod-111 — you need a summary of incidents and near-misses since the (mock) last council.
- Guardrail effectiveness numbers from mod-108, monitoring digest from mod-109, red-team run reports from mod-104 — these are evidence pointers on decision slides.

## Problem statement

Pick a **mock council date** (an absolute date, e.g., 2026-Q3 council on 2026-09-24). Assemble the full packet the risk council would receive one week in advance, plus the minutes filed 48 hours after the meeting.

The packet has **three decisions** the council is being asked to make. Choose them to span the shape of real council work:

1. **A tier promotion.** Approve promotion of one system in the programme from one tier to the next (e.g., T2 → T3). Uses the deployment-tiering shape from chapter 04.
2. **An above-appetite acceptance.** Approve a residual-risk exception where a scored row from the register sits above the appetite band, for a fixed window, with named mitigations and monitors.
3. **A mute or retirement decision.** Approve muting or retiring a monitor, a guardrail, or a tripwire — the class of decision that quietly erodes the programme if it happens without council visibility.

If your running system does not have three such decisions naturally, invent them consistently with the system's real state (say so in a "scenario is illustrative" note at the top of the packet). Do not invent register numbers or evaluation metrics — cite `<!-- needs-research: ... -->` where a real number is unknown.

## Requirements

Produce four artefacts, all in one folder `mod-112-ex02-council-packet-2026-qN/`:

### 1. Pre-read deck — `pre-read.md` (or slides.md)

- 2-4 **context slides**: programme scope + systems in scope; changes since last council; register-health heat map thumbnail; incident summary since last council.
- **Three decision slides**, each populated in the seven-field template from chapter 02:
  - Decision requested (one imperative sentence).
  - Scope (system, population, time window).
  - Evidence pointers (register rows, evaluation reports, monitor extracts, red-team findings — cited by ID and hash where available).
  - Options and recommendation (A / B / C or approve / defer / reject; recommendation with ≤ 3 lines of rationale).
  - Residual-risk claim (level after the decision if approved, per the programme's rubric).
  - Escalation dependency (any decision this depends on).
  - Rollback / reversibility (how the decision is reversed, under what condition).
- 2-4 **signal slides**: monitoring digest; obligations landing next quarter; staffing / capability posture. No decisions asked on signal slides.

The deck is versioned and hashed. Include a header line: `Pre-read version: v1, date: YYYY-MM-DD, hash: sha256:...` (a placeholder hash is fine for the drill; note that a real packet would hash the rendered PDF).

### 2. One-pager — `one-pager.md`

Single side of A4 (or landscape slide). Sits at the top of the packet. Follow the seven-section template from chapter 02:

- Header (programme name, council date, version, hash).
- Decisions requested (numbered list, one line each, linked to the corresponding decision slide).
- Position on register health (heat map thumbnail; number of rows above appetite; delta since last council).
- Position on incidents (counts by severity; open post-mortems).
- Position on obligations (any change to residual claim or obligation status).
- Escalations pending (notifications, not decisions).
- Signatories (risk engineer as author; risk council chair as post-meeting countersign).

### 3. Decision log entries — `decision-log-deltas.yaml`

One YAML entry per council decision, in the schema from chapter 02. Include at minimum:

- `decision_id`, `title`, `requested_by`, `scope`, `decision` (approve / approve-with-conditions / defer / reject), `conditions`, `decided_at`, `council_members_present`, `dissents`, `evidence_pack_ref`, `evidence_pack_hash`, `minutes_ref`, `linked_register_rows`, `linked_obligations`, `followups`, `supersedes` (if any), `next_review`.

Three entries — one per decision. Where a decision is `defer`, the entry still records the deferral (the council decided to defer, and the reason).

### 4. Minutes — `minutes.md`

Follow the minutes template from chapter 02:

- Header (date, chair, attendees, apologies, quorum, pre-read version+hash, one-pager version+hash).
- Standing items (register health, incident summary, obligations update).
- Decisions section (one line per decision-log ID with outcome).
- Discussion section (non-decision items surfaced from signal slides).
- Actions section (numbered action IDs with owner and due date).
- Sign-off section (chair, risk engineer, one other attendee).
- Attachments section (hashes for pre-read pack, one-pager, decision-log deltas).

## Starter guidance

- Do the one-pager **last**. It is a decision index, not a summary; you cannot index decisions until the decision slides are drafted.
- On decision slides, do the seven fields in order. If a field is empty, the slide is not ready — chapter 02's discipline. Better to drop a decision from the packet and note it as `not ready` than to circulate an incomplete slide.
- On the above-appetite decision, the rollback field is *how the acceptance expires* — a date, a condition that revokes the exception, or an evidence event that closes the row. "The exception is time-boxed to 90 days" is a rollback; "we will revisit" is not.
- On the mute / retirement decision, the reversibility field is *what un-mutes the monitor* — the exact condition that re-activates it. This is the field auditors read most carefully, because muted monitors are how programmes go silent.
- Evidence pointers are IDs, not descriptions. `HR-CS-INJ-01` and `ART-CS-GRD-Q2-01` beat "the injection row and the guardrail report." If you invent an ID for the drill, use the naming convention from your programme.
- If the running system is illustrative rather than real, say so at the top of the pre-read — but keep all the evidence pointer *shapes* consistent with what a real packet would carry.
- The minutes reference the pre-read and one-pager **by hash**, not by name. Placeholder hashes are fine for the drill; the discipline is that a hash exists.

## Acceptance criteria

The packet is accepted when:

- The pre-read deck carries context slides, three complete decision slides (all seven fields populated on every slide), and signal slides — with no mixing of context / decision / signal.
- The one-pager is a single page, indexed to the decision slides, and versioned with the pre-read.
- The decision log has three entries in the chapter 02 schema; every field is populated; `evidence_pack_ref` and `evidence_pack_hash` cite the pre-read pack; `linked_obligations` cites at least one framework citation per decision.
- The minutes reference the pre-read and one-pager by hash, distinguish decisions from discussion, and carry at least three signatures (chair, risk engineer, one other).
- Every approved decision has a **rollback / reversibility clause** — a decision without one is not accepted (chapter 02's failure mode: missing rollback).
- The above-appetite decision has an expiry (date or condition), not an open-ended acceptance.
- The mute / retirement decision names the exact condition under which the monitor / guardrail / tripwire is re-activated.
- A council member who reads only the one-pager can walk into the meeting knowing what is being decided; a council member who reads the full pack can vote inside the meeting.

## Stretch goals

- Add a **dissent scenario** to one of the three decisions — one attendee (name the role) records a reservation. Populate the reservation text in the decision-log entry and reflect it in the minutes. This is the shape a council record takes when the vote is not unanimous; it is the field an auditor reads for governance-of-governance evidence.
- Draft the **council charter excerpts** the packet cites — quorum rule, attendance requirements, decision-authority thresholds. The packet references these in the minutes header; a real programme has them in a linked council charter. Even a two-page draft closes the loop.
- Do the **audit read-out drill**: pretend to be an ISO/IEC 42001 lead auditor arriving on-site. Read the packet cold and write a one-paragraph audit note per artefact — "does the pre-read distribution meet clause 9.3 evidence expectations; do the minutes carry the sign-off required by the council charter; is the decision-log queryable." The gaps you find are the packet's real acceptance criteria.
- If the mock decision includes a **framework obligation** (e.g., EU AI Act Article 13 transparency), draft the addendum the decision-log entry would cite — the transparency notice, the operator-facing disclosure, or the customer-facing residual claim. This is what mod-101 ch 04 obligations look like when they cash out in a council-approved artefact.

## Deliverable format

A folder `mod-112-ex02-council-packet-2026-qN/` containing:

- `pre-read.md`
- `one-pager.md`
- `decision-log-deltas.yaml`
- `minutes.md`
- `README.md` — one paragraph explaining the mock scenario, the three decisions, and any `illustrative` disclaimers.

## Submission

Commit the folder to the programme's council-packet repository (or the equivalent workspace). If the drill is done inside a real programme, do not check placeholder hashes into the audit-facing repo — mark the folder `drill-YYYY-QN` and file it in a training subfolder. A drill packet in the audit-facing pack contaminates the evidence trail.
