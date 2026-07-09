# mod-108-guardrail-engineering-and-effectiveness: Guardrail Engineering and Effectiveness Measurement

> Scaffolded by `aicg org execute-plan`. Lecture chapters and exercise content are authored on subsequent autonomous cycles.

**Estimated effort:** 16 hours

## Learning objectives

- Design an input / output guardrail architecture for an LLM application — input filters, output validators, tool-invocation policy, safe-completion / refusal shaping — that composes cleanly with the underlying model without silent single-points-of-failure
- Ship guardrail code with NVIDIA NeMo Guardrails (rule + flow), Guardrails AI (schema / validator + Hub), and a custom lightweight layer that can be dropped into a FastAPI / LangChain / LangGraph app
- Ship classifier-based guardrails with Meta Llama Guard, Google ShieldGemma, OpenAI Moderation, and Anthropic Constitutional Classifiers — and know how to compose classifier + rule + retrieval-based defence in depth
- Ship PII guardrails with Presidio and integrate them into ingest / retrieval / output paths
- Measure guardrail effectiveness — true positive / false positive / true negative / false negative on a bounded adversarial set — with explicit cost and latency accounting, and produce an FP/FN report a product manager can read
- Evaluate third-party guardrail vendors (Lakera Guard, Arthur AI, Robust Intelligence, Calypso AI, HiddenLayer, Protect AI) with a defensible build-vs-buy matrix; feed the app-side measurement contract to `ai-eval-engineer` (peer, level 30)
- Map every guardrail slot to an OWASP LLM Top 10 entry, an MITRE ATLAS tactic, and a Google SAIF pillar — so the coverage story is defensible

## Structure

- `01-…md` … `0N-…md`: lecture chapters.
- `exercises/`: per-exercise prompts.
- `labs/`: long-form hands-on labs.
- `quizzes/`: knowledge checks.
- `resources.md`: external references.
