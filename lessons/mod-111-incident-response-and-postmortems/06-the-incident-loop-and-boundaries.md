# The Incident Loop, the SecOps / DFIR Boundary, and Level-40 Escalation

The five preceding chapters defined the pieces: taxonomy and triage, root-cause analysis, post-mortem authoring, fixture back-feed, regulator-facing packages. This chapter closes the loop — how the pieces compose into a running incident-response practice, where the boundary with SecOps / DFIR sits (this role does not do intrusion analysis or malware forensics), and what escalates to `agentic-safety-engineer` at level 40 for frontier-agent incidents. It is also the chapter that names the *learning-propagation* interface with the risk council (mod-112).

## Motivation

Two failure modes at the loop level:

- **The one-shot incident.** The team runs a great triage, an excellent RCA, and a defensible post-mortem — and then does not update the harm model, the register, the fixtures, or the monitoring. Six months later a variant re-produces the incident and it looks like the first time. The loop is broken.
- **The scope-creep incident.** The AI-risk engineer treats every AI incident as also-a-security-incident and takes on intrusion analysis, packet-capture forensics, or malware reverse-engineering. Investigations stall on skill gaps that SecOps / DFIR would clear in an afternoon. Post-mortems miss deadlines. The role's scope loses shape.

The loop-and-boundary shape below is designed to prevent both.

## The incident-response loop

The loop has eight stages, each with an owner and each with a hand-off artifact. Every stage's hand-off is a versioned file or ticket comment — not a chat thread.

```
signal (mod-109)
   |
   v
triage (chapter 01)          --> incident record + class + SEV + routing
   |
   v
RCA (chapter 02)             --> RCA evidence pack + causal chain paragraph
   |
   v
mitigation                   --> release-manifest entry + monitoring signal
   |
   v
post-mortem (chapter 03)     --> single post-mortem file, three audience extracts
   |
   v
register update (mod-103 ch 06) --> row edit + sign-off
   |
   v
fixture back-feed (chapter 04)  --> fixture in red-team + app-eval suites
   |
   v
regulator package (chapter 05)  --> extract(s) submitted, submission log
   |
   v
learning propagation (mod-112)  --> council packet entry; harm-model / rubric updates
```

The loop is not linear in wall-clock time — several stages run in parallel (mitigation, regulator clock, RCA) — but each stage has an artifact that has to exist by the time the next stage's sign-off closes. If any stage's artifact is missing when the next stage signs off, the loop has broken and the audit reads it as such.

### Stage owners and cadence

| Stage | Primary owner (role) | Latest artifact-due signal |
|---|---|---|
| Signal | Monitoring (mod-109) / on-call | Signal itself |
| Triage | Incident lead / risk engineer | Incident record + class + SEV + routing set on-ticket within on-call cadence |
| RCA | Risk engineer + `ai-eval-engineer` (L35) + SOC IR (if engaged) | Causal-chain paragraph and evidence pack signed before mitigation is declared complete |
| Mitigation | Product / platform engineering | Release manifest + monitoring confirmation of no recurrence within the mitigation window |
| Post-mortem | Risk engineer | Draft within an internal cadence (commonly 5-10 business days after mitigation); sign-off routes per section 10 of chapter 03 |
| Register update | Risk engineer | Row edit + sign-off before post-mortem is closed |
| Fixture back-feed | Risk engineer + `ai-eval-engineer` (L35) | Fixture IDs recorded on the CAP; both suites gated on release |
| Regulator package | Risk engineer authors; legal / compliance submits | Per the timing envelopes in chapter 05 |
| Learning propagation | Risk engineer + governance owner | Council packet entry (mod-112) at the next scheduled cadence; harm-model / rubric changes filed |

The loop is exercised in the exercises, but it lives day-to-day as the on-call runbook + the register operations doc + the risk-council packet cadence.

## The SecOps / DFIR boundary — what this role is *not*

The AI-risk-engineer role does not do digital forensics and incident response (DFIR) in the SecOps sense. Two teams do adjacent work, and drawing the boundary crisply prevents scope-creep and skill-gap failures.

| Work | Owned by | This role's interface |
|---|---|---|
| Network traffic capture and analysis | SOC / network security | Names the outbound endpoints of concern; consumes the SOC finding |
| Endpoint forensics (host / container / VM images) | DFIR | Names the trace window; consumes the DFIR finding |
| Malware reverse-engineering | DFIR / threat intel | Names the artifact (if one is retrieved); consumes the reverse-engineering finding |
| IAM audit — who invoked what credential | Security / IAM | Names the tool-call log and the credential of interest; consumes the IAM audit |
| Intrusion attribution (who did it) | Threat intel | Provides trace context; consumes the attribution finding |
| Data-exfiltration reconstruction (what left, to where) | SOC + data-loss-prevention | Names the exfiltrated content shape and volume; consumes the DLP finding |
| Model / prompt / RAG-corpus tampering forensics | This role + platform eng | Owns — trace-store discipline lives here |
| Guardrail-effectiveness and safety-classifier RCA | This role + mod-108 owner | Owns — the effectiveness suite is a risk-engineering artifact |
| Model-output behavioural RCA | This role + `ai-eval-engineer` (L35) | Owns — trace diffing and RCA workflow live here |

The interface artifact between this role and SecOps / DFIR is the *evidence hand-off record* (chapter 02) — trace IDs, tool-call arguments, timestamps, endpoints of concern, expected callback. Do not do their work in the RCA scratch file; hand it off cleanly.

The boundary the other direction is also crisp: SecOps / DFIR should not be reasoning about model output quality, retrieval provenance, or guardrail-effectiveness. When SOC returns a finding, the risk engineer folds it into the AI-side RCA — the reverse hand-off — and completes the causal chain.

## Level-40 escalation — frontier-agent incidents

Some incidents involve capability surfaces above this role's floor. The frontier-agent depth — autonomous, multi-step planning; long-horizon tool chaining; sub-agent spawning; goal reformulation under environmental pressure; resource acquisition; power-seeking-shaped behaviour patterns — belongs to `agentic-safety-engineer` at level 40, and its incident-response depth is that role's, not this one's.

Triggers that escalate an incident to L40:

- The incident involves a system whose deployment tier (mod-102 ch 04, frontier-lab-shaped tiers) puts it above the enterprise's non-frontier threshold.
- The failure mode includes an autonomous action outside the authorised scope of the system — planner produces a plan that steps outside its declared goal, sub-agent is spawned outside the declared topology, credentials are used outside their intended scope on a chain of the model's own construction.
- The incident matches a frontier-risk category enumerated in the enterprise's tiering (borrowed from Anthropic RSP, OpenAI Preparedness, Google DeepMind FSF — mod-102 ch 04) — CBRN uplift, cyber uplift with autonomous execution, autonomous replication, undermining oversight.
- The incident's downstream effect is on model-provider-level safeguards (a bypass that would work on the provider's model as shipped, not just on the deployment's configuration).

The L25 role's job at escalation:

- Continue to run the L25 loop for the deployment-side pieces (mitigation, harm-model / register update, fixture back-feed for the deployment-specific fixture, deployment-side regulator packages).
- Hand off the frontier-agent-specific RCA and the frontier-tier post-mortem section to L40; sign onto the hand-off record.
- Ensure the L40 output is folded back into the deployment-side post-mortem (it is one incident with two authors on the frontier-agent section, not two incidents).
- Route the frontier-agent-specific communication (frontier-lab tier disclosure, government AI safety institute liaison where applicable) through L40 and the governance leadership above L25.

The L40 role's job is the frontier-agent-specific investigation, the frontier-lab / RSP-shape reporting, and the model-provider-level engagement. The L25 role does not attempt any of these unilaterally.

## Learning propagation to the risk council (mod-112 interface)

The loop's final stage is *learning propagation*. The risk council (mod-112) consumes:

- **The incident summary** — the governance-council extract from chapter 03 section by section.
- **Aggregation trends** — mod-112 rolls up incidents by class, by system, by CAP age, by residual movement; open CAPs past due date surface as escalations.
- **Harm-model / rubric changes** — additions to the harm model (mod-102), rubric-version bumps to the risk register (mod-103), coverage-grid gaps that were exposed.
- **Programme-scale actions** — CAPs that span multiple systems (e.g., a shared component's incident triggers CAPs across every deployment that consumes it); the council sponsors and cadences those.
- **External-corpus updates** — where AIID / OECD.AI entries the council should be aware of match the incident class, the council packet cites them.

The interface artifact is the council-packet section this role owns. Its shape is defined by mod-112; this role's job is to produce the incident-derived content for it on schedule.

## The loop as an audit exhibit

At audit time, the loop is walked end-to-end for a sample of incidents. What the auditor checks:

- Every incident has a triage record with class, SEV, and routing decisions logged at triage time.
- Every incident has an RCA with a causal-chain paragraph and an evidence pack whose artifacts resolve.
- Every incident has a post-mortem with all ten sections; audience extracts exist for the ones that applied.
- Every register row referenced in the post-mortem shows the incident-triggered update; sign-off matches.
- Every confirmed exploit / regression has a fixture in both the red-team and app-eval suites; fixture IDs cross-reference back to the incident.
- Every reportable incident's regulator package exists and its submission log matches the timing envelope.
- Every incident appears in the council packet at the next scheduled cadence after post-mortem sign-off.

A missing artifact at any stage of the loop is a finding. The engineering discipline is to *not have that missing artifact*, not to reason about why the missing artifact was acceptable.

## Anti-patterns at the loop level

- **The half-loop.** Great incident work stops at post-mortem; register, fixtures, monitoring, council are not updated. Fix: the loop stages have named owners and their artifacts are checked before the incident is closed.
- **The scope-creep loop.** The risk engineer takes on packet capture, container forensics, and malware analysis. Fix: hand-off record to SOC / DFIR is a mandatory artifact whenever those chains are engaged.
- **The frontier-agent silence.** An incident with frontier-capability signals is treated at L25. Fix: the escalation triggers above are on the triage checklist (chapter 01) and route to L40 at triage time, not after RCA.
- **The council-only loop.** Post-mortems land in the council packet but the fixtures and register updates do not follow. Fix: the council packet is the *output* of the loop; the register + fixtures + monitor updates are the *upstream* stages and are gated first.

## Cross-references

- **Signal source** — mod-109 continuous monitoring feeds the loop.
- **Triage / RCA / post-mortem / fixture / regulator** — chapters 01-05 of this module.
- **Register update** — mod-103 chapter 06 event-triggered cadence.
- **Supply-chain incident shape** — mod-110 owns third-party-component and dataset-provenance incidents; those enter this loop with mod-110 as the surface for step 3's upstream / supply-chain delta.
- **Council** — mod-112 consumes the governance-council extract on its cadence.
- **Level-40 role** — `agentic-safety-engineer` for frontier-agent incidents; the boundary is documented on the enterprise's deployment-tier register (mod-102 ch 04).

## Summary

- The incident-response loop has eight stages: signal, triage, RCA, mitigation, post-mortem, register update, fixture back-feed, regulator package, learning propagation. Every stage has an artifact; missing artifacts are audit findings.
- The SecOps / DFIR boundary is crisp — this role does not do intrusion analysis, endpoint forensics, malware reverse-engineering, or IAM audit. Hand off cleanly via an evidence hand-off record.
- Frontier-agent incidents escalate to `agentic-safety-engineer` (L40); the L25 role continues the deployment-side loop and hands off the frontier-agent RCA and post-mortem section.
- Learning propagation to the risk council (mod-112) is the loop's closing stage — incident summary, aggregation trends, harm-model / rubric changes, programme-scale CAPs, external-corpus updates.
- At audit time the loop is walked end-to-end; the discipline is to *have* every artifact, not to justify missing ones.
- The module's five exercises exercise the loop stages in order — triage, RCA, post-mortem, fixture, regulator package — and produce the loop-shaped evidence trail a real programme runs on.
