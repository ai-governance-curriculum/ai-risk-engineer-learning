# Exercise 04 — AIRO and NIST GenAI Profile Cross-Tag

**Estimated effort:** 2 hours

## Objective

Complete the taxonomy overlay stack on your harm catalog: add **AIRO ontology** categorisation, **NIST AI RMF Generative AI Profile (AI 600-1)** risk numbers, **ISO/IEC 23894** risk sources, and **AIR-Bench 2024** policy-derived risk categories to every row. Also produce the *twelve-row applicability sheet* for AI 600-1 (per mod-101 ch 02) reconciled against this harm model. The output is a harm catalog whose every row is tagged from all four overlays and whose GenAI applicability sheet is defensible.

## Prerequisites

- Chapter 03 (this module) read; mod-101 ch 02 read.
- Exercise 03 completed — the MIT AIR-tagged harm catalog is the input.
- Access to:
  - NIST AI 600-1 (Generative AI Profile) — <https://airc.nist.gov/AI_RMF_Knowledge_Base/AI_RMF/Uses/GenAI-Profile>
  - AIRO — <https://w3id.org/airo>
  - AIR-Bench 2024 — Stanford CRFM publication / dataset (locate the current published paper and repository)
  - ISO/IEC 23894:2023 Annex A — via ISO national body or licensed implementer materials

## Problem statement

For every row of the harm catalog, add tags from:

1. **NIST AI 600-1** — the risk number (1-12) plus the risk short-name. Multiple risks per row are welcome when the scenario genuinely spans; mark one as *primary* and the others as *secondary*.
2. **ISO/IEC 23894 risk sources** — from Annex A. Cite the annex sub-heading identifier. Row-source associations are usually 1-3 per row; if you find yourself listing more than three, the row is probably under-specified — split it.
3. **AIRO ontology** — at minimum populate the four fields introduced in chapter 03: `hasRiskSource`, `hasConsequence`, `hasImpactedStakeholder`, `hasMitigation` (the mitigation field may reference an as-yet-unbuilt mitigation ID; that is fine).
4. **AIR-Bench 2024** — the policy-derived category code that most closely matches. If the scenario spans multiple categories, note the primary and secondary codes. If no category matches after a genuine search, mark *no AIR-Bench code — reason: <one sentence>*.

Additionally, produce or update the **twelve-row applicability sheet** for AI 600-1 (mod-101 ch 02) for the running system, reconciled with the harm catalog: every "in" row of the sheet must map to at least one harm-catalog row, and every harm-catalog row must map to at least one "in" row of the sheet.

## Requirements

- Output as `mod-102-ex04-taxonomy-cross-tag.md` — either amending the exercise-03 overlay in-place with new columns, or as a separate document that references the exercise-03 row identifiers.
- Every catalog row carries a NIST AI 600-1 primary risk number (and secondary if applicable), one or more ISO 23894 risk source identifiers, an AIRO tag block with the four fields populated, and an AIR-Bench 2024 category code or explicit *no code* justification.
- The AI 600-1 twelve-row applicability sheet is present, complete (twelve rows), reconciled against the catalog, and each row's *reason* fits on one line.
- The AIRO block is present as a structured block (YAML or the ontology's Turtle-lite abbreviation is welcome) rather than prose.
- Any tag that is empty must carry an explicit reason. Silent empties are unacceptable.
- Cite versions: AI 600-1 publication date, AIRO release / URI version, AIR-Bench release, ISO/IEC 23894 edition consulted.

## Starter guidance

- Do the NIST AI 600-1 pass first — most rows will have an obvious primary. Rows without an obvious primary are usually classification / decision-support harms not GenAI-specific; mark those with a clear *no primary AI 600-1 risk — reason:* line.
- Do the ISO 23894 pass in the same sitting — you already know from chapter 05 whether the row is data-, model-, use-, or oversight-originated, and Annex A gives you the sub-headings.
- For AIRO, start from the *consequence* class: what harm class describes the outcome? Then work outward to risk source, impacted stakeholder, mitigation.
- For AIR-Bench 2024, the fastest path is to skim the category hierarchy for the top-level bucket (e.g., security, harmful content, privacy, bias) that matches, then descend into the sub-categories to find the specific code. Cite the code exactly as the release prints it.
- For the applicability sheet, run through the twelve risks and answer *for the running system* whether each is in-scope. Each "in" row must have at least one harm-catalog row supporting it; if a risk is "in" but you have no catalog row, that is a *harm-model coverage gap* to close before submission.
- If your reconciliation reveals a harm-catalog row that doesn't fit any of the twelve GenAI risks, that is legitimate — it means your system covers non-GenAI risks that AI 600-1 doesn't enumerate. Note the row and cite an alternative source (ISO 23894 alone, mod-101 ch 05 sector obligations, etc.).

## Acceptance criteria

The submission is accepted when:

- Every catalog row has NIST AI 600-1, ISO 23894, AIRO, and AIR-Bench 2024 tags (or explicit reasoned empties).
- All version citations are present.
- The AI 600-1 twelve-row applicability sheet is complete and reconciled.
- No harm-catalog row is unmatched to at least one "in" applicability row (or the row's non-GenAI justification is present).
- No "in" applicability row is unsupported by at least one harm-catalog row.
- AIRO blocks are structured (not prose) and each field has a value or a reasoned empty.

## Stretch goals

- Convert the AIRO tags to a Turtle / JSON-LD serialisation aligned with AIRO's published vocabulary; if your organisation has a risk-graph platform, load the file.
- For every "in" AI 600-1 risk, cross-reference to the mod-101 exercise-01 RMF sub-category map — the sub-category that a `MEASURE` control for this risk would live under. This bridges exercises 04 and 05 and preps mod-103.
- Add a *conflict* section listing rows where the four overlays disagree on granularity or scope (e.g., AIRO consequence class is broader than the AIR-Bench category or vice versa). Note how you reconciled each.

## Deliverable format

A single markdown document with the following top-level sections:

1. *Version citations* — AI 600-1, AIRO, AIR-Bench 2024, ISO 23894 edition.
2. *Cross-tagged catalog* — the harm catalog with the four overlays applied.
3. *AI 600-1 twelve-row applicability sheet* — one row per risk, reconciled.
4. *Reasoned empties* — a small table of tags that came up empty, with reasons.
5. *Conflicts* (stretch) — where overlays disagree and how you reconciled.

## Submission

Attach the markdown (and YAML / Turtle if produced) to the module submission channel.
