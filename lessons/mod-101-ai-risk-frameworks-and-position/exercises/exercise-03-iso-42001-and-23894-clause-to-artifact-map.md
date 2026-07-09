# Exercise 03 — ISO/IEC 42001 & 23894 Clause → Artifact Map

**Estimated effort:** 2 hours

## Objective

Build the ISO clause-to-artifact map for your running system. When you are done, an ISO/IEC 42001 lead auditor should be able to walk in and, for every clause and every Annex A control you claim is applicable, be pointed at a specific piece of engineering evidence. You will also produce the ISO/IEC 23894 risk-source table for the same system.

## Prerequisites

- Chapter 03 (this module) read.
- Access to (or a working summary of) **ISO/IEC 42001:2023**, **ISO/IEC 23894:2023**, and — where you can reach it — **ISO/IEC 42005:2025**. Formal copies are paywalled through your ISO national body or through a reseller; if you cannot access the paid text, you may work from the *publicly available* summaries and outline documents published by ISO on `iso.org` and from published lead-implementer / lead-auditor training curricula.
- Exercise 01 and exercise 02 completed for the same running system.
- If you have no ISO access at all, note it at the top of your submission and work from the *publicly documented clause and control names*; do not fabricate control text.

## Problem statement

Produce three deliverables:

1. A **42001 clause-to-artifact map** covering **clauses 4-10** — for each clause requirement, name the engineering artifact that satisfies it and its path.
2. A **42001 Annex A control-to-artifact map** for at least **ten Annex A controls** you claim as applicable — for each, cite the control identifier, the control objective (one sentence), and the artifact + path + signer.
3. A **23894 risk-source table** — for at least **eight risk sources** drawn from ISO/IEC 23894 Annex A, cite the risk source, whether it is *in scope* for your system, the lifecycle stage (Annex B mapping), and the row(s) in your risk register (from exercise 01 / exercise 02) that address it.

## Requirements

- All three maps use the *running system* from exercise 01.
- Every artifact you cite must already appear in your exercise 01 / exercise 02 map — this exercise is about *re-filing existing evidence against ISO*, not producing new evidence. If an ISO clause has no candidate artifact, mark it **GAP** and file it as a backlog row.
- For every Annex A control cited, distinguish between:
  - *Implemented* — control is in place and evidence exists.
  - *Planned* — control is designed but not yet run.
  - *Not applicable* — with a stated reason.
- Version-stamp the ISO editions you cite (e.g., "ISO/IEC 42001:2023").
- Where you use publicly available *summaries* instead of the paywalled standard text, cite the summary source and note "clause-level identifiers only; text paraphrased."

## Starter guidance

- Do the clause-to-artifact map first; the Annex A map follows naturally once you have listed which artifacts you produce.
- Use the "one artifact files against several standards at once" insight from chapter 03. A single red-team report is likely to file against 42001 Annex A operational controls, 23894 as treatment evidence, 42005 as impact-assessment robustness section, and 25059 as *robustness* characteristic evidence.
- 23894 Annex A gives you the *risk sources* (bias, opacity, complexity, autonomy, ML-specific issues, environmental impact, security, safety, privacy, and more). If your risk register from exercise 02 uses different names, the crosswalk column in the risk-source table is where you reconcile.
- When in doubt about a clause number, use the ISO/IEC 42001 outline document ISO publishes freely, or the CEN-CENELEC JTC 21 workplan references.

## Acceptance criteria

- Clause 4-10 map is complete — every clause has either an artifact or a **GAP** row.
- ≥10 Annex A controls covered with implementation status.
- ≥8 23894 risk sources covered in the risk-source table with in-scope ruling and lifecycle stage.
- Every artifact citation resolves to a specific path (`evidence/…`, `risk-register/…`, `model-cards/…`).
- Every *paraphrased* control text is marked as paraphrased; no invented control text.
- Where a peer role owns the artifact (e.g., `ai-eval-engineer` owns groundedness methodology), the peer is named.

## Stretch goals

- Extend the map to include ISO/IEC 42005:2025 impact-assessment sections, and produce a *skeleton impact assessment* pointing at your evidence.
- Add ISO/IEC 25059 quality characteristics as a column on the Annex A map — every Annex A control you implement should map to one or more 25059 characteristics (robustness, transparency, fairness, controllability, etc.).
- Add an ISO/IEC 27005 crosswalk — because your organisation's information-security risk register is likely 27005-shaped, this shows how AI-risk plugs in without a schema fight.

## Deliverable format

A single markdown document `mod-101-ex03-iso-map.md` with three sections corresponding to the three deliverables. Each section begins with a one-paragraph reading-guide for a lead-implementer / lead-auditor audience.

## Submission

Attach the markdown file. Do not paste ISO standard text verbatim in the submission (licensing); paraphrase, cite the clause identifier, and reference the ISO edition.
