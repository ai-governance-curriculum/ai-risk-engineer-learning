# Briefing the AI Governance Council — Pre-Read, One-Pager, Decision Log, Minutes

The AI governance council is the standing forum where risk-engineering evidence turns into organisational decisions. The council is *not* the risk engineer's meeting — it is chaired by `head-of-ai-governance` (level 60) with attendance from legal, security, product, and business owners. Your job at level 25 is to *brief the council*: assemble a pre-read that lets the council decide inside the meeting, a one-pager that survives the ten-minute skim before it, a decision-log entry that captures each ruling, and minutes that hold up under external audit.

## Motivation

Council meetings fail in two directions. In one, they turn into *read-out sessions* — the risk engineer walks the council through 40 slides of what happened last quarter, and no decision is taken. In the other, they turn into *ambush sessions* — a decision is put in front of the council that the council has no context to make, so the decision is deferred and the programme stalls.

Both failures are pre-read failures. When the pre-read is right, the meeting decides. The pre-read is not a report; it is a *forcing function*.

Council-facing artefacts also need to be *audit-durable*. If the ISO/IEC 42001 audit or the EU AI Act post-market conformity check asks "who decided the residual-risk acceptance for system X on date Y and what evidence was considered," the answer is a decision-log row with pointers to a versioned pre-read and versioned minutes. If that record does not exist, the finding is that the AIMS is not operating.

## The pre-read deck — "one decision per slide"

The pre-read deck is the primary artefact. It carries three kinds of slide, in this order:

1. **Context slides** (2-4 slides) — programme scope, systems in scope, changes since last council, headline register state (heat map from mod-103 ch 05), incident summary since last council (mod-111 log).
2. **Decision slides** (one per decision requested — typically 3-6) — each slide *asks the council a single yes/no or A/B/C question*, gives the evidence, and names the recommendation.
3. **Signal slides** (2-4 slides) — no decision asked; monitoring digest (mod-109), forward-look on obligations landing in the next quarter, staffing / capability posture.

The load-bearing discipline is *one decision per slide*. A slide that says "and there is also this other thing the council should know" is a signal slide, not a decision slide. Mixing them produces decisions that were not really made — the meeting nods at the slide and moves on.

### The decision-slide template

Every decision slide has the same seven fields, in the same order. Deviating slows the council down; consistency lets a council member read four decision slides in eight minutes.

| Field | Content |
|---|---|
| Decision requested | One sentence, in the imperative. "Approve promotion of the agentic ticket-triage variant from T1 to T2." |
| Scope | Which system(s), which population, which time window |
| Evidence pointers | Direct links to the register rows, evaluation reports, monitor extracts, red-team findings underneath the request |
| Options and recommendation | A / B / C (or approve / defer / reject); the recommendation from the risk engineer and the rationale in ≤ 3 lines |
| Residual-risk claim | The one-line residual claim (mod-103 ch 04 output), with the level after the decision if approved |
| Escalation dependency | Any decision this depends on (a legal ruling not yet closed, an obligation not yet mapped) |
| Rollback / reversibility | If approved, how the decision is reversed, and under what condition |

If any of the seven fields is empty, the slide is not ready for the council. Pull it, close the gap, and hold it for the next cycle.

### A worked decision slide (short form)

> **Decision requested:** Approve promotion of the customer-support RAG assistant from T2 to T3 (general external customer availability).
>
> **Scope:** Customer-support assistant, English-language cohort, EU + US customers, six-month promotion window. Excludes the agentic ticket-triage variant (T1 -> T2 in a separate decision).
>
> **Evidence pointers:** Register rows HR-CS-INJ-01 through HR-CS-DRIFT-11 (all at or below appetite after Q2 treatment); Q2 red-team run report `2026-q2-cs-rag-external.md` (external partner engagement, 3 findings closed); guardrail effectiveness report Q2 (all guardrails ≥ 0.92 F1 on retention corpus, mod-108); monitor coverage grid Q2 (all critical monitors green, mod-109 digest §3); DPIA supplementary Q2 (privacy team sign-off attached).
>
> **Options and recommendation:** (A) Approve as scoped; (B) Approve subject to a 60-day monitored ramp with revert at 5 percent traffic; (C) Defer to Q4 pending external audit conclusion. Recommendation: **B** — retains the promotion but attaches a formal ramp with pre-registered rollback criteria.
>
> **Residual-risk claim:** Inherent risk on the six above-appetite rows was reduced to below-appetite by four controls credited in Q2 (mod-108 F1 numbers cited above). Residual is stable-below-appetite at T3 subject to the mod-109 monitor set staying green during the ramp.
>
> **Escalation dependency:** None; the outstanding legal question on children's data (Q1) does not apply to this cohort.
>
> **Rollback / reversibility:** Ramp revert to T2 on any of: (i) a customer-visible incident above severity 2; (ii) monitor `mon-cs-jailbreak-rate` above 0.5 percent for 3 consecutive days; (iii) any regression on retention corpus > 5 percent. Revert is a config flip in the release pipeline, tested weekly.

That is the shape. Every decision slide reads the same way, so a council member can absorb the ask in under two minutes.

## The one-pager

The one-pager is a *single side of A4* (or a single slide in landscape) that sits at the top of the pre-read pack. Council members who skim only the one-pager should walk into the meeting able to say what is being decided.

The one-pager is not a summary of the deck; it is a *decision index*. A working shape:

| Section | Content |
|---|---|
| Header | Programme name; council date; version and hash of the pre-read pack |
| Decisions requested | Numbered list; one line per decision; each links to the corresponding decision slide |
| Position on register health | Heat-map thumbnail; number of rows above appetite; delta since last council |
| Position on incidents | Number of incidents this quarter by severity; number of near-misses; open post-mortems |
| Position on obligations | Any obligation status change (new EU AI Act delegated act; new sector overlay; ISO/IEC 42001 audit dates); any material change to residual claim |
| Escalations pending | Anything the council is being *notified of* rather than asked to decide, but which will need attention next cycle |
| Signatories | Risk engineer (author); risk council chair (countersign after the meeting) |

The one-pager is versioned with the pre-read pack. A one-pager without a version is a memo; a one-pager with a version is an artefact.

## The decision log

The decision log is the *single ledger* of every decision the council has taken. It is not the meeting minutes and it is not the pre-read; it is the durable list.

One entry per council decision, with these fields:

```yaml
- decision_id: DEC-2026-Q2-004
  title: "Promote customer-support RAG assistant T2 -> T3 (with monitored ramp)"
  requested_by: ai-risk-engineer-L25 (Ada Chen)
  scope:
    system: cs-rag-assistant
    variant: base
    cohort: EN, EU + US customers
    window: 2026-07-01 to 2026-12-31
  decision: approve-with-conditions
  conditions:
    - 60-day monitored ramp with revert at 5 percent traffic
    - rollback triggers as pre-registered (see pre-read slide 12)
  decided_at: 2026-06-24
  council_members_present: [chair, legal, security, product, business-owner-cs, cio-representative]
  dissents: none
  evidence_pack_ref: council/2026-q2/pre-read.pdf#slide-12
  evidence_pack_hash: sha256:...
  minutes_ref: council/2026-q2/minutes.md#DEC-2026-Q2-004
  linked_register_rows: [HR-CS-INJ-01, ..., HR-CS-DRIFT-11]
  linked_obligations: [NIST-RMF-MEASURE-2.5, ISO-42001-A.6.2.4, EU-AIA-Art-26]
  followups:
    - description: "Attach ramp monitor spec to mod-109 register"
      owner: ai-risk-engineer-L25
      due: 2026-07-08
      status: open
    - description: "Report ramp status weekly for first month"
      owner: ai-risk-engineer-L25
      due: 2026-07-31
      status: recurring
  supersedes: DEC-2026-Q1-002        # earlier T1 -> T2 promotion; superseded, not replaced
  next_review: 2026-Q4 council
```

The log lives *in the register repo* (or the GRC platform), not in a slide deck. Its purpose is to be queried: give me every above-appetite acceptance in the last four quarters, give me every decision tied to a specific system, give me every decision linked to NIST AI RMF MEASURE 2.5.

Decision-log discipline is what mod-103 ch 06's sign-off routing looks like at council scope. The entries in the log are the register's council-level sign-offs, made queryable.

## Minutes that survive audit

Meeting minutes are not the decision log — they are the *record of the meeting*. Audit reads them to confirm the council actually met, had the required attendees, considered the pre-read, and reached the decisions the log records.

A working minutes template:

```markdown
# AI Governance Council — 2026 Q2 meeting

**Date:** 2026-06-24
**Chair:** [head of AI governance]
**Attendees:** [named list, roles]
**Apologies:** [named list, roles]
**Quorum:** Yes / No (per council charter §X)
**Pre-read version:** council/2026-q2/pre-read.pdf @ sha256:...
**One-pager version:** council/2026-q2/one-pager.pdf @ sha256:...

## Standing items
- Register health review — heat map presented; no material change from pre-read.
- Incident summary — 1 sev-2, 3 sev-3, 7 near-misses this quarter; post-mortems attached (mod-111).
- Obligations update — noted EU AI Act delegated act on transparency landing Q3.

## Decisions
- DEC-2026-Q2-001 [tier promotion internal-limited variant] — approved as recommended; log entry filed.
- DEC-2026-Q2-004 [tier promotion RAG assistant T2 -> T3] — approved with conditions (monitored ramp); dissent: none; log entry filed with pre-registered rollback.
- DEC-2026-Q2-006 [above-appetite fairness delta acceptance, 90-day window] — approved with reservations from [legal partner]; recorded in log with reservation text.

## Discussion (non-decision)
- Signal slide on obligations pipeline; council noted the Q3 delegated act as a standing item for the Q3 packet.

## Actions
- ACT-2026-Q2-01 [attach ramp monitor spec to mod-109 register] — owner: risk engineer; due: 2026-07-08.
- ACT-2026-Q2-02 [refresh RMF crosswalk for Q3 delegated act] — owner: risk engineer + architect (L50); due: 2026-08-15.

## Sign-off
- Chair: [signature, date]
- Risk engineer (author): [signature, date]
- Secretary (or role): [signature, date]

## Attachments (hashes)
- Pre-read pack — sha256:...
- One-pager — sha256:...
- Decision-log deltas — sha256:...
```

The three properties that make minutes audit-durable: (i) they *reference* the pre-read and one-pager by hash, (ii) they *distinguish* decisions from discussion, (iii) they carry *sign-off* from more than one attendee. Missing any one of the three tends to become a finding under ISO/IEC 42001 clause 9.3 (management review) or under EU AI Act Article 61-style post-market surveillance evidence expectations.

<!-- needs-research: the specific ISO/IEC 42001 clause references and audit-language conventions for council minutes should be cross-checked against the ISO/IEC 42001 lead-implementer / lead-auditor guidance current at authoring time. -->

## Common failure modes and their fixes

| Failure mode | Symptom | Fix |
|---|---|---|
| Read-out session | 40-slide walkthrough; no yes/no votes | Cap decision slides at 6; move context into pre-read only; use meeting for decisions |
| Ambush decision | Decision requested with no evidence pack | Pre-read distribution deadline (5-7 days pre-meeting); no decision may be added the day of |
| Silent above-appetite | Above-appetite rows aggregated in the pre-read but no decision requested | Make above-appetite acceptance a *forced* decision slide when appetite is breached; no drift by silence |
| Log-in-minutes | Decisions recorded only in minutes; no queryable log | The decision log is a separate artefact in the register repo; every council-approved decision is an entry |
| Missing rollback | Approved decisions with no reversibility clause | Rollback / reversibility is a required decision-slide field; a decision without one is not ready |
| Signature-of-one | Only the risk engineer signs the minutes | Council charter requires at least chair + risk engineer (author) + one other; no exceptions |
| Evidence-hash absence | Minutes reference "the pre-read" without a hash | Pre-read and one-pager are hashed and pinned in the minutes; if they change post-meeting, that is a corrigendum, not a silent edit |

## Interfaces to earlier modules

- **mod-101** — the framework mapping determines which obligations attach to each decision-log entry (`linked_obligations` field).
- **mod-102** — harm-model rows are what evidence pointers on a decision slide typically resolve to.
- **mod-103** — the register aggregation reports (ch 05) are the raw material for the pre-read's context slides; the sign-off routing (ch 06) is the pattern this chapter's council-level entries extend.
- **mod-104 / 105** — red-team and adversarial-ML run reports are direct evidence on many decision slides.
- **mod-106 / 107** — fairness and privacy findings often *are* the decisions above appetite that the council rules on.
- **mod-108** — guardrail effectiveness numbers are core evidence pointers.
- **mod-109** — the monitoring digest is a signal slide; the mute of a tripwire monitor is itself a council-owned decision.
- **mod-110** — MRM tiering rulings and third-party assurance evidence appear as decision slides or signal slides depending on materiality.
- **mod-111** — incident summary is a standing agenda item; post-mortems that raise programme-level questions become decision slides at the next council.

Chapter 03 continues into the external-facing side of the same artefact chain — model / system / dataset cards.

## Summary

- The council is chaired by the head of AI governance; the risk engineer at level 25 *briefs* the council with a pre-read deck, a one-pager, and decision-log entries.
- Pre-read discipline is *one decision per slide*, seven fields, no mixing of context and decision.
- The one-pager is a decision index, versioned and hashed with the pre-read pack.
- The decision log is a queryable ledger in the register repo (or GRC platform), separate from minutes.
- Minutes are audit-durable when they reference pre-read and one-pager by hash, distinguish decisions from discussion, and carry multi-party sign-off.
- The seven common failure modes each have a mechanical fix; adopting the mechanical fix is what makes the audit trail hold under review.
- Exercise 02 walks a full pre-read pack, one-pager, decision log, and minutes for a mock council session on the running system.
