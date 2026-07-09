# Frontier-Lab Risk-Tier Frameworks as Enterprise Deployment-Tiering Templates

Anthropic's Responsible Scaling Policy, OpenAI's Preparedness Framework, and Google DeepMind's Frontier Safety Framework are public documents from the frontier model developers. Each pairs *capability thresholds* with *deployment / safeguard obligations*, so that as capability crosses a threshold, safeguards escalate. You are not running a frontier lab — but the *shape* of these documents is directly reusable for enterprise deployment tiering. This chapter teaches how to lift the shape.

## Motivation

Most enterprise AI deployment programs jump straight from "the model works" to "ship it," with no formal tier between "internal proof of concept" and "customer-facing production." That creates two failure modes:

- **Tier collapse.** A demo assistant with hallucinations gets promoted to a customer-facing decision-support system with no incremental safeguard review, because there is no policy that names the tiers or their obligations.
- **Cargo-culting.** A team reads Anthropic's ASL definitions or OpenAI's Preparedness thresholds and tries to *apply the actual thresholds* — CBRN uplift, autonomous replication — to an internal RAG chatbot. Those thresholds are for frontier models; a customer-support assistant does not need to be judged against autonomous-replication triggers.

The productive path is to borrow the *shape* — capability thresholds paired with obligations, pre-committed before deployment — while writing thresholds appropriate to your system. The frontier documents are shape templates, not spec sheets. This chapter shows you how to read them that way.

## The four documents you have to read

### Anthropic Responsible Scaling Policy (RSP)

Anthropic's RSP defines a set of **AI Safety Levels (ASL)** analogous to biosafety levels, each with a capability description and a corresponding set of deployment and containment commitments. As a model's capabilities cross into an ASL, additional evaluations, safeguards, and deployment restrictions are triggered. The policy is public and versioned; you can read the current published version and the prior versions on Anthropic's site.

<!-- needs-research: cite the current published RSP version and its date; the policy is updated periodically and the specific ASL threshold language changes across versions. -->

- Key shape properties to extract:
  - **Capability thresholds** (what the model can do that requires escalation).
  - **Pre-triggered safeguards** (safeguards that must be in place *before* the threshold is crossed).
  - **Evaluation cadence** (how the policy determines whether the threshold has been crossed).
  - **Board-level accountability** (who is responsible if a threshold is crossed without safeguards).

### OpenAI Preparedness Framework

OpenAI's Preparedness Framework defines **tracked risk categories** — with cybersecurity, biological, radiological / nuclear, chemical, persuasion, and model-autonomy areas among those historically named — and thresholds (e.g., low / medium / high / critical) with pre-committed policy actions when a model crosses a threshold. The framework is versioned; the initial "beta" was released in December 2023 and subsequent updates have restructured the tracked categories.

<!-- needs-research: cite the current published version of the Preparedness Framework and its threshold structure exactly as printed; category names and threshold labels change across versions. -->

- Key shape properties to extract:
  - **Category grid** (what harm dimensions are tracked separately rather than lumped together).
  - **Threshold levels** (how the graduated response is structured).
  - **Deployment-blocking thresholds** (which levels forbid release without additional mitigation).
  - **Cross-team review** (which internal body signs off on threshold determinations).

### Google DeepMind Frontier Safety Framework (FSF)

DeepMind's FSF defines **Critical Capability Levels (CCLs)** — capability thresholds at which enhanced testing and mitigation are required — and pairs them with an evaluation and mitigation plan. The framework covers autonomy, biosecurity, cybersecurity, and machine-learning R&D categories among its tracked capability areas, and it commits to specific evaluation and safeguard practices when a CCL is approached or crossed.

<!-- needs-research: cite the current published version of the FSF, its exact CCL naming, and its tracked capability categories at the version referenced. -->

- Key shape properties to extract:
  - **Capability level definition style** (how each CCL is specified as an operational test).
  - **Evaluation vs. mitigation split** (which practices are for detection vs. which for response).
  - **Approach-vs.-crossed differentiation** (many FSF-style frameworks distinguish "we are getting close" from "we are here").

### Frontier Model Forum publications

The Frontier Model Forum (Anthropic, Google, Microsoft, OpenAI as founding members; membership evolving) publishes issue briefs, technical reports, and best-practice notes on frontier model safety. These are not risk-tier frameworks themselves; they are cross-industry synthesis materials that document areas of emerging consensus and disagreement. Reading them is how you find out where the industry currently sits on questions like "what evaluations should exist before deployment" and "how are external safety testers engaged."

<!-- needs-research: pull the specific FMF issue briefs that are the current canonical references at authoring time; the publication list grows. -->

- Key shape properties to extract:
  - **Areas of published consensus** (things that would be difficult to defend *not* doing in your program).
  - **Areas of open work** (things where reasonable people disagree, so your program has room to make its own call).
  - **Terminology** (adopt the industry terms rather than inventing your own).

## The comparative read — six questions to ask each document

When you read the three risk-tier frameworks (RSP, Preparedness, FSF) side by side, use six comparison questions rather than reading top-to-bottom in each:

1. **What capability categories does it track?** (For example: cybersecurity, CBRN, autonomy, persuasion, R&D acceleration.)
2. **How are the thresholds defined?** (Level names, operationalisation, pass/fail criteria.)
3. **What safeguards attach at each threshold?** (Pre-triggered vs. reactive.)
4. **Who decides that a threshold has been crossed?** (Internal evaluation team, external auditor, board committee.)
5. **What is the deployment consequence of crossing a threshold?** (Halt, delay, additional evaluations, restricted access.)
6. **How is the framework itself updated?** (Cadence, who reviews changes, transparency commitments.)

Six-column table with one row per framework is exactly the artifact exercise 05 asks you to produce.

## Lifting the shape to your enterprise deployment tiering

Your enterprise system will not have frontier capability triggers. Your triggers will be things like "the system will render advice as final answer to a customer without a human in the loop," or "the system will make credit / hiring / benefits decisions," or "the system will use autonomous multi-step tool calling to complete workflows." But the *shape* of the frontier-lab framework applies:

- Enumerate **capability tiers** for the system class (POC, internal-limited, limited-external, general-external, high-stakes-external).
- For each tier, name the **pre-triggered safeguards** — the harm-model rows that must be closed, the guardrails that must be deployed, the monitors that must be live, the human-oversight design that must be in place.
- For each tier, name the **decision authority** — who signs off on the promotion from tier to tier.
- For each tier, name the **evaluation cadence** and **eval scope** required to keep the tier.
- Pre-commit the tiering *before* a specific model / deployment is up for promotion. Post-hoc tier definition is a red flag.

A skeleton enterprise deployment-tiering table for the customer-support RAG assistant example:

| Tier | System state | Pre-triggered safeguards | Decision authority | Evaluation cadence |
|---|---|---|---|---|
| T0 — POC | Internal engineering only | Trace store enabled; abuse contact set | Team lead | On demand |
| T1 — internal limited | Internal users, non-decision use | Guardrail baseline; incident-response playbook; red-team suite pilot | Engineering + risk lead | Per release |
| T2 — limited external | Small external cohort, human-in-the-loop | Full harm-model row closure; guardrail effectiveness above threshold; monitor spec live; opt-out available | Risk council | Per release + weekly monitor review |
| T3 — general external | General customer availability, human-in-the-loop | External red-team engagement; regulator-facing evidence packet complete; post-market surveillance plan | Risk council + release-assurance sign-off (`ai-evaluation-engineer` L35) | Per release + monthly programme review |
| T4 — high-stakes | Decisions or actions with material user impact | External audit; fairness / privacy / adversarial-ML deep evaluations; SR 11-7 / EU AI Act obligations mapped | Head of AI governance + legal + business owner | Continuous with quarterly deep-dive |

The point is not the exact tiers or thresholds; the point is that the *shape* — pre-committed tiers, pre-triggered safeguards, named authorities, named cadence — is the reusable pattern. You are borrowing rigour, not thresholds.

<!-- needs-research: the tier / safeguard schematic above is illustrative and should be adapted per organisation and per sector (financial services, healthcare, public sector) with reference to the applicable regulatory obligations; the specific safeguard-per-tier language should not be lifted verbatim into any real program without local counsel. -->

## Where you *don't* copy from the frontier docs

- Frontier capability thresholds (CBRN uplift measurements, autonomous replication trigger criteria) belong to the frontier developer. If you are in scope for such thresholds, you are working at the frontier and the escalation is to `agentic-safety-engineer` (level 40) or the frontier developer itself (mod-102 ch 06).
- Specific evaluation suites named in the frontier documents (some are non-public; some depend on frontier-developer infrastructure). Adopt the shape, not the internal harness.
- Public commitments to external auditors and to governments. Those are the developer's commitments; your program makes its own.

Being explicit about what you don't copy is what makes the shape lift defensible rather than cargo-culted.

## Alignment with mod-101 chapter 04 (EU AI Act)

The EU AI Act Article 55 covers general-purpose AI models with systemic risk and imposes risk-assessment, mitigation, and reporting obligations on their providers. The frontier-lab documents largely map to Article 55 concerns at the *provider* side. Your enterprise tiering is at the *deployer* / *downstream provider* side (Articles 26 and, in some cases, 25). Reading the frontier documents helps you *anticipate* what a general-purpose model you consume is already required to deliver, and thus what you can *demand as input* to your own harm model.

Mod-101 chapter 04 covered the article-by-article deliverables; this chapter's job is to note the connection point.

## Alignment with mod-101 chapter 05 (sector flavours) and mod-110 (MRM)

For a US regulated financial-services shop, the enterprise deployment-tiering table above must be reconciled with SR 11-7 / OCC 2011-12 / SR 23-4 obligations — model-risk tiering by materiality, third-party risk lifecycle management. For an FDA-regulated health SaMD, GMLP and PCCP obligations apply. You are not choosing between frontier-lab shape and regulatory shape — you are *composing* them: use the frontier-lab shape as an internal engineering pattern; overlay sector obligations as external constraints; and let the register (mod-103) be the reconciled ledger.

Mod-110 will pick up MRM + supply-chain in depth.

## What good looks like at the end of this chapter

A six-column comparative read of RSP + Preparedness + FSF (exercise 05), plus a first-draft enterprise deployment-tiering table for your running system that names the tiers, the pre-triggered safeguards, the decision authority, and the evaluation cadence. Signed off by the risk council before any specific system is being tiered — because *pre-commitment is the point*.

## Summary

- Anthropic RSP (ASL), OpenAI Preparedness (level grid), and Google DeepMind FSF (CCLs) are shape templates: capability thresholds paired with pre-committed safeguards.
- Lift the shape, not the thresholds — your enterprise triggers are advisory-vs.-decision, human-in-the-loop-vs.-autonomous, internal-vs.-external, not CBRN uplift.
- The comparative read uses six columns: categories tracked, threshold definition, attached safeguards, decision authority, deployment consequence, framework update mechanism.
- The Frontier Model Forum publications are cross-industry synthesis; adopt the vocabulary and consensus, be explicit where consensus is absent.
- Frontier-lab shape composes with EU AI Act Article 55 (upstream) and sector obligations (SR 11-7 / FDA GMLP + PCCP; mod-110 territory).
- Exercise 05 walks the comparative read and the first-draft enterprise tiering.
