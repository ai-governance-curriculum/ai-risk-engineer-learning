# Exercise 01 — NIST AI RMF Obligation → Engineering Artifact Map

**Estimated effort:** 2 hours

## Objective

Produce a *reviewable map* from a chosen slice of NIST AI RMF 1.0 sub-categories to concrete engineering artifacts, for one AI system you have picked as the module's running example. The output is the deliverable this whole module builds on — every later exercise re-uses the same running system.

## Prerequisites

- Chapter 01 (this module) read.
- Access to the **NIST AI RMF Playbook** (`airc.nist.gov/AI_RMF_Knowledge_Base/Playbook`) and to a copy of AI 100-1.
- A running example: one AI system with enough public documentation for you to reason about it. Suggested options:
  - a public-facing coding assistant (e.g., a hosted code-completion API you have used),
  - a customer-support LLM assistant on a public product,
  - a document-summarisation service you have deployed or contributed to at work,
  - a public bank / insurance / health decision-support system that has published a model card.
- If you cannot use a real system, use the *fictional* case study `CarePilot`: a hospital-triage support assistant for a mid-sized regional health system, RAG-based over triage protocols, human-in-the-loop before every recommendation. Document it briefly (one page) as your running system.

## Problem statement

Pick **one AI system** (real or the fictional `CarePilot`). For **at least ten NIST AI RMF sub-categories** — at least three from MEASURE and at least three from MANAGE (the rest may come from GOVERN or MAP) — produce a *reviewable map* that answers, in five sentences per sub-category (as in chapter 01), the following:

1. What the sub-category *says* (verbatim outcome statement).
2. What it *means* for your specific system.
3. How you *test* it (tool, dataset, or eval).
4. What you *treat as pass* (threshold, and where the threshold is defined).
5. What you *file as evidence* (artifact name, path, signer).

## Requirements

- Output as a single markdown document `mod-101-ex01-rmf-map.md` in your working directory. This is your submission.
- Cover at least **10 sub-categories** in total; **at least 3 from MEASURE and at least 3 from MANAGE**.
- For each sub-category, include the *outcome statement verbatim* and cite the version of the Playbook you used (AI 100-1 §, Playbook entry date if visible).
- Any sub-category where you conclude an artifact does not yet exist must be marked **OPEN** and turned into a *TODO row in the risk register* — not silently skipped.
- All artifact paths and threshold references should be plausible even if the artifact isn't yet built (e.g., `evidence/measure-2.7/<release>/report.json` and `risk-register/2025/CarePilot.yaml`). The reviewer needs to see that you have designed the *shape* of the artifact.
- Where the sub-category is answered by a peer role, mark it explicitly: `hand-off → ai-eval-engineer (L30) for methodology; consumed metric = groundedness`.

## Starter guidance

- Start from the sub-category list, not the tools. Do not begin by browsing garak / PyRIT / Fairlearn. Read the sub-category outcome first; then decide what tool tests it.
- Prefer the concrete over the aspirational — "attack-success-rate on garak at commit `<hash>` ≤ target defined in risk register" beats "we evaluate against jailbreaks."
- If a sub-category is out of your system's scope (e.g., MEASURE 2.11 fairness for a purely internal code-completion tool), still write a one-sentence *reason* — the auditor's next question is *why*.
- Refer to chapter 01's table for the concrete pattern to reproduce.

## Acceptance criteria

The submission is accepted when:

- The map covers **≥10 sub-categories** including **≥3 MEASURE and ≥3 MANAGE**.
- Every row has all five fields populated OR is marked **OPEN** with a linked risk-register TODO shape.
- Every threshold cites its *source* (risk register, chapter, standard).
- Every artifact has a *path* and *signer*.
- The Playbook version used is cited at the top of the document.
- A reviewer with no prior exposure to your running system can, in under 15 minutes, look at the map and understand what tests run, what passes, and what evidence would be produced.

## Stretch goals

- Cover **all** MEASURE sub-categories for your system, not just three, and identify *which sub-categories share a single artifact* (that convergence is a real production win — evidence you can point at is more valuable than 40 tiny reports).
- Cross-walk each of your MEASURE rows to the corresponding ISO/IEC 42001 Annex A control (chapter 03) — you will need this in exercise 03.
- Add a `residual_risk_after_control` column and, for each row, describe how the residual risk is bounded — this feeds mod-103.

## Deliverable format

A single markdown document with the following top-level sections:

1. *Running system* — one-paragraph description, links to public documentation.
2. *Playbook version cited* — date, URL.
3. *Sub-category map* — one row per sub-category.
4. *Open items* — table of everything marked OPEN, with a target date for closing.
5. *Peer hand-offs* — table of every metric consumed from `ai-eval-engineer` / `model-evaluation-engineer` / `ai-infra-mlops`.

## Submission

Attach the markdown file to the module submission channel; do not paste raw payload data or secrets. If your running system is under NDA, redact and note the redaction.
