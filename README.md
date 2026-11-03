# AI Governance · AI Risk Engineer — Learning Repository

<!-- aicg:site-banner -->
> 🎓 Part of the free, open-source **AI Career Curriculum** ecosystem — [Infrastructure](https://github.com/ai-infra-curriculum) · [ML Engineering](https://github.com/ml-engineering-curriculum) · [AI Engineering](https://github.com/ai-engineering-curriculum) · [Governance](https://github.com/ai-governance-curriculum) · [Startup](https://github.com/ai-startup-curriculum) · [Workforce](https://github.com/ai-workforce-curriculum).
<!-- /aicg:site-banner -->

<!-- aicg:sponsor -->
> 💜 **[Sponsor this curriculum](https://github.com/sponsors/ai-governance-curriculum)** — sponsorships keep the whole open-source AI Career Curriculum free and moving.
<!-- /aicg:sponsor -->

**Role level:** 25 (hands-on engineering specialist — AI Governance family)

Operationalise AI risk end-to-end as an engineer: author harm models grounded in real AI incidents, quantify inherent and residual risk, engineer LLM / agent red-team programs and adversarial-ML evaluations, ship fairness / privacy / guardrail engineering, wire monitoring into an AI risk register, apply SR 11-7 / FDA GMLP / EU AI Act Article 9 shape to regulated ML/AI, run AI-specific incident response with permanent-regression-fixture back-feed, and own a cross-functional AI risk program at team scope.

> **Status**: planned. This bootstrap packet lands the requirements catalog and the module / project plan authored from [`.aicg/curriculum-plan.json`](.aicg/curriculum-plan.json). Lessons and projects will be drafted by subsequent autonomous content cycles. Live posting evidence was deferred (WebSearch / WebFetch was not granted in the bootstrap session) — see [`JOB_REQUIREMENTS.md`](JOB_REQUIREMENTS.md) `Status`.

## What you get

Planned commitment: **170 hours across 12 modules** + **100 hours across 3 projects** = **~270 hours**.

- **Modules 101 & 102** — AI risk frameworks read as an engineer would, and harm-model authoring grounded in the AI Incident Database, OECD.AI incidents, and the MIT AI Risk Repository.
- **Module 103** — risk quantification (impact × likelihood × exposure) and populating an AI risk register in Credo AI / Holistic AI / ModelOp shape.
- **Module 104** — LLM / agent red-team engineering with garak, PyRIT, Promptfoo red-team, Inspect, HarmBench, AgentDojo, InjecAgent.
- **Module 105** — adversarial-ML engineering with ART, TextAttack, CleverHans, Counterfit mapped to NIST AI 100-2 and MITRE ATLAS.
- **Module 106** — fairness / bias / explainability engineering with Fairlearn, AIF360, Holistic AI, SHAP, Alibi, LIME cleared against NYC LL144 and EEOC adverse-impact obligations.
- **Module 107** — privacy-risk engineering with Opacus DP-SGD, ML Privacy Meter, Presidio; Shokri MIA + Carlini LiRA baselines.
- **Module 108** — guardrail engineering and effectiveness with NeMo Guardrails, Guardrails AI, Llama Guard, ShieldGemma, OpenAI Moderation, Constitutional Classifiers; vendor build-vs-buy.
- **Module 109** — continuous monitoring and risk-register integration; EU AI Act Article 72 post-market-surveillance report authoring.
- **Module 110** — SR 11-7 / OCC 2011-12 / SR 23-4 applied to ML/AI, FDA GMLP + PCCP, AI supply-chain provenance (ML-BOM, SPDX AI, SLSA, Sigstore, ModelScan, picklescan).
- **Module 111** — AI-specific incident response with post-mortem authoring and permanent-regression-fixture back-feed.
- **Module 112** — cross-functional AI risk program, governance-council briefing, model-card risk slice, Preparedness / RSP-style deployment tiering; certifications portfolio.
- **Three projects** — a harm-model + red-team package for one system; an adversarial + fairness + privacy eval bundle for one model; an end-to-end AI risk assessment for one regulated use case (US-bank SR 11-7, US-health FDA GMLP + PCCP, or EU AI Act high-risk).

## Level ladder — where this track fits

- **level 10** — [`ai-infra-junior-engineer-learning`](https://github.com/ai-infra-curriculum/ai-infra-junior-engineer-learning) (assumed engineering-craft prerequisites)
- **level 15** — [`ai-governance-analyst-learning`](https://github.com/ai-governance-curriculum/ai-governance-analyst-learning) (operational analyst work; assumed familiarity)
- **level 20** — [`ml-engineer-learning`](https://github.com/ml-engineering-curriculum/ml-engineer-learning) (ML fundamentals; assumed)
- **level 25** — **`ai-risk-engineer-learning`** (this track) — hands-on engineering of AI risk
- **level 30 (peers)** — [`ai-eval-engineer-learning`](https://github.com/ai-engineering-curriculum/ai-eval-engineer-learning) (application-layer eval), [`model-evaluation-engineer-learning`](https://github.com/ml-engineering-curriculum/model-evaluation-engineer-learning) (statistical / benchmark methodology)
- **level 35** — [`ai-evaluation-engineer-learning`](https://github.com/ai-governance-curriculum/ai-evaluation-engineer-learning) (release-assurance / audit-trail methodology), [`security-learning`](https://github.com/ai-infra-curriculum/security-learning) / [`ai-infra-security-learning`](https://github.com/ai-infra-curriculum/ai-infra-security-learning) (deep ML/AI security)
- **level 40** — [`agentic-safety-engineer-learning`](https://github.com/ai-governance-curriculum/agentic-safety-engineer-learning) (frontier-agent red-team methodology depth)
- **level 50** — [`senior-ai-governance-architect-learning`](https://github.com/ai-governance-curriculum/senior-ai-governance-architect-learning) (architecture)
- **level 60** — [`head-of-ai-governance-learning`](https://github.com/ai-governance-curriculum/head-of-ai-governance-learning) (program leadership)
- **level 70** — [`chief-ai-officer-learning`](https://github.com/ai-governance-curriculum/chief-ai-officer-learning) (executive)

See [`CURRICULUM.md`](CURRICULUM.md) for the ownership rule this track applies and [`PREREQUISITES.md`](PREREQUISITES.md) for the assumed lower-level curriculum.

## Layout

```
ai-risk-engineer-learning/
├── lessons/mod-XXX-*/        modules with lectures, exercises, labs, quizzes
├── projects/project-XXX-*/   multi-module capstones
├── CURRICULUM.md             role-level coverage map
├── PREREQUISITES.md          assumed entry skills
├── VERSIONS.md               release history
├── JOB_REQUIREMENTS.md       requirements catalog with cited authoritative references
├── .aicg/job-requirements.json    machine-readable requirements & references
├── .aicg/curriculum-plan.json     machine-readable module / exercise / project plan
└── README.md                 this file
```

## Paired Solutions Repo

[`ai-risk-engineer-solutions`](https://github.com/ai-governance-curriculum/ai-risk-engineer-solutions) carries the reference implementations.

---

<!-- aicg:maintained-by -->
Maintained by [VeriSwarm.ai](https://veriswarm.ai)
