# Handoffs at the Level Ladder: L15 Intake Upstream, L40 Frontier-Agent Depth Downstream

The harm model has two edges where the level-25 role hands off. Below it, the analyst at level 15 (`ai-governance-analyst`) produces the *intake* that your harm model consumes. Above it, the safety engineer at level 40 (`agentic-safety-engineer`) picks up the *frontier-agent depth* the harm model refers to but does not itself resolve. Getting these two interfaces right is what keeps you from accidentally doing analyst work or accidentally under-covering an area that should have escalated. Mod-101 chapter 06 introduced the ladder; this chapter drills the two harm-model-specific interfaces.

## Motivation

The harm model is not authored from scratch. It reads *from* intake — what the system does, who uses it, what data flows through it, what regulatory context applies — and it writes *toward* mitigation and register. If the intake is bad, your harm model is speculative. If you take frontier-agent scenarios that require dangerous-capability evaluation depth into your own scope, you either produce a shallow evaluation (and the auditor calls it out) or you take on work that displaces your MEASURE-side output for the rest of the release cycle.

Both edges are policed by explicit interface contracts. The rest of this chapter defines them.

## Upstream: what you consume from `ai-governance-analyst` (L15)

Before you open the harm-model repo for a system, the level-15 analyst has already produced (or is expected to have already produced):

1. **System intake record.** What the system does, who its users are, what capability surfaces it exposes, what data types it ingests, what tools it can call, where the deployment sits geographically, what regulated jurisdictions apply. Format is usually a structured record in a governance platform (GRC tool, internal registry, or a YAML file in the risk-engineering repo depending on organisation maturity).
2. **First-draft AI impact assessment.** The ISO/IEC 42005-shaped impact assessment, first-drafted from a template. It names the intended purpose, foreseeable misuse, affected stakeholders, and mitigation strategies at a governance level of detail.
3. **Framework-crosswalk legwork.** Which NIST AI RMF sub-categories apply, which ISO/IEC 42001 Annex A controls the system inherits, which EU AI Act articles are triggered, and which sector overlays are in play (mod-101 ch 05).
4. **Inventory record.** The system's row in the AI inventory with its ID, owner, business use case, deployment status, and change-control state.
5. **Regulatory tracking.** Any new jurisdictional obligation that would affect this system, tracked at the analyst side and surfaced to you when it lands.

If any of these five inputs is missing on the ground, do not silently produce them yourself. Instead: file a *coverage gap* against the analyst function; harm-model work waits on the missing input; escalate visibility if the input has been missing for more than one release cycle.

### Field-by-field intake ↔ harm-model interface

The harm model's fields at chapter 01 pull from the intake as follows:

| Harm-model field | Intake source |
|---|---|
| System scope (which capability surfaces exist) | Intake record |
| Which parties can be harmed | Impact assessment (stakeholder map) |
| Which taxonomies apply (NIST GenAI Profile, ISO 23894, AIRO, AIR-Bench) | Framework-crosswalk legwork |
| Which regulatory obligations flow into `mitigation_owner` | Regulatory tracking + crosswalk |
| Which existing inventory record the harm model is attached to | Inventory record |

Every harm-model repo entry cites the intake record ID. If the intake record moves or is deprecated, the harm-model row updates or archives with it. Silent drift in either direction is an audit finding.

### What you write *back* to the analyst

The interface is not read-only. As you author the harm model, you produce:

- **Corrections** — the intake said the system doesn't have a `refund` tool but the codebase says it does. File back.
- **Enrichments** — additional stakeholder classes, additional capability surfaces, additional jurisdictions triggered by an integration you found while modelling.
- **First-draft impact-assessment amendments** — the ISO 42005 fields the analyst first-drafted often need the harm-model author's engineering-side detail before they are final.

Route these back through the analyst rather than editing the governance record directly. That is what keeps the analyst's records defensible.

### An intake-shape template

A minimally useful intake shape (YAML fragment, illustrative) that supports harm-model authoring:

```yaml
system_id: cs-rag-assistant-01
name: Customer Support RAG Assistant
purpose: Answer customer support questions using indexed KB
capability_surfaces:
  - text_generation
  - retrieval
  - tool_calling
  - multimodal_input
  - state_across_turns
users:
  - class: paying_customer
  - class: support_agent
data_ingest:
  - class: user_message
  - class: kb_document
  - class: previous_conversation
tool_topology:
  - name: lookup_order
    permissions: read_only
  - name: escalate_ticket
    permissions: write_ticket
regulated_contexts:
  - eu_ai_act_role: deployer
  - us_state: ca
  - sector: consumer_saas
inventory_record_id: inv-2025-014
```

Whatever the format your organisation uses, this is the minimum information the harm-model author needs. If the intake shape doesn't carry these fields, negotiate an update with the analyst function before you start harm modelling — otherwise you will be inventing intake as you go, which is exactly what the ladder is trying to prevent.

## Downstream: what you hand off to `agentic-safety-engineer` (L40)

The level-40 role owns *frontier-agent* depth — dangerous-capability evaluations, RSP-shape frontier-model interaction, agentic multi-step / multi-agent scenarios at the depth a safety-engineering specialist covers. There are five kinds of scenario your harm model *touches* but does not *resolve*, and each escalates.

### 1. Frontier-model capability triggers

If your system consumes a general-purpose model that is nearing frontier-lab capability thresholds (chapter 04), harm scenarios that reason about *the model's frontier capability* are not yours to evaluate. Your harm model *names* them and routes them to the provider's evaluation (a level-40 concern to negotiate) or to `agentic-safety-engineer` for dangerous-capability evaluation methodology.

### 2. Multi-step autonomous agent scenarios

For agents that plan and execute long chains of tool calls, delegate to sub-agents, spawn processes, or interact with the real world through browsers / robotic control, harm scenarios that require *dangerous-capability evaluation of agent traces at depth* — not just "the agent called the wrong tool," but "the agent exhibits goal-directed behaviour outside its instruction envelope" — escalate. Your harm model names the risk and cites the escalation.

### 3. Frontier red-team methodology

You cover *enterprise* red-team methodology in mod-104. Frontier red-team methodology — dangerous-capability elicitation, sandbagging detection, evaluation of situation-aware behaviour, deceptive-alignment probing — is the level-40 depth. Harm-model rows that would only be measured by frontier methodology escalate.

### 4. RSP-shape deployment tiering at frontier scale

Your enterprise deployment tiering (chapter 04) borrows the shape. When a system genuinely operates at frontier scale — internal model developer with frontier capabilities, safety-testing lab, national-security-adjacent context — the tiering itself is a level-40 (or above) concern.

### 5. Cross-organisation industry coordination

Frontier Model Forum-style industry coordination, joint evaluation programmes with government AI safety institutes, coordinated responsible-disclosure of dangerous capabilities — these escalate. Your harm model may reference them; it does not run them.

### The escalation shape

For each scenario that escalates, the harm-model row's `mitigation_owner` field carries the escalation:

- `mitigation_owner: escalated_to = agentic-safety-engineer (L40)`
- `escalation_reason: dangerous-capability evaluation methodology required`
- `residual_owner_in_scope: <your name / role>` — you carry the residual until the escalation resolves.

That last line matters. Escalation is not the same as offloading. You still track the residual; the depth expansion is offloaded.

## The two edges together

The upstream edge (L15 → you) is *consume with contract*. The downstream edge (you → L40) is *route with residual*. The two together mean the harm model has:

- A clean provenance for every fact it asserts about system context (intake source cited).
- A clean escalation for every scenario that exceeds its scope (residual owner + escalation reason).

That is the artifact-level version of the ladder contract from mod-101 ch 06. Applied specifically to harm modelling.

## An interface contract you actually write

The contract you produce is a small YAML at `docs/interfaces.yaml` alongside the harm model. Illustrative:

```yaml
harm_model:
  system_id: cs-rag-assistant-01
  intake_record_id: inv-2025-014
  intake_owner_role: ai-governance-analyst-L15
  intake_last_synced: 2026-05-14
  intake_gaps_open: []
  frontier_escalations:
    - row_id: HR-CS-AGT-04
      reason: multi-step tool-chain autonomy exceeds L25 methodology scope
      escalated_to: agentic-safety-engineer-L40
      residual_owner_in_scope: ai-risk-engineer-L25
      residual_review_cadence: monthly
```

Each release cycle, this file is diffed. The diff is a first-order artifact for release evidence.

## Anti-patterns

Three anti-patterns to catch yourself in:

- **Silently authoring intake.** You are on a deadline; the analyst hasn't produced the intake record; you improvise. Don't. Author the *harm-model draft* against a clearly-marked *provisional intake* and open the coverage gap. If your harm model looks solid but the intake underneath was invented, the whole artifact is upstream of trust.
- **Silently absorbing frontier scope.** A single scenario in your harm model requires dangerous-capability evaluation methodology. You take it on because "we're not that big." Two months later the evaluation is inadequate and audit flags it. Escalate the methodology; keep the residual.
- **Duplicating impact-assessment work.** Your harm model has two long paragraphs about stakeholder categories and jurisdictions. That belongs in the impact assessment, not here. Your harm model *cites* those; it doesn't rewrite them.

## Alignment with mod-101 ch 06 and mod-112

The ladder framing here is a specialisation of mod-101 ch 06's overall deferral contract to the harm-model artifact. Mod-112 will build the full cross-functional contract at program scope. This chapter's job is to make the two harm-model-adjacent interfaces first-class.

## Summary

- Upstream (L15 → you): consume system intake, first-draft impact assessment, framework crosswalk, inventory record, regulatory tracking. If missing, file a coverage gap; don't invent intake.
- Every harm-model row cites its intake source; drift in either direction is an audit finding.
- Downstream (you → L40): frontier-model capability triggers, multi-step autonomous-agent scenarios, frontier red-team methodology, frontier-scale deployment tiering, industry coordination all escalate.
- Escalation is not offloading — the residual owner remains in scope on your side, with a review cadence.
- The interface contract is a small YAML alongside the harm model that names intake sources, gaps, and frontier escalations — first-order release evidence.
- Anti-patterns to avoid: silently authoring intake, silently absorbing frontier scope, duplicating impact-assessment paragraphs.
