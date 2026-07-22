# The Level-Ladder Handoff — Working Eval vs. Runtime Hardening

Every other chapter in this module builds *the working eval* — attacks, defence baselines, numbers, register updates, monitors. But the deployed inference endpoint has an *outer perimeter* — WAF-style filtering, rate-limit orchestration, endpoint attestation, artifact signing — that this role does not own. That perimeter is owned by the platform-security peer at level 35 in the sibling `security-learning` / `ai-infra-security` track. This chapter draws the exact line, defines the handoff contract, and calls out what a level-25 AI risk engineer *specifies* vs. what the peer *implements and runs*.

## Motivation

Two failure modes when the boundary is fuzzy:

- **Double-owned controls.** Both this role and the platform-security peer think they own the rate-limit at the model endpoint. Neither ends up owning it operationally; when it fails, nobody's on-call rotation triggers. The auditor finds an unmaintained control.
- **Orphan-owned controls.** Neither role thinks it owns the "signed model artifact" control. The AI-risk role assumes it's a platform concern; the platform-security peer assumes it's an ML concern. The control is not implemented; a supply-chain poisoning attack later succeeds through the unsigned artifact path.

Drawing the boundary explicitly — in a contract that both roles review — is what prevents both.

## The two roles and the level-ladder position

- **This role** — level 25, `ai-risk-engineer`. Owns the *adversarial-ML eval*, the *defence baselines*, the *residual-risk update*, and the *specification* of runtime controls. Does not own the runtime infrastructure that enforces those controls.
- **The peer** — level 35, an `ai-infra-security` / platform-security engineer in the sibling `security-learning` track. Owns the *runtime hardening at the inference edge* — API gateway configuration, WAF rules for ML endpoints, rate-limit enforcement, endpoint attestation, artifact signing, secret-store integration, network-layer isolation.

The two roles collaborate on:

- **Query-rate control.** Level-25 specifies the required rate limit per caller (from the extraction / MIA analysis in chapter 04). Level-35 implements the enforcement at the API gateway and produces the evidence that it is in place.
- **Query-pattern anomaly detection.** Level-25 specifies the metric (aggregate query entropy, per-caller pattern deviation) and the alert threshold. Level-35 wires the metric into the platform-security monitoring stack.
- **Output redaction / perturbation.** Level-25 specifies the redaction shape (top-k only, confidence bucket count, non-argmax noise magnitude) based on the extraction-attack delta. Level-35 (or the ML platform team) implements the redaction in the serving path.
- **Signed model artifacts.** Level-25 specifies that all model artifacts consumed by the deployed system must be provenance-verifiable. Level-35 implements the signing / verification pipeline and produces the artifact-manifest evidence.
- **Endpoint attestation.** Level-25 requires that the model behind the API is the one on the release manifest (defence against silent swap by an insider or a supply-chain attacker). Level-35 implements the attestation mechanism.

## The handoff contract — one table

The contract you write for the running system in exercise 05 (or as a companion artifact to it) fills this table. Every row is a control; every row names the specifier, the implementer, the operator, and the evidence.

| Control | Specifier (level 25) | Implementer (level 35 or ML platform) | Operator (on-call) | Evidence artifact |
|---|---|---|---|---|
| Per-caller query rate limit at the inference API gateway | Specifies the rate value and the auth-token dimension | Implements enforcement in the gateway | Platform-security on-call | Gateway config in a versioned repo; audit-log of rate-limit hits |
| Aggregate query-pattern anomaly monitor | Specifies the metric formula and threshold | Instruments the metric into the platform SIEM | Platform-security on-call | Monitor definition in monitoring repo; last-N-days alert history |
| Output confidence redaction | Specifies the redaction shape | Implements in the serving path | ML platform on-call | Serving-path unit test; production sample verification |
| Model-artifact signing / provenance | Specifies the requirement | Implements the signing pipeline and the verification hook at load-time | Platform-security on-call | Signed manifest of the release; verification-log entry per model load |
| Endpoint attestation (model identity at the endpoint) | Specifies the requirement | Implements the attestation mechanism | Platform-security on-call | Attestation-log entry per deploy |
| Adversarial-training on the model itself | Specifies the requirement and the target robustness | Implemented by ML team during training | ML on-call | Training-config commit hash; robustness-eval report (chapter 05) |
| DP-training on the model | Specifies the requirement, target ε, hyperparameters | Implemented by ML team during training | ML on-call | Training-config commit hash; DP-accounting log (co-owned with mod-107) |
| Runtime backdoor detector (STRIP-style) in the serving path | Specifies the detector and the operating point (detection rate vs. FPR) | Implemented by ML platform team | ML platform on-call | Serving-path config; last-N-days detection log |

Two things this table makes explicit:

- **Specifier ≠ Implementer.** This role specifies; peers implement. If the specification is unclear or unmeasurable, that is *this role's* fault — not the implementer's.
- **The evidence artifact is a first-class deliverable.** A control that ships with no evidence pointer is not a shippable control.

## What this role owns end-to-end (specification through report)

- The **attack panel** — which attacks are in scope, at which threat models, at which budgets. Chapters 02-04.
- The **defence baseline suite** — which defences the eval measures, in which order, against which attack panel. Chapters 02-04.
- The **robustness-eval report** — the artifact that carries the numbers. Chapter 05.
- The **register update** — inherent / residual / control-defeated columns updated with quantified numbers. mod-103 lives at the peer chapter; this module produces the input.
- The **monitor specifications** for the controls that ship. mod-109 wires them.
- The **incident-response runbook branches** for the control-defeated variants. mod-111 files the runbooks.

## What the peer owns end-to-end (runtime hardening)

- The **API gateway configuration** for the inference endpoint.
- The **WAF rules and ML-specific request filters** — request-shape enforcement, header requirements, payload-size limits, content-type restrictions.
- The **rate-limit enforcement mechanism** — token buckets, sliding windows, per-caller counters.
- The **query-pattern anomaly monitor implementation** — the metric collectors, SIEM integration, alerting pipeline.
- The **artifact signing / verification pipeline** — key management, signing keys, verification hooks.
- The **endpoint attestation mechanism** — the attestation protocol and the verification of it on every deploy.
- The **secret management** for model API keys, upstream data-store credentials, and any inference-time secrets.
- The **network-layer isolation** — VPC posture, egress restrictions, internal service mesh routing.
- The **on-call rotation** for all of the above.

## What is co-owned (and how)

- **The runtime backdoor detector (STRIP or similar).** This module baselines the detector and picks the operating point. The ML platform team implements it in the serving path. The ML platform on-call operates it. The residual-risk owner (this role) revisits when the detector's clean-FPR drifts.
- **The output confidence redaction.** This module picks the shape. The serving path implements it. Both roles revisit when the redaction is proposed to be relaxed for a business reason.
- **The DP-training defence.** This module baselines the utility-privacy tradeoff. mod-107 owns the formal (ε, δ) accounting. The ML team implements the training config. All three review when the ε target changes.
- **Supply-chain poisoning defence.** This module baselines behavioural checks on downloaded checkpoints. The peer at level 35 owns provenance controls (signed checkpoints, hash-pinned registry). mod-110 owns the MRM / supplier-risk view of the same pipeline.

## The escalation path

- **`ai-evaluation-engineer` (level 35).** Release-gate decisions that turn on the eval numbers ("do we ship this model despite the PGD residual?") escalate to this role. This module produces the number; that role owns the gate call.
- **`agentic-safety-engineer` (level 40).** Attacks on agent-topology (multi-agent poisoning, tool-use injection, planner subversion) sit above this module's ceiling. Where an attack is genuinely agent-scale, escalate.
- **`senior-ai-governance-architect` (level 50).** Frameworks and program-level policy on adversarial-robustness commitments live at level 50. This module's evidence and reports are inputs.
- **`head-of-ai-governance` (level 60).** Enterprise appetite and cross-portfolio adversarial-risk posture live at level 60. This module's roll-up numbers are inputs.

## The out-of-scope list

- **General cybersecurity hygiene** — patching, endpoint protection, IAM basics, network segmentation. Owned by the broader security organisation, not by this module.
- **Cryptographic protocol design.** Model-endpoint attestation uses well-known crypto primitives; this module does not invent new ones.
- **Physical infrastructure security.** Data-centre access controls, hardware supply chain — out of scope.
- **Legal / regulatory interpretation.** This module names what is technically the case; interpretation of "was this a reportable incident under [jurisdiction]" is legal and level-40+ governance territory.

## Cross-references

- **mod-101 (frameworks and role scope)** — this chapter operationalises the deferral-contract shape mod-101 introduces.
- **mod-102 (harm modelling)** — the ownership assignment on each harm-model row (mitigation_owner column) is what this chapter formalises for adversarial-ML rows.
- **mod-103 (risk quantification)** — the register carries the ownership assignment per row.
- **mod-107 (privacy-risk engineering)** — DP-training epsilon-target-setting handoff.
- **mod-108 (guardrail engineering)** — runtime detectors and output-redaction guardrails.
- **mod-109 (continuous monitoring)** — every co-owned control has a monitor; wired at mod-109 with an owner named per this chapter's contract.
- **mod-110 (MRM / supply-chain)** — third-party model consumption and vendor-model risk.
- **mod-111 (incident response)** — runbook ownership per control-defeated variant follows the contract shape here.
- **`security-learning` / `ai-infra-security` (level 35)** — the peer track that owns the runtime hardening described in this chapter.

## Summary

- This module owns the working adversarial-ML eval, the defence baselines, the residual-risk update, and the *specification* of runtime controls.
- The peer at level 35 in `security-learning` / `ai-infra-security` owns the *runtime hardening* — gateway config, WAF rules, rate-limit enforcement, artifact signing, endpoint attestation.
- The handoff contract is one table: control, specifier, implementer, operator, evidence artifact.
- Co-owned controls (runtime detectors, output redaction, DP-training) get an explicit review cadence and a named owner per state.
- Escalation goes to `ai-evaluation-engineer` (L35) for release-gate calls, `agentic-safety-engineer` (L40) for agent-topology attacks, and higher governance roles for program-level policy.
- The eval you produce in exercise 05 is not shippable without the co-authored contract that this chapter defines.
