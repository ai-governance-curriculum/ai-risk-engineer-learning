# ISO/IEC 42001, 23894, 42005, 25059 — Standards Your Program Plugs Into

NIST is guidance. ISO is what a certifier audits you against. If your organisation ever wants an *AI management system certificate* the way it holds an ISO/IEC 27001 information-security certificate, this is the family you engineer toward. Even if certification is not in scope, the ISO family gives you the *nouns* — control, objective, treatment, residual risk — that senior architecture and executive audiences already speak.

## Motivation

Governance analysts and auditors will hand you the standards as pdf documents. Your job is to know which one asks *what*, so that a piece of evidence you produce (a red-team report, a fairness scorecard, an incident post-mortem) can be filed against the right clause without a two-week interpretation cycle.

## Four standards, four jobs

| Standard | Published | Job |
|---|---|---|
| **ISO/IEC 42001** | 2023 | Management-system requirements for AI (the auditable, certifiable AIMS) |
| **ISO/IEC 23894** | 2023 | Risk-management *guidance* specific to AI — how ISO 31000 applies to AI systems |
| **ISO/IEC 42005** | 2025 | Guidance on AI-system impact assessment (what to include, how to structure it) |
| **ISO/IEC 25059** | 2023 | AI-specific extension of the SQuaRE quality model (ISO/IEC 25010) — quality characteristics for AI |

Two adjacent standards matter for context:

- **ISO/IEC 27005:2022** — information-security risk-management guidance. Existing enterprise risk registers are usually shaped by 27005; AI risk plugs into that shape (mod-103).
- **ISO/IEC 23053** — the reference framework for AI systems using ML; it defines the vocabulary the other standards assume.
- **ISO/IEC TR 24028** — trustworthiness of AI (informational, but the taxonomy appears in review docs).

## 42001 — the AIMS (AI Management System)

ISO/IEC 42001 defines requirements for establishing, implementing, maintaining, and continually improving an *AI Management System*. It mirrors the Annex SL high-level structure that ISO/IEC 27001 (ISMS) uses, so if you have seen the 27001 certificate shape, the layout is familiar:

- **Clauses 4-10** are the auditable management-system requirements: context, leadership, planning, support, operation, performance evaluation, improvement.
- **Annex A** is the list of *reference controls* — the AI-specific analogue of ISO/IEC 27001 Annex A.
- **Annex B** contains implementation guidance.
- **Annex C / D** contain informational objectives and considerations that pin the standard to trustworthy-AI principles.

### What a level-25 engineer produces against 42001

You do **not** author the Statement of Applicability (that is a senior-architect / governance-leadership job). You *supply* the evidence that the reference controls actually work. That evidence looks like:

- Control-implementation records — a config, a code diff, a signed report — for the Annex A controls the SoA has marked "applicable."
- Operation-clause evidence: model registry entries, evaluation runs, incident post-mortems, monitoring configurations.
- Performance-evaluation evidence: internal-audit findings closed, metrics against risk-treatment plans, MRR (management review) inputs.

A useful heuristic: every clause-9 (performance evaluation) requirement you get audited on is answered by *something a monitor or an eval produces*. Chapters 01-02 already trained the eval / monitor / evidence shape — 42001 is where those artifacts get filed.

## 23894 — AI risk-management *guidance*

ISO/IEC 23894 is not a requirements standard. It is guidance on how to apply the general ISO 31000 risk-management principles to an AI system. It gives you:

- an AI-specific view of the risk-management process — *scope*, *risk identification*, *risk analysis*, *risk evaluation*, *risk treatment*, *communication*, *monitoring and review*,
- **Annex A** — a catalogue of AI-related *risk sources* (bias, opacity, complexity, autonomy, machine-learning-specific issues, environmental impact, security, safety, privacy, and more),
- **Annex B** — a mapping from those risk sources onto ISO/IEC 23053 lifecycle stages.

### What a level-25 engineer produces against 23894

- A risk register (mod-103) whose *risk sources* column is drawn from Annex A. Any risk you register that doesn't map to an Annex A source is a candidate for a "we defined our own; here's why."
- A lifecycle-stage matrix (Annex B shape) that shows which risks apply at data acquisition, model training, deployment, and monitoring.
- A residual-risk statement per risk after your controls are in place. 23894 explicitly asks for residual-risk documentation, not just inherent risk.

You will meet 23894's terminology again in chapter 04 when EU AI Act Article 9 asks for a risk-management system: the harmonised standard family that supports Article 9 draws on 23894's vocabulary.

## 42005 — AI system impact assessment

ISO/IEC 42005 (published 2025) is guidance on how to conduct an *AI system impact assessment*. It is the ISO analogue of the impact-assessment expectations you see in the EU AI Act (see chapter 04) and in the Colorado AI Act (chapter 05).

It structures the impact assessment around:

- purpose and context of the AI system,
- affected individuals and groups,
- potential impacts (positive and negative) across the lifecycle,
- assessment approach and evidence,
- mitigation and monitoring,
- documentation and communication.

### What a level-25 engineer produces against 42005

- The *evidence sections* of an impact assessment: model card, dataset card, eval results, red-team results, fairness slice, privacy-risk report, guardrail configuration.
- A trace from every impact statement in the impact assessment back to a piece of engineering evidence. If a statement in the impact assessment reads "the system does not disparately impact protected groups," there is a signed Fairlearn / AIF360 report behind it.

The impact assessment is *authored* by governance (level 15 does the first draft; level 25 fills in the engineering sections; level 35 signs and archives it).

## 25059 — AI quality model

ISO/IEC 25059 extends the SQuaRE (ISO/IEC 25010) product-quality model with AI-specific quality characteristics. The characteristics it adds and refines include:

- functional adaptability,
- functional correctness,
- user controllability,
- transparency,
- robustness,
- intervenability,
- fairness / non-discrimination.

You will not usually be *audited to* 25059 directly. What 25059 gives you is a *shared vocabulary* for how quality metrics ladder up into quality characteristics. When you write a model card or a release scorecard, using 25059 characteristic names ("robustness", "transparency") lets a senior architect (level 50) cross-walk your evidence into their control library without a naming-fight.

## How the four standards map to your engineering work

| You produce | 42001 uses it as | 23894 uses it as | 42005 uses it as | 25059 uses it as |
|---|---|---|---|---|
| Risk register (mod-103) | Clause 6.1 evidence of planning | The risk-process artifact itself | Impact-assessment risk section | — |
| Harm model (mod-102) | Clause 8 operational-planning input | Risk identification output | Impact-assessment context | — |
| Red-team report (mod-104) | Annex A operational control evidence | Analysis + treatment evidence | Impact-assessment robustness section | Evidence of *robustness* characteristic |
| Adversarial-ML report (mod-105) | Annex A operational control evidence | Analysis + treatment evidence | Impact-assessment robustness section | Evidence of *robustness* characteristic |
| Fairness report (mod-106) | Annex A control evidence | Bias-source treatment record | Impact-assessment fairness section | Evidence of *fairness / non-discrimination* |
| Privacy-risk report (mod-107) | Annex A control evidence | Privacy-source treatment record | Impact-assessment privacy section | — |
| Guardrail configs (mod-108) | Annex A control evidence | Treatment record | — | Evidence of *user controllability* / *intervenability* |
| Monitoring wiring (mod-109) | Clause 9 performance-evaluation evidence | Monitoring & review record | — | — |
| Post-mortem (mod-111) | Clause 10 improvement evidence | Communication and consultation record | — | — |
| Model card / dataset card (mod-112) | Clause 7.5 documented-information | Communication record | Impact-assessment documentation | Transparency-characteristic evidence |

The point of the table is that a single engineering artifact usually files against *several* standards at once. That is on purpose. You never build "the ISO evidence set" as a separate thing — you build engineering evidence that happens to file against the standards you have to report against.

## Interaction with certification and audit

- **42001 certification** is issued by an accredited certification body. You do not run the audit; you are audited.
- **The Statement of Applicability** (which Annex A controls are in scope) is drafted by governance / architecture. You provide implementation evidence for the ones marked applicable.
- **Internal audit** under clause 9.2 usually goes through you first (in your review committee, mod-112) before it escalates.

## Summary

- 42001 is the certifiable AI Management System. You supply control-implementation evidence, not the SoA.
- 23894 is AI risk-management guidance. Its Annex A is a checklist of risk sources you will draw from in the register.
- 42005 is impact-assessment guidance. You author the engineering sections.
- 25059 gives you shared vocabulary for AI quality characteristics — use its names in model cards to interoperate with senior architecture.
- Every engineering artifact you produce usually files against multiple standards at once. Exercise 03 walks through the clause-to-artifact map.
