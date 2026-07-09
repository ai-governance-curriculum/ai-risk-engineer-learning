# The NIST GenAI Profile as a Harm-Model Starting Checklist

The Generative AI Profile of the AI RMF — **NIST AI 600-1**, published July 2024 — is what you point at the day someone tells you to "assess the GenAI system." It enumerates twelve risks unique to or exacerbated by generative AI, and for each risk it suggests actions across the four RMF functions. You use it as a *checklist against which your harm model is authored* — not as the harm model itself.

## Motivation

Ad-hoc harm modelling produces harm models that miss things auditors and red-teamers routinely find. Every one of the twelve risks below has been observed in shipped systems and is asked about in review. Starting from the twelve gives you a baseline the auditor cannot claim you skipped.

The harm model itself — misuse personas, cascading harms, incident-corpus grounding, AIR-Bench 2024 alignment — is taught in **mod-102**. This chapter's job is to give you the *checklist* and the discipline of using it.

## The twelve risks (verbatim short forms from AI 600-1)

AI 600-1 §2 names the following twelve GenAI risks. Names below match the document; the "engineering read" column is your work.

| # | AI 600-1 risk | Engineering read (one sentence you write per system) |
|---|---|---|
| 1 | **CBRN Information or Capabilities** — lowered barrier of entry or increased uplift for chemical, biological, radiological, or nuclear harm | Refusal-eval on CBRN-uplift prompt suites; capability-limit tests where the system can generate procedural detail |
| 2 | **Confabulation** — false, inaccurate, or misleading content ("hallucination") | Groundedness / factuality metric + retrieval-alignment metric + citation-required guardrail where applicable |
| 3 | **Dangerous, Violent, or Hateful Content** | Output-safety classifier (Llama Guard, ShieldGemma, OpenAI Moderation, or equivalent) — mod-108 |
| 4 | **Data Privacy** — leakage or inference of personal data | MIA / LiRA baselines (mod-107) + PII detection / redaction pipeline (Presidio) at input, retrieval, and output |
| 5 | **Environmental Impacts** — energy, water, carbon | Training / inference footprint disclosure; typically defer measurement to infra team but you file the disclosure |
| 6 | **Harmful Bias or Homogenization** | Group-fairness slices (mod-106); homogenisation checks on decision-recommendation outputs |
| 7 | **Human-AI Configuration** — miscalibrated reliance, insufficient oversight, unclear boundaries | Human-oversight design review — this is where you meet EU AI Act Article 14 (see chapter 04) |
| 8 | **Information Integrity** — synthetic content flooding, election / civic mis-info | Provenance guardrail (C2PA / watermarking where the system is capable); output-tag conventions |
| 9 | **Information Security** — expanded attack surface (prompt injection, jailbreak, tool-abuse, model-extraction) | LLM red-team suite (mod-104); NIST AI 100-2 adversarial-ML coverage (mod-105) |
| 10 | **Intellectual Property** — memorised training data, style-appropriation, licence violation | Memorisation eval; licence-flag guardrail on generated outputs; training-data provenance record (mod-110) |
| 11 | **Obscene, Degrading, and/or Abusive Content** — including NCII / CSAM production | Output classifier + policy-level refusal; escalation-to-Trust-and-Safety hook |
| 12 | **Value Chain and Component Integration** — third-party model / dataset risk propagating in | ML-BOM (mod-110); ModelScan / picklescan gate on model registry ingestion |

Read AI 600-1 §3 (suggested actions) for each risk. That section is written against the same GOVERN / MAP / MEASURE / MANAGE structure as AI 100-1 — so each risk decomposes into things you *do*, not just things you *worry about*.

## How you use the twelve

Not every risk applies to every system. A code-review copilot is not usually a CBRN uplift concern; a public-facing character chatbot is not usually a value-chain concern in the same way as a hedge-fund GenAI pipeline. But you must **explicitly rule each risk in or out**, and the reason has to hold up.

The three-column applicability sheet is the artifact:

| Risk # | Applicability to *this system* | Reason |
|---|---|---|
| 1 | Out | System has no domain expertise or refusal-behavior touching CBRN topics; it is a code assistant on internal repos |
| 2 | In | System summarises internal knowledge-base articles; hallucination directly harms downstream decisions |
| 3 | In | Public-facing chat; standard content-safety classifier applies |
| ... | ... | ... |

The sheet has to be *complete* — twelve rows every time — and *versioned*, because rulings change as the system's use case grows. A ruling of "out" that later becomes "in" without a new applicability row is an audit finding.

## Two example decompositions

### Risk 2 — Confabulation for a customer-support RAG assistant

*Applicability:* In. The assistant reads product docs and returns instructions to end users.

*What you measure:* groundedness (does every claim in the answer trace to a retrieved passage) and retrieval-alignment (does the retrieved passage actually support the answer). Both metrics are owned by `ai-eval-engineer` at level 30 — you consume the metric and *set the risk threshold*.

*What you gate:* if groundedness < threshold on the CI eval-set, the model candidate does not ship. If groundedness drops in production monitoring, an incident-candidate is filed (mod-111).

*What you file as evidence:* groundedness scorecard per release, plus the citation-required guardrail's config (mod-108).

### Risk 9 — Information Security for a customer-support RAG assistant

*Applicability:* In. Both direct prompt injection (a user pasting an instruction into the chat window) and indirect prompt injection (a KB article containing an attacker-authored instruction) are plausible.

*What you measure:* attack-success-rate on a garak + PyRIT direct-injection suite and an AgentDojo / InjecAgent-style indirect-injection suite. The suite is pinned; harmful payloads are hosted per the workflow taught in mod-104.

*What you gate:* attack-success-rate ≤ target on the pinned suite. Any release-breaking finding is turned into a permanent regression fixture (mod-111).

*What you file as evidence:* red-team report signed by the run pipeline, suite version pinned, guardrail configs versioned, prompt-injection-detection monitor spec (mod-109).

## Where the GenAI Profile stops

AI 600-1 does not:

- rank the twelve risks against each other,
- set thresholds for any of them,
- tell you which tools to use,
- assign accountability inside your org.

Those are all *your* engineering / governance decisions. AI 600-1 is a checklist, not a specification.

## Interaction with other frameworks

The twelve GenAI risks are not a taxonomy in the sense of ISO/IEC 23894's risk sources — they are a *starting checklist*. In your harm model (mod-102) you will cross-walk each of the twelve to:

- MIT AI Risk Repository top-level categories,
- OECD.AI incident types,
- AIR-Bench 2024 policy-derived categories,
- ISO/IEC 23894 Annex A risk sources.

That crosswalk is what a senior architect (level 50) inherits when they build the org-wide control library.

## Summary

- AI 600-1 (NIST GenAI Profile, July 2024) enumerates twelve GenAI-specific risks and suggests actions per risk.
- You use it as a *checklist* your harm model must clear, not as the harm model itself.
- Every system requires a *twelve-row applicability sheet*; "out" rulings must state a reason and get re-visited when scope changes.
- Each in-scope risk decomposes into a metric, a threshold, a guardrail, a monitor, and an evidence artifact (same shape as chapter 01).
- Exercise 02 walks the twelve for a real system.
