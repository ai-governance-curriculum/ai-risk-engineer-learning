# Exercise 01 — Risk Programme Charter for One Team

**Estimated effort:** 2 hours

## Objective

Author the **programme charter** for the running system's team — the single artefact that names scope, cadence, artefact bill, decision matrix, sign-off routing, RACI, and escalation criteria for a level-25 AI risk programme. The charter is the load-bearing artefact chapter 01 keeps citing: everything downstream (council packet in exercise 02, card in exercise 03, tiering in exercise 04, crosswalk in exercise 05) refers back to what the charter *authorises the programme to own*.

## Prerequisites

- Chapter 01 read closely — the four-tier cadence, the artefact bill of materials, the decision matrix, the sign-off graph, and the RACI shape are the skeleton the charter fills in.
- Mod-101 chapter 06 (deferral contract) re-skimmed — the charter needs to name what escalates to `ai-evaluation-engineer` (L35), `agentic-safety-engineer` (L40), and `head-of-ai-governance` (L60).
- Mod-103 chapter 06 (register operations) re-skimmed — the charter's sign-off routing extends the register's row-level sign-off pattern up to system and programme scope.
- Access to the running system(s) — the same system used in mods 101-111. If your programme covers more than one system, list them all; the charter is for the *programme*, not a single system.
- Enough visibility into the team to name the actual attendees for each cadence tier — you can redact names but not roles.

## Problem statement

Draft a programme charter for the team that owns the running system(s), covering all six sections below. The charter is the artefact a new joiner reads to answer *what does this programme own right now, on what cadence, with whom, and what escalates*.

The charter is **prescriptive** — it names the cadence, attendees, and sign-off routing your team commits to. It is **not** aspirational — do not include cadence tiers your team will not actually hold, or artefacts that do not exist upstream.

## Requirements

The charter is a single markdown document, `mod-112-ex01-programme-charter.md`, with these sections:

### 1. Scope

- Programme name.
- Systems in scope (one line per system: system name, deployment tier, business owner, risk-engineering owner). If the programme has more than one system, use the same shape for each.
- Systems explicitly *out of scope* — if there are adjacent AI systems in the organisation that this programme does *not* cover, name them and name who does own them. This closes the "who owns X" question that comes up at every council.
- Regulatory / obligation surface currently in scope (per mod-101 ch 04-05) — e.g., EU AI Act articles for the system's classification, SR 11-7 attributes, sector overlays, jurisdictional carve-outs.

### 2. Cadence

Populate a table matching chapter 01's four-tier cadence, but with *your team's actuals*:

| Cadence | Duration | Chair | Standing attendees | Rotating attendees | Standing agenda | Refreshed artefacts |
|---|---|---|---|---|---|---|
| Weekly | | | | | | |
| Bi-weekly | | | | | | |
| Monthly | | | | | | |
| Quarterly | | | | | | |

If your team cannot actually hold four cadence tiers (small team, low system count, part-time role), collapse them — but *say so explicitly* and note which tier absorbs the collapsed agenda. A three-tier collapsed cadence with a named rationale is defensible; four tiers on paper that no-one attends is not.

### 3. Artefact bill of materials

For each artefact in chapter 01's bill of materials (fifteen entries: harm model, register, red-team programme, adversarial-ML evaluation, fairness / bias / explainability, privacy-risk evaluation, guardrail effectiveness, monitoring digest, MRM / supply-chain evidence, incident log and post-mortems, programme charter, council-facing packet, model / system / dataset cards, deployment tiering, framework crosswalks), record:

- **Status**: exists / partial / not yet built.
- **Owner**: named role (redact person if needed).
- **Refresh cadence**: on which tier does it get updated.
- **Location**: repo path, GRC platform reference, or wiki link.
- **Upstream module**: which of mods 101-111 authored the underlying evidence.

If an artefact is `partial` or `not yet built`, add a `next-refresh-target` field with a date. The bill of materials is what the charter commits to holding *live*.

### 4. Decision matrix

Instantiate chapter 01's decision matrix for your team. Rows are decisions; columns are `owner at team scope`, `escalates to`, and `escalation trigger`. At minimum, cover the thirteen decision classes from chapter 01. Add any additional decisions specific to your programme (e.g., "approve a new data-source contract with lawful-basis implications").

Every row must resolve — no decision is left unowned. If a decision does not apply to your programme (e.g., you have no agentic variants and no autonomy-escalation decisions), delete the row and note the deletion in a "decisions not applicable" section below the matrix.

### 5. Sign-off routing

Draw the sign-off graph as a nested list (or a diagram if you prefer). At minimum:

- Row-level decisions: two signatures minimum. Name the two roles.
- System-level decisions: three signatures minimum. Name the three roles.
- Programme-level decisions: risk council countersign on top of system-level three. Name the council contact.
- Escalation exits: legal, frontier / dangerous-capability, control-library architecture. Name the escalation contact per exit.

The routing is **append-only** (mod-103 ch 06). Note this discipline explicitly — a revoked sign-off is a new sign-off event, never an edit.

### 6. RACI across the six functions

Fill out chapter 01's RACI table with your team's actual role names. If a function is not represented on your team (e.g., no dedicated infra / MLOps partner because ML lead covers it), collapse the columns and note the collapse.

Every work item must have exactly one `A` (accountable). Multiple `R` (responsible) is fine; multiple `A` is a governance smell.

## Starter guidance

- Chair the exercise from *what your team actually does*, not what a template says a programme should do. A charter that overstates cadence is worse than one that admits a three-tier collapse — the auditor and the new joiner will catch the fiction faster than they will fix it.
- If you cannot fill a cell (unknown owner, unclear escalation contact, missing artefact), write `<!-- needs-research: ... -->` in the cell rather than inventing. This is what the module's chapters do; the charter should do the same.
- Convert relative dates to absolute — "quarterly" is fine as a cadence label, but "next refresh: end of Q3" should be "next refresh: 2026-09-30".
- Do not invent decisions your programme does not own. If your team does not touch red-team engagement scope (a security partner owns it end-to-end), delete that row and note the escalation.
- The escalation contacts to `agentic-safety-engineer` (L40), `senior-ai-governance-architect` (L50), and `head-of-ai-governance` (L60) may be role labels rather than named individuals — that is fine, but the *route* must be named ("escalate via risk council chair to head of AI governance"), not implied.

## Acceptance criteria

The charter is accepted when:

- All six sections are populated for the actual team and system(s), with no `TBD` cells (either `needs-research` or a real value).
- The four-tier cadence table has actual attendees, chairs, and agendas — no template placeholders.
- The artefact bill of materials names every artefact with status, owner, cadence, location, and upstream module; any `partial` or `not yet built` entry has a next-refresh-target date.
- The decision matrix covers at least chapter 01's thirteen decision classes; every row resolves; deletions are noted.
- The sign-off routing names actual roles per tier, not "TBD" or "someone from security."
- The RACI table has exactly one accountable per work item; any function collapse is noted with rationale.
- A new joiner reading the charter can answer, without asking: *which system(s) does this programme own, on which cadence, with which artefacts live, who signs off row / system / programme decisions, and what escalates where.*

## Stretch goals

- Add an **operating maturity self-assessment** — for each artefact in the bill of materials, rate maturity as `initial / repeatable / defined / managed / optimising` (or equivalent). This is the input a level-15 analyst uses when the programme is asked to *report on itself* to the council. Charts of maturity by artefact are what a head of AI governance reads.
- Draft the charter's **change log** header — the discipline that says "this charter is versioned; every material change is recorded with rationale, date, and sign-off." This is what makes the charter itself an audit-durable artefact rather than a wiki page.
- Author a **stress-test scenario**: pick one decision from the matrix that has never actually been exercised (e.g., "mute of a tripwire monitor" or "approve promotion into T3"), walk it through the routing you drew, and note any place the routing broke. Charter revisions come from stress tests, not from calm cycles.
- If your programme covers more than one system, add a **cross-system dependencies section** — where one system's failure would trigger a review of another system (e.g., shared embedding model, shared guardrail library). This is what mod-110 ch on shared-component review reads.

## Deliverable format

A single markdown document with these sections, in this order:

1. *Programme name and scope* — one paragraph.
2. *Systems in / out of scope* — table.
3. *Cadence* — table.
4. *Artefact bill of materials* — table.
5. *Decision matrix* — table with deletions noted.
6. *Sign-off routing* — nested list or diagram.
7. *RACI* — table.
8. *Escalation contract summary* — one paragraph naming the L35 / L40 / L50 / L60 routes.
9. *Charter version, date, and sign-off* — one block.
10. *Open questions and `needs-research` items* — bulleted list.

## Submission

Commit the charter to the running-system programme repo (or the GRC platform's programme workspace). Circulate for review to at least one other role named in the RACI — the review event itself is the first sign-off on the charter, and the review's comments are the first stress test.
