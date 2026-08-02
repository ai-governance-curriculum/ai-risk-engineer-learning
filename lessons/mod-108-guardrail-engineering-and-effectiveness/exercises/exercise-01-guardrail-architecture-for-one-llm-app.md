# Exercise 01 — Guardrail Architecture for One LLM Application

**Estimated effort:** 2 hours

## Objective

Design the **five-slot guardrail architecture** (chapter 01 §"The slot set") for **one** running LLM application you carry through the module, then produce the **slot × attack-class coverage table** that maps every slot to OWASP LLM Top 10, MITRE ATLAS, and Google SAIF. The output of this exercise is the reference artefact every subsequent exercise updates: exercise 02 ships the rule / flow controls into these slots, exercise 03 adds classifiers, exercise 04 adds Presidio, exercise 05 measures, exercise 06 evaluates vendors against the slot set defined here.

## Prerequisites

- Chapter 01 read; chapter 02–06 skimmed at the level of the summary tables so you know what mechanisms will land in the slots.
- The **running system** you carried through mods 101–107 — a specific LLM application (a customer-support RAG assistant, a developer copilot, an internal HR bot, a claims-processing agent — pick one and stay with it for every exercise in the module).
- The **harm model** rows produced in mod-102 for this system.
- The **risk register** rows produced in mod-103 for this system, including the inherent / residual / control-defeated columns.
- Version identifiers to pin at authoring time: the OWASP LLM Top 10 revision you cite (v1 2023 vs. v2025 renumbering — pick and pin), the MITRE ATLAS matrix version, and the Google SAIF pillar set. Write these version identifiers into the coverage table's header so the artefact is reproducible.

## Problem statement

Instantiate the chapter-01 slot set for the running system and produce the coverage artefact that binds slots to the three external frameworks. The artefact must survive an auditor asking "for scenario `X` in the harm model, which slots address it, what happens to the residual number if slot `Y` is defeated, and what OWASP / ATLAS / SAIF cells are you claiming coverage on?" for every scenario in the harm model.

You are **not** writing code in this exercise. You are producing the **design artefact** that every subsequent control implementation must map to.

### The five slots (from chapter 01)

Instantiate each with a concrete responsibility for your application:

1. **Input filter** — between caller and model.
2. **Retrieval filter** — between retriever and context assembler (omit only if the system has no retrieval; if it has none, say so explicitly and cross-reference to how the harm-classes that a retrieval filter would have addressed are handled elsewhere).
3. **Tool-invocation policy** — between the model's tool-call intent and the tool executor (omit only if the system has no tool calls; same discipline as above).
4. **Output validator** — between the model output and the caller / downstream.
5. **Safe-completion / refusal shaping** — the last thing before bytes leave the application.

For each slot present in your architecture, name a **concrete mechanism candidate** (e.g., "NeMo GR input rail + Llama Guard action" or "custom middleware + Presidio scan"). You will validate these choices when you actually ship the controls in exercises 02–04; for now this is a design commitment, not shipped code.

For each slot **absent** from your architecture (either because the system has no retrieval / no tools, or because the design deliberately omits a slot), write a **why** paragraph that either explains why the slot's responsibility is covered elsewhere or names the harm-model rows that will therefore be scored control-defeated in the register.

### The coverage table

The coverage table has one row per slot present in your architecture. Columns:

| Slot | OWASP LLM Top 10 (version pinned) | MITRE ATLAS tactic(s) | Google SAIF pillar(s) | Chosen mechanism (candidate) | Named failure mode |
|---|---|---|---|---|---|

Fill every cell explicitly. Cells that name multiple entries (e.g., "LLM01 + LLM05 partial") must qualify the "partial" — write the sub-scenario the slot addresses. Cells that are empty mean "this slot does not address this class"; write the empty cell explicitly, do not skip.

### The composition test

After the table, walk the composition test from chapter 01 §"Defence in depth" for **three harm-model scenarios**:

- **Scenario A.** Pick a scenario for which the harm model says at least two slots address it. Name those slots. Describe what happens to the residual number if each slot is defeated one at a time, then both together. If any single defeat collapses the residual to inherent, that scenario has a silent SPOF and needs a design fix.
- **Scenario B.** Pick a scenario that is addressed by only one slot in your current design. Either add a second slot with a *different mechanism* and re-walk, or file the scenario as open work with a treatment plan.
- **Scenario C.** Pick a scenario where the four silent-SPOF anti-patterns from chapter 01 §"The silent single-point-of-failure anti-pattern" could apply (shared-model bypass, fail-open on error, streaming bypass, debug bypass). Name which anti-pattern is most plausible for your system and what design or code discipline prevents it.

## Requirements

Produce all six items:

1. **Architecture diagram** — `risk/mod-108/<system-id>/architecture/slots.md` (or a rendered image alongside). Shows request flow through the five slots (or the subset you ship) with the mechanism per slot annotated. A single-figure sketch is enough; the diagram is a communication aid, not the specification.
2. **Slot specification** — `risk/mod-108/<system-id>/architecture/slot-spec.md`. One section per slot in your architecture: named responsibility (one sentence), named failure mode (one sentence from chapter 01 or your own if the failure mode is application-specific), chosen mechanism candidate (one line), version-pinning notes (which library / model / vendor version this will bind to when shipped in exercises 02–04).
3. **Coverage table** — `risk/mod-108/<system-id>/architecture/coverage.md`. The row-per-slot table shape defined above. Header names the OWASP LLM Top 10 version, the MITRE ATLAS matrix version, and the SAIF pillar set used. Cells that are addressed carry the entry code; cells that are empty are marked "—" not blank so the reader knows the empty cell was considered.
4. **Composition walk-through** — `risk/mod-108/<system-id>/architecture/composition.md`. Sections A, B, C as above; each names a specific mod-102 harm-scenario ID and walks the defeat sequence.
5. **Slot-to-scenario matrix** — `risk/mod-108/<system-id>/architecture/scenario-slot-matrix.md`. Rows are the harm-model scenarios for this system; columns are the slots. Each cell notes whether the slot addresses the scenario (Y / N / partial with note). Any scenario whose row is entirely empty is a control-defeated row for the register — flag it explicitly.
6. **Register handoff note** — `risk/mod-108/<system-id>/architecture/register-handoff.md`. One page. For every scenario where the coverage table adds a new mechanism candidate, name which mod-103 register row's residual and control-defeated columns will be updated once the mechanism is measured (exercise 05). Do not update the register yet — the residual credit requires measurement.

Cross-cutting rules:

- **No naked "high/medium/low."** Every rating or coverage claim carries either a citation (harm-scenario ID, OWASP LLM code, ATLAS tactic ID, SAIF pillar name) or a defended one-sentence note. Naked ratings turn the artefact into vibes (chapter 06 §"Build-vs-buy matrix").
- **Empty cells are stated, not hidden** (chapter 01 §"Slot × attack-class coverage table"). A scenario a slot does not address is a stated fact; do not omit rows to hide gaps.
- **Version-pin the frameworks at the top of the coverage table.** OWASP LLM Top 10 v1 2023 and v2025 use different category codes; the ATLAS matrix version and the SAIF pillar set both evolve. Pin what you cite.
- **This is design, not code.** Do not stub any mechanism in this exercise; that is exercise 02–04. Do not claim measured coverage; that is exercise 05.

## Starter guidance

- **Start from the mod-102 harm model, not from the slots.** Enumerate the harm-scenario rows first; the slot set is what you build to *address* those rows. If you start from the slot list, you will produce an architecture that looks complete but leaves harm rows uncovered.
- **The tool-invocation policy slot is deterministic — treat it that way.** Chapter 02 §"When rule-based fits" is explicit: tool-call policy is never classifier-based. If your candidate mechanism for slot 3 is "Llama Guard on the tool call," that is a design bug; the mechanism there is an allow-list + argument schema in custom middleware.
- **Retrieval filter is easy to under-scope.** It has to catch indirect prompt injection (LLM01), sensitive information disclosure through retrieved PII (LLM06), and embedding-space attacks (LLM08). If your candidate mechanism for the retrieval slot only names one of these, expand.
- **Safe-completion / refusal shaping is the slot most commonly ignored.** It is not "the classifier decided to refuse." It is the *content* the caller sees when the classifier refuses — a refusal template that does not leak policy and does not restate the disallowed content. Chapter 01 §"The slot set" describes the failure mode explicitly.
- **The four silent-SPOF anti-patterns are always worth checking against.** Even a well-designed slot set has one of them latent; naming which anti-pattern most plausibly bites this system is the composition walk-through's payoff.
- **Reuse the working example.** Chapter 01 §"Worked example — customer-support RAG assistant" is a full instantiation you can mirror for your own system. It is a shape to reproduce, not to copy; the mechanism choices there are illustrative, not prescriptive for your system.

## Acceptance criteria

The submission is accepted when:

- All six artefacts exist under `risk/mod-108/<system-id>/architecture/` with the file names above.
- Every slot present in the architecture has a stated responsibility, failure mode, and mechanism candidate.
- Every slot absent from the architecture has a written justification and cross-reference to how the harm-classes are handled.
- The coverage table pins the OWASP LLM Top 10 version, the MITRE ATLAS matrix version, and the SAIF pillar set used, with every cell either populated or marked "—."
- The composition walk-through covers scenarios A, B, and C from a specific harm-model row ID each, and walks the defeat sequence explicitly.
- The scenario-slot matrix flags every scenario with zero coverage as an open row for the register.
- The register handoff note names the mod-103 rows that will be updated after exercise 05 measurement, without pre-crediting them.

## Stretch goals

- **Session / agent extension slots.** If your system has session state (memory) or agent sub-invocations (LangGraph nodes spawning sub-agents), extend the slot set with the two additional slots chapter 01 §"The slot set" mentions and walk their responsibilities and failure modes.
- **The streaming path.** If your application ships a streaming completion endpoint, describe how the output validator (slot 4) runs on a streamed response. Chapter 01 §"The silent single-point-of-failure anti-pattern" case 3 (streaming bypass) is explicit that this is a common defect; the stretch is to document the design that prevents it in your system.
- **Mod-104 handshake.** Skim the mod-104 exercises and identify which of your slots the mod-104 red-team payloads target most directly. Name the top-two payload categories per slot so exercise 05 can consume the mod-104-hosted set with an already-attached slot label.
- **Cross-module handshake with mod-109.** Draft a one-paragraph "monitor selection for these slots" note — for each slot, name the signal you would expect the mod-109 monitor to track (e.g., "input filter FP rate, weekly" or "tool-invocation policy allow-list violation count, real-time"). This is a shape for mod-109 chapter 01 to consume, not a mod-109 deliverable.
- **The refusal string audit.** Sample five refusal strings the system currently emits (or would emit under a plausible design). Score each against chapter 01's failure mode for slot 5 — does the string leak internal policy or restate the disallowed content? File any that fail as a fix for the exercise-04 output slot.

## Deliverable format

A single markdown document `mod-108-ex01-slots.md` that indexes the artefacts. The index document contains:

1. *System summary* — one paragraph naming the running system, the harm-model source, and the mod-103 register file.
2. *Framework versions pinned* — OWASP LLM Top 10 version, MITRE ATLAS matrix version, SAIF pillar set.
3. *Architecture diagram* — inline figure or a `slots.md` pointer.
4. *Slot specification* — the block from `slot-spec.md`.
5. *Coverage table* — the block from `coverage.md`.
6. *Composition walk-through* — scenarios A, B, C from `composition.md`.
7. *Scenario-slot matrix* — the block from `scenario-slot-matrix.md`, with any uncovered row flagged.
8. *Register handoff note* — the list of mod-103 rows queued for post-exercise-05 update.
9. *Open items* — anything the composition walk-through surfaced that needs a treatment plan (e.g., a scenario with only one addressing slot that cannot be given a second mechanism inside this module — escalates to mod-112).

## Submission

Attach the index markdown and the six architecture artefacts to the module submission channel. The artefacts are the reference set every subsequent exercise builds on; keep them under version control from now on and treat any change as a formal revision, not an edit.
