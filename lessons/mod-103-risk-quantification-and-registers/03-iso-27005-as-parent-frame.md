# ISO/IEC 27005 as the Parent Frame: Plugging AI Risk into the Enterprise ERM

AI risk is not a green-field practice inside most organisations that already run one. It is a new *risk domain* inside an ERM function that already has a shape, a vocabulary, and an audited process. That shape almost always comes from **ISO/IEC 27005** — the information-security risk-management guidance that sits inside the ISO 27000 family — because the org already runs 27001-adjacent risk management. If the AI risk register you produce cannot be *read* by a 27005-trained analyst without translation, you will lose a schema fight you did not need to enter.

This chapter says: reuse the 27005 shape. Show what the ISO/IEC 23894 (AI-specific) additions look like against it. Land the AI risk register as one more source of rows into the enterprise ERM, not a parallel practice.

## Motivation

Three patterns force AI risk toward compatibility with 27005:

- **Auditors already know 27005.** ISO 27001-certified organisations run internal and external audits against a risk-management process shaped by 27005. Introducing a differently-shaped AI risk process doubles the audit burden and confuses evidence trails.
- **ERM committees already own risk-appetite language.** The enterprise risk-appetite statement is anchored to the ERM's ordinal scale. If your AI register uses a different scale, portfolio roll-ups need a translation matrix, and the risk council reads two heat maps side-by-side.
- **Integration and supply-chain risk are already 27005-shaped.** AI-system supply-chain risks (mod-110) — third-party model providers, dataset vendors, retrieval components — will already be represented in the security-side 27005 register. Duplicating them in a separate AI register wastes work.

Reusing 27005 does not mean AI risk is a subset of information-security risk. It means the *process shape* — establish context, identify, analyse, evaluate, treat, monitor / review, communicate / consult — is inherited and *the content* is AI-specific per ISO/IEC 23894 (AI risk-management guidance) and per the mod-102 taxonomies.

## The ISO/IEC 27005 process shape (as a reused parent)

ISO/IEC 27005:2022 (Information security, cybersecurity and privacy protection — Guidance on managing information security risks) specialises ISO 31000 for information-security risk. Its process is a repeatable loop:

<!-- needs-research: verify against ISO/IEC 27005:2022 (or the current published edition) the exact clause headings for each step below; the shape given here reflects the general 27005 process but is not a direct quotation. -->

| ISO/IEC 27005 process step | What the step establishes | The AI-risk register's slot into this step |
|---|---|---|
| Establish context | Scope, criteria (risk-acceptance criteria, evaluation criteria, impact criteria), organisation, stakeholders | Which AI systems are in scope; risk-appetite bands; register rubric versions |
| Risk identification | Enumerate risk scenarios, assets, threats, vulnerabilities, consequences | Import mod-102 harm-model rows; run the persona × capability-surface enumeration |
| Risk analysis | Analyse likelihood and consequences of each scenario, existing controls, level of risk | Score inherent / residual / control-defeated per chapter 01; use the chapter 02 axes |
| Risk evaluation | Compare analysed risk against acceptance criteria; prioritise | Compare residual against appetite bands; produce treatment queue |
| Risk treatment | Select treatment options (modify, retain, avoid, share); produce a plan | Assign guardrails (mod-108), monitors (mod-109), and mitigation owners |
| Monitoring and review | Track control performance; re-evaluate on change | Feed monitor signal back into the register (mod-109); trigger re-scoring on system changes |
| Communication and consultation | Stakeholder engagement, sign-off routing, reporting | Cadence, ownership, sign-off routing (chapter 06); risk-council reporting |

Every AI-risk activity in this module can be filed against one of the seven boxes above. If you produce an artifact that does not slot cleanly, you have either (a) invented an AI-specific step the parent frame doesn't need or (b) mis-named the artifact. Either is fixable in review.

## What ISO/IEC 23894 adds without breaking the shape

**ISO/IEC 23894:2023 — Information technology — Artificial intelligence — Guidance on risk management** is the AI-specific guidance. It builds on ISO 31000 (the same general risk-management standard that 27005 specialises) and adds AI-specific content:

- AI-specific *risk sources* (data, model, use, human oversight, lifecycle, broader environment — see mod-102 ch 03).
- AI-specific *consequences* (bias, over-reliance, drift, discrimination, opacity).
- AI-specific *lifecycle stages* (data acquisition, model training, deployment, monitoring, decommissioning) that shape when a risk is identified.
- Annexes with worked examples showing how ISO 31000 activities map to AI systems.

Because both 23894 and 27005 specialise 31000, they *share the process shape* — the differences are in the *content* (AI-specific vs. infosec-specific). The pattern this module recommends: run the 27005-shaped process with 23894-shaped content. The AI risk register lives in the same schema as the security risk register, with fields filled from 23894 sources.

<!-- needs-research: confirm against ISO/IEC 23894:2023 that Annex A / Annex B directly map risk-management activities to ISO 31000 clauses; adjust the phrasing above once verified via the standard text. -->

## The schema you actually publish

A register schema that plugs into 27005 has the following fields as a floor. This is the schema mod-103 asks you to produce; chapter 06 shows how it lands in Credo AI, Holistic AI, ModelOp, Monitaur, or the spreadsheet fallback.

```yaml
row_id: HR-CS-INJ-01           # ties to mod-102 harm-model row
system_id: cs-rag-assistant-01
system_inventory_ref: inv-2025-014

# Establish-context fields
rubric_version: v1.3-cs-rag-2026-q2
appetite_band: enterprise-appetite-v2

# Risk-identification fields (from mod-102)
scenario: |
  Indirect prompt injection via a retrieved KB article causes the assistant
  to open an escalate_ticket to a mailbox controlled by the attacker.
persona: upstream_data_supplier          # mod-102 ch 01
capability_surface: retrieval + tool_call
taxonomy_tags:                            # mod-102 ch 03
  nist_genai_profile: [risk-9-information-security]
  iso_23894_risk_source: [data, use]
  airo:
    hasRiskSource: [data, model_deployment_context]
    hasConsequence: [security_breach, information_integrity_loss]
  air_bench_2024: [category-tbd-verify]
cause_bucket: integration                 # mod-102 ch 05

# Risk-analysis fields (from chapters 01-02)
inherent: { impact: 4, likelihood: 4, exposure: 4, combined: 5 }
residual: { impact: 4, likelihood: 2, exposure: 4, combined: 3 }
control_defeated:
  - control_id: CTL-CS-RAG-PROV-01
    likelihood: 3
    combined: 4
    monitor_ref: MON-CS-RAG-FN-01
  - control_id: CTL-CS-RAG-ALLOWLIST-01
    likelihood: 4
    combined: 5
    monitor_ref: MON-CS-RAG-ALLOWLIST-01
credited_controls:
  - id: CTL-CS-RAG-PROV-01
    evidence_ref: mod-108-report-2026-q2#prov-fnrate
  - id: CTL-CS-RAG-ALLOWLIST-01
    evidence_ref: release-2026-05-22#allow-list-hash

# Risk-evaluation fields
above_appetite: false
appetite_evaluation_comment: |
  Residual (Medium=3) is within appetite band 'medium'; a control-defeated
  case exceeds appetite and drives monitoring priority.

# Risk-treatment fields
treatment: modify
mitigation_owner: role/ai-risk-engineer
control_owner: team/customer-support-platform
target_residual: 2
treatment_due_date: 2026-09-30

# Monitoring-and-review fields
monitor_refs: [MON-CS-RAG-FN-01, MON-CS-RAG-ALLOWLIST-01]
review_cadence: quarterly
last_reviewed: 2026-06-15
next_review_due: 2026-09-15

# Communication-and-consultation fields
sign_off_routing:
  - role: ai-risk-engineer (L25)
  - role: system-owner
  - role: security-risk-partner (27005 side)
  - role: risk-council (enterprise)
change_log_ref: register-changelog-2026-q2
```

Every field maps to one of the 27005 process steps in the table above. That mapping is the argument that lets you file this register alongside the security-side one.

## Reconciling with the security-side 27005 register

Two register-scale reconciliations are always required — do these once and then keep them fresh.

### 1. Shared assets, shared threat sources

Many AI-adjacent risks are already recorded in the security register: the RAG store as a data store, the customer-support platform as an application, the LLM API vendor as a third-party supplier, prompt injection as a class of application-layer input attack. Do not duplicate. Reference:

- If a scenario in your AI register is *already covered* by a 27005 row, cite the 27005 row-ID and record only the AI-specific *additions* (the guardrail, the AI-specific control-defeated cases). The AI row *complements* the 27005 row; it does not replace it.
- If a scenario in your AI register has *no equivalent* on the security side — a fairness harm, a hallucination, an over-reliance harm — the AI register is authoritative. Note explicitly that the security register does not carry it, so a joint audit does not go looking for a phantom row.

### 2. Appetite-band alignment

Your AI register uses the *enterprise* appetite bands (chapter 05), which are shared with the security-side register. If the AI register uses a different band scale, the roll-up is broken. Two failure modes to watch:

- The AI register uses finer bands than the enterprise ("we split High into High-A and High-B"). Fine, but map to the enterprise bands on export; the risk council reads enterprise bands.
- The AI register uses a wider range than the enterprise ("we go up to Catastrophic"). Fine, but map to the enterprise's top band on export or negotiate a scale change with the ERM function.

## What 23894 changes about *how* the process runs (versus 27005 alone)

A 27005 process that has never operated against AI systems will produce these gaps, which 23894 fills:

- **AI-specific risk sources are not on the security-team's shelf.** A 27005-only analyst will not enumerate labelling error, dataset drift, or human-oversight failure. 23894 catalogues them.
- **AI-specific consequences are subtle.** Bias, homogenisation, over-reliance, opacity — these do not show up in a security-side CIA triad. 23894 names them.
- **AI-specific lifecycle stages produce risks that pure infosec risk management misses.** Data acquisition and model training are risk-carrying phases; 23894 walks them.
- **AI-specific controls have non-classical performance characteristics.** A prompt-injection filter does not have a well-defined false-negative rate the way a signature-based IPS does. The control-defeated column of chapter 01 exists because AI controls degrade silently.

Read 23894 as the *content-side companion* that lets you run the 27005 process without under-covering AI-specific risk sources and consequences. It is not a replacement process.

## An anti-pattern: the parallel AI risk register

A common first move is to spin up a "GRC-for-AI" tool and let the AI register live there, disconnected from the enterprise 27005 register. Consequences:

- The risk council reads two heat maps and cannot say what the *combined* enterprise risk is.
- Auditors see a supplier (LLM API vendor) in the AI register and a supplier (same vendor) in the security register with different residual scores.
- The appetite conversation happens twice, differently.
- A control failure that affects both registers requires manual coordination during incident response.

The mitigation: even if you keep the AI register in a specialised GRC-for-AI platform, the *schema* is 27005-compatible and roll-ups are exported to the enterprise ERM's shape. Chapter 06 shows the export.

## Handoffs with adjacent roles inside the 27005 process

- **Establish-context** is a joint step with the CISO / infosec-risk function and with `ai-governance-analyst` (L15) providing intake (mod-102 ch 06).
- **Risk identification** is where the mod-102 harm model produces the rows; the security-side threat catalogue may contribute for shared assets.
- **Risk analysis** is where you do the L25 work in this module.
- **Risk treatment** is where mod-108 (guardrails) and mod-109 (monitoring) supply controls.
- **Monitoring and review** is the mod-109 signal loop.
- **Communication and consultation** is the mod-112 program-level artifact — cadence, sign-off routing, risk-council reporting.

If a step in the 27005 shape has no clear owner between you, the security-risk partner, the analyst, and the risk council, that unclarity is your escalation to `senior-ai-governance-architect` (L50) or `head-of-ai-governance` (L60) — this is the ladder from mod-101 ch 06.

## Summary

- ISO/IEC 27005 supplies the *process shape* your enterprise's existing risk-management function already uses; reuse it so AI risk plugs in without a schema fight.
- ISO/IEC 23894 supplies the *AI-specific content* — risk sources, consequences, lifecycle stages, control characteristics — that a pure 27005 process would under-cover.
- Both specialise ISO 31000, so the process steps line up 1:1 and only the content changes.
- Publish a register schema that fills every 27005 process step; reuse shared assets and threat sources with the security register; align appetite bands with the enterprise's language.
- The failure mode to avoid is the parallel AI risk register that never composes with the enterprise ERM.
- Exercise 03 walks the alignment on real rows.
