# Exercise 03 — ISO/IEC 27005 Parent-Frame Alignment

**Estimated effort:** 2 hours

## Objective

File the register you built through exercises 01 and 02 against the ISO/IEC 27005 process shape so it composes with the enterprise ERM without a schema fight, and reconcile with the security-side 27005 register on shared assets, shared threat sources, and appetite bands. The output is the artifact that lets a 27005-trained analyst read your AI register without translation.

## Prerequisites

- Chapter 03 read.
- Chapter 04 skimmed — the 25059 / MEASURE tags are populated in exercise 03 lightly and revisited in exercise 04.
- Exercises 01 and 02 completed — the three-state rows and the versioned scoring scheme carry through.
- Access to (a) your enterprise's security-side 27005 register or a redacted excerpt; (b) the ISO/IEC 27005:2022 process-step list (via licensed access through your ISO national body — public summaries are also acceptable for exercise purposes); (c) any ERM policy that names the enterprise appetite bands.
- If none of the above is accessible, use the ISO/IEC 27005 process-step list published on the ISO landing page and construct a plausible 3-row *provisional* security register for the same system, clearly marked as provisional.

## Problem statement

Produce two artifacts:

1. A **process-step filing matrix** — for each of the seven ISO/IEC 27005 process steps (Establish context → Risk identification → Risk analysis → Risk evaluation → Risk treatment → Monitoring and review → Communication and consultation), name the concrete AI-register artifacts you produced (in exercises 01, 02, and mod-102) that fill that step. Every artifact you have should file against at least one step; every step should have at least one artifact.
2. A **cross-register reconciliation table** for shared assets and shared threat sources between your AI register and the security-side 27005 register. For each shared item, name whether the AI register defers (cites the security row-ID), extends (adds AI-specific control-defeated variants alongside the security row), or is authoritative (the security register does not carry it). Include appetite-band alignment notes.

Also produce a **schema-alignment YAML** — a version of your row schema that shows the fields chapter 03 requires, filled in for one of your three rows from exercise 02, with every field's provenance to a 27005 process step written inline.

## Requirements

- Output as `mod-103-ex03-iso-27005-alignment.md` plus a single `row-schema-aligned.yaml` for the demonstration row.
- The process-step filing matrix has seven rows (one per 27005 step) and at least two columns: *AI-register artifact* and *where it lives* (file path, GRC-platform ID, or "not yet produced"). Any 27005 step with no artifact is called out as a coverage gap with a plan to close.
- The cross-register reconciliation table has at least *four* rows covering: one shared *asset* (e.g., a data store, an LLM API vendor, a customer-support platform), one shared *threat source* (e.g., prompt injection as a class of input attack, insider misuse), one *AI-authoritative* item (a fairness or over-reliance harm the security register does not carry), and one *appetite-band alignment* item (either your bands match, or a translation matrix is defined).
- The schema-alignment YAML fills every chapter 03 field for one row, and each field carries a `# from 27005 step: <step name>` comment inline. Fields with no step (e.g., internal row metadata) are marked `# metadata`.
- The reconciliation table names the *specific* security-register row IDs (or provisional IDs if you constructed a provisional register) so the trace is unbroken.
- If the enterprise appetite bands and your rubric bands are mismatched, publish the translation matrix (yours → theirs on export) and note the ERM function you would negotiate with to align.
- The document includes a short *parallel-register anti-pattern check* — a paragraph confirming the AI register cites the security register where it can and does not silently duplicate rows.

## Starter guidance

- Fill the process-step matrix by walking your existing artifacts, not by walking the 27005 steps. You already produced *things* in mods 101-102 and exercises 01-02; the matrix is a filing exercise, not an authoring exercise. Only after filing what you have do you look at empty steps and plan how to close them.
- On the shared-asset row: the LLM API vendor is almost always on the security-side register (as a third-party supplier). Cite that row and add only the *AI-specific* additions (e.g., the model-output-integrity control, the AI-specific control-defeated variants). Do not restate the vendor's security posture; the security register already carries that.
- On the shared-threat row: prompt injection is now widely enough understood that it will often already appear as an input-validation risk on the security-side register at some level. Cite it and add the AI-specific consequences.
- On the AI-authoritative row: fairness harms, over-reliance, hallucinations, and human-AI configuration risks do not show up on a classical security register. Note explicitly that the security register does not carry them; the AI register is the source of truth.
- On appetite alignment: if your rubric goes up to "Very Severe" and the enterprise stops at "High," you do not have a fight; you have a mapping — Very Severe rolls up to High on export. Publish that mapping and move on. If the mapping loses information the risk council needs, that is a scale-negotiation with the ERM function.
- Do *not* introduce AI-specific process steps that break the 27005 shape (e.g., "AI risk identification" as a new step). ISO/IEC 23894 fills 27005 with AI-specific content within the same steps; adding steps is the anti-pattern.

## Acceptance criteria

The submission is accepted when:

- The seven-step filing matrix is complete; any empty step is a named coverage gap with an owner and a close plan.
- The reconciliation table covers the four required item types (shared asset, shared threat source, AI-authoritative, appetite alignment) with specific row-ID citations.
- The schema-alignment YAML fills every chapter 03 field and comments each field's 27005-step provenance.
- The parallel-register anti-pattern check is present and honest — if you found any duplication in your existing draft, it is called out and the resolution planned.
- A 27005-trained analyst can read your document and answer "which of my process steps does the AI register plug into where, and where does it duplicate or defer to my register?" without asking follow-up questions.

## Stretch goals

- Add a *reverse* reconciliation — take three rows from the *security-side* 27005 register that touch AI systems and note whether they should acquire AI-specific additions your AI register can supply. This positions the AI register as an *upstream* of enrichment for the security register too.
- Draft the ISO/IEC 23894-to-27005 content-overlay note: for each 27005 step, name the AI-specific content 23894 recommends adding (23894 Annex A / Annex B examples). This becomes reusable guidance for future systems.
- If your ERM tool supports it, prototype the export: emit your AI register in the ERM's ingestion format and verify that the enterprise heat map correctly places your rows. If it does not, note where the schema translation broke.
- If sector overlays apply (SR 11-7, FDA GMLP, EU AI Act), produce a mini-matrix showing which 27005 step each sector obligation flows through — this closes the loop back to mod-101 ch 04 / ch 05.

## Deliverable format

A single markdown document with the following sections, plus the YAML file:

1. *Process-step filing matrix* — 7 rows × 2+ columns.
2. *Cross-register reconciliation table* — ≥ 4 rows spanning asset / threat / AI-authoritative / appetite.
3. *Schema-alignment YAML* — reference to `row-schema-aligned.yaml`; brief note per field group on its 27005-step provenance.
4. *Appetite-band mapping* — either the bands align or the translation matrix is published.
5. *Parallel-register anti-pattern check* — one paragraph.
6. *Coverage gaps and close plans* — one bullet per gap.
7. *Reverse-reconciliation notes* (stretch), *23894 content overlay* (stretch), *export prototype notes* (stretch), *sector-overlay flow* (stretch).

## Submission

Attach the markdown and YAML to the module submission channel. Where your organisation's security register is confidential, redact identifiers and preserve the reconciliation shape; a reviewer must be able to see the pattern of deference vs. extension vs. authoritative even if the row contents are blanked.
