# Exercise 04 — EU AI Act Articles → Engineering Deliverables

**Estimated effort:** 2 hours

## Objective

Produce the article-to-artifact map for Articles 9, 10, 14, 15, 55, 61, and 72 of the EU AI Act (Regulation (EU) 2024/1689) against your running system. When you are done, a compliance reviewer should be able to walk each article and, without further translation, be pointed at a specific engineering artifact (or, for Article 55, at the reason it is not in scope).

## Prerequisites

- Chapter 04 (this module) read.
- The **EU AI Act consolidated text** on `eur-lex.europa.eu` open in a tab; you will re-read Articles 9, 10, 14, 15, 55, 61, and 72 in this exercise.
- Exercises 01-03 completed for the same running system. This exercise re-files the *same* engineering evidence against a new frame.
- If your running system would clearly not be a "high-risk AI system" under Annex III (say, a research prototype only used internally), state that at the top and choose one of two options:
  - proceed anyway as a "what would we build *if* we placed this on the EU market" exercise, or
  - substitute `CarePilot` (fictional health-triage assistant from exercise 01) as a Annex III high-risk analogue.

## Problem statement

Produce a single **article-to-artifact map** with one section per article listed below. Each section must state, in order:

1. The obligation, one paragraph, paraphrased (do not paste article text verbatim).
2. Whether the obligation is *in scope* for your running system, and the reason.
3. The engineering artifact(s) that satisfy the obligation, with paths and signers.
4. The traceability line — which exercise-01 / exercise-02 / exercise-03 row does this share evidence with?
5. Open items — if the obligation is not fully discharged, what remains, on what timeline.

## Requirements

- Cover **all seven** articles: 9, 10, 14, 15, 55, 61, 72.
- Article 55 (systemic-risk GPAI) may be marked *out of scope* for most running systems — if so, still write the paragraph and state why. Do not skip.
- Article 61 (registration) requires *technical fields* to be enumerated even if the actual registration form is out of scope for this exercise; enumerate what you would supply.
- For Article 15 (accuracy / robustness / cybersecurity) produce a *traceability table* — every sentence in the future model-card accuracy declaration must be traceable to an eval-set + result + evaluation run.
- Every artifact citation must resolve to a path that also appears in exercise 01 or exercise 03. New artifacts are OK but must be *filed back* into exercise 01 as an addendum.
- Cite the article numbering source (OJ published text vs. consolidated text) with a link.
- Any obligation you cannot fully discharge is marked **OPEN** with a target date.

## Starter guidance

- Article 9's risk-management-system obligation is *your risk register from exercise 01 / 02*. Don't build a second thing; re-file the first.
- Article 10 is a *data* article — the dataset card, bias audit, and data-lineage manifest are the artifacts. Bias-audit depth is taught in mod-106; here you just point at where the artifact will live.
- Article 14 is a *design* article — the artifact is the human-oversight design document and the intervention API. Product / UX contribute; you author the technical section.
- Article 15 is where *all* your evaluation-engineering work lands. Expect this section to be the longest.
- Article 55 applies only to GPAI models classified with systemic risk. If you are inside a provider organisation whose model has that classification, you have a lot more to build. Most learners will mark this out of scope with a citation to why.
- Article 61 needs *technical* fields, not the full registration record. Enumerate what your engineering evidence would supply.
- Article 72 is post-market monitoring — the mod-109 wiring is the artifact.

## Acceptance criteria

- All seven articles covered.
- Each section has all five required paragraphs / tables.
- Article-15 traceability table is complete for every accuracy claim.
- Every artifact resolves to a path also present in exercises 01 or 03 (with the note *in exercise 01, row N*, or *in exercise 03, Annex A control N*).
- Article numbering is cited to the source (consolidated OJ text URL).
- Open items are visible, dated, and owned.

## Stretch goals

- Add Article 13 (transparency and information to deployers) and Article 12 (record-keeping / logs) to the map — both directly consume engineering evidence and often surface together with 14 and 15.
- Add Article 27 (fundamental-rights impact assessment) — this connects the AI Act to the ISO/IEC 42005 shape from exercise 03.
- Track *which harmonised standards* (CEN-CENELEC JTC 21 workplan) will eventually formalise the obligations you have covered, and add a `harmonised-standard` column with the placeholders. Mark as `<!-- needs-research -->` while the standard is still in draft.

## Deliverable format

A single markdown document `mod-101-ex04-eu-ai-act-map.md` structured by article. Include a *front-matter* section citing article numbering source, running system, and classification assumption (high-risk under Annex III / GPAI / out-of-scope / other).

## Submission

Attach the markdown file. Do not include paywalled harmonised-standard drafts; cite instead. If your running system is under NDA, redact model names but retain the structure.
