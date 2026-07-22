# mod-109 — Continuous Monitoring, Post-Market Surveillance, and Risk-Register Integration

**Estimated effort:** 12 hours
**Position in the track:** ninth module; assumes mod-101 (frameworks & role scope), mod-102 (harm model), mod-103 (risk register), mod-104 (evals), mod-105 (adversarial ML), mod-106 (fairness & explainability), mod-107 (privacy), and mod-108 (guardrails) are done.

By mod-108 you have guardrails that fire, register rows that carry residual scores, and a harm model that names the scenarios you care about. This module builds the *wire* — the signal contract that binds every monitor to a specific register field, at a defended cadence, with a defended threshold, escalating to named owners. You then use WhyLabs (whylogs), Evidently AI, and Alibi Detect as the open-source working toolset, cite where Fiddler AI, Arthur AI, and Holistic AI fit as commercial platforms, produce an EU AI Act Article 72 post-market monitoring report generated *from* the register + monitor pipeline rather than composed by hand, and ship an alert routing where on-call, security, and the governance council all read the same source of truth.

The boundaries you write down in this module are the ones the audit reads first. The monitoring pipeline itself is owned by `ai-infra-mlops` (peer, level 25); the application-layer online evaluation loop is owned by `ai-eval-engineer` (peer, level 30). This module cites both boundaries in every affected signal contract row — it does not re-own them.

## Learning objectives

- Design the **risk-signal contract** — which monitor feeds which risk-register field, at what cadence, with what threshold, escalating to which owner; write it as a versioned artifact with sign-off, not as a dashboard.
- Wire **model / data drift, quality regression, safety-monitor signal, near-miss data, cost / latency anomalies, and jailbreak / prompt-injection detection** into the AI risk register with deterministic register-field effects and per-slice discipline.
- Use **WhyLabs (whylogs), Evidently AI, and Alibi Detect** as the open-source working toolset, and cite **Fiddler AI, Arthur AI, and Holistic AI** as commercial platforms — knowing that deep MLOps monitoring platform depth belongs to `ai-infra-mlops` (peer, L25) and application-eval online-loop depth to `ai-eval-engineer` (peer, L30).
- Author a **EU AI Act Article 72 post-market monitoring report** for a high-risk system — data logged, thresholds, incidents, remediation, corrective action plan — generated as a reduction over the register plus monitor pipeline, and dovetailed with Article 73 serious-incident reporting.
- Ship an **incident-alert routing** that reaches on-call, security, and the governance council from the same source of truth, with the incident-candidate queue as the single wire into mod-111.
- **Cite the peer boundaries** to `ai-eval-engineer` (L30) on the app-eval online loop and to `ai-infra-mlops` (L25) on the monitoring pipeline in every affected signal contract row.

## Chapters

1. [`01-the-risk-signal-contract.md`](01-the-risk-signal-contract.md) — the contract: signal-to-register-field mapping, cadence, threshold, escalation; why an ad-hoc dashboard does not count as a monitor; versioning and sign-off.
2. [`02-drift-and-quality-monitoring.md`](02-drift-and-quality-monitoring.md) — model / data drift (KS, PSI, MMD, JS divergence), quality regression, per-slice discipline; whylogs, Evidently AI, Alibi Detect as the working toolset; boundary to `ai-infra-mlops` (L25).
3. [`03-safety-and-application-signal.md`](03-safety-and-application-signal.md) — guardrail firing rate, jailbreak / prompt-injection detection, PII leakage detection, near-miss data, cost / latency anomalies; wiring these into register fields; boundary to `ai-eval-engineer` (L30).
4. [`04-post-market-surveillance-and-article-72.md`](04-post-market-surveillance-and-article-72.md) — EU AI Act Article 61 / 72 post-market monitoring plan and report shape; dovetail with Article 73 serious-incident reporting; FDA SaMD post-market surveillance as the sector variant; the report as a reduction over the register + monitors.
5. [`05-alert-routing-and-single-source-of-truth.md`](05-alert-routing-and-single-source-of-truth.md) — routing an alert to on-call, security, and the governance council from the same source of truth; the anti-pattern of a Slack-only alert; the incident-candidate queue and the governance-council digest.
6. [`06-level-ladder-and-vendor-boundaries.md`](06-level-ladder-and-vendor-boundaries.md) — Fiddler AI, Arthur AI, WhyLabs, Holistic AI as commercial platforms vs. the open-source starting points; the mod-108 upstream wire and the mod-111 / mod-112 downstream wires; the peer boundaries at L25 (`ai-infra-mlops`) and L30 (`ai-eval-engineer`).

## Exercises

- [`exercises/exercise-01-risk-signal-contract-drill.md`](exercises/exercise-01-risk-signal-contract-drill.md) — draft the signal contract for the running system; one row per (monitor, register-field) pair with all required columns filled.
- [`exercises/exercise-02-drift-and-quality-monitoring-with-whylabs-and-evidently.md`](exercises/exercise-02-drift-and-quality-monitoring-with-whylabs-and-evidently.md) — wire whylogs and Evidently signals into the register; defend the metric, cadence, and threshold choices per row.
- [`exercises/exercise-03-safety-monitor-into-risk-register-wire.md`](exercises/exercise-03-safety-monitor-into-risk-register-wire.md) — wire mod-108 guardrail fires into the register as safety-monitor signal contract rows; include the near-miss and cost / latency families.
- [`exercises/exercise-04-eu-ai-act-article-72-report-drill.md`](exercises/exercise-04-eu-ai-act-article-72-report-drill.md) — generate a full Article 72 post-market monitoring report for a high-risk system, keyed to the plan and to signal contract identifiers.
- [`exercises/exercise-05-alert-routing-and-single-source-of-truth.md`](exercises/exercise-05-alert-routing-and-single-source-of-truth.md) — set up the routing so on-call, security, and the governance council read the same tripped-signal records; write the runbooks; walk the incident-candidate queue.

## Labs & quizzes

- [`labs/`](labs/) — a longer hands-on lab is scheduled here; see the lab README for the current status.
- [`quizzes/`](quizzes/) — knowledge-check quiz for the module.

## Resources

See [`resources.md`](resources.md) for primary sources — every reference is a standards body, regulator, official project page, or the original publication of a drift-metric statistic. Commercial vendor references point at the vendor's own product page.

## How to work through the module

1. Continue with the **same running system** used in mods 101-108. Every exercise in this module writes to and reads from the register you have been carrying since mod-103.
2. Read chapters 01-06 in order. Each is short and pins one axis of the signal contract or its downstream artifacts.
3. Do exercises 01-05 in order. They stack:
   - 01 drafts the empty contract for the system,
   - 02 fills the drift-and-quality rows using whylogs and Evidently,
   - 03 fills the safety-monitor, near-miss, and cost / latency rows using mod-108 guardrails as the upstream source,
   - 04 generates the Article 72 report as a reduction over the register + monitors,
   - 05 sets up the routing, the runbooks, and the incident-candidate queue so on-call, security, and the governance council share truth.
4. Take the quiz.

By the end of the module you should be able to answer, for any register row that depends on a runtime signal: which monitor supplies the signal, what its semantics are, what threshold trips it, what the register-side effect is, who is on call, which peer track owns the pipeline, and where the tripped-signal record lands as evidence. If any of those seven answers is missing, the row is not shippable — and the audit will find it before you do.
