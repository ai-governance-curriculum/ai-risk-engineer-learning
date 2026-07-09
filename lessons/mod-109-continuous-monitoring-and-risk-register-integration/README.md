# mod-109-continuous-monitoring-and-risk-register-integration: Continuous Monitoring, Post-Market Surveillance, and Risk-Register Integration

> Scaffolded by `aicg org execute-plan`. Lecture chapters and exercise content are authored on subsequent autonomous cycles.

**Estimated effort:** 12 hours

## Learning objectives

- Design the risk-signal contract — which monitor feeds which risk-register field, at what cadence, with what threshold, escalating to which owner
- Wire model / data drift, quality regression, safety-monitor signal, near-miss data, cost / latency anomalies, and jailbreak / prompt-injection detection into the AI risk register
- Use WhyLabs (whylogs), Evidently AI, Alibi Detect, Fiddler AI, Arthur AI, and Holistic AI as the working toolset — knowing that deep MLOps monitoring platform depth belongs to `ai-infra-mlops` (peer, level 25) and application-eval online-loop depth belongs to `ai-eval-engineer` (peer, level 30)
- Author a EU AI Act Article 72 post-market-surveillance report for a high-risk system — data logged, thresholds, incidents, remediation, corrective action plan
- Ship an incident-alert routing that reaches the on-call engineer, the security team, and the governance council with the same source of truth
- Cite the boundary to `ai-eval-engineer` (peer, level 30) on the app-eval online-loop and to `ai-infra-mlops` (peer, level 25) on the monitoring pipeline itself

## Structure

- `01-…md` … `0N-…md`: lecture chapters.
- `exercises/`: per-exercise prompts.
- `labs/`: long-form hands-on labs.
- `quizzes/`: knowledge checks.
- `resources.md`: external references.
