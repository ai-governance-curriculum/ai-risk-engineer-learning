# The AI Incident Taxonomy, Severity Rubric, and Triage Routing

A general software / security incident-response process — the SEV rubric, the on-call rotation, the war-room ritual, the post-mortem template — was designed for outages and intrusions. AI systems introduce a distinct class of incident that does not always look like an outage and does not always look like an intrusion. This chapter names those classes, ranks them on a severity rubric your existing SEV process can consume, and defines the routing decision that comes first when the pager goes off: is this product IR, security IR, governance IR, or all three at once.

## Motivation

Two failure modes an AI risk engineer inherits from the classical process:

- **Miscategorised as a defect.** A model regression that quietly biases refunds against a protected group is filed as a JIRA ticket labelled "model quality regression — P3." No incident channel is opened, no post-mortem is written, and the regulator later asks for the incident file that does not exist.
- **Miscategorised as a breach.** A prompt-injection exploit that surfaces private context from the RAG store is opened as a security SEV1 and routed to SOC. SOC treats it as a data-exfiltration incident with unknown TTPs, spends two days looking for a network vector, and never touches the app-side trace store that holds the evidence.

Both failure modes have the same root cause: no one wrote down the taxonomy that says *this is what an AI incident is, this is the severity, and this is the routing*. This chapter is that write-down.

## The AI-specific incident taxonomy

Every AI-specific incident this role handles falls into one of the classes below. The class labels drive the triage routing and the RCA playbook, so treat the labels as first-class enum values, not prose. Add new classes deliberately, with a review, not by editing severity language in place.

| Class | Short description | Primary evidence surface | Typical routing |
|---|---|---|---|
| **Jailbreak / prompt-injection exploitation** | A user or an upstream-content path induces the model to bypass its safety layer or produce output outside policy | Trace store: prompt, retrieval context, model output, guardrail verdicts | Product IR + governance IR; security IR if data exfiltration is confirmed |
| **Model regression** | A new model, prompt, retriever, or decoding config makes a previously-passing eval or fixture fail; live behaviour degrades against the pre-release envelope | Eval-suite deltas, shadow-eval report, trace store, release manifest | Product IR + governance IR |
| **Biased-outcome escalation** | A protected-group slice's outcome metric moves outside its documented band, or a specific complaint chain shows disparate impact | Fairness slice report (mod-106), grievance channel, trace store | Product IR + governance IR; legal for jurisdictions where disparate impact is actionable |
| **Hallucination-driven harm** | A confabulated fact is surfaced to a user or a downstream system, and a real-world action follows (a bad refund, a wrong medical instruction, a misfiled legal citation) | Trace store, downstream system log, user report | Product IR + governance IR |
| **Unauthorised tool use** | A model / agent invokes a tool with parameters that were not authorised (wrong scope, wrong tenant, wrong side-effect class) | Tool-call log, planner trace, credential-scope audit | Product IR + governance IR + security IR |
| **Membership-inference or training-data exposure** | Model output reproduces training data, or an inference-time probe recovers the training-membership of a record | Model output, red-team probe log, dataset-manifest link | Product IR + governance IR + security IR + privacy office |
| **PII leakage** | Personal data flows into an output, a log, a downstream store, or a model-provider prompt where it should not | Trace store, log-pipeline audit, DPIA record | Product IR + governance IR + privacy office; security IR if the leak crossed a trust boundary |

Additional classes appear in the frontier-agent depth (autonomous tool chains, self-directed goal reformulation, resource acquisition) and escalate to `agentic-safety-engineer` at level 40 — chapter 06 draws the line. For this role's day-to-day, the seven classes above are the taxonomy floor.

The class labels are cross-walked to the mod-102 harm-model row that predicted the scenario. If an incident has no matching harm-model row, that is itself a finding — it means the harm model missed the scenario, and mod-102 gets an update through the incident-triggered path.

## Severity rubric — a rubric your SEV process can consume

The existing SEV1 / SEV2 / SEV3 process at most organisations grades on customer-visible outage minutes and revenue impact. AI incidents rarely present as outage minutes; a hallucinated medical instruction to one user can be a SEV1 harm while the service is 100% available. The rubric therefore has to grade on *harm shape*, not on outage-minute proxies.

A defensible rubric that plugs into the standard SEV levels:

| Severity | Harm-shape criteria (any one triggers the level) | Response posture |
|---|---|---|
| **SEV1** | Confirmed serious harm to a named party or protected class; confirmed data exposure that triggers a breach-notification obligation; confirmed regulator-reportable event under EU AI Act Article 73 or an equivalent sector rule (SR 11-7 material MRM incident, FDA MDR-triggering SaMD event, HIPAA breach); confirmed autonomous / agentic action outside the authorised scope of the system | War-room, executive sponsor, regulatory-reporting clock starts, all three IR tracks engaged |
| **SEV2** | Confirmed unsafe output pattern reaching users at scale without confirmed serious harm; model regression whose live metrics violate the release envelope on a production slice; guardrail-effectiveness delta breaches its floor on a production model; jailbreak that bypasses a shipped safety control but has not yet been observed to cause harm | On-call incident lead, product IR + governance IR engaged, security IR conditional on evidence chain |
| **SEV3** | Repeatable near-miss caught by a monitor or a guardrail; single-user harm complaint with unclear reproducibility; upstream corpus of a shared component reports a class of exploit that has not yet been observed here | Async incident channel, RCA on business-hours cadence, post-mortem written to the same shape as SEV1/2 |
| **SEV4** | Signal-level anomaly that has not been reproduced; drift telemetry that has not yet crossed a threshold | Investigation ticket, no incident channel; if the anomaly reproduces or crosses a threshold, escalate to SEV3 |

Two rubric rules worth pinning explicitly:

- **A single user's serious harm can be a SEV1.** Availability-shaped severity would grade it a P3. Do not let the classical rubric downgrade harm-shape incidents.
- **A confirmed regulator-reportable event is always at least a SEV1.** Do not litigate reportability inside the SEV rubric; that is the compliance track's decision (chapter 05).

The rubric is versioned in the incident-response repo alongside the general SEV rubric, and every AI SEV maps to a general SEV band so the enterprise IR dashboard is not blind to AI severity.

## The routing decision — product IR vs security IR vs governance IR

Every AI incident is triaged along three axes at the same time, not one at a time. The three axes are independent — an incident can require zero, one, two, or all three tracks.

| Track | It is engaged when… | What it does |
|---|---|---|
| **Product IR** | The system's user-facing behaviour is (or could be) wrong — bad output, wrong tool call, degraded quality, biased outcome | Owns the reproduction, the app-side evidence chain, the immediate-mitigation change, the user-communication draft |
| **Security IR (SOC-side)** | Confidentiality, integrity, or availability of the system's data or infrastructure has been touched — data exfiltration, credential misuse, model / prompt-store tampering, DoS on the model provider | Owns the SOC-side evidence chain (network, host, IAM, provider logs), classical containment / eradication / recovery, the security post-mortem contribution |
| **Governance IR** | The incident is a candidate obligation trigger — EU AI Act Article 73 serious incident, SR 11-7 MRM incident, FDA MDR-triggering event, HIPAA breach, a policy-council-visible harm — regardless of whether product or security is engaged | Owns the reportability decision, the regulator-package clock (chapter 05), the risk-register update (chapter 03), the risk-council briefing (mod-112) |

A working routing matrix for the classes in the taxonomy:

| Incident class | Product IR | Security IR | Governance IR |
|---|---|---|---|
| Jailbreak / prompt injection (no data touched) | Yes | Conditional | Yes |
| Jailbreak / prompt injection (data exfiltrated) | Yes | Yes | Yes |
| Model regression (safety metric) | Yes | No | Yes |
| Model regression (quality-only, non-safety) | Yes | No | Conditional |
| Biased-outcome escalation | Yes | No | Yes |
| Hallucination-driven harm (single user) | Yes | No | Yes |
| Hallucination-driven harm (class of users, downstream action) | Yes | Conditional | Yes |
| Unauthorised tool use | Yes | Yes | Yes |
| Membership-inference / training exposure | Yes | Yes | Yes |
| PII leakage across a trust boundary | Yes | Yes | Yes |

"Conditional" means a documented decision at triage time — not a default. The decision is recorded on the incident ticket and is itself audit evidence.

## Interface with the general SEV process

The AI-specific taxonomy does not replace the general SEV process. It plugs into it in three specific places:

- **Signal ingestion.** Monitors from mod-109 (drift, guardrail-effectiveness delta, safety-classifier signal, tool-call anomaly) create incident candidates against this taxonomy the same way SRE monitors create availability candidates against the classical taxonomy.
- **SEV declaration.** The AI severity rubric maps into the general SEV bands so the incident is visible on the enterprise IR dashboard. An AI SEV1 is a general SEV1; do not create a shadow rubric.
- **Post-mortem.** The general post-mortem template gets AI-specific extensions (chapter 03) — model version diff, prompt / retriever / decoding config diff, guardrail-effectiveness delta, harm-model row reference, register-row reference. The general template does not have those fields; you add them.

The AI-specific process is *additive*. Removing the general process because "AI is different" is an anti-pattern — the on-call rotation, the incident-channel discipline, and the executive-sponsor pattern are still doing real work.

## Mock triage walkthrough — a jailbreak with retrieval exfiltration

To make the taxonomy concrete, walk one incident through the triage:

- **Signal.** A monitor from mod-109 fires: "prompt-injection-detected count > threshold on retrieval path in the last hour."
- **Class assignment.** Cross-walk against the taxonomy — the pattern is jailbreak / prompt-injection exploitation.
- **Evidence quick-look.** Pull one trace; the retrieved KB chunk contains injection strings that induce a `send_email` tool call. Trace shows the tool call was executed with a customer email in the body.
- **Severity assignment.** A named customer's email address was exfiltrated to an attacker-controlled mailbox — confirmed data exposure. SEV1 under the harm-shape rubric; reportability check begins immediately.
- **Routing.** Product IR (app-side reproduction, immediate mitigation), Security IR (SOC-side confirmation, containment on the tool-call path, IAM audit on the outbound mailbox), Governance IR (Article 73 reportability check, register-row update to `HR-CS-INJ-01`, risk-council notification).
- **Harm-model cross-walk.** Row `HR-CS-INJ-01` predicted this scenario. Row exists; the incident is a scenario firing, not a taxonomy miss. Register update tracks residual movement (chapter 03 cross-reference).

That walkthrough is the shape exercise-01 trains at scale.

## Cross-references

- **Signal source** — mod-109 continuous-monitoring signals create the candidates on this taxonomy.
- **Harm-model row** — mod-102's harm-model row is the *predicted* form of the class; the incident is the *observed* form. Missing cross-walk is an audit finding.
- **Guardrail-effectiveness delta as evidence** — mod-108 owns the effectiveness measurement; the incident RCA (chapter 02) reads its delta.
- **Register update** — mod-103's incident-triggered cadence receives the residual update and the corrective-action-plan (chapter 03 of this module).
- **Regulator-package clock** — chapter 05 of this module owns the timing envelopes; the taxonomy row above is what starts the clock.

## Summary

- The AI-specific incident taxonomy has seven classes at the floor: jailbreak / prompt-injection exploitation, model regression, biased-outcome escalation, hallucination-driven harm, unauthorised tool use, membership-inference / training-data exposure, and PII leakage.
- Severity grades on *harm shape*, not on outage-minute proxies. A single user's serious harm can be a SEV1. A confirmed regulator-reportable event is always at least SEV1.
- Routing is a three-axis decision — product IR, security IR, governance IR — made at triage time and recorded on the ticket. "Conditional" is a decision, not a default.
- The AI taxonomy plugs into the general SEV process at signal ingestion, SEV declaration, and post-mortem; it does not replace it.
- Every AI incident cross-walks to a harm-model row (mod-102) and a register row (mod-103); a missing cross-walk is itself a finding.
- Exercise-01 drills the triage and routing decision on a mixed queue of candidate incidents.
