# Running an AI Risk Programme at Team Scope

Modules 101-111 taught engineering craft one dimension at a time — frameworks, harm modelling, quantification, red-team, adversarial ML, fairness, privacy, guardrails, monitoring, MRM, incident response. This chapter is where those dimensions become a *programme*: a cadence, a set of artefacts, a decision surface, and a sign-off graph that spans product / ML / infra / security / governance / legal at team scope. It is the first chapter of the integrator module on purpose — everything downstream is a specific instrument played inside this ensemble.

## Motivation

Two failure modes recur whenever a team has done the engineering work but has not built the programme around it:

- **All-artefact, no-cadence.** The harm model exists (mod-102). The register exists (mod-103). Guardrail effectiveness numbers exist (mod-108). Monitor dashboards exist (mod-109). But nothing is *reviewed on a schedule*, nothing is *signed off*, and the artefacts drift out of alignment with the running system. Audit finding: "the risk-management process is documented but not operating."
- **All-meeting, no-artefact.** The team has a weekly risk stand-up and a monthly council review. Meetings produce minutes; minutes produce action items; action items produce more meetings. But no versioned artefact carries the current position on the risk, so a new joiner cannot answer *what does this programme own right now*.

The programme this chapter defines closes both gaps: cadence *drives* artefact refresh, artefacts *anchor* decisions, decisions *route through* a named sign-off graph. The risk engineer at level 25 does not chair the council (that is `head-of-ai-governance` at level 60), but does own the programme *at team scope* — one to a handful of systems, one product line, one engineering group.

## The four-tier cadence

At team scope, four cadences carry the programme. Each has a different purpose, a different attendee list, and a different set of artefacts on the table.

| Cadence | Purpose | Attendees | Artefacts refreshed |
|---|---|---|---|
| Weekly (30 min, standing) | Operational triage — new findings, monitor alerts, near-misses, red-team hits | Risk engineer, ML lead, platform / MLOps lead, security partner | Register row updates (event-triggered), incident log entries, monitor exception list |
| Bi-weekly (60 min) | Cross-functional working session — treatment work in flight, control-effectiveness review | Weekly attendees plus product owner, legal partner (rotating) | Guardrail effectiveness reports (mod-108), treatment plans, integration hand-off contracts |
| Monthly (60-90 min) | Programme review — register state across all systems in scope, above-appetite exceptions, monitoring digest | Bi-weekly plus system owners, governance analyst (L15), release-assurance partner (L35) | Register aggregation reports (mod-103 ch 05), monitoring digest, incident-and-near-miss summary, evidence-manifest health |
| Quarterly (2-3 hours) | Council-facing review — pre-read deck for the risk / governance council; framework and standard reconciliation; deployment-tier posture | Monthly plus AI governance council chair, architect (L50), external audit liaison as needed | Council pre-read deck, one-pager, decision-log entries, RMF / 42001 crosswalk refresh, deployment-tier posture |

The weekly is the heartbeat; the quarterly is the visible-to-executives artefact. The bi-weekly and monthly are what make sure the quarterly is not an emergency write-up. Skipping any one tier tends to collapse work into the next tier down (weekly becomes triage-plus-strategy, monthly becomes triage-plus-council-prep) and something falls off.

<!-- needs-research: the exact minute counts and attendee lists above are illustrative; align to the organisation's operating cadence and the number of systems in the programme's scope. -->

## The artefact bill of materials

The programme has *one* set of live artefacts. Every meeting reads from the same set; nothing lives in a private notebook. The bill of materials, at team scope:

- **Harm model** (mod-102) — one per system, versioned, taxonomy-tagged, evidence-linked.
- **Risk register** (mod-103) — a set of scored rows per system, with operational fields (ownership, cadence, sign-off, evidence-manifest) as in mod-103 ch 06.
- **Red-team programme** (mod-104) — the persona catalogue, the attack corpus, the last N run reports, the permanent-regression suite.
- **Adversarial-ML evaluation** (mod-105) — the evaluation harness, the last run's robustness deltas, the shared-component notes.
- **Fairness / bias / explainability** (mod-106) — the subgroup slice definitions, the last measurement, the caveats sheet, the explainability tool output where relevant.
- **Privacy-risk evaluation** (mod-107) — the memorisation / MIA / linkage findings, the DPIA input trail, the data-flow map.
- **Guardrail effectiveness** (mod-108) — per-guardrail effectiveness reports, coverage grid, defeat-mode notes.
- **Monitoring digest** (mod-109) — the wired signals, the SLO / alert configuration, the current-quarter monitor summary, the mute-list with justifications.
- **MRM / supply-chain evidence** (mod-110) — SR 11-7 tiering (where applicable), SBOM/model-BOM entries, third-party assurance evidence.
- **Incident log and post-mortems** (mod-111) — chronological log; a post-mortem per incident; regression-fixture back-pointers.
- **Programme charter** (this module, exercise 01) — the operating agreement: scope, cadence, attendees, sign-off routing, escalation criteria.
- **Council-facing packet** (this module, ch 02 and exercise 02) — pre-read, one-pager, decision-log entries, minutes.
- **Model / system / dataset cards** (this module, ch 03 and exercise 03) — the external-facing evidence face of everything above.
- **Deployment tiering** (this module, ch 04 and exercise 04) — pre-registered thresholds, rollback criteria, tripwires.
- **Framework crosswalks** (this module, ch 05 and exercise 05) — RMF MEASURE / MANAGE and ISO/IEC 42001 evidence mappings.

Every one of those artefacts has been earned in an earlier module. This chapter's job is to say *how they refresh and how they hang together*.

## The decisions the programme owns

A programme is defined by the decisions it makes. Below is a working decision matrix for team scope. Rows above the line are owned; rows below the line escalate.

| Decision | Owner at team scope | Escalates to (and when) |
|---|---|---|
| Add a new harm-model row | Risk engineer + system owner | — (unless new harm crosses jurisdictional obligation not yet mapped) |
| Credit / drop a control on a register row | Risk engineer + control owner | — (unless residual moves above appetite) |
| Accept residual at or below appetite | System owner (co-signed by risk engineer) | — |
| Accept residual *above* appetite for a fixed window | System owner + risk council (this module ch 02) | Risk council; ultimately `head-of-ai-governance` (L60) |
| Approve promotion between deployment tiers T0 / T1 / T2 (mod-102 ch 04) | Team lead + risk engineer | — |
| Approve promotion into T3 / T4 (external, high-stakes) | Risk council + release-assurance sign-off (`ai-evaluation-engineer` L35) | Head of AI governance (L60) |
| Approve a red-team engagement scope (mod-104) | Risk engineer + security partner | — (unless it hits external targets or third-party systems) |
| Mute / retire a monitor (mod-109) | Risk engineer + monitor owner (with justification recorded) | — (mute of a control-defeated tripwire escalates to risk council) |
| Declare an AI incident (mod-111) | Risk engineer or on-call, per playbook | Auto-notifies risk council for severity ≥ high |
| Publish a model / system / dataset card externally | Risk engineer + comms / legal review | Head of AI governance for tier ≥ T3 |
| Sign off a quarterly council packet | Risk engineer authors; risk council chair countersigns | — |
| Legal-opinion decision (obligation compliance ruling) | *Never owned here* | Legal counsel |
| Frontier / dangerous-capability threshold ruling | *Never owned here* | `agentic-safety-engineer` (L40) and beyond |
| Enterprise control-library architecture change | *Never owned here* | `senior-ai-governance-architect` (L50) |

Exercise 01 asks you to instantiate this matrix for a specific team. The matrix is the load-bearing artefact — meetings without a matrix devolve into who-decides-what conversations that never converge.

## The sign-off routing graph

Sign-off is where a decision becomes durable. The programme's sign-off graph is a small directed graph. A defensible team-scope shape:

- **Row-level decisions** (add harm row, credit control, residual within appetite): risk engineer + system owner + affected control owner. Two-signature minimum.
- **System-level decisions** (deployment tier promotion, red-team scope, mute of a tripwire monitor): risk engineer + system owner + security partner. Three-signature minimum.
- **Programme-level decisions** (accept above-appetite exception, publish external card, framework-crosswalk version bump): risk council countersign on top of the system-level three.
- **Escalation exits** (legal ruling, frontier threshold, control-library architecture): decision leaves the programme; the graph records the *escalation event*, not the decision itself.

Sign-off is *append-only* (mod-103 ch 06). A revoked sign-off is a new sign-off event; you never edit the record of the prior one.

## The RACI shape across the six functions

Team-scope programme work spans product, ML, infra, security, governance, and legal. RACI (Responsible / Accountable / Consulted / Informed) is a well-worn frame; the shape below is the one worth defending:

| Work item | Product | ML | Infra / MLOps | Security | Governance (this role) | Legal |
|---|---|---|---|---|---|---|
| Harm model authoring | C | C | I | C | R / A | I |
| Register row scoring | C | C | I | C | R / A | I |
| Guardrail specification (mod-108) | C | C | R (implementation) | C | A (spec + effectiveness) | I |
| Monitor wiring (mod-109) | I | I | R (pipeline) | C | A (signal spec) | I |
| Red-team engagement | I | C | I | R (execution partner) | A (scope + report) | C (if external) |
| Deployment-tier promotion | R (business case) | C | R (release pipeline) | C | A (risk sign-off) | C (obligations) |
| Council pre-read (this module ch 02) | C | C | I | C | R / A | C |
| Model card publication (this module ch 03) | C | C | I | C | R / A | R (external review) |
| RMF / 42001 crosswalk (this module ch 05) | I | I | I | C | R / A | I |
| Incident post-mortem (mod-111) | C | C | R | C | A | C |
| Legal ruling on obligation | I | I | I | C | C | R / A |

*R / A on the risk-engineering column captures the "authors and owns the artefact" state; the accountable executive above the risk engineer is the risk council or head of AI governance depending on the programme's shape.*

## A worked example — the customer-support RAG assistant programme

The running example continues from mods 102-111. A single risk engineer at level 25 owns the programme for a customer-support RAG assistant with two adjacent systems (an internal knowledge-search variant and an experimental agentic ticket-triage variant). The team is: two ML engineers, one MLOps engineer, one product owner, a security partner assigned 20 percent, a legal partner assigned 10 percent, an analyst at L15 handling intake, and a rotating engineering-manager sponsor.

Weekly cadence: the risk engineer chairs a 30-minute stand-up. Standing items: new monitor alerts (mod-109), red-team run outcomes (mod-104), incidents and near-misses (mod-111), register row updates queued for sign-off (mod-103 ch 06). One-off items: any harm-model additions from new features shipping that week (mod-102).

Bi-weekly: a 60-minute working session. Standing: treatment work in flight for above-appetite exceptions, guardrail-effectiveness deltas since last session (mod-108), evidence-manifest health (any broken pointers, per mod-103 ch 06 register-hygiene checks). Rotating: one deep-dive into one system (three systems, six weeks between deep-dives).

Monthly: 90-minute programme review. Standing: register aggregation reports (heat map, above-appetite list, control-defeated list, shared-component list, coverage grid, jurisdiction exceptions — mod-103 ch 05); monitoring digest; incident-and-near-miss summary. Attendees include the system owners for all three systems and the L15 analyst.

Quarterly: 2.5 hours. Council pre-read is circulated a week in advance. The one-pager sits at the top of the pack. The decision log carries three items this quarter: tier promotion for the agentic triage variant from T1 to T2, an above-appetite exception on a fairness delta for the RAG variant, and a supplier-context change on the embedding model (mod-110 shared-component report). Minutes filed within 48 hours.

The programme's *charter* — the artefact that names all of the above and the escalation criteria — is exercise 01.

## Interfaces to earlier modules — the integrator picture

- **mod-101** — the framework mapping and role-scope contract are the parent constraint on the whole programme; the cadence and sign-off routing here are the operating expression of the mod-101 ch 06 deferral contract.
- **mod-102** — harm model is the seed artefact; every cadence tier refreshes some slice of it.
- **mod-103** — register is the ledger; the aggregation reports (ch 05) are what feeds the monthly and quarterly reviews.
- **mod-104 / 105** — red-team and adversarial-ML run reports land in weekly triage and refresh scored rows.
- **mod-106 / 107** — fairness and privacy findings feed the monthly deep-dives and, when they cross appetite, the quarterly council packet.
- **mod-108** — guardrail effectiveness is the control-credit input to the register; the bi-weekly reads its deltas.
- **mod-109** — monitors are the weekly triage feed; the digest lands in the monthly.
- **mod-110** — MRM tiering and supply-chain evidence flow into the monthly and the quarterly council review; escalations under SR 11-7 obligations go straight to the council.
- **mod-111** — incidents interrupt the cadence (event-triggered review); post-mortems and regression fixtures land back in mods 103-108 artefacts.

Chapters 02-06 of this module pick up the specific programme-level artefacts (council packet, model card, tiering contribution, framework crosswalk, certifications posture).

## Summary

- The programme is four cadences deep — weekly triage, bi-weekly working session, monthly programme review, quarterly council packet — each with a defined attendee list and refreshed artefacts.
- The bill of materials is finite and every entry maps to a specific earlier module; there is no artefact in the programme that is not earned in mods 101-111 or produced in this module.
- The decision matrix names which decisions the team owns and which escalate; anything not in the matrix should not be decided in a weekly stand-up.
- Sign-off routing is append-only and graduated by decision scope (row, system, programme, escalation).
- The RACI shape spans six functions; the risk engineer is responsible and accountable for risk-engineering artefacts and consulted almost everywhere else.
- Exercise 01 walks the programme charter for one team; every subsequent exercise in the module plugs into the cadence and artefact set defined here.
