# mod-111 — AI-Specific Incident Response and Post-Mortem Engineering

**Estimated effort:** 10 hours
**Position in the track:** eleventh module; assumes mod-101 through mod-110 are done.

A general software / security incident-response process was designed for outages and intrusions. AI systems introduce a distinct class of incident — jailbreak / prompt-injection exploitation, model regression, biased-outcome escalation, hallucination-driven harm, unauthorised tool use, membership-inference exposure, PII leakage — that does not always look like an outage and does not always look like an intrusion. This module teaches the engineering discipline that sits *on top of* the existing SEV process: how to triage AI-specific incidents, how to conduct trace-diff root-cause analysis, how to author a post-mortem that survives an engineering team, a governance council, and a regulator, how to turn every confirmed exploit into a permanent regression fixture in the red-team and app-eval suites, and how to package the regulator-facing summary in the EU AI Act Article 73, SR 11-7, FDA MDR, and (where applicable) HIPAA shapes.

Incidents feed *back* into the artifacts built earlier in the track: the harm model (mod-102) gains rows and edits; the risk register (mod-103) gains residual updates and corrective-action-plans; the red-team suite (mod-104), guardrail-effectiveness suite (mod-108), monitoring signals (mod-109), and MRM / supply-chain records (mod-110) all move on the incident's evidence. The module closes the loop and hands the learning to the risk council (mod-112).

## Learning objectives

- Triage AI-specific incidents — jailbreak / prompt-injection exploitation, model regression, biased-outcome escalation, hallucination-driven harm, unauthorised tool use, membership-inference exposure, PII leakage — and route them against a general software / security incident process (product IR vs security IR vs governance IR, or all three).
- Conduct technical root-cause analysis using the trace store, model version diffs, prompt / retriever / decoding config diffs, guardrail-effectiveness deltas, and drift signals — coordinating with `ai-eval-engineer` (L35) on the app-side evidence chain and with security on the SOC-side chain.
- Write a post-mortem that a governance council, a regulator, and the engineering team can each act on; feed the incident back into the risk register (mod-103) with an updated residual-risk claim and a corrective-action-plan.
- Turn every confirmed exploit / regression into a permanent regression fixture in the red-team suite (mod-104) and the app-eval suite — using the AI Incident Database + OECD.AI incidents as a public compare-point.
- Package the regulator-facing incident summary in the EU AI Act Article 73 shape for serious-incident reporting, in an SR 11-7 shape for FI MRM incidents, and in the FDA MDR (21 CFR Part 803) shape for AI / SaMD incidents; know where HIPAA and GDPR Article 33 run concurrently.
- Run the incident-response loop end-to-end (signal → triage → RCA → mitigation → post-mortem → register update → fixture back-feed → regulator package → learning propagation) and know the SecOps / DFIR boundary and the level-40 escalation to `agentic-safety-engineer` for frontier-agent incidents.

## Chapters

1. [`01-ai-incident-taxonomy-and-triage.md`](01-ai-incident-taxonomy-and-triage.md) — the AI-specific incident taxonomy, the harm-shape severity rubric, and the three-axis routing decision (product IR / security IR / governance IR); the interface with the general SEV process.
2. [`02-trace-diff-root-cause.md`](02-trace-diff-root-cause.md) — technical root-cause analysis on AI systems: the six evidence surfaces (trace store, model version, prompt / retriever / decoding config, guardrail-effectiveness delta, drift, supply-chain change log), the "what changed since the incident began" workflow, and the coordination interface with `ai-eval-engineer` (L35) and security.
3. [`03-post-mortem-authoring.md`](03-post-mortem-authoring.md) — the ten-section post-mortem shape that survives engineering, governance-council, and regulator audiences; the incident-into-register update; the corrective-action-plan with named owners and evidence-of-completion artifacts.
4. [`04-regression-fixture-back-feed.md`](04-regression-fixture-back-feed.md) — turning every confirmed exploit / regression into a pinned, versioned fixture in the red-team suite (mod-104) and the app-eval suite; fixture format, back-feed workflow, fixture hygiene; AIID + OECD.AI as public compare-points.
5. [`05-regulator-facing-summaries.md`](05-regulator-facing-summaries.md) — EU AI Act Article 73 serious-incident summary shape; SR 11-7-shaped FI MRM incident memo; FDA MDR (21 CFR Part 803) for SaMD incidents; HIPAA breach notification and GDPR Article 33 concurrency; the "one incident, three regulator-facing packages" extract pattern.
6. [`06-the-incident-loop-and-boundaries.md`](06-the-incident-loop-and-boundaries.md) — the incident-response loop end-to-end; the SecOps / DFIR boundary that this role does *not* cross; the level-40 escalation to `agentic-safety-engineer` for frontier-agent incidents; learning propagation to the risk council (mod-112).

## Exercises

- [`exercises/exercise-01-ai-incident-triage-and-routing-drill.md`](exercises/exercise-01-ai-incident-triage-and-routing-drill.md) — drill the triage classification, severity assignment, and routing decision against a mixed queue of candidate incidents.
- [`exercises/exercise-02-trace-diff-based-root-cause-analysis.md`](exercises/exercise-02-trace-diff-based-root-cause-analysis.md) — run the "what changed since the incident began" workflow on a mock incident with all six evidence surfaces present, producing a causal-chain paragraph and an evidence pack.
- [`exercises/exercise-03-post-mortem-authoring-for-a-mock-jailbreak-incident.md`](exercises/exercise-03-post-mortem-authoring-for-a-mock-jailbreak-incident.md) — author the ten-section post-mortem for the mock jailbreak used in exercise-02 and generate the three audience-specific extracts.
- [`exercises/exercise-04-regression-fixture-back-feed-workflow.md`](exercises/exercise-04-regression-fixture-back-feed-workflow.md) — extract the pinned fixture record from the RCA, land it in both suites, and wire the register-row credit.
- [`exercises/exercise-05-regulator-facing-incident-summary-authoring.md`](exercises/exercise-05-regulator-facing-incident-summary-authoring.md) — author the EU AI Act Article 73 extract and the SR 11-7-shaped MRM incident memo from the post-mortem; run the cross-package consistency check.

## Labs & quizzes

- [`labs/`](labs/) — a longer hands-on lab is scheduled here; see the lab README for the current status.
- [`quizzes/`](quizzes/) — knowledge-check quiz for the module.

## Resources

See [`resources.md`](resources.md) for the primary sources — every reference is a link to the regulator, standards body, incident-corpus maintainer, government AI safety institute, or original publication of the engineering practice.

## How to work through the module

1. Read chapters 01-06 in order. Each is short and pins one stage or one boundary of the incident-response loop.
2. Pick a *single AI system* (real or documented) to be the running example — ideally the same running system used in mod-102 / mod-103 so the incident's harm-model and register-row cross-references are live artifacts you already have.
3. Do exercises 01-05 in order. They stack:
   - 01 drills the triage / severity / routing decision on a mixed queue,
   - 02 runs the trace-diff RCA on one incident from that queue,
   - 03 authors the post-mortem for the same incident and generates the audience extracts,
   - 04 back-feeds the fixture into both suites and wires the register credit,
   - 05 generates the regulator-facing extracts and runs the consistency check.
4. Take the quiz.

By the end of the module you should be able to run the full incident-response loop for an AI-specific incident — from signal through learning-propagation — with every stage's artifact in hand and each cross-reference (harm-model row, register row, fixture ID, regulator submission, council packet entry) live. The audit exhibit for the loop is what the module trains you to produce; there is no partial credit for a broken stage.
