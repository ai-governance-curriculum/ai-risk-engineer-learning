# Exercise 01 — Misuse Persona and Capability Surface Mapping

**Estimated effort:** 2 hours

## Objective

Produce the first two columns of the harm-model artifact for your running system: a **complete persona × capability-surface matrix** with concrete scenario stubs in each occupied cell. The output is the frame on which exercises 02-05 will hang evidence, taxonomy tags, causal attribution, and downstream tiering.

## Prerequisites

- Chapter 01 (this module) read.
- Access to whatever intake documentation exists for your running system (mod-101 exercises may already have produced this; if not, write a one-page provisional intake and mark it *provisional*).
- If you cannot use a real system, use one of the fictional case studies from mod-101 (e.g., `CarePilot` — a hospital-triage support assistant) as your running system.

## Problem statement

Pick **one AI system** — the same one you used through mod-101 is the recommended default. Produce a persona × capability-surface matrix that:

- Names *all four* misuse personas from chapter 01 (adversarial user, insider, upstream data supplier, downstream integrator) along one axis.
- Names *at least eight* capability surfaces from your system along the other axis. Draw from the checklist in chapter 01, but add / drop rows to fit your system (an agentic system will have more; a classifier will have different ones).
- In each occupied cell, writes a *concrete* scenario stub — a one-sentence description of what a persona of that type could do against that capability surface, in *your* system.
- Marks empty cells with an explicit reason ("this persona has no reachable action against this capability surface in this system").

## Requirements

- Output as a single markdown document `mod-102-ex01-persona-capability-matrix.md` in your working directory.
- Personas × capability surfaces ≥ 4 × 8 = 32 cells. Occupied cells carry a scenario stub; empty cells carry a reason.
- Each scenario stub is *specific to your system* — no generic "prompt injection" language. Name the input surface, the observable behaviour, and the immediate harm in one sentence.
- Assign a stable `id` to each occupied scenario cell in the form `HR-<system>-<surface>-<n>` (e.g., `HR-CS-RTV-01` for the customer-support assistant, retrieval surface, first scenario). These IDs are what exercises 02-05 will cite; they do not change once assigned.
- For each occupied cell, include a *one-sentence* rationale: why this persona × surface combination produces a meaningful risk in your system.
- Include an explicit *provisional or verified* marker at the top of the document showing whether your intake is verified against a real intake record or written provisionally.

## Starter guidance

- Start from the capability-surface axis. List every capability surface *the system actually has* before you think about who could exercise it. If you list "tool calling" but the system has no tools, drop the row.
- Then walk each surface across the four personas. Personas are cheap to iterate; capability surfaces are the substantive work.
- For any cell you can't quickly fill, ask: is this persona actually able to reach this surface in this deployment? If reachability is unclear, mark the cell as *open — reachability unknown* and open a note to check with the application team. This is more useful than pretending.
- If two of your cells describe the *same* underlying scenario at different granularities, merge them and cite both cells' IDs.
- If a scenario cell requires two personas at once (e.g., an insider *and* an adversarial user), split it into two rows with a shared `id_group` — that maintains the personas-are-single invariant from chapter 01.

## Acceptance criteria

The submission is accepted when:

- The matrix has ≥ 4 personas × ≥ 8 capability surfaces, populated per the rules above.
- Every occupied cell has a scenario stub, a stable `id`, and a one-sentence rationale.
- Every empty cell has an explicit reason.
- The intake `provisional / verified` marker is present and correct.
- A reviewer with no prior exposure to your system can, in under 15 minutes, look at the matrix and understand what surfaces exist, what personas can reach them, and where the harm-model rows will land.

## Stretch goals

- Add a fifth persona category *specific to your system* if it fits (e.g., *regulator or external auditor* for a system that produces audit-relevant outputs; *automated downstream agent* for an agent-to-agent integration). Justify the addition briefly.
- Draft the immediate → secondary → cascading → systemic harm chain (chapter 01) for the top three occupied cells that look most severe on inspection. This is not required at this exercise, but it seeds exercise 03's overlay work.
- Cross-reference each capability surface with the AIRO ontology's capability class (chapter 03). This is a stretch tag; empty is acceptable at this exercise.

## Deliverable format

A single markdown document with the following top-level sections:

1. *Running system* — one-paragraph description; intake reference or provisional marker.
2. *Capability surfaces* — bulleted list with a one-sentence description each.
3. *Persona × capability-surface matrix* — the table, with scenario stubs in occupied cells and reasons in empty cells.
4. *Assigned IDs* — flat list of every occupied cell's `id` and its scenario stub.
5. *Open reachability questions* — bulleted list of the cells you couldn't confidently classify, and who you need to check with.

## Submission

Attach the markdown file to the module submission channel. If your running system is under NDA, redact identifying details but preserve the capability-surface names — the reviewer cannot review a matrix whose surfaces are blank.
