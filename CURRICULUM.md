# AI Risk Engineer Curriculum

**Role level:** 25 (hands-on engineering specialist — AI Governance family)
**Family:** AI Governance
**Status:** planned — modules and projects below are the planned scope authored from [`.aicg/curriculum-plan.json`](.aicg/curriculum-plan.json). Lessons and projects will be drafted by subsequent autonomous content cycles.

## Overview

This track teaches the *engineering craft* of AI risk end-to-end for a hands-on specialist at level 25: reading AI risk frameworks (NIST AI RMF + GenAI Profile, ISO/IEC 23894, ISO/IEC 42001, ISO/IEC 42005, EU AI Act, SR 11-7, FDA GMLP/PCCP, Colorado AI Act) as an engineer would → authoring harm models grounded in the AI Incident Database, OECD.AI incidents, and the MIT AI Risk Repository → quantifying inherent, residual, and control-defeated risk on defensible scoring schemes → engineering LLM and agent red-team programs (garak, PyRIT, Promptfoo red-team, HarmBench, AgentDojo, InjecAgent, Inspect) → adversarial-ML engineering (ART, TextAttack, CleverHans, Counterfit) mapped to NIST AI 100-2 and MITRE ATLAS → fairness / bias / explainability engineering (Fairlearn, AIF360, Holistic AI, SHAP, Alibi, LIME) that clears NYC LL144, EEOC adverse-impact, and BABL AI audit shapes → privacy-risk engineering (MIA / LiRA, DP-SGD with Opacus, ML Privacy Meter, Presidio) → guardrail engineering and effectiveness measurement (NeMo Guardrails, Guardrails AI, Llama Guard, ShieldGemma, OpenAI Moderation, Constitutional Classifiers, Lakera / Arthur / Robust Intelligence / Calypso AI / HiddenLayer) → wiring drift and safety-monitor signal into the AI risk register and producing EU AI Act Article 72 post-market-surveillance reports → applying SR 11-7 / OCC 2011-12 / SR 23-4 to ML/AI at regulated FIs (and FDA GMLP + PCCP at health systems) → engineering AI supply-chain provenance (ML-BOM, SPDX AI, SLSA, Sigstore, ModelScan, picklescan) → running AI-specific incident response with permanent-regression-fixture back-feed → owning a cross-functional AI risk program at team scope with governance-council briefings and a risk-engineering slice of model / system / dataset cards.

It is a **hands-on engineering specialist track at level 25**, positioned in the **AI Governance family**, between the AI Governance Analyst (level 15, operational analyst work — reads, tracks, files) and the AI Evaluation Engineer / Senior AI Governance Architect (levels 35+, release-assurance and architecture). The differentiator against the peer `ai-eval-engineer-learning` (AI Engineering family, level 30 — application-layer evaluation depth) and against `model-evaluation-engineer-learning` (ML Engineering family, level 30 — statistical / benchmark methodology depth) is *risk-engineering ownership*: this curriculum builds the harm model, generates the red-team data, engineers the adversarial-ML / fairness / privacy / guardrail evaluations, quantifies risk, and turns governance policy into testable code + evidence.

Total planned commitment: **170 hours across 12 modules** + **100 hours across 3 projects** = **~270 hours**.

## Ownership rule

Following the project-wide ownership rule, this curriculum:

- **Owns** the engineering craft of AI risk end-to-end — engineering-grade fluency with AI risk frameworks, harm-model authoring, risk quantification and register engineering, LLM / agent red-team engineering, adversarial-ML engineering, fairness / bias / explainability engineering, privacy-risk engineering, guardrail engineering and effectiveness measurement, monitoring-to-risk-register integration, MRM and AI supply-chain assurance, AI-specific incident response, and a cross-functional AI risk program at team scope.
- **Defers down** to [`ai-governance-analyst-learning`](https://github.com/ai-governance-curriculum/ai-governance-analyst-learning) (level 15) for the operational analyst legwork this role consumes — AI-system intake and inventory, jurisdictional regulatory tracking, first-draft impact-assessment authoring, framework-crosswalk legwork, reading eval / audit evidence into governance records; to [`ml-engineer-learning`](https://github.com/ml-engineering-curriculum/ml-engineer-learning) (level 20) for classical ML / PyTorch / sklearn / MLflow / packaging fundamentals; to [`ai-infra-junior-engineer-learning`](https://github.com/ai-infra-curriculum/ai-infra-junior-engineer-learning) (level 10) for engineering-craft prerequisites (Linux, Git, Python, packaging, HTTP APIs).
- **Defers sideways** to [`ai-eval-engineer-learning`](https://github.com/ai-engineering-curriculum/ai-eval-engineer-learning) (peer, level 30, AI Engineering family) for application-layer LLM/agent evaluation-engineering depth (traces, judges, RAG eval, eval-gated CI/CD, online eval, cost/latency/quality); to [`model-evaluation-engineer-learning`](https://github.com/ml-engineering-curriculum/model-evaluation-engineer-learning) (peer, level 30, ML Engineering family) for benchmark, statistical, and judge-vs-human calibration methodology depth; to [`ai-infra-mlops-learning`](https://github.com/ai-infra-curriculum/ai-infra-mlops-learning) (peer, level 25, Infra family) for MLOps automation, model registry, and CI/CD-for-ML depth; to peer AI Engineering tracks [`llm-application-developer-learning`](https://github.com/ai-engineering-curriculum/llm-application-developer-learning), [`rag-engineer-learning`](https://github.com/ml-engineering-curriculum/rag-engineer-learning), [`applied-ai-engineer-learning`](https://github.com/ai-engineering-curriculum/applied-ai-engineer-learning), [`fine-tuning-engineer-learning`](https://github.com/ml-engineering-curriculum/fine-tuning-engineer-learning), and [`training-pipeline-engineer-learning`](https://github.com/ml-engineering-curriculum/training-pipeline-engineer-learning) for the systems being assessed for risk.
- **Defers up** to [`ai-evaluation-engineer-learning`](https://github.com/ai-governance-curriculum/ai-evaluation-engineer-learning) (peer / next-up, level 35, Governance family) for release-assurance / audit-trail / regulator-facing methodology depth — this role produces the risk-engineering evidence that peer packages into release-assurance reviews; to [`security-learning`](https://github.com/ai-infra-curriculum/security-learning) and [`ai-infra-security-learning`](https://github.com/ai-infra-curriculum/ai-infra-security-learning) (level 35) for deep ML/AI security at platform scale (model-extraction defence at inference edge, judge supply-chain, eval-set exfiltration prevention, MLSecOps); to [`agentic-safety-engineer-learning`](https://github.com/ai-governance-curriculum/agentic-safety-engineer-learning) (level 40) for frontier-agent red-team methodology and dangerous-capability evaluation depth; to [`senior-ai-governance-architect-learning`](https://github.com/ai-governance-curriculum/senior-ai-governance-architect-learning) (level 50) for control-library architecture, policy taxonomy, and cross-jurisdiction reconciliation; to [`head-of-ai-governance-learning`](https://github.com/ai-governance-curriculum/head-of-ai-governance-learning) (level 60) and [`chief-ai-officer-learning`](https://github.com/ai-governance-curriculum/chief-ai-officer-learning) (level 70) for program leadership, board-level reporting, and regulator engagement.
- **Out of scope** to legal counsel (this role does not deliver legal opinion) and to security operations / DFIR (SOC-side incident handling; this role feeds AI-specific signal to SecOps).

See [`JOB_REQUIREMENTS.md`](JOB_REQUIREMENTS.md) for the requirements-to-coverage map and the cited public references the catalog is grounded in.

## Module Plan

| Module | Title | Hours | Status |
|---|---|---|---|
| mod-101-ai-risk-frameworks-and-position | AI Risk Frameworks and the Risk-Engineer Position on the Governance Ladder | 12 | planned |
| mod-102-harm-modelling-and-risk-taxonomy | Harm Modelling and Risk Taxonomy Engineering | 14 | planned |
| mod-103-risk-quantification-and-registers | Risk Quantification, Scoring, and Enterprise AI Risk Registers | 12 | planned |
| mod-104-llm-red-teaming-and-attack-engineering | LLM and Agent Red-Team Engineering | 18 | planned |
| mod-105-adversarial-ml-and-robustness | Adversarial ML and Robustness Engineering | 16 | planned |
| mod-106-fairness-bias-and-explainability-engineering | Fairness, Bias, and Explainability Engineering | 16 | planned |
| mod-107-privacy-risk-engineering | Privacy-Risk Engineering for ML and LLM Systems | 14 | planned |
| mod-108-guardrail-engineering-and-effectiveness | Guardrail Engineering and Effectiveness Measurement | 16 | planned |
| mod-109-continuous-monitoring-and-risk-register-integration | Continuous Monitoring, Post-Market Surveillance, and Risk-Register Integration | 12 | planned |
| mod-110-model-risk-management-and-supply-chain | Model Risk Management (MRM) for ML/AI and AI Supply Chain | 16 | planned |
| mod-111-incident-response-and-postmortems | AI-Specific Incident Response and Post-Mortem Engineering | 10 | planned |
| mod-112-cross-functional-ai-risk-program-and-communication | Cross-Functional AI Risk Program and Communication | 14 | planned |

## Project Plan

| Project | Title | Hours | Status |
|---|---|---|---|
| project-101-harm-model-and-red-team-package-for-one-system | Harm Model + Red-Team Package for One AI System | 30 | planned |
| project-102-adversarial-fairness-and-privacy-eval-bundle | Adversarial + Fairness + Privacy Eval Bundle for One Model | 30 | planned |
| project-103-end-to-end-ai-risk-assessment-for-one-regulated-use-case | End-to-End AI Risk Assessment for One Regulated Use Case | 40 | planned |

## Module summaries

### mod-101 — AI Risk Frameworks and the Risk-Engineer Position
Read NIST AI RMF (with Playbook), NIST GenAI Profile, ISO/IEC 42001 / 23894 / 42005 / 25059, and EU AI Act Articles 9-15, 55, 61, 72 as an engineer — mapping each obligation to concrete tests, thresholds, guardrails, monitors, and evidence artifacts. Position SR 11-7 / OCC 2011-12 / SR 23-4, FDA GMLP + PCCP, and the Colorado AI Act as sector / geography flavours of the same shape. Diagram this role's exact scope on the level ladder — what `ai-governance-analyst` (level 15) already owns and what `ai-evaluation-engineer` (level 35), `agentic-safety-engineer` (level 40), `senior-ai-governance-architect` (level 50), and `head-of-ai-governance` (level 60) inherit or extend.

### mod-102 — Harm Modelling and Risk Taxonomy Engineering
Author a harm model for a concrete AI system — misuse personas (adversarial user, insider, upstream data supplier, downstream integrator), capability surfaces, immediate / secondary / cascading harms — grounded in the AI Incident Database, OECD.AI incidents, and the MIT AI Risk Repository. Layer the NIST GenAI Profile, ISO/IEC 23894 risk sources, AIRO, and AIR-Bench 2024 policy-derived categories. Read Anthropic RSP, OpenAI Preparedness, and Google DeepMind Frontier Safety Framework as shape templates for enterprise deployment tiering.

### mod-103 — Risk Quantification, Scoring, and Enterprise AI Risk Registers
Design a defensible scoring scheme (impact × likelihood × exposure) for AI risk with inherent / residual / control-defeated distinctions, calibration to enterprise risk appetite, and portfolio aggregation. Align with ISO/IEC 27005 so AI risk plugs into ERM without a schema fight. Populate an AI risk register in a GRC-for-AI platform (Credo AI, Holistic AI, ModelOp, Monitaur) and the spreadsheet-shaped fallback. Deep statistical calibration methodology is linked out to `model-evaluation-engineer` (peer, level 30).

### mod-104 — LLM and Agent Red-Team Engineering
Author production-grade LLM / agent red-team programs with garak, PyRIT, Promptfoo red-team, and Inspect (UK AISI). Engineer jailbreak evaluations across direct and indirect prompt injection, encoding / obfuscation, multi-turn refusal-bypass, role-play / persona-hijack, GCG-style adversarial suffixes, and many-shot in-context; engineer tool-abuse evaluations with AgentDojo / InjecAgent. Adopt HarmBench, AIR-Bench 2024, and a policy-derived internal harm-eval suite as reference eval-sets; keep harmful payloads out of the source repo. Coordinate with `ai-eval-engineer` (peer, level 30) on the app-side safety measurement contract; defer frontier-agent depth to `agentic-safety-engineer` (level 40).

### mod-105 — Adversarial ML and Robustness Engineering
Read NIST AI 100-2 (e2025) as the working taxonomy — evasion, poisoning, extraction, inversion, membership inference, availability — mapped to MITRE ATLAS and OWASP ML Top 10. Engineer evasion attacks with ART / TextAttack (CleverHans as classical compare-point); engineer data / label poisoning with detection baselines (Spectre, activation clustering, STRIP); engineer extraction / inversion attacks well enough to defend the deployed API with rate control, DP training, and output-perturbation. Ship a robustness-eval report with residual-risk delta. Deep platform-scale defence at inference edge is linked to `security-learning` / `ai-infra-security` (level 35).

### mod-106 — Fairness, Bias, and Explainability Engineering
Ship group-fairness evaluations (demographic parity, equalised odds, equal opportunity, calibration by group, disparate impact / four-fifths, predictive parity) with Fairlearn / AIF360; know the mathematical incompatibilities. Ship individual-fairness and causal-fairness diagnostics where the family fits. Ship mitigation loops with the accuracy / fairness trade-off documented. Author explainability slices with SHAP + Alibi + LIME. Deliver NYC Local Law 144 shape bias-audit reports and EEOC adverse-impact evaluations comfortable running the ForHumanity / BABL AI credential shape.

### mod-107 — Privacy-Risk Engineering
Engineer membership-inference evaluations at production quality (Shokri MIA, Carlini LiRA); engineer attribute-inference / model-inversion / memorisation evaluations; engineer DP-SGD training with Opacus and TensorFlow Privacy — defending (ε, δ) / noise multiplier / clipping choices; ship PII detection / redaction pipelines with Presidio integrated into ingest, retrieval, and output paths. Author a privacy-risk report mapped to GDPR Article 22 and the NIST Privacy Framework; route legal opinion out of scope to counsel.

### mod-108 — Guardrail Engineering and Effectiveness
Design an input / output guardrail architecture for an LLM application (input filters, output validators, tool-invocation policy, safe-completion). Ship code with NeMo Guardrails, Guardrails AI, a custom drop-in layer, and classifier-based guardrails (Llama Guard, ShieldGemma, OpenAI Moderation, Constitutional Classifiers). Measure guardrail effectiveness with FP / FN reports and cost / latency accounting. Evaluate third-party vendors (Lakera, Arthur, Robust Intelligence, Calypso AI, HiddenLayer, Protect AI) with a build-vs-buy matrix. Feed the app-side measurement contract to `ai-eval-engineer` (peer, level 30).

### mod-109 — Continuous Monitoring and Risk-Register Integration
Design the risk-signal contract — which monitor feeds which risk-register field, at what cadence, with what threshold. Wire drift, quality regression, safety-monitor signal, near-miss data, cost / latency anomalies, and jailbreak / prompt-injection detection into the AI risk register using WhyLabs (whylogs), Evidently AI, Alibi Detect, Fiddler AI, Arthur AI, and Holistic AI. Author EU AI Act Article 72 post-market-surveillance reports. Defer deep MLOps monitoring platform depth to `ai-infra-mlops` (peer, level 25) and application-eval online-loop depth to `ai-eval-engineer` (peer, level 30).

### mod-110 — Model Risk Management and AI Supply Chain
Apply SR 11-7 / OCC 2011-12 / SR 23-4 to ML/AI at a regulated FI — validation-vs-development independence, conceptual soundness, ongoing monitoring, outcome analysis, effective challenge. Apply FDA GMLP + PCCP where the AI is a medical device. Engineer AI supply-chain provenance (Hugging Face model cards, ML-BOM with CycloneDX, SPDX 3.0 AI profile, SLSA levels, Sigstore weights signing); detect malicious model artifacts (Protect AI ModelScan, picklescan) and gate model-registry ingestion.

### mod-111 — AI-Specific Incident Response
Triage AI-specific incidents (jailbreak / prompt-injection exploitation, model regression, biased-outcome escalation, hallucination-driven harm, unauthorised-tool-use, MIA exposure) and route them against the general software / security incident process. Conduct technical root cause using the trace store, version diffs, config diffs, guardrail-effectiveness deltas, and drift signals. Write post-mortems that a council, a regulator, and engineering can act on. Turn every confirmed exploit / regression into a permanent regression fixture. Package EU AI Act Article 73 serious-incident reports and SR 11-7-shape FI MRM incidents.

### mod-112 — Cross-Functional AI Risk Program and Communication
Own an AI risk program at team scope across product / ML / infra / security / governance / legal — cadence, artifacts, decisions, sign-off routing. Brief the AI governance council with pre-read decks, one-pagers, and decision-log templates. Author the risk-engineering slice of model / system / dataset cards (Hugging Face model-card format or internal template). Contribute to Preparedness / RSP-style deployment tiering. Reconcile evidence to NIST AI RMF MEASURE / MANAGE and ISO/IEC 42001 controls; hand release-assurance work to `ai-evaluation-engineer` (peer / next-up, level 35) and architecture to `senior-ai-governance-architect` (level 50). Position IAPP AIGP, ForHumanity, BABL AI, ISO/IEC 42001 LI/LA, and IAPP CIPP as the credentials portfolio.

## Assessment

Each module ships **1 quiz** plus the exercises and a lab listed in [`.aicg/curriculum-plan.json`](.aicg/curriculum-plan.json). Each project ships a portfolio-grade README, a reproducibility bundle (risk-eval configs, red-team suites hosted per the harmful-payload workflow taught in mod-104, dataset / model / prompt / decoding / seed hashes, guardrail configs, monitoring-to-register mapping), and an explicit rubric covering harm-model completeness, risk-quantification defensibility, red-team-suite quality, adversarial / fairness / privacy coverage, guardrail-effectiveness reporting, monitoring-into-register wiring, MRM / supply-chain evidence, incident-response artifacts, and cross-audience communication.

## Where to go after this curriculum

- **`ai-evaluation-engineer-learning`** (peer / next-up, level 35, Governance family) — release-assurance / audit-trail / regulator-facing methodology; consumes this curriculum's risk-engineering evidence.
- **`security-learning`** / **`ai-infra-security-learning`** (level 35) — deep ML/AI security at platform scale.
- **`agentic-safety-engineer-learning`** (level 40) — frontier-agent red-team methodology and dangerous-capability evaluation depth.
- **`senior-ai-governance-architect-learning`** (level 50) — architectural scope on the Governance ladder; owns control-library architecture, policy taxonomy, cross-jurisdiction reconciliation.
- **`head-of-ai-governance-learning`** (level 60) — leadership scope; owns program leadership, board reporting, regulator engagement.
- **`chief-ai-officer-learning`** (level 70) — executive scope.
- **`ai-eval-engineer-learning`** (peer, level 30, AI Engineering family) — natural cross-training for the application-layer evaluation-engineering peer.
- **`model-evaluation-engineer-learning`** (peer, level 30, ML Engineering family) — natural cross-training for statistical / benchmark methodology depth.

<!-- needs-research: backfill industry-frequency evidence into JOB_REQUIREMENTS.md once the autonomous research loop runs with WebSearch / WebFetch exercised against live job boards; demote any module or exercise whose underlying requirement does not show up in ≥3 in-window postings. -->
