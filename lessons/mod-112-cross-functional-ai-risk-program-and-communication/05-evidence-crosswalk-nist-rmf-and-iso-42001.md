# Reconciling Evidence to NIST AI RMF MEASURE / MANAGE and ISO/IEC 42001

This chapter is where the risk-engineering evidence produced across mods 102-111 gets reconciled to the two standards that most enterprise programmes report against: NIST AI RMF 1.0 (specifically the MEASURE and MANAGE functions) and ISO/IEC 42001:2023 (the AI Management System). The reconciliation is a *crosswalk* — a table that, for every RMF sub-category and every ISO 42001 control the programme claims to satisfy, points at the specific evidence artefact that satisfies it.

The crosswalk is a load-bearing artefact for two reasons. Externally, it is what an ISO/IEC 42001 lead auditor reads first when arriving on-site. Internally, it is how the programme detects coverage gaps — a sub-category with no evidence pointer is a gap, and the gap has to be either filled or explicitly waived with rationale.

## Motivation

Framework crosswalks fail in three familiar ways:

- **Prose crosswalk.** The mapping is a document of paragraphs — "we address NIST AI RMF MEASURE 2.5 through our red-team programme" — with no direct pointer to a specific evidence artefact and no version. An auditor cannot verify from the document alone that the claim holds today.
- **N-to-1 collapse.** The same artefact is pointed at from every sub-category — "the register" satisfies fifty different MEASURE sub-categories — with no per-sub-category evidence granularity. This is not defensible under audit; the auditor asks *which row* of the register satisfies MEASURE 2.5 and the programme has to search.
- **1-to-1 duplication.** The programme authors a *separate* artefact for each sub-category, producing redundant evidence stores that drift and disagree. The evidence-source-of-truth question becomes a governance debate every quarter.

The productive shape is what mod-101 chapter 03 named the "one artefact files against several standards at once" pattern. A single artefact — a red-team run report, a guardrail effectiveness report, a monitor spec, a register row, a decision-log entry — is pointed at by *multiple* framework rows in the crosswalk, and each pointer is *granular* (row-level, section-level, version-hash pinned).

## What the crosswalk covers

At level 25, the two families the crosswalk covers are the RMF (US programme baseline; the framework most enterprise AI programmes converge on for their internal risk language) and ISO/IEC 42001 (the international AIMS standard, the accreditable one). Other frameworks (EU AI Act articles, SR 11-7, FDA GMLP + PCCP, Colorado AI Act) are typically handled either as *overlays* to the same crosswalk (adding columns for jurisdictional citations) or as separate specialist crosswalks maintained by the analyst (L15) with the risk engineer as reviewer.

| Framework family | Level 25 posture |
|---|---|
| NIST AI RMF 1.0 (MEASURE, MANAGE) | Owned by this role at team scope; crosswalk authored here |
| ISO/IEC 42001:2023 (AIMS controls) | Owned by this role at team scope; crosswalk authored here |
| ISO/IEC 23894 / 25059 / 42005 | Supporting; used as taxonomic scaffolding (mod-103 ch 03; mod-102 ch 03) but not always separately crosswalked |
| EU AI Act (articles 9, 10, 14, 15, 55, 61, 72) | Overlay column on the crosswalk (obligations mapped in mod-101 ch 04) |
| SR 11-7 / OCC 2011-12 / SR 23-4 / FDA GMLP + PCCP / Colorado AI Act | Overlay columns for in-scope programmes; ownership shared with the L15 analyst and reviewed by the L50 architect |
| Enterprise ISO/IEC 27001 / SOC 2 / internal control library | Consumed as parent, not authored here; escalates to `senior-ai-governance-architect` (L50) |

The crosswalk this chapter builds is *the risk-engineering evidence table for team scope*. It is not the enterprise control library (that is level-50 territory) and it is not the release-assurance evidence package (that is level-35 territory).

## The NIST AI RMF MEASURE and MANAGE functions

The AI RMF 1.0 defines four functions — GOVERN, MAP, MEASURE, MANAGE — each with categories and sub-categories. GOVERN and MAP are largely policy / intake / harm-modelling work (handled at levels 15 and above; mod-101, mod-102). MEASURE and MANAGE are where the engineering evidence lives.

- **MEASURE** categories cover the *measurement* of identified risks: methodology selection (MEASURE 1), applying appropriate methods (MEASURE 2), monitoring and tracking (MEASURE 3), gathering feedback (MEASURE 4). Sub-categories under these get granular ("MEASURE 2.5" for adversarial robustness; the exact wording is in AI 100-1 and the Playbook).
- **MANAGE** categories cover *acting on measured risks*: prioritisation (MANAGE 1), risk-response strategy (MANAGE 2), residual-risk documentation (MANAGE 3), incident response and communication (MANAGE 4). Again, sub-categories get granular.

Every MEASURE and MANAGE sub-category should map to at least one evidence artefact from mods 102-111. The Playbook language is deliberately non-prescriptive; the risk engineer's job is to translate each sub-category into a specific test-threshold-guardrail-monitor-evidence artefact (mod-101 ch 01's five-column translation) and then point the crosswalk at that artefact.

<!-- needs-research: cite the specific MEASURE / MANAGE sub-category identifiers used in the running examples below against the current AI 100-1 / Playbook wording; sub-category numbering has been stable but the Playbook prose evolves. -->

## ISO/IEC 42001:2023 — the controls the crosswalk covers

ISO/IEC 42001 is the auditable AI Management System standard. Its controls sit in Annex A and its clauses (4-10) shape the management-system frame: context, leadership, planning, support, operation, performance evaluation, improvement.

At level 25, the controls the risk engineer engages with most heavily are those in the "operation" and "performance evaluation" clusters, and Annex A control families covering AI system life-cycle management, resources for AI systems, AI system impact assessment, verification and validation, and third-party relationships. The exact Annex A control identifiers should be pinned per audit cycle.

<!-- needs-research: pin exact ISO/IEC 42001:2023 Annex A control identifiers and titles at authoring time; the working set spans multiple control families and audit citations must be verbatim. -->

The ISO/IEC 42001 crosswalk *complements* the RMF one: many artefacts satisfy an RMF sub-category and a 42001 control simultaneously. That is what makes the crosswalk economical to maintain — one column per framework, one row per artefact, cell content is the framework citation.

## Crosswalk shape

A working crosswalk is a table (spreadsheet, YAML index in the register repo, or a view over the GRC-for-AI platform). One row per *evidence artefact*, columns for framework citations.

| Artefact ID | Artefact type | Version / hash | RMF MEASURE sub-cats | RMF MANAGE sub-cats | ISO/IEC 42001 controls | EU AI Act articles | SR 11-7 attributes | Owner | Last refresh |
|---|---|---|---|---|---|---|---|---|---|
| ART-CS-RED-Q2-01 | Red-team run report (mod-104) | v2.3, sha256:... | MEASURE-2.5, MEASURE-2.6, MEASURE-3.1 | MANAGE-2.3 | A.6.2.4, A.6.2.6 | Art 15 | — | risk-eng-L25 | 2026-06-01 |
| ART-CS-GRD-Q2-01 | Guardrail effectiveness report (mod-108) | v3.1, sha256:... | MEASURE-2.5, MEASURE-2.7 | MANAGE-2.2 | A.6.2.4 | Art 15 | — | risk-eng-L25 | 2026-06-15 |
| ART-CS-MON-2026 | Monitor spec + digest (mod-109) | v1.4, sha256:... | MEASURE-3.2, MEASURE-3.3, MEASURE-4.1 | MANAGE-4.1 | A.6.2.5, A.6.2.8 | Art 15, Art 72 | — | risk-eng-L25 | 2026-06-30 |
| ART-CS-FAIR-Q2 | Fairness-slice report (mod-106) | v2.0, sha256:... | MEASURE-2.11, MEASURE-2.12 | MANAGE-2.4 | A.6.2.4 | Art 10 | — | risk-eng-L25 | 2026-06-20 |
| ART-CS-PRIV-Q2 | Privacy-risk report (mod-107) | v2.0, sha256:... | MEASURE-2.10 | MANAGE-2.4 | A.7.2, A.7.3 | Art 10 | — | risk-eng-L25 | 2026-06-20 |
| ART-CS-REG-2026-Q2 | Register aggregation report (mod-103 ch 05) | v1, sha256:... | MEASURE-1.1, MEASURE-2.1 | MANAGE-1.1, MANAGE-1.4, MANAGE-3.1 | A.6.2.2, A.6.2.3, A.6.2.7 | Art 9 | Attribute: model-tier | risk-eng-L25 | 2026-06-30 |
| ART-CS-COUNCIL-Q2 | Council decision-log deltas (this module ch 02) | v1, sha256:... | MANAGE-3.2, MANAGE-4.3 | (as above) | A.5.2, A.5.3 (leadership) | Art 9 | — | risk-eng-L25 | 2026-06-24 |
| ART-CS-CARD-2026-06 | Model / system / dataset cards (this module ch 03) | v3.0, sha256:... | MEASURE-4.2 | MANAGE-4.3 | A.6.2.8 | Art 13 | — | risk-eng-L25 | 2026-06-15 |
| ART-CS-TIER-2026 | Deployment-tier contract (this module ch 04) | v2.1, sha256:... | MEASURE-1.3 | MANAGE-1.2, MANAGE-2.1 | A.6.2.4 | Art 15 | Attribute: model-tier | risk-eng-L25 | 2026-06-24 |
| ART-CS-INC-2026 | Incident log + post-mortems (mod-111) | rolling, sha256:... | MEASURE-4.3 | MANAGE-4.1, MANAGE-4.2 | A.6.2.5, A.10.1 (improvement) | Art 72 | — | risk-eng-L25 | rolling |
| ART-CS-SBOM-2026 | Model-BOM / SBOM (mod-110) | v1.2, sha256:... | MEASURE-2.9 | MANAGE-1.3 | A.6.2.7, A.9 (third-party) | Art 10 | Attribute: third-party-risk | risk-eng-L25 + supply-chain partner | 2026-06-30 |

*Sub-category and control identifiers in the table are placeholders; the actual mappings must be pinned to the current published AI 100-1 sub-category numbering and the ISO/IEC 42001:2023 Annex A identifiers.*

<!-- needs-research: the exact mappings in the table above are indicative and should be reviewed against the current AI RMF sub-category wording and the ISO/IEC 42001:2023 Annex A verbatim identifiers before use in any live audit-facing artefact. -->

The two important properties:

- **Many-to-many.** One artefact points at multiple framework citations. One framework citation may be pointed at by multiple artefacts (an ISO/IEC 42001 impact-assessment control might be satisfied by a combination of the harm model and the DPIA). That is by design.
- **Version-hash pinned.** Every artefact pointer is a hash. When the artefact is refreshed, the crosswalk row updates. When the framework is revised (a new AI RMF revision, an ISO/IEC 42001 amendment), the *citation* column updates and any newly uncovered sub-category becomes a visible gap.

## Coverage gap detection

The crosswalk's second job is to *find gaps*. A gap is a framework sub-category with no artefact pointing at it. The gap-detection query is straightforward against the crosswalk:

- Enumerate every in-scope MEASURE / MANAGE sub-category the programme claims.
- Enumerate every in-scope ISO/IEC 42001 Annex A control the programme claims.
- Subtract the set of sub-categories / controls with at least one crosswalk row.
- Report the remainder as *gaps*.

Every gap has one of three dispositions:

- **Fill.** The gap corresponds to work the programme *should* be doing; the response is to schedule the missing artefact.
- **Waive with rationale.** The gap corresponds to a sub-category or control that is out-of-scope for this system (e.g., a control on autonomous-agent oversight for a system with no autonomous capabilities). The waiver is written up with rationale, dated, signed off, and stored in the crosswalk.
- **Escalate.** The gap corresponds to work above level 25 (release-assurance methodology owned by L35; control-library architecture at L50). The gap is *marked as escalated*, with the escalation contract from mod-101 ch 06 cited.

A crosswalk with unmarked gaps is not shippable to audit. Every framework cell must be one of: filled, waived, or escalated.

## The "one artefact, many standards" pattern — worked

Consider the guardrail effectiveness report `ART-CS-GRD-Q2-01`. A single artefact, one authoring pass, files against:

- **NIST AI RMF MEASURE-2.5** (application of appropriate measurement methods to adversarial robustness / prompt-injection defeat).
- **NIST AI RMF MEASURE-2.7** (application of measurement methods across relevant risk categories).
- **NIST AI RMF MANAGE-2.2** (risk-response strategies for identified risks).
- **ISO/IEC 42001 Annex A** — the control family covering verification and validation of AI systems.
- **EU AI Act Article 15** (accuracy, robustness, cybersecurity).
- **The programme's internal deployment-tier contract** (this module ch 04) — as a threshold input.
- **The model / system card** (this module ch 03) — as an "Evaluation" section input.
- **The council pre-read decision slides** (this module ch 02) — as evidence pointers.

One artefact, seven distinct downstream uses. That is what makes the "one artefact, many standards" pattern the economic engine of a defensible programme: authoring effort scales with the number of *evidence artefacts*, not with the product of artefacts and framework citations.

## Handing off release-assurance work to `ai-evaluation-engineer` (level 35)

Level 35 in the same governance family owns release-assurance / audit-trail methodology depth. The clean hand-off contract from level 25 into level 35:

| Level-25 responsibility | Level-35 responsibility |
|---|---|
| Author the crosswalk and per-artefact citations | Package the crosswalk + underlying artefacts as an audit-facing evidence bundle |
| Maintain evidence-artefact freshness (per operational cadence) | Design the release-assurance methodology (release gate, sign-off circuit, deviation handling) |
| Compute per-artefact hashes | Design the evidence-bundle integrity scheme (chain-of-custody, timestamping, immutable log) |
| Reconcile RMF / 42001 for team-scope systems | Reconcile release-assurance evidence across systems and audit cycles |
| File findings back into the register | Regulator-facing evidence discipline (post-market surveillance packet, external-audit response) |

At level 25 you produce the crosswalk and the underlying artefacts *up to* the release-assurance boundary. Beyond that boundary — packaging for regulator, methodology for the release gate, immutability of the evidence bundle — is L35 work.

## Handing off control-library architecture to `senior-ai-governance-architect` (level 50)

Level 50 owns the enterprise control-library architecture. The clean hand-off:

| Level-25 responsibility | Level-50 responsibility |
|---|---|
| Cite existing enterprise controls in the crosswalk | Author and version the enterprise control library itself |
| Report gaps / duplications encountered in team-scope work | Rationalise the control library across systems and jurisdictions |
| Overlay sector obligations (SR 11-7, FDA GMLP, Colorado AI Act) into the crosswalk | Design the Statement of Applicability (SoA) for ISO/IEC 42001 at organisation scope |
| Ingest updates from framework revisions into the crosswalk | Design the ingestion process and the versioning discipline for the control library itself |

Framework revisions land as work at both levels. At L25 you refresh your crosswalk; at L50 the architect refreshes the enterprise control library and re-scopes the SoA.

## Maintenance discipline

The crosswalk drifts silently unless there is a refresh cadence:

- **Monthly** — hash-check every artefact pointer. Any stale hash flags the row.
- **Per framework revision** — when the AI RMF or the ISO/IEC 42001 Playbook / Annex A gets a documented update, re-check every citation and report new gaps.
- **Per system change** — when a system's scope or capability changes, the artefacts referring to it may change and the crosswalk must be re-checked.
- **Per audit cycle** — regenerate the crosswalk into the audit-facing form (this module ch 02 pre-read pack shape) with narrative for each row.
- **Per council packet** — the crosswalk delta since last packet is a signal slide at the quarterly.

CI check on the crosswalk file: every citation cell resolves; every referenced artefact exists at the pinned hash; no citation is left blank without a waiver entry.

## Interfaces to earlier modules

- **mod-101** — ch 01 (RMF as an engineer), ch 03 (ISO standards the programme plugs into), ch 04 (EU AI Act articles for the overlay columns), ch 05 (sector overlays), ch 06 (deferral contract to L35 / L50) — all directly relevant.
- **mod-102** — the harm model is an artefact in the crosswalk; taxonomy tags (NIST GenAI Profile, ISO 23894) provide cross-references.
- **mod-103** — the register is an artefact; the aggregation reports (ch 05) are artefacts; the operational fields (ch 06) provide the version-hash discipline.
- **mods 104-108** — every per-module report is an artefact row.
- **mod-109** — the monitor spec and digest are artefacts covering MEASURE-3 / MANAGE-4.
- **mod-110** — the SBOM / model-BOM and MRM evidence cover third-party and supply-chain controls.
- **mod-111** — the incident log and post-mortem set covers MEASURE-4.3 and MANAGE-4 in full.

Chapter 06 continues into the certifications portfolio and career shape that closes the module and the track.

## Summary

- The crosswalk is one row per evidence artefact, columns for framework citations (RMF sub-categories, ISO/IEC 42001 controls, EU AI Act articles, sector overlays where in scope).
- The "one artefact, many standards" pattern means authoring effort scales with artefacts, not with the framework product.
- Gaps are detected mechanically and dispositioned three ways: fill, waive with rationale, or escalate; unmarked gaps are non-shippable.
- Release-assurance methodology, evidence-bundle integrity, and regulator packaging escalate to `ai-evaluation-engineer` (L35).
- Enterprise control-library architecture and the Statement of Applicability at organisation scope escalate to `senior-ai-governance-architect` (L50).
- Maintenance is a cadence (monthly hash check, per-revision citation refresh, per-audit regeneration); CI enforces integrity.
- Exercise 05 walks the crosswalk build for the running system across mods 102-111 artefacts and pins the level-35 / level-50 escalations.
