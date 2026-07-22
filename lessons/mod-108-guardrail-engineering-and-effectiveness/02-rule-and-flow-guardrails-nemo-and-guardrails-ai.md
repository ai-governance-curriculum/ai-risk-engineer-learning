# Rule and Flow Guardrails — NVIDIA NeMo Guardrails, Guardrails AI, and the Custom Middleware Layer

Chapter 01 fixed the slot set. This chapter ships the *rule and flow* layer that populates several of those slots — the deterministic, cheap, auditable checks that run first. Three tools cover almost every enterprise deployment: **NVIDIA NeMo Guardrails** for dialog-shaped rails and multi-step flows, **Guardrails AI** for schema-plus-validator output enforcement, and a custom lightweight middleware for the cases where you want a small blast radius and no framework lock-in. This chapter teaches when each fits and how they compose with the classifier layer that lands in chapter 03.

## Motivation

Two ways this layer is commonly shipped wrong:

- **All classifier, no rule.** Every slot is a call to a moderation classifier. Runtime cost is a function of prompt length and model choice; latency dominates the tail; when the classifier drifts, nothing else catches. A cheap deterministic filter at the front would have handled 60–80% of the traffic for 0.5 ms and let the classifier see only what needed judgement.
- **All rule, no flow.** Every slot is a regex. Novel jailbreaks bypass in the first week. The rule set grows unmaintainably. There is no place to encode "if the user asks about topic X, route to canned answer Y" — that is a dialog concern, not a regex concern.

The right layer is a *rule-and-flow* layer: deterministic checks where deterministic is defensible, structured flow orchestration where the check needs multi-step state, and a schema / validator step on the output. Classifiers (chapter 03) sit on top.

## When rule-based fits and when classifier-based fits

Before either tool, the rule / classifier decision. A summary table for the working set of common checks:

| Check | Rule-based fits when… | Classifier-based fits when… |
|---|---|---|
| PII redaction | The PII patterns are enumerable (Presidio recognisers, chapter 04) | The PII is contextual — "the customer's employer" — and requires a semantic classifier |
| Prompt-injection detection | Signatures for known attack strings are cheap to add (denylist) | The attack is paraphrased, obfuscated, or embedded in a benign envelope (LLM01 variants) |
| Topic scoping | The allowed topics are enumerable ("account, billing, product") | The disallowed topics are broader than the allowed and can be paraphrased |
| Output schema | The output is a structured object (JSON, tool call, form) | The output is free-text and needs semantic validation |
| Toxicity / harmful content | Slur / phrase denylist catches the low-hanging fruit | Nuanced content classification (Llama Guard categories, ShieldGemma dimensions) |
| Tool-call policy | Allow-list on destinations, argument schema | Never — tool-call policy is deterministic by definition |
| Refusal shaping | Templated refusal string for a named category | Never — refusals are deterministic content |

Rule-based wins on cost, latency, auditability, and predictability. Classifier-based wins on coverage. Ship both.

## NVIDIA NeMo Guardrails — Colang rails, flows, and the dialog manager

NeMo Guardrails ("NeMo GR") is an open-source framework from NVIDIA for adding programmable guardrails to LLM applications. It ships as a Python library with a config-file interface and a small DSL called **Colang** for writing dialog flows.

Key concepts:

- **Rails.** Named guardrails scoped to a stage of the request. NeMo GR ships input rails (run on user input), output rails (run on model output), dialog rails (steer the conversation), retrieval rails (run on retrieved chunks), and execution rails (run around tool calls). Each rail corresponds directly to one of the chapter-01 slots.
- **Colang.** A DSL for describing user intents, bot responses, and flows. Colang lets you say "if the user asks about competitor products, respond with the canned decline" without writing that as a regex or a classifier.
- **Actions.** Python functions the flow can call. Actions are how a Colang flow reaches out to a classifier, a PII library, or a tool-call check.
- **Dialog manager.** The runtime that interprets the Colang flows, decides which rails fire, and stitches the model call together with the rail results.

<!-- needs-research: pin the current stable Colang version (Colang 2.x is the direction of travel) and note any breaking changes since the last released version at authoring time. -->

A minimal NeMo GR config looks like:

- `config.yml` — models, rail definitions, and knobs.
- `rails/input.co` and `rails/output.co` — Colang flows for input and output rails.
- `actions.py` — Python actions the flows call.

The auditable trace out of NeMo GR names each rail that fired, each action invoked, and the verdicts produced. That trace is what mod-109 consumes as monitor signal, and what mod-111 consumes at incident time.

**Where NeMo GR fits.** Applications with a dialog shape (assistants, chatbots, structured intake flows) where you need multi-step flow logic and a shared rail configuration across environments. It is also strong when you have several deployments and want a shared rail package.

**Where it does not fit.** Single-shot completion endpoints where a full dialog manager is overkill; libraries embedded inside a larger application where a small blast radius is more important than a full framework; extremely latency-sensitive paths where the framework overhead is meaningful.

## Guardrails AI — schema, validators, and the Validator Hub

Guardrails AI ("Guardrails AI") is an open-source library that treats guardrails as *validators* wrapped around the LLM call. It ships with a schema / validator DSL and a **Validator Hub** — a public registry of community and vendor validators for common checks (PII, competitor mentions, toxicity, prompt injection, JSON schema, etc.).

Key concepts:

- **Guard.** A wrapper around one or more validators applied to input or output.
- **Validators.** Individual checks. Each validator has an `on_fail` behaviour — reask the model, refrain (return an error), fix (transform the value), filter (drop the offending portion), or noop (log-only).
- **Validator Hub.** The registry from which you install additional validators. Hub validators include community and vendor-authored checks; treat each one as third-party code and pin versions.
- **RAIL / structured schema.** Guardrails AI supports schema-driven output validation where the schema itself carries validators per field.

<!-- needs-research: confirm the current API surface (the library has moved between `rail` XML-flavoured configs and Pydantic-model-driven guards); cite the current published documentation as authoritative. -->

**Where Guardrails AI fits.** Output-slot validation where the output is structured (JSON, form, tool arguments) and the validators are field-scoped. It is also strong as a way to reuse community validators quickly for common cases.

**Where it does not fit.** Complex multi-turn dialog logic (NeMo GR is better); tool-call policy that must live outside the LLM wrapper (build that in your own middleware).

## The custom lightweight middleware layer

Neither NeMo GR nor Guardrails AI is a good fit for every case. Two cases where a custom, small, in-repo middleware layer wins:

- **Tool-invocation policy.** The allow-list on destinations, argument schema on tool arguments, and quota / rate-limit enforcement per tool are all *inside* your application code — they know about your service registry, your allow-lists, and your tenants. Wrapping the tool executor with a middleware function you own is cheaper and clearer than routing through a rails framework for a check that is deterministic.
- **Latency-critical or embedded slots.** A slot in a synchronous inference path where a framework's overhead is a large fraction of the budget, or a slot inside a library that will be embedded in many callers, is better served by a small, focused middleware.

A custom middleware shape that drops into FastAPI / LangChain / LangGraph looks like:

- A pre-model middleware that runs the input filter (regex / signature check + PII scan + rate limit) and short-circuits on rejection.
- A retrieval hook (LangChain / LangGraph retriever wrapper) that runs the retrieval filter on returned chunks and rejects or transforms.
- A tool-executor wrapper (LangChain tool decorator / LangGraph tool node) that enforces allow-list, argument schema, and destination policy.
- A post-model middleware that runs the output validator and the refusal-shaping template.

Each of those is a function you own, tested against fixtures, versioned in the repo. It composes cleanly with NeMo GR (which can call your middleware from Colang actions) and with Guardrails AI (which can wrap your model call).

## Composing the three tools

Three composition patterns that recur:

- **NeMo GR at the dialog boundary, Guardrails AI on structured outputs, custom middleware around tools.** NeMo GR handles the input / retrieval rails and the dialog-scoped policy; Guardrails AI validates any structured output (JSON schema, tool arguments, form fills); the custom middleware enforces tool-invocation policy. Each tool sits at the boundary it is best at.
- **NeMo GR only, with actions calling classifiers.** For applications small enough to live inside one NeMo GR config, this is the simplest. Colang flows call Llama Guard / ShieldGemma / Presidio through actions; the schema layer is a Colang output rail.
- **Custom middleware only, with vendor calls for classifiers.** For latency-critical single-turn endpoints, no framework — just your middleware calling OpenAI Moderation / Llama Guard / Presidio directly. You lose the auditable trace NeMo GR gives you; log the same fields yourself.

The choice is not once-per-track; it is once-per-application. Applications built around agents (LangGraph) often go NeMo-GR-plus-custom-middleware; applications built around structured extraction often go Guardrails-AI-plus-custom-middleware; small single-endpoint applications often go custom-middleware only.

## A worked composition — customer-support RAG assistant

Continuing the running scenario. Slot-by-slot mechanism assignment:

| Slot | Chosen mechanism | Reason |
|---|---|---|
| Input filter | NeMo GR input rail: signature denylist + Llama Guard action | Dialog shape; want shared rail config across staging / prod |
| Retrieval filter | NeMo GR retrieval rail: provenance check + ShieldGemma action | Sits naturally in NeMo GR; shares config with input |
| Tool-invocation policy | Custom middleware wrapping the tool executor | Deterministic, needs allow-list from internal service registry, latency-critical |
| Output validator | Guardrails AI guard on the response JSON (fields: `answer`, `citations`, `escalate_ticket`) with PII validator per field + Presidio wrapper | Structured output, field-level validators |
| Refusal shaping | NeMo GR output rail: refusal-template action | Named categories, deterministic templates |

The trace out of NeMo GR names every rail; the Guardrails AI guard produces a per-field validation report; the custom middleware logs allow-list decisions with the destination attempted. All three log the same request identifier so the monitor in mod-109 can reconstruct the full chain.

## Testing the rule / flow layer

Two testing disciplines this layer specifically requires:

- **Fixture-driven, per-rail.** Every rail has a fixture set — known-bad inputs it must catch and known-good inputs it must pass. Fixtures are versioned in the repo. The mod-104 red-team output feeds new adversarial fixtures in; the mod-111 incident post-mortem feeds regression fixtures in.
- **Composition tests, not just unit tests.** A fixture that passes a rail unit test may still be handled wrong at composition (the input rail catches it, then the retrieval rail re-introduces it through a chunk it should have dropped). Ship end-to-end fixtures that exercise the full slot chain.

## Anti-patterns

Four anti-patterns specifically at this layer:

- **Regex explosion.** A rail's rule set grows to 300 regexes; nobody remembers what each one catches; retiring a regex is scary because the coverage is not measured. Fix: every rule has a fixture that fails without it and passes with it.
- **Silent action failure.** A Colang action calls a classifier that returns an error; the action returns `None`; the flow interprets `None` as "no verdict" and continues. Fix: actions are typed; errors bubble; the rail fails closed.
- **Hub-validator sprawl.** The Validator Hub is easy to install from; six validators of unknown provenance end up in the output guard. Fix: every Hub validator is treated as third-party code with a pinned version, a change log, and a fixture set.
- **Middleware that reaches around itself.** A custom middleware that runs the guardrail check, then also exposes a "debug bypass" flag consumed by an internal caller. The bypass is silent; the middleware author knows about it, no-one else does. Fix: the bypass is a flag on the request that is logged and rate-limited and firewall-gated.

## Cross-references

- **Mod-103 ch 01** — the register credits controls only when they have live evidence. NeMo GR rail traces, Guardrails AI validation reports, and middleware allow-list decisions *are* the evidence.
- **Mod-104** supplies the adversarial payloads that populate the per-rail fixture sets.
- **Chapter 03** supplies the classifier actions the rails call.
- **Chapter 04** supplies the Presidio recognisers the PII validators wrap.
- **Chapter 05** measures the effectiveness of the full rail + classifier + PII composition on the bounded adversarial set.
- **Mod-109** consumes the rail trace as monitor signal; the trace format decision made here is what the monitor sees.
- **Mod-111** consumes the trace at incident time; if the trace does not name the rail that fired, the incident cannot be routed.

## Summary

- Ship a rule and flow layer *before* the classifier layer; deterministic checks are cheaper, faster, and more auditable than model calls.
- NeMo Guardrails fits dialog-shaped applications; Guardrails AI fits structured-output validation; a custom middleware wins for tool-call policy and latency-critical paths. Compose them.
- Every rail has fixtures, a pinned version, and an auditable trace. Actions fail closed on error, not open.
- Anti-patterns: regex explosion, silent action failure, Validator Hub sprawl, debug-bypass middleware.
- Exercise 02 puts you through a NeMo GR + Guardrails AI composition on the running system.
