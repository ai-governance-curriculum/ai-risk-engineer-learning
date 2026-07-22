# Guardrail Architecture and Composition — Slots, Defence in Depth, and the Coverage Table

A guardrail is not a library import. It is a *slot* in the request / response path of an LLM application, with a named responsibility, a named failure mode, and a measured effectiveness number. This chapter defines the slot set every LLM application ships, the composition rules that keep the slots from becoming a silent single-point-of-failure, and the coverage table that maps each slot to OWASP LLM Top 10, MITRE ATLAS, and Google SAIF so the story is defensible in review.

## Motivation

Two failure modes an auditor spots inside a minute:

- **The "one guardrail" architecture.** The application ships a single output classifier and calls it "the guardrail." When the classifier misses, nothing else catches. When the classifier is bypassed, the operator does not know until an incident lands.
- **The "many guardrails, no coverage" architecture.** The application ships six libraries — a moderation call here, a Presidio pass there, a schema validator on the JSON output. Nobody can say which slot each library sits in, which attack class it addresses, or what the residual number is if one fails. Coverage is asserted, not evidenced.

Both fail because the underlying artifact — the slot set with named responsibilities and named failure modes — was never built. This chapter builds it.

## The slot set

An LLM application in production has five guardrail slots. Some applications ship additional slots (memory-scoped guardrails on session state, guardrails on sub-agent spawning, guardrails on the human-in-the-loop review surface); those are extensions of the five. If a slot is *absent*, the harm classes that slot addresses are unmitigated and must be scored control-defeated in the register.

| Slot | Where it sits | Primary responsibility | Named failure mode |
|---|---|---|---|
| **Input filter** | Between user / caller and the model | Reject or transform obviously harmful, injection-shaped, or policy-violating inputs before they reach the model | Bypassed by novel or obfuscated attack patterns; drifts as the attack surface changes |
| **Retrieval filter** | Between the retriever and the context assembler | Reject or transform retrieved chunks that carry injection payloads, disallowed content, or PII the caller must not see | Trusts unauthenticated upstream text; misses injection payloads that survive chunking |
| **Tool-invocation policy** | Between the model's tool-call intent and the tool executor | Enforce allow-list, argument schema, quotas, side-effect scope, and destination validation on every tool call | Allow-list drift; argument-schema bypass; destination validation weakened for a "known enterprise customer" |
| **Output validator** | Between the model output and the caller / downstream | Enforce schema, policy, and safety on the generated output; catch what the input and retrieval filters missed | Schema-only validation with no policy layer; validation off in streaming path |
| **Safe-completion / refusal shaping** | Fused into the output validator or into the model system prompt | Ensure that when the model refuses or produces a partial answer, the refusal is calibrated, non-leaky, and does not itself become an attack surface | Refusal string that leaks internal policy or provides a step-by-step "what would have been said" restatement of the disallowed content |

Every request that touches the model passes through slots 1 → 5 in order (input → retrieval, if any → tool policy, if any → output validation → refusal shaping). Every slot's presence collapses inherent risk (mod-103 ch 01) to residual risk on the scenarios it addresses; every slot's absence produces a control-defeated variant.

### The order matters

Slot ordering is not decorative. Two ordering discipline points:

- **Filter before you spend model tokens.** Input filters cheap and fast enough to run pre-model save token cost and reduce blast radius. If the input is obviously injection-shaped, you never call the model.
- **Validate after you get model tokens.** Output validators run *after* the model produces text. Structural validation (JSON schema, allowed enums, length bounds) is cheap; safety validation (classifier, policy match) is more expensive and is what makes the output layer defence-in-depth.

The safe-completion / refusal shaping step is the last thing before the caller receives bytes. If it is inlined into slot 4 as "if the classifier flags, replace with `Sorry, I can't help with that.`", it is still a distinct concern with a distinct failure mode — treat it as slot 5 in the architecture diagram even when the code sits in the same function as slot 4.

## Defence in depth — the composition rule

Defence in depth is the rule that no single slot may be the only thing standing between the attacker and the harm. Concretely:

- **Every harm class named in the mod-102 harm model has *at least two* slots that address it.** Prompt injection is caught at input, at retrieval (for indirect injection through retrieved content), and at output. PII leakage is caught at ingest, at retrieval, and at output. Disallowed content is caught at input and at output.
- **The two slots use different mechanisms.** If both the input filter and the output validator are the same OpenAI Moderation call with the same threshold, the classifier failing once bypasses both. A composition of *rule + classifier + retrieval-based check* survives a single classifier miss.
- **Each slot has an independent evidence trail.** The measurement in chapter 05 produces a TP / FP / TN / FN row per slot per adversarial category. If two slots share a measurement (because they share a mechanism), the composition is not defence in depth even if two config blocks exist.

The composition test the auditor applies is: *pick a harm class from the harm model, name the slots that address it, and describe how the residual number changes as each of those slots is defeated one at a time.* If the number collapses to inherent the moment the first slot is defeated, the harm class has one real control, not several.

## The silent single-point-of-failure anti-pattern

Four common failure modes that produce a silent SPOF even when the architecture diagram shows several slots:

1. **Shared model, shared bypass.** Input classifier and output classifier are the same fine-tune of the same base model. A jailbreak that fools the input classifier fools the output classifier for the same reason.
2. **Fail-open on error.** The guardrail classifier returns a 500; the middleware treats the missing verdict as "allow." The application is now unguarded for the duration of the outage, and no alert fires because "no verdict" and "clean verdict" look identical downstream.
3. **Streaming bypass.** The output validator runs on the completed response, but the streaming path forwards tokens as they arrive. If the caller consumes the streaming path, the validator never runs on the delivered bytes.
4. **Debug bypass.** A "debug=true" query parameter, session flag, or internal caller identity disables one or more slots for troubleshooting. The flag persists in production and is set by an integration test that runs against production data.

All four are *silent*. The residual score in the register still reflects a working control set, and the monitor is not sized to catch the defeat inside the tolerable window. Every one of them is caught by the discipline in mod-103 ch 01 (control-defeated column per named control) and the monitor selection in mod-109. This chapter's contribution is enumerating the modes so the harm-model → guardrail translation names them explicitly.

## Slot × attack-class coverage table

The following table is the shape of the coverage artifact `exercise-01` produces. Rows are the five slots (plus optional session / agent slots for applications that have them); columns are OWASP LLM Top 10 category, MITRE ATLAS tactic, and Google SAIF pillar. Cells name the entries the slot addresses.

| Slot | OWASP LLM Top 10 (v2025) | MITRE ATLAS tactic | Google SAIF pillar |
|---|---|---|---|
| Input filter | LLM01 Prompt Injection; LLM05 Improper Output Handling (partial, upstream); LLM09 Misinformation (input side) | Initial Access; ML Model Access; ML Attack Staging | Extend strong security foundations to the AI ecosystem; Extend detection and response |
| Retrieval filter | LLM01 Prompt Injection (indirect); LLM06 Sensitive Information Disclosure; LLM08 Vector and Embedding Weaknesses | ML Supply Chain Compromise; Initial Access via a compromised data source | Automate defences to keep pace with new and existing threats; Adapt controls to adjust mitigations and create faster feedback loops |
| Tool-invocation policy | LLM06 Sensitive Information Disclosure; LLM07 System Prompt Leakage (agent path); LLM10 Unbounded Consumption (partial) | Execution; Exfiltration; Impact | Contextualise AI system risks in surrounding business processes; Extend strong security foundations |
| Output validator | LLM01 (post-hoc); LLM05 Improper Output Handling; LLM06 Sensitive Information Disclosure; LLM09 Misinformation | Exfiltration; Impact (Evade ML Model detection layer inverted for defence) | Automate defences to keep pace; Harmonise platform-level controls |
| Safe-completion / refusal shaping | LLM07 System Prompt Leakage; LLM09 Misinformation (calibrated refusal instead of confident wrong answer) | Impact (reduce blast radius on refusal); Defence Evasion (deny leaking policy to the attacker) | Harmonise platform-level controls; Contextualise AI system risks |

<!-- needs-research: pin the exact OWASP LLM Top 10 version identifier current at authoring time; the 2025 revision reorders and re-numbers some categories from the 2023 v1. Same for the current MITRE ATLAS matrix version. -->

Two coverage disciplines:

- **The cells are addressed, not claimed.** A cell means the slot has an implemented mechanism that measurably catches instances of the attack class on the bounded adversarial set (mod-104). "Address" without a measurement row is a claim.
- **Empty cells are enumerated, not hidden.** If a harm class from the harm model has no slot addressing it, that is a hole in the coverage table — one that gets escalated to the risk council in mod-112 with a treatment plan.

## Worked example — customer-support RAG assistant

Reusing the running scenario `HR-CS-INJ-01` from mod-102 (indirect prompt injection through a retrieved KB article that induces `escalate_ticket` to an attacker-controlled destination). The slot set instantiated for this application:

| Slot | Concrete control | Mechanism |
|---|---|---|
| Input filter | Reject user turns containing known injection signatures + Llama Guard input classification | Rule + classifier |
| Retrieval filter | Provenance check on retrieved chunks; injection-signature scan; ShieldGemma pass on chunk text | Rule + classifier |
| Tool-invocation policy | Allow-list on `escalate_ticket` destinations; argument schema; destination-registry lookup | Rule |
| Output validator | JSON schema + policy classifier on the response body; Presidio pass to strip PII fields the caller must not see | Rule + classifier + PII |
| Safe-completion / refusal shaping | Refusal-string template that names the policy category without quoting the disallowed content | Rule |

For `HR-CS-INJ-01`, three slots contribute to the residual number: the retrieval filter (drops chunks whose provenance fails), the tool-invocation policy (allow-list on the destination), and the output validator (Presidio strip on the response body). Slot-defeat scores per mod-103 ch 01:

- Retrieval filter defeated (bypass through a novel injection pattern the signatures do not match): tool allow-list still restricts destinations, so residual moves from Medium to High but not to inherent.
- Tool allow-list disabled (change-management gap during a debug session): residual moves to Very High because the injection can now direct `escalate_ticket` to an arbitrary destination.
- Output validator's Presidio pass turned off: the response body may leak PII fields; residual on the leakage side of the scenario moves to Very High even if the tool allow-list is still in force.

Those three defeat scores are what the monitors in mod-109 must catch inside the tolerable window.

## Composition patterns

Three composition patterns that recur:

- **Rule → classifier → policy.** Cheap rule matches obvious cases (regex, denylist, size / rate); classifier catches the shaped cases the rule misses; policy layer enforces business rules the classifier does not know about. Ordering left-to-right by cost.
- **Classifier + retrieval + rule for indirect injection.** Classifier flags known injection shapes; retrieval-side provenance rule rejects untrusted chunks; rule at the tool-call layer restricts side effects even if a chunk gets through. Three independent mechanisms per harm class.
- **Deterministic before probabilistic.** A structural check (schema, allow-list, size bound) is deterministic and fast. Run it first; the probabilistic checks (classifiers) run on what survives.

The rule-versus-classifier trade-off is developed in chapter 02 (rule / flow layer) and chapter 03 (classifier layer). The PII layer is developed in chapter 04. The composition of all three is measured in chapter 05.

## Cross-references

- **Mod-102** (harm model) supplies the scenarios each slot must address. If a harm-model row has no slot addressing it, that is either open work here or an out-of-scope escalation.
- **Mod-103 ch 01** (inherent / residual / control-defeated) supplies the vocabulary in which slot coverage is scored. Each slot's presence collapses inherent to residual; each slot's absence is one control-defeated row.
- **Mod-104** (LLM red team) supplies the bounded adversarial set the effectiveness measurement runs against — do not build a private eval set at this level; consume the hosted one.
- **Mod-105** (adversarial ML) supplies the classifier-attack literature that shapes how the classifier-based slots degrade; several slots here reuse classifier robustness lessons from that module.
- **Mod-107** (privacy risk) supplies the PII taxonomy that the retrieval / output PII checks in chapter 04 enforce.
- **Mod-109** (monitoring) consumes the slot definition and the control-defeated score to size the monitor for each named control.
- **Mod-111** (incident response) consumes the slot map so that when an incident lands, the runbook branch is already keyed to "which slot defeated."

## Summary

- A guardrail is a *slot* — input filter, retrieval filter, tool-invocation policy, output validator, safe-completion / refusal shaping — with named responsibility, failure mode, and measurement.
- Defence in depth: every harm class is addressed by at least two slots using different mechanisms with independent evidence trails.
- The silent SPOF anti-patterns are shared-model bypass, fail-open on error, streaming bypass, and debug bypass.
- The slot × attack-class coverage table maps every slot to OWASP LLM Top 10, MITRE ATLAS, and Google SAIF cells; addressed means measured, not claimed.
- The slot set is the vehicle by which mod-102 harm rows become mod-103 residual numbers, mod-104 fixtures become effectiveness measurements, and mod-109 monitors become live signal.
- Exercise 01 turns this chapter into the slot set + coverage table for your running system.
