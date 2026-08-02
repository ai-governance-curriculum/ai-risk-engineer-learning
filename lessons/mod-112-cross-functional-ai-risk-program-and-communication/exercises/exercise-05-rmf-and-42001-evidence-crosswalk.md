# Exercise 05 — RMF and ISO/IEC 42001 Evidence Crosswalk

**Estimated effort:** 2 hours

## Objective

Build the **framework evidence crosswalk** for the running system's programme: one row per evidence artefact, columns for NIST AI RMF MEASURE / MANAGE sub-categories, ISO/IEC 42001 Annex A controls, EU AI Act articles (overlay), and any in-scope sector overlays. Then run **gap detection** — every framework citation is either filled by an artefact, waived with rationale, or explicitly escalated to `ai-evaluation-engineer` (L35) or `senior-ai-governance-architect` (L50). The output is what an ISO/IEC 42001 lead auditor reads first on-site.

## Prerequisites

- Chapter 05 read closely — the crosswalk shape, the "one artefact, many standards" pattern, gap-detection discipline, the L35 / L50 hand-off table.
- NIST AI RMF 1.0 (AI 100-1) and the RMF Playbook — MEASURE and MANAGE sections — <https://airc.nist.gov/AI_RMF_Knowledge_Base/Playbook>. Pin the current published wording of the sub-categories you cite.
- ISO/IEC 42001:2023 — Annex A controls and clauses 4-10. Access is paywalled through national bodies; the ISO landing page has publicly available scope summaries. Cite exact Annex A control identifiers only if you have licensed access; otherwise cite them with `<!-- needs-research: pin verbatim control identifier from ISO/IEC 42001:2023 -->` markers.
- EU AI Act (Regulation (EU) 2024/1689) articles 9, 10, 13, 14, 15, 55, 61, 72 — <https://eur-lex.europa.eu/eli/reg/2024/1689/oj>.
- Any in-scope sector overlays from mod-101 chapter 05 (SR 11-7 for financial services; FDA GMLP + PCCP for medical devices; Colorado AI Act; NYC LL 144).
- The artefact set you have accumulated across mods 102-111 (harm model; register; red-team run reports; adversarial-ML evaluation; fairness slice reports; privacy-risk reports; guardrail effectiveness reports; monitor spec and digest; MRM / supply-chain evidence; incident log; post-mortems). Plus the artefacts from this module's exercises 01-04 (programme charter, council packet, cards, tiering block).

## Problem statement

Produce a crosswalk table with **one row per evidence artefact** the running system's programme currently has. For each row, populate the framework-citation columns and mark each cell as one of:

- **Filled** — the artefact satisfies the cited sub-category / control at the pinned hash.
- **Waived** — the cited sub-category / control is out-of-scope for this system; a rationale is attached.
- **Escalated** — the work belongs above level 25; the escalation contract is cited.

At the end of the crosswalk, run gap detection: enumerate every in-scope RMF sub-category, ISO/IEC 42001 control, and obligation from the overlays; subtract the set with at least one filled artefact; report the remainder as gaps with their disposition (fill, waive, escalate).

## Requirements

Produce two artefacts in a folder `mod-112-ex05-crosswalk/`:

### 1. The crosswalk itself — `crosswalk.md` (or `crosswalk.yaml`)

A table (markdown table, spreadsheet, or YAML index — pick one; if YAML, follow the schema below). At minimum these columns:

| Column | Content |
|---|---|
| Artefact ID | Stable identifier (e.g., `ART-CS-RED-Q2-01`) |
| Artefact type | Which module authored it (red-team run report, guardrail effectiveness, monitor spec, etc.) |
| Version / hash | The pinned version identifier and hash |
| RMF MEASURE sub-cats | Comma-separated sub-category identifiers (e.g., `MEASURE-2.5, MEASURE-2.7`) |
| RMF MANAGE sub-cats | Comma-separated sub-category identifiers |
| ISO/IEC 42001 controls | Annex A control identifiers |
| EU AI Act articles | Article identifiers (Art 9, 10, 13, 14, 15, 55, 61, 72 as applicable) |
| Sector overlays | Any in-scope overlay citations (SR 11-7 attributes, FDA guidance sections, NYC LL 144, Colorado AI Act) |
| Owner | Role that owns the artefact refresh |
| Last refresh | Absolute date |

Populate at least **ten rows** — the artefacts chapter 05's worked table lists are a good starting set: red-team run report, guardrail effectiveness report, monitor spec + digest, fairness-slice report, privacy-risk report, register aggregation report, council decision-log deltas, model / system / dataset cards, deployment-tier contract, incident log + post-mortems, model-BOM / SBOM. If your programme does not yet have one of these, mark the row with `<!-- gap: artefact not yet produced; see gap register -->` and record it in the gap register (below).

The table is **many-to-many**: one artefact points at multiple framework citations, and one framework citation may be pointed at by multiple artefacts. That is by design.

Every artefact pointer is **version-hash pinned**. Placeholder hashes are fine for the drill; the discipline is that a hash exists.

### 2. The gap register — `gaps.md`

For every in-scope framework citation that has no filled artefact pointer, one entry with:

- **Framework citation** (RMF sub-category ID / ISO 42001 control ID / EU AI Act article / overlay citation).
- **Disposition**: `fill`, `waive`, or `escalate`.
- **If `fill`**: which artefact needs to be produced, by whom, by when.
- **If `waive`**: rationale (why out-of-scope for this system), signed off by whom, dated.
- **If `escalate`**: which higher-level role owns the work (L35 for release-assurance / audit-trail methodology; L50 for enterprise control-library architecture; L60 for programme-scale policy). Cite the escalation contract from mod-101 chapter 06.

The gap register is the artefact that turns a crosswalk from "we file some evidence" into "we file every applicable citation." A crosswalk without a gap register is not shippable.

### 3. Coverage summary — appended to `crosswalk.md`

At the end of the crosswalk, a short summary:

- Total in-scope RMF MEASURE sub-categories: N; filled: X; waived: Y; escalated: Z.
- Total in-scope RMF MANAGE sub-categories: N; filled: X; waived: Y; escalated: Z.
- Total in-scope ISO/IEC 42001 Annex A controls: N; filled: X; waived: Y; escalated: Z.
- Total in-scope EU AI Act articles: N; filled: X; waived: Y; escalated: Z.
- Any overlays: N; filled: X; waived: Y; escalated: Z.

Coverage summary is what a council reads on the quarterly signal slide (per chapter 02).

## Starter guidance

- Do the crosswalk **artefact-first**, not framework-first. Enumerate the artefacts you have; then for each artefact, list every framework citation it satisfies. This is the "one artefact, many standards" economy chapter 05 describes.
- On RMF sub-categories, the Playbook language is deliberately non-prescriptive. Your job is to translate each sub-category into the *evidence artefact* that satisfies it (chapter 05's five-column translation from mod-101 ch 01). If the sub-category talks about "measuring risks," the artefact is your evaluation report; if it talks about "prioritising risk response," it is your register aggregation report.
- On ISO/IEC 42001 Annex A controls, pin the *identifier and title* verbatim from the standard. If you do not have licensed access at the moment of the exercise, use `<!-- needs-research: pin verbatim identifier from ISO/IEC 42001:2023 -->` and populate the crosswalk with the *category* (verification and validation; impact assessment; third-party relationships; resources) rather than the exact control identifier. This lets the crosswalk take shape now and be sharpened when the licensed access is available.
- On EU AI Act articles, cite by article number and title. Article 9 is risk management; Article 10 is data and data governance; Article 13 is transparency; Article 14 is human oversight; Article 15 is accuracy, robustness, cybersecurity; Article 55 is obligations for general-purpose AI models with systemic risk; Article 61 is corrective actions and duty of information; Article 72 is post-market monitoring.
- On gaps, be honest. If the programme has no monitor spec yet (mod-109 work not done), that is a gap under RMF MEASURE-3.x — do not paper over it with a paragraph on a wiki page. Record the gap with a `fill` disposition and a target date.
- On escalations, use mod-101 chapter 06's deferral contract wording. "Release-assurance methodology depth" escalates to L35; "enterprise control-library architecture" escalates to L50; "programme-scale tiering policy or external commitments" escalates to L60. The escalation is a *pointer*, not a hand-off — the L25 crosswalk records it and the L25 role continues to maintain the citation while the higher-level work happens.
- Do not invent sub-category numbers. If you do not remember whether MEASURE 2.5 or 2.7 is the right citation for guardrail effectiveness, look it up in the Playbook. Fabricated citations are worse than gaps.

## Acceptance criteria

The crosswalk and gap register are accepted when:

- The crosswalk table has at least ten artefact rows for the running system.
- Every row has an artefact ID, artefact type, version / hash, at least one framework citation, an owner, and a last-refresh date.
- Every citation cell is *specific* — sub-category or control identifier, not a paragraph.
- The gap register enumerates every unfilled in-scope framework citation with a disposition (fill / waive / escalate).
- Every `waive` disposition has a rationale, a sign-off role, and a date.
- Every `escalate` disposition cites the destination (L35 / L50 / L60) and the mod-101 ch 06 escalation contract.
- The coverage summary counts every framework family the programme is in scope for.
- No cell in the crosswalk is left blank without a corresponding gap-register entry.
- An ISO/IEC 42001 lead auditor arriving on-site can (i) read the crosswalk in ten minutes, (ii) see which controls are covered by which artefact, (iii) see the gap register and its dispositions, (iv) know which work is above your paygrade and where it lives.

## Stretch goals

- Wire a **CI check** on the crosswalk file: for every artefact row, a script resolves the artefact reference (file path, GRC platform link) and checks that the pinned hash matches. Any stale hash flags the row. Chapter 05 names this as the maintenance discipline; making it a CI check is what forces the discipline to hold.
- Run the **gap-disposition sign-off cycle** end-to-end: circulate the gap register to the roles that would sign off the waivers and escalations (system owner for out-of-scope waivers; risk council chair for escalations); record their sign-offs in the register. This is what turns a crosswalk from a draft into an audit-ready artefact.
- Add a **framework-revision playbook**: a short note that says "when the AI RMF publishes a revision, run steps X-Y-Z"; "when ISO/IEC 42001 publishes an amendment, run steps A-B-C." Populate with the actual steps you would take (re-check every citation column; report newly uncovered sub-categories to the gap register; re-hash affected artefact rows). This is the maintenance discipline made explicit.
- Produce a **sector-overlay slice** as a separate view: if your programme is in-scope for SR 11-7 or FDA GMLP + PCCP or Colorado AI Act or NYC LL 144, extract just the overlay column into its own document with narrative for each obligation, ready for the sector regulator's evidence request. This is the shape L15 analyst-supported overlay work takes when done well.
- Draft the **hand-off packet to L35** for one specific artefact: pick one row of the crosswalk (say, the red-team run report) and package the artefact plus its framework citations, its refresh cadence, and its hash-integrity discipline into a hand-off packet that a level-35 release-assurance engineer could pick up and bundle into a regulator-facing evidence pack. This is the level-25-to-level-35 seam made concrete.

## Deliverable format

A folder `mod-112-ex05-crosswalk/` containing:

- `crosswalk.md` (or `crosswalk.yaml`) — the crosswalk table plus coverage summary.
- `gaps.md` — the gap register.
- `README.md` — one paragraph on the running system, the frameworks in scope, and any overlays.

## Submission

Commit the folder to the programme's crosswalk / evidence-index workspace. If your programme uses a GRC-for-AI platform (Credo AI, Holistic AI, ModelOp, Monitaur) that has a native crosswalk view, mirror the file-based crosswalk into the platform and note the correspondence in the README. The file-based crosswalk is what an auditor reads offline; the platform view is what the programme operates against day to day.
