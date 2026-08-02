# Exercise 02 — NeMo Guardrails and Guardrails AI Hands-On

**Estimated effort:** 3 hours

## Objective

Ship the **rule and flow** layer for the running system's guardrail architecture (exercise 01) using **NVIDIA NeMo Guardrails**, **Guardrails AI**, and a **small custom middleware** for the tool-invocation policy slot. Each of the three tools lands in the slot chapter 02 §"Composing the three tools" argues fits best. Prove the composition on a small **fixture set** — known-bad inputs each rail must catch and known-good inputs each rail must pass — and produce an auditable trace out of every rail invocation. This exercise ships the deterministic layer; exercise 03 stacks classifiers on top; exercise 05 measures the composed stack.

## Prerequisites

- Exercise 01 completed. The slot specification and coverage table are the reference.
- Chapter 02 read; chapter 03 skimmed at the summary-table level so you know which classifier goes where (their bodies land in exercise 03).
- Python environment with:
  - **NVIDIA NeMo Guardrails** — pinned version installed; a working LLM backend it can call (any provider your organisation permits — local Llama, OpenAI, Anthropic, Bedrock). The exercise is about the rails, not the model.
  - **Guardrails AI** — pinned version; a Validator Hub token if you consume Hub validators; at least one custom `Validator` authored in-repo so you exercise the SDK, not just the Hub.
  - **FastAPI / LangChain / LangGraph** — whichever wraps your running system today. The exercise integrates against that; do not swap frameworks for the exercise.
- A **fixture set** — at least 10 known-bad and 10 known-good inputs per rail. Pull known-bads from mod-104's hosted adversarial set where the category matches; if a rail addresses a category mod-104 does not cover, author your own from the harm-model rows. Known-goods must look like real traffic your running system serves (paraphrase real user turns; do not synthesise obviously benign strings that give the rail nothing to distinguish).

## Problem statement

Instantiate the rule / flow layer for the running system. Do not add classifier calls yet — those land in exercise 03. Ship the deterministic checks, the schema / validator step on structured outputs, and the tool-invocation policy in the custom middleware. Every rail emits an auditable trace naming which rail fired, which action ran, and the verdict.

### The three tools and where they land

Follow chapter 02 §"A worked composition":

- **NeMo Guardrails** — input rail (signature denylist plus a stub action that will be swapped for Llama Guard in exercise 03), retrieval rail (provenance check plus a stub action for ShieldGemma in exercise 03), refusal-shaping output rail (templated refusals per named category). Use Colang for the flows; author actions in Python.
- **Guardrails AI** — output guard on the response body. If your application emits structured JSON (tool arguments, form fills, `answer` / `citations` split), use a schema-driven guard with per-field validators. If the application emits free text, use a validator guard with at least one custom validator you authored (do not fill the guard with only Hub validators — you must exercise the SDK).
- **Custom middleware for the tool-invocation policy** — a small module (in FastAPI: a dependency; in LangChain / LangGraph: a tool decorator or node wrapper) that enforces allow-list on destinations, argument schema on tool arguments, and per-tool quota / rate limit. Chapter 02 §"The custom lightweight middleware layer" is the reference shape. Do not route this through NeMo GR — the exercise's point is that a deterministic policy lives in code you own.

### The fixture harness

Each rail has a fixture set of known-bad and known-good inputs. Write a small test harness (pytest or a plain runner) that walks the fixtures through the composed stack and asserts:

- Every known-bad is caught (either rejected by a rail or transformed / redacted, per the rail's design). Not-caught counts as a failure.
- Every known-good passes end-to-end. Blocked known-goods count as a failure.
- Every rail invocation logs a trace record (JSON) with fields: `request_id`, `slot`, `rail_name`, `action_name`, `verdict`, `latency_ms`, `config_version`.

The harness reports the raw pass / fail counts per rail (not FP / FN rates yet — the labelled adversarial-and-legitimate set for that measurement is exercise 05).

### One deliberate anti-pattern probe

For each of chapter 02 §"Anti-patterns", write one small negative test that would trigger the anti-pattern in your design, and confirm the design does not. The four anti-patterns:

- **Regex explosion.** Confirm every regex in your rail set has at least one fixture that fails without it and passes with it.
- **Silent action failure.** Force one of your NeMo GR actions to raise; confirm the flow fails closed (the input is rejected or the response is refused), not open.
- **Hub validator sprawl.** If you consume any Hub validators, list each with pinned version and one fixture that exercises it. If you use zero Hub validators, state that.
- **Middleware debug bypass.** Confirm your custom middleware does not expose an unlogged bypass flag. If it does expose a bypass at all (e.g., for internal test callers), the bypass is logged, rate-limited, and gated by network policy.

## Requirements

Produce all seven items:

1. **NeMo GR config** — `risk/mod-108/<system-id>/rails/nemo/` containing `config.yml`, `rails/input.co`, `rails/retrieval.co`, `rails/output.co` (refusal-shaping), and `actions.py`. Actions may be stubs at this stage; the stubs will be replaced by real classifier calls in exercise 03. Pin the NeMo GR version and the Colang version in `config.yml`.
2. **Guardrails AI guard** — `risk/mod-108/<system-id>/rails/guardrails_ai/guard.py` plus any per-field validator files. Includes at least one custom in-repo validator (not only Hub validators). Pin Guardrails AI version.
3. **Custom middleware** — `risk/mod-108/<system-id>/rails/middleware/tool_policy.py`. Enforces allow-list, argument schema, quota. Wired into the actual FastAPI / LangChain / LangGraph surface the running system uses. Unit tests for allow-list and schema violations.
4. **Fixture set** — `risk/mod-108/<system-id>/rails/fixtures/` with subdirectories per rail. Each fixture is a small JSON or YAML file naming the rail, expected verdict, source (mod-104 payload ID, or harm-model row ID, or paraphrased real traffic).
5. **Fixture harness** — `risk/mod-108/<system-id>/rails/harness/run_fixtures.py`. Walks fixtures through the composed stack; asserts pass / fail per fixture; emits a per-rail summary and the JSON trace file.
6. **Rail trace log format** — `risk/mod-108/<system-id>/rails/traces/README.md`. Documents the JSON schema of the trace records (fields, types, semantics). Mod-109 will consume this format; the schema decision made here is what the monitor sees.
7. **Anti-pattern probe report** — `risk/mod-108/<system-id>/rails/anti-patterns.md`. Four sections, one per anti-pattern from chapter 02. Each section states the negative test and the observed behaviour. Any anti-pattern the design does not resist is filed as a fix, not silently glossed.

Cross-cutting rules:

- **Actions fail closed, not open** (chapter 02 §"Anti-patterns"). A NeMo GR action that errors must produce a "reject" verdict; a `None` return is not "no verdict."
- **Fixture-driven, per-rail and end-to-end** (chapter 02 §"Testing the rule / flow layer"). Unit fixtures per rail *and* at least one end-to-end fixture that exercises the whole slot chain — an injection payload that must be caught by the input rail *or* by the retrieval rail *or* by the output guard, and the end-to-end fixture confirms at least one of the three catches it.
- **Every rail has a pinned version and a fixture set** — NeMo GR framework version, Colang version, Guardrails AI version, every Hub validator pinned. Middleware pins its own dependency versions.
- **Trace fields include `config_version`.** When the rail config changes, the trace records reflect it so the mod-109 monitor can detect the change.
- **No classifier calls in the rail bodies yet.** Stubs are fine (`def llama_guard_action(...): return {"verdict": "safe"}` with a `# TODO exercise-03`). This exercise ships the deterministic layer; classifiers land next.

## Starter guidance

- **Follow the worked composition literally at first, then adjust.** Chapter 02 §"A worked composition — customer-support RAG assistant" instantiates all three tools for the reference system; if your running system is a different shape, adjust the mechanism-per-slot cell but keep the pattern.
- **Colang is a DSL, not Python.** If a Colang flow gets complex, factor logic out to a Python action; do not try to write branching business logic in Colang. The DSL is for dialog structure and rail invocation.
- **Guardrails AI's API surface has moved.** The Pydantic-model-driven guard is current at authoring time but the RAIL XML shape is still supported in some versions. Cite the docs version you install against; do not follow a mixed-version tutorial. See resources.md's `needs-research` note on this.
- **Middleware for the tool policy is the smallest component but the highest-blast-radius one.** A missing allow-list check on `escalate_ticket`, `send_email`, or `pay_invoice` lets an indirect-injection payload cause a real-world side effect. Test the allow-list negative cases first: payload attempts `send_email` to an external domain the allow-list rejects → the middleware raises and the request path returns a refusal.
- **Structured-output guards give you the best per-field policy story.** If your model returns JSON with an `answer` field and a `citations` field, a per-field validator lets you have different policies on each — Presidio on `answer` (redact PII the caller is not entitled to see), URL-shape check on `citations`, schema on tool arguments. This will pay off in exercise 04 (per-field Presidio) and exercise 05 (per-field measurement).
- **The trace format decision is load-bearing for mod-109.** If you skip `request_id`, mod-109 cannot join across slots. If you skip `config_version`, mod-109 cannot detect that a config change moved the FP / FN numbers. Author the trace format now with mod-109's needs in mind.
- **Do not over-scope the fixture set.** 10 bad + 10 good per rail is enough to prove the plumbing. The bounded labelled adversarial-and-legitimate set for the FP / FN measurement is a *different* set that exercise 05 consumes from mod-104; keep the two separate so the fixture set stays small and quick.

## Acceptance criteria

The submission is accepted when:

- The NeMo GR config, the Guardrails AI guard, and the custom middleware all exist and are wired into the running system's actual serving path.
- The Guardrails AI guard includes at least one custom in-repo validator.
- The custom middleware enforces allow-list, argument schema, and quota; each has a unit test.
- The fixture set has at least 10 known-bad and 10 known-good per rail, with sources cited.
- The fixture harness runs to completion and asserts every fixture's expected verdict; any regression is a build failure.
- Every rail invocation emits a trace record whose format is documented under `traces/README.md`.
- The anti-pattern probe report covers all four anti-patterns; any pattern the design does not resist is filed with a follow-up.
- All tool versions (NeMo GR, Colang, Guardrails AI, Presidio-if-touched, middleware dependencies) are pinned in the artefacts.

## Stretch goals

- **A second rail composition.** In addition to the primary composition (NeMo GR at the dialog boundary, Guardrails AI on structured output, custom middleware for tools), author a *minimal alternative*: custom-middleware-only, no NeMo GR, no Guardrails AI. Run the same fixture set through both compositions and produce a small comparison of trace richness, code size, and where each shape wins. Chapter 02 §"Composing the three tools" is the reference.
- **The debug-bypass negative test.** If your custom middleware exposes any bypass path (a test-tenant flag, a `X-Debug: true` header, a specific IP allow-list), add a fixture that fires the bypass and confirms it is logged, rate-limited, and network-gated. If the bypass is silently unrestricted, that is the bug the stretch surfaces.
- **A composition test that mirrors the mod-102 harm-scenario chain.** Take one indirect-prompt-injection harm scenario (chapter 01 §"Worked example" has one; your mod-102 will have your equivalent). Author an end-to-end fixture that walks the payload from user turn through retrieval into tool invocation; confirm which of the three rails caught it. If none caught it, the classifier layer (exercise 03) has to; note that dependency explicitly.
- **A regression fixture from a real internal incident.** If your organisation has had any internal LLM incident where a rule-layer control could have caught it, add the regression fixture. Chapter 02 §"Testing the rule / flow layer" is explicit that mod-111 post-mortems feed regression fixtures.
- **A NeMo GR trace consumer stub.** Write a small script that reads the NeMo GR trace file and produces a per-rail pass / fail summary for the last N runs. This is a mod-109 monitor stub; the metric plumbing lands in mod-109, but the intake shape is decided here.

## Deliverable format

A single markdown document `mod-108-ex02-rails.md` that indexes the artefacts. The index document contains:

1. *System summary* — one paragraph naming the running system, the framework it uses (FastAPI / LangChain / LangGraph), and the mechanism per slot from exercise 01.
2. *Tool versions pinned* — NeMo GR, Colang, Guardrails AI, every Hub validator, middleware dependencies.
3. *NeMo GR config* — inline the `config.yml` or a pointer, plus one representative Colang flow.
4. *Guardrails AI guard* — the guard construction plus the custom validator.
5. *Custom middleware* — the tool-policy shape plus the allow-list / schema / quota tests.
6. *Fixture harness output* — the per-rail pass / fail summary and the trace file location.
7. *Trace format spec* — the JSON schema documented in `traces/README.md`.
8. *Anti-pattern probe report* — the four sections.
9. *Open items* — any anti-pattern the design does not fully resist, any classifier stub that is not yet wired (exercise 03 pickup), any measurement that is deferred to exercise 05.

## Submission

Attach the index markdown plus the seven artefacts to the module submission channel. Keep the artefacts under version control; the fixture harness and trace format are the load-bearing contracts every subsequent exercise depends on.
