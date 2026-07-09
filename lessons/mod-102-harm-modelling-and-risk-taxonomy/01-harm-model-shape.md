# The Harm Model as an Engineering Artifact

A *harm model* is not a paragraph in an impact assessment. It is a versioned engineering artifact that lists, for one AI system, the specific ways that system can cause harm, who has to be present for each harm to occur, which capability of the system enables it, and how the first-order harm propagates outward. Everything downstream — red-team scope, guardrail selection, monitor thresholds, incident-triage playbooks — reads off of this file. This chapter defines the artifact.

## Motivation

Ad-hoc harm modelling produces two failure modes an auditor immediately spots:

- **Vibes-based enumeration.** The harm model reads like a threat brainstorm: "the model could be jailbroken," "biased outputs," "hallucinations." No adversary, no capability, no downstream chain, no evidence. The auditor asks *"how did you decide these are the harms"* and there is no answer.
- **Framework-copy enumeration.** The harm model is literally the NIST AI 600-1 risk list pasted in. Every system inherits the same twelve rows regardless of what it does. When the auditor asks *"where does risk #7 actually show up in your system,"* it turns out nobody knows.

The engineering way out is to model the harm model itself — treat it as a data structure with required fields, produce it from evidence, and version it. Then the downstream artifacts have something concrete to consume.

## The artifact's shape

A harm model for one AI system is a table. Each row is one *harm scenario*, and each column is a required field. The minimum column set:

| Column | What it holds |
|---|---|
| `id` | Stable identifier used by every downstream artifact (register row, red-team fixture, guardrail config, monitor spec) |
| `persona` | Which misuse persona instantiates the scenario (see below) |
| `capability_surface` | Which capability of *this* system is being exercised |
| `immediate_harm` | The first-order harm that lands on a named party |
| `secondary_harm` | The follow-on harm that lands on a related party or on the operating environment |
| `cascading_or_systemic_effect` | Downstream / population-level effect if the scenario is repeated at scale |
| `evidence_refs` | AIID / OECD.AI incident IDs, MIT AI Risk Repository row IDs, published cases (mod-102 ch 02) |
| `taxonomy_tags` | NIST GenAI Profile risk #, MIT AIR domain + causal codes, ISO 23894 risk source, AIRO class, AIR-Bench 2024 category (mod-102 ch 03) |
| `cause_bucket` | Operator / model / data / integration (mod-102 ch 05) — routes ownership |
| `mitigation_owner` | The role that will own the fix (yourself, an ML/eng team, a peer track, out-of-scope escalate) |

Rows without evidence refs, taxonomy tags, and a cause bucket are *drafts* — not part of the shippable harm model. Chapters 02, 03, and 05 close each of those fields respectively.

## Misuse personas — the four you must cover

A harm scenario is instantiated by *someone doing something*. The persona names the "someone." Level-25 harm-modelling covers four personas as a floor. If any of the four is absent from your harm model, an auditor will ask why.

### 1. Adversarial user

The end user is trying to make the system do a thing it should not do. Prompt injection into a chat window, uploading a document that carries jailbreak instructions, chaining a benign-looking sequence of requests until the model complies, wearing an adversarial patch to fool an image classifier. This is the persona most systems remember to model; it is also the one that hides the least.

### 2. Insider

An operator-side actor with authorised access uses that access to reshape the system's behaviour. Examples: a prompt engineer swaps the system prompt to disable a refusal; a data scientist swaps a fine-tuning dataset without going through review; an ops engineer replaces a guardrail model with a permissive one; a support agent uses an "admin bypass" mode more broadly than intended. Insider personas produce the harms that *pass the red-team*, because the red-team assumes production configuration.

### 3. Upstream data supplier

Any party whose data enters the system's training, fine-tuning, retrieval, or context. Examples: a public-website author who plants indirect-prompt-injection strings that later appear in RAG chunks; a labelling vendor whose annotators systematically misclassify a demographic; a pre-trained-checkpoint publisher who ships a backdoored embedding model; a synthetic-data generator whose distribution drifts. Upstream personas are the ones the ordinary threat model *skips*, and they are the ones that surface in supply-chain incidents.

### 4. Downstream integrator

A party that consumes the system's output and puts it into a real-world context you don't fully control. Examples: an application team that turns advisory model output into an auto-approve loop; a downstream chatbot that concatenates your model's output into another model's prompt (compounding failure); a partner product that surfaces your recommendations as "expert advice" without the caveats you shipped. Downstream personas are how a well-behaved model still causes harm — because the integrator disabled the guardrail you shipped or removed the context that gave the output its meaning.

Every scenario row names *exactly one* of these personas. Scenarios that need two personas (e.g., an adversarial user *and* an insider who left a config file exposed) become two rows sharing an `id_group` — do not collapse them, because the mitigations are different.

## Capability surfaces — the "against what"

The capability surface is the specific behaviour of *this* system that the persona is exercising. Naming this precisely is what stops the harm model from being generic.

A working checklist of capability surfaces for a typical LLM-based enterprise application:

| Capability surface | Concrete instance in a customer-support RAG assistant |
|---|---|
| Text generation | Answers user's question in natural language |
| Instruction following | Executes multi-step instructions given in the user message |
| Retrieval | Selects documents from a KB and inserts them into context |
| Tool calling | Calls internal APIs (lookup order, escalate ticket, refund) |
| Memory / state | Preserves user-provided context across turns |
| Multimodal input | Accepts screenshots / PDFs from the user |
| Output rendering | Renders as HTML into a support-agent console |
| Human-oversight surface | Provides "approve / edit" affordance to the human agent |
| Vendor / model configuration | Model, temperature, system prompt, safety-classifier settings |
| Auditability surface | Trace store, retention window |

The harm scenario names *which* of these it exercises. "Prompt injection" is vague; "indirect prompt injection via retrieved KB article that induces `escalate_ticket` tool call to a mailbox controlled by attacker" is a harm scenario.

For agentic systems the capability surface list expands (tool topology, planner outputs, sub-agent spawning, credential propagation, browser control). The frontier-agent depth of that expansion is where `agentic-safety-engineer` (level 40) takes over — chapter 06 draws the exact line.

## The immediate → secondary → cascading → systemic chain

A single harm rarely lands on one party and stops. The harm model has to trace at least three steps.

| Step | Question it answers |
|---|---|
| **Immediate harm** | Who is directly injured when the scenario succeeds once? |
| **Secondary harm** | Who else is affected as a foreseeable consequence — related party, operating environment, system operator? |
| **Cascading effect** | What happens when the scenario is repeated across many users or many systems that use this component? |
| **Systemic effect** | What happens at population, market, or public-institution scale if this scenario is common? |

Tracing this chain forces the model to reason about *scale* explicitly rather than pricing every scenario as "one incident." An SR 11-7 / SR 23-4 shop needs the cascade view for third-party concentration analysis; an EU AI Act filer needs it for the "risk to fundamental rights" argument in the Article 9 file. Both consumers ask the same underlying question — *how bad does this get when it happens a lot* — and the harm model is where you answer.

### Worked example — customer-support RAG assistant, one scenario

- `id`: `HR-CS-INJ-01`
- `persona`: Upstream data supplier — an author of a public help-forum thread whose contents get indexed into the KB.
- `capability_surface`: Retrieval → context assembly → tool calling (`escalate_ticket`).
- `immediate_harm`: Attacker-crafted post is retrieved during a legitimate user's session; the injected instruction causes the assistant to open an `escalate_ticket` call whose body exfiltrates the user's redacted account identifier to a mailbox the attacker monitors.
- `secondary_harm`: The support agent who reads the escalated ticket is misled into treating the exfiltration as a valid workflow step; internal trust in the ticket queue drops.
- `cascading effect`: If the injection pattern is common in indexed content, many sessions leak account identifiers over a rolling week; ticket-queue integrity is questioned across accounts; a whole-index reindex is required.
- `systemic effect`: If similar injection patterns are widespread in publicly indexed content and are not mitigated across many products, RAG assistants become a *class* of exfiltration channel and regulators respond at policy level.
- `evidence_refs`: (mod-102 ch 02) AIID entries on indirect prompt injection; OWASP LLM Top 10 LLM01; Greshake et al. indirect-prompt-injection paper.
- `taxonomy_tags`: (mod-102 ch 03) NIST GenAI Profile risk #9 (Information Security); MIT AIR domain "Privacy & Security"; AIRO consequence class `SecurityBreach`; AIR-Bench 2024 category tag TBD in ch 03.
- `cause_bucket`: (mod-102 ch 05) integration-caused (retrieval pipeline trusts unauthenticated upstream text).
- `mitigation_owner`: You (spec) + application team (implement); guardrail = provenance filter + tool-call allow-list.

That one row is enough to spawn a red-team fixture (mod-104), a guardrail config (mod-108), a monitor (mod-109), and an incident-triage playbook entry (mod-111). That is what "the harm model is upstream of everything" means in practice.

## What a good harm model is *not*

To keep the artifact useful, three anti-patterns to avoid:

- **Not a risk register.** The harm model enumerates scenarios and evidence; the register (mod-103) quantifies likelihood, severity, and residual risk. The two artifacts feed each other but are not the same file.
- **Not a threat model of the network stack.** The AI harm model covers AI-specific harms — the model output, the training data, the retrieval pipeline, the tool topology, the human-in-the-loop configuration. Classical network-and-endpoint threats route to SecOps.
- **Not the impact assessment.** The impact assessment (owned upstream by `ai-governance-analyst` at level 15, standardised in ISO/IEC 42005) is a governance document; the harm model is an engineering artifact that the impact assessment consumes and cites.

The three artifacts share fields but have distinct owners; drawing that boundary cleanly is chapter 06's job.

## Versioning and change control

The harm model is versioned in the risk-engineering repo, not in a governance portal. Every change gets a diff and a reviewer. Changes are triggered by:

- a new use case or capability being enabled,
- a new incident (yours or someone else's) landing in AIID / OECD.AI that a review says *would apply here*,
- a taxonomy update (NIST GenAI Profile revision, MIT AIR Repository update, AIR-Bench release),
- a red-team run producing a novel attack pattern (mod-104),
- an incident post-mortem that names an unmodelled scenario (mod-111).

Silent additions or deletions are audit findings. The changelog goes into the release evidence packet.

## Summary

- The harm model is a versioned engineering artifact: one system, many scenario rows, ten required fields per row.
- Four misuse personas are floor coverage — adversarial user, insider, upstream data supplier, downstream integrator.
- The capability surface names the specific behaviour the persona exercises; keep it precise or the harm model becomes generic.
- Every scenario traces immediate → secondary → cascading → systemic harm; this is how the same file serves both an SR 11-7 shop and an EU AI Act filer.
- The harm model is not the risk register (mod-103), not the network threat model, and not the impact assessment (owned at level 15).
- Exercise 01 turns this shape into a filled sheet of personas × capability surfaces for your running system.
