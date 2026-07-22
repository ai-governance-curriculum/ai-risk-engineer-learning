# mod-108 — Guardrail Engineering and Effectiveness Measurement

**Estimated effort:** 16 hours
**Position in the track:** eighth module; assumes mods 101–107 are done.

Mods 101–103 fixed the frameworks, the harm model, and the register that quantifies each scenario. Mods 104–107 populated the harm side: LLM red-team payloads (104), adversarial-ML failure modes (105), fairness / bias / explainability findings (106), and privacy-risk positions (107). This module turns those findings into *running code*. You design an input / output guardrail architecture for a concrete LLM application, ship it with NVIDIA NeMo Guardrails, Guardrails AI, Meta Llama Guard, Google ShieldGemma, OpenAI Moderation, Anthropic Constitutional Classifiers, and Microsoft Presidio, then measure its true positive / false positive / true negative / false negative rate on a bounded adversarial set with explicit cost and latency accounting. You produce an FP/FN report a product manager can read, evaluate the third-party guardrail vendors, and hand a measurement contract to `ai-eval-engineer` at level 30.

The guardrails you build here are the *controls* whose presence collapses inherent risk to residual risk in the mod-103 register, and whose failure produces the control-defeated scores. If a guardrail is not shipped here, the residual number in the register cannot credit it.

## Learning objectives

- Design an input / output guardrail architecture for an LLM application — input filters, retrieval filter, tool-invocation policy, output validators, safe-completion / refusal shaping — that composes cleanly with the underlying model without silent single-points-of-failure.
- Ship guardrail code with **NVIDIA NeMo Guardrails** (Colang rails, flows, dialog manager), **Guardrails AI** (schema + Validator Hub), and a custom lightweight middleware layer droppable into a FastAPI / LangChain / LangGraph app.
- Ship classifier-based guardrails with **Meta Llama Guard**, **Google ShieldGemma**, **OpenAI Moderation**, and **Anthropic Constitutional Classifiers**, and compose classifier + rule + retrieval-based defence in depth.
- Ship PII guardrails with **Microsoft Presidio** (analyser + anonymiser) and integrate them into ingest, retrieval, and output paths.
- Measure guardrail effectiveness — TP / FP / TN / FN on a bounded adversarial set — with explicit cost and latency accounting, and produce an FP/FN report a product manager can read.
- Evaluate third-party guardrail vendors (**Lakera Guard**, **Arthur AI**, **Robust Intelligence**, **Calypso AI**, **HiddenLayer**, **Protect AI**) with a defensible build-vs-buy matrix, and hand the app-side measurement contract to `ai-eval-engineer` (peer, level 30).
- Map every guardrail slot to an **OWASP LLM Top 10** entry, a **MITRE ATLAS** tactic, and a **Google SAIF** pillar — so the coverage story is defensible.

## Chapters

1. [`01-guardrail-architecture-and-composition.md`](01-guardrail-architecture-and-composition.md) — the input / retrieval / tool / output / refusal-shaping slot set; defence in depth; the silent single-point-of-failure anti-pattern; slot-to-attack-class mapping across OWASP LLM Top 10, MITRE ATLAS, and Google SAIF.
2. [`02-rule-and-flow-guardrails-nemo-and-guardrails-ai.md`](02-rule-and-flow-guardrails-nemo-and-guardrails-ai.md) — NVIDIA NeMo Guardrails (Colang rails, flows, dialog manager); Guardrails AI (schema + Validator Hub); a custom lightweight middleware layer for FastAPI / LangChain / LangGraph; when rule-based fits and when classifier-based fits.
3. [`03-classifier-guardrails-llama-guard-shieldgemma-moderation-constitutional.md`](03-classifier-guardrails-llama-guard-shieldgemma-moderation-constitutional.md) — Meta Llama Guard (3 / 4), Google ShieldGemma, OpenAI Moderation, Anthropic Constitutional Classifiers; how each is trained and where each fits; classifier + rule composition.
4. [`04-pii-guardrails-with-presidio.md`](04-pii-guardrails-with-presidio.md) — Microsoft Presidio (analyser + anonymiser); integration into ingest, retrieval, and output paths; recogniser tuning; false-positive / false-negative accounting; the "PII already in the RAG store" mitigation.
5. [`05-effectiveness-measurement-fp-fn-cost-latency.md`](05-effectiveness-measurement-fp-fn-cost-latency.md) — the effectiveness eval: bounded adversarial set (hosted per the mod-104 harmful-payload workflow), TP / FP / TN / FN reporting, cost and latency accounting; the FP/FN report a product manager reads; per-slot residual-risk update into the mod-103 register.
6. [`06-vendor-build-vs-buy-and-app-eval-contract.md`](06-vendor-build-vs-buy-and-app-eval-contract.md) — third-party vendor evaluation (Lakera Guard, Arthur AI, Robust Intelligence, Calypso AI, HiddenLayer, Protect AI); build-vs-buy matrix rows (coverage, latency, cost, deployment shape, portability, evidence trail, price); the app-side measurement contract handed to `ai-eval-engineer` (peer, L30).

## Exercises

- [`exercises/exercise-01-guardrail-architecture-for-one-llm-app.md`](exercises/exercise-01-guardrail-architecture-for-one-llm-app.md) — design the slot set for your running LLM application; produce the OWASP / ATLAS / SAIF coverage table.
- [`exercises/exercise-02-nemo-guardrails-and-guardrails-ai-hands-on.md`](exercises/exercise-02-nemo-guardrails-and-guardrails-ai-hands-on.md) — ship a minimal NeMo Guardrails config + a Guardrails AI validator against the same fixture set.
- [`exercises/exercise-03-classifier-guardrails-llama-guard-and-shieldgemma.md`](exercises/exercise-03-classifier-guardrails-llama-guard-and-shieldgemma.md) — wire Llama Guard and ShieldGemma into the input / output slots and compose with the rule layer.
- [`exercises/exercise-04-presidio-pii-guardrail-integration.md`](exercises/exercise-04-presidio-pii-guardrail-integration.md) — integrate Presidio into ingest, retrieval, and output; tune a recogniser; measure FP / FN.
- [`exercises/exercise-05-guardrail-effectiveness-fp-fn-report.md`](exercises/exercise-05-guardrail-effectiveness-fp-fn-report.md) — produce the FP/FN + cost + latency report a product manager can read; back-feed residual-risk updates into the mod-103 register.
- [`exercises/exercise-06-vendor-guardrail-build-vs-buy-matrix.md`](exercises/exercise-06-vendor-guardrail-build-vs-buy-matrix.md) — evaluate the third-party vendors against your slot set and produce the defensible build-vs-buy matrix.

## Labs & quizzes

- [`labs/`](labs/) — a longer hands-on lab is scheduled here; see the lab README for the current status.
- [`quizzes/`](quizzes/) — knowledge-check quiz for the module.

## Resources

See [`resources.md`](resources.md) for primary sources — every reference is a link to the standards body, the tool's official documentation, the classifier's model card / paper, or the vendor's own site.

## How to work through the module

1. Reuse the running system you carried through mods 101–107. The exercises are cumulative and every one uses this system.
2. Read chapters 01–06 in order. Each pins one dimension of a shippable guardrail programme.
3. Do exercises 01–06 in order. They stack:
   - 01 designs the slot set and the coverage table,
   - 02 ships the rule / flow layer,
   - 03 ships the classifier layer and composes with 02,
   - 04 ships the PII layer across three paths,
   - 05 measures effectiveness and updates the register,
   - 06 closes the build-vs-buy question and hands the measurement contract sideways to `ai-eval-engineer`.
4. Take the quiz.

By the end of the module you should be able to show an auditor a guardrail architecture where every slot links to (a) an OWASP LLM Top 10 entry, a MITRE ATLAS tactic, and a Google SAIF pillar, (b) at least one running control (rule, classifier, or PII recogniser) with pinned versions, (c) a measured TP / FP / TN / FN + cost + latency row on the bounded adversarial set, and (d) a residual-risk update against the mod-103 register. Slots that cannot answer all four are open work.
