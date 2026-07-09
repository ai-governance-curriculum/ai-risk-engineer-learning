# mod-111-incident-response-and-postmortems: AI-Specific Incident Response and Post-Mortem Engineering

> Scaffolded by `aicg org execute-plan`. Lecture chapters and exercise content are authored on subsequent autonomous cycles.

**Estimated effort:** 10 hours

## Learning objectives

- Triage AI-specific incidents — jailbreak / prompt-injection exploitation, model regression, biased-outcome escalation, hallucination-driven harm, unauthorised-tool-use, membership-inference exposure — and route them correctly against a general software / security incident process
- Conduct technical root-cause analysis using the trace store, model version diffs, prompt / retriever / decoding config diffs, guardrail-effectiveness deltas, and drift signals — coordinating with `ai-eval-engineer` on the app-side evidence chain and with security on the SOC-side chain
- Write a post-mortem that a governance council, a regulator, and the engineering team can each act on; feed the incident back into the risk register with an updated residual-risk claim and a corrective-action-plan
- Turn every confirmed exploit / regression into a permanent regression fixture in the red-team suite and the app-eval suite — using the AI Incident Database + OECD.AI incidents as a public compare-point
- Package the regulator-facing incident summary in the EU AI Act Article 73 shape for serious-incident reporting and in an SR 11-7 shape for FI MRM incidents

## Structure

- `01-…md` … `0N-…md`: lecture chapters.
- `exercises/`: per-exercise prompts.
- `labs/`: long-form hands-on labs.
- `quizzes/`: knowledge checks.
- `resources.md`: external references.
