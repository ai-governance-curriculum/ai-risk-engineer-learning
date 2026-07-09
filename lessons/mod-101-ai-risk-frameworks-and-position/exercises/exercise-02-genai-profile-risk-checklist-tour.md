# Exercise 02 — NIST GenAI Profile 12-Risk Checklist Tour

**Estimated effort:** 2 hours

## Objective

Walk the twelve GenAI risks in **NIST AI 600-1** for your running example, produce the *applicability sheet* required to defend "in-scope" vs "out-of-scope" decisions, and decompose every in-scope risk into a metric / threshold / guardrail / monitor / evidence set.

## Prerequisites

- Chapter 02 (this module) read.
- The **NIST AI RMF Generative AI Profile** (AI 600-1, published July 2024) in front of you — freely available from `nvlpubs.nist.gov` / `airc.nist.gov`.
- The same running system chosen in exercise 01. If your running system is *not* a generative-AI system, either (a) substitute the fictional `CarePilot` from exercise 01 (which is generative), or (b) explicitly identify a generative *component* of your system (e.g., a summarisation service that fronts a classical model).

## Problem statement

Produce two documents:

1. A **12-row applicability sheet** — one row per AI 600-1 risk — with an in-scope / out-of-scope ruling and the reason. No rows may be skipped.
2. For each **in-scope** row, a five-sentence decomposition (as in chapter 02) into: what the risk *means for this system*, what you *measure*, what you *gate on*, what you *guard against*, and what you *file as evidence*.

## Requirements

- Applicability sheet must contain exactly twelve rows corresponding to AI 600-1's twelve risks. Use the AI 600-1 short names verbatim.
- Every applicability decision — in-scope OR out-of-scope — must include a *reason* of at least one sentence. "Not applicable" without a reason is rejected.
- For every in-scope risk, produce the five-sentence decomposition described above.
- For every in-scope risk, name the *tool* you would use for the measurement (garak, PyRIT, Fairlearn, ML Privacy Meter, Presidio, Llama Guard, ShieldGemma, OpenAI Moderation, HarmBench, AgentDojo, InjecAgent, etc.). If no tool exists, mark **NEW-TOOLING-NEEDED** and describe what would need to be built (this becomes a mod-104/108 backlog item).
- Cross-reference every in-scope row to one or more of the NIST AI RMF sub-categories mapped in exercise 01. In-scope risks that map to *no* sub-category are a gap in your exercise-01 output — go back and add them.

## Starter guidance

- Read AI 600-1 §2 (risks) end-to-end before starting the applicability sheet. Don't rule anything out prematurely.
- Read AI 600-1 §3 (suggested actions) for each in-scope risk. Many of your five-sentence decompositions will crib directly from §3.
- Some risks (data privacy, information security) are almost always in-scope. Some (environmental impacts, CBRN, obscene / abusive content) are context-dependent — be honest about the ruling.
- Where a risk applies but the mitigation is genuinely upstream (e.g., CBRN uplift is mitigated by a base-model provider's RLHF, not by your app), say so — and file the *inherited-control* reasoning as your evidence, not silence.

## Acceptance criteria

- 12-row applicability sheet complete; no gaps; every ruling justified.
- In-scope rows each have a five-sentence decomposition with named tooling.
- Every in-scope decomposition cross-references at least one RMF sub-category from exercise 01.
- Every measurement and gate can be executed by a peer reading only your document — no hidden assumptions.
- Any risk marked **NEW-TOOLING-NEEDED** is filed as a backlog item into an accompanying `mod-101-ex02-backlog.md`.

## Stretch goals

- Map each of your in-scope AI 600-1 risks onto (a) MIT AI Risk Repository categories and (b) OWASP LLM Top 10 entries. This gives the harm model in mod-102 a head start.
- For risks where mitigation is upstream (e.g., inherited from a base-model provider), file a *supply-chain evidence request* to that provider — this seeds the mod-110 supply-chain work.
- Add a *residual risk after control* one-liner per row and a *severity rating* using whatever qualitative scale (Low / Medium / High / Critical) your organisation already uses; this seeds mod-103's scoring exercise.

## Deliverable format

Two markdown files:

- `mod-101-ex02-applicability.md` — the twelve-row sheet plus in-scope decompositions.
- `mod-101-ex02-backlog.md` — the backlog of any NEW-TOOLING-NEEDED and inherited-control-evidence items.

## Submission

Attach both files. Do not include raw jailbreak payloads or adversarial prompts in the applicability sheet — payloads live in the mod-104 harmful-payload workflow.
