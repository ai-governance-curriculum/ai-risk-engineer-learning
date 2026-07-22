# NIST AI 100-2 (e2025) as the Working Adversarial-ML Taxonomy

Every attack you engineer, every defence you baseline, and every row you drop into the risk register in this module needs a stable, defensible name. **NIST AI 100-2 e2025 — "Adversarial Machine Learning: A Taxonomy and Terminology of Attacks and Mitigations"** is the taxonomy you use. It is authored jointly by NIST and community researchers, it is refreshed on a slow cadence, and it is the one document a regulator, an auditor, and a peer red-teamer will all recognise. This chapter reads it as the working spine and lays out how each attack category maps out to **MITRE ATLAS** tactics (the operational threat-landscape view) and **OWASP ML Top 10** entries (the application-security view). Exercise 01 has you fill the cross-tag sheet for a real system on top of the shape defined here.

## Motivation

Adversarial-ML literature is enormous and inconsistent. The same attack goes by three different names across three different papers; the same defence is called a "mitigation," a "hardening," and a "robustification" in the same week. If the risk register borrows whichever term the last paper used, the register drifts and the auditor cannot follow the trail from harm scenario to eval to residual score.

The fix is to *pin one taxonomy* and route every other name through it. NIST AI 100-2 is chosen for three reasons:

- It is a **standards-body document**, so it is defensible in front of an auditor without argument.
- It **partitions the attack surface** into a small number of categories that each map cleanly to a specific defence family — evasion attacks to input-space robustness, poisoning to training-pipeline integrity, extraction to API-surface controls, and so on.
- It is **maintained** — the e2025 revision refreshes the pre-2020 shape with generative-AI-era attacks and terminology, which means the vocabulary you tag with today is still the vocabulary in the next audit cycle.

MITRE ATLAS and OWASP ML Top 10 are *not* replacements for the NIST taxonomy — they cover overlapping ground with different framings. ATLAS is a TTP-style adversarial matrix; OWASP ML Top 10 is a risk-ranked list oriented at application security teams. The cross-tag ensures the same harm scenario reads correctly in all three vocabularies without any of the three getting to be the authoritative one.

## The six categories in AI 100-2 e2025

AI 100-2 e2025 organises attacks by the *attacker's objective*, not by the technique. The six categories that anchor this module:

| Category | Attacker objective | Where in the ML lifecycle | Typical concrete techniques |
|---|---|---|---|
| **Evasion** | Cause the deployed model to misclassify / mis-generate on a specific input | Inference time | FGSM, PGD, Carlini & Wagner (C&W), TextAttack transformations, universal perturbations |
| **Poisoning** | Corrupt the model at training / fine-tuning so it behaves adversarially at inference | Data pipeline, training pipeline | Label-flipping, targeted data poisoning, backdoor / BadNets trigger insertion, clean-label poisoning |
| **Extraction (model stealing)** | Reconstruct a functional copy of the deployed model from its query interface | Inference API | Query-based extraction (Tramèr et al.), Knockoff Nets, active-learning extraction |
| **Inversion (attribute / reconstruction)** | Recover training-data inputs or sensitive attributes from model outputs / gradients | Inference API, model weights | Fredrikson model inversion, gradient-inversion attacks, feature reconstruction |
| **Membership inference (MIA)** | Determine whether a specific record was in the training set | Inference API | Shokri shadow-model MIA, LiRA (likelihood-ratio attack), loss-threshold MIA |
| **Availability** | Degrade the service — latency, cost, or quality — at scale | Inference API, resource layer | Sponge examples, adversarial-cost attacks, energy-latency attacks |

<!-- needs-research: confirm that AI 100-2 e2025 keeps this six-way partition and check the exact section numbering; the pre-e2025 draft used a similar but not identical split. -->

Two properties of this partition worth internalising:

- **Category is defined by objective**, so an attack that is technically a "PGD variant" can appear either under evasion (if it degrades classification on one input) or under poisoning-adjacent categories (if the same optimisation is used at training time). Tag by what the attacker is trying to *achieve*.
- **Categories are not mutually exclusive at the incident level**. A real adversary chains evasion probes to characterise the model, then extraction to build a surrogate, then evasion on the surrogate to craft a stronger attack. In the register you tag each *scenario* with its primary category and note the chain in the row's notes.

## The threat model rubric AI 100-2 uses

Each attack in AI 100-2 is described against three axes. Any technique you introduce in the module has to be described in these terms — this is how the register row stays unambiguous.

| Axis | Options | What it constrains |
|---|---|---|
| **Attacker knowledge** | White-box (full model access), grey-box (partial — architecture, distribution), black-box (query-only) | The attack budget; the transferability question; the defence surface |
| **Attacker capability** | Passive observer, active querier, training-data contributor, model-supply-chain participant | Where in the pipeline the defence has to live |
| **Attacker goal** | Targeted (specific misclassification), untargeted (any misclassification), availability, information extraction | The success metric for the eval |

When you write a robustness-eval report (chapter 05), you name those three axes explicitly per attack. "PGD on a customer-support classifier" is not enough; "*white-box, active-querier, untargeted* PGD with L∞ perturbation budget ε=8/255" is a threat model that can be reviewed.

## MITRE ATLAS as the operational overlay

MITRE ATLAS mirrors the ATT&CK matrix for ML systems. Its columns are *tactics* (attacker objectives at operational granularity) and its cells are *techniques* (concrete methods) with real case studies attached. Where NIST AI 100-2 tells you *what the attack is called at a taxonomy level*, ATLAS tells you *what the attacker's operational chain looks like* and links to published incident write-ups.

The NIST → ATLAS mapping that anchors the cross-tag exercise:

| NIST AI 100-2 category | Representative ATLAS tactics | Notes |
|---|---|---|
| Evasion | Reconnaissance, ML Attack Staging, Evade ML Model | ATLAS breaks the attack into a chain — recon of the model, staging perturbations, delivery — which the taxonomy collapses into "evasion" |
| Poisoning | ML Supply Chain Compromise, Poison Training Data | ATLAS explicitly separates the supply-chain path (poisoned pretrained checkpoint) from the training-data path |
| Extraction | Reconnaissance, ML Model Access, Exfiltration via ML Inference API | Extraction sits under the exfiltration tactic in ATLAS's chain view |
| Inversion | ML Model Access, Exfiltration via ML Inference API | Similar tactic chain to extraction; different technique cells |
| Membership inference | ML Model Access, Exfiltration via ML Inference API | Same — the outputs a MIA needs are the same category of ATLAS cell |
| Availability | Impact (Denial of ML Service / Cost Harvesting) | ATLAS has explicit Impact-tactic cells for the availability category |

<!-- needs-research: verify the exact current ATLAS tactic names and technique IDs against the live matrix; ATLAS occasionally renames or reorganises cells between releases. -->

The point is not to memorise ATLAS IDs but to be able to walk from *scenario name in the harm model* → *NIST category* → *ATLAS tactic and technique ID* → *published case study* in one motion. That chain is what makes the harm scenario auditable.

## OWASP ML Top 10 as the application-security overlay

OWASP ML Top 10 is the application-security community's ranked list of adversarial-ML risks. Its category set overlaps with NIST AI 100-2 but is framed for AppSec — the concerns are how the risk shows up at the API surface, in the SDLC, and in third-party model consumption. The Top 10 evolves; anchor to the current published list.

<!-- needs-research: confirm the exact current numbering and titles of OWASP ML Top 10 entries; the list is versioned and re-ranked between refreshes. -->

The cross-tag pattern:

| NIST AI 100-2 category | Typical OWASP ML Top 10 entries | Notes |
|---|---|---|
| Evasion | Input Manipulation Attack | The AppSec framing of adversarial examples |
| Poisoning | Data Poisoning Attack; Supply Chain Vulnerabilities | Split across data and supply-chain rows in the Top 10 |
| Extraction | Model Theft | Direct match |
| Inversion | Model Inversion Attack | Direct match |
| Membership inference | Membership Inference Attack | Direct match |
| Availability | Denial of Service (adapted) | Availability is less explicitly ranked in ML Top 10 — cite the general availability entry and note the mapping |

Do **not** collapse NIST and OWASP into a single label per scenario. The two labels answer different review questions — NIST for the taxonomy defensibility, OWASP for the AppSec routing (which security team owns the mitigation? whose SDLC does it belong in?).

Note as well that the **OWASP LLM Top 10** is a *different* list — it covers LLM-application-specific risks (prompt injection, sensitive information disclosure, insecure output handling). The LLM Top 10 is the primary overlay for mod-104 (LLM red-teaming); the ML Top 10 is the primary overlay for this module. Scenarios that touch both worlds (e.g., a fine-tuned LLM classifier that is adversarially probed and also prompt-injected) get tagged in both.

## The cross-tag table shape

This is the artifact exercise 01 has you build. One row per attack scenario in scope for the system in question.

| Scenario id | NIST AI 100-2 category | Attack technique | Attacker knowledge | Attacker capability | Attacker goal | MITRE ATLAS tactic(s) + technique id(s) | OWASP ML Top 10 entry | OWASP LLM Top 10 entry (if applicable) | Harm-model row (mod-102) | Register row (mod-103) |
|---|---|---|---|---|---|---|---|---|---|---|

Two invariants that must hold for the sheet to be shippable:

- **Every scenario row cross-links back to a harm-model row (mod-102) and a register row (mod-103).** If there is no mod-102 row, either the harm model is incomplete or the attack scenario is out of scope. If there is no mod-103 row, the register does not know about the risk.
- **Every attack you *implement* in exercises 02-04 must appear as a row here first.** The taxonomy is upstream of the tooling — if you cannot name the attack in NIST + ATLAS + OWASP terms, don't run it yet.

## What this chapter does *not* cover

- **The `security-learning` / `ai-infra-security` peer track (level 35)** covers platform-scale defence at the inference edge — WAF-style query filtering, API rate-limit orchestration, model-endpoint attestation. This module cites the tactics; that peer owns the runtime. Chapter 06 draws the exact handoff.
- **LLM-specific attack categories** (prompt injection, jailbreaks, indirect prompt injection) are covered in mod-104 (LLM red-teaming). Where a scenario is genuinely dual-domain (e.g., an adversarial suffix that is *both* an evasion attack on a safety classifier *and* a jailbreak), tag it in both taxonomies and note the cross-reference in the row.
- **Privacy-metric depth for MIA** — LiRA calibration curves, DP-training epsilon accounting, membership-advantage confidence intervals — sits in mod-107 (privacy-risk engineering). This module runs the attacks well enough to establish *that* there is a privacy risk; mod-107 quantifies it against a formal privacy budget.

## Cross-references

- **mod-102 (harm modelling)** — every scenario row on the cross-tag sheet points at a harm-model row; a scenario with no harm-model row is either missing from mod-102 or out of scope.
- **mod-103 (risk quantification)** — the register consumes the cross-tag sheet and produces the inherent / residual / control-defeated scoring.
- **mod-104 (LLM red-teaming)** — dual-domain scenarios get an OWASP LLM Top 10 tag as well as an ML Top 10 tag.
- **mod-107 (privacy-risk engineering)** — the MIA and inversion rows here are the intake for mod-107's formal privacy-metric work.
- **mod-108 (guardrail engineering)** — the defence baselines you build in this module inform the guardrail catalogue mod-108 selects from.
- **mod-109 (continuous monitoring)** — an evasion-attack-success-rate monitor and an extraction-query-rate anomaly monitor both originate from the cross-tag scenarios here.

## Summary

- NIST AI 100-2 e2025 is the working taxonomy; six categories — evasion, poisoning, extraction, inversion, membership inference, availability — organised by attacker objective.
- Every technique is described against three threat-model axes: knowledge (white / grey / black-box), capability (passive / active / training-contributor / supply-chain), goal (targeted / untargeted / availability / extraction).
- MITRE ATLAS is the operational overlay — tactics and techniques with published case studies.
- OWASP ML Top 10 is the AppSec overlay — different framing, different owning team, do not collapse it into the NIST tag.
- The cross-tag sheet has one row per attack scenario and links back to the harm model (mod-102) and register (mod-103); no orphan rows.
- Exercise 01 builds the sheet for the running system; exercises 02-04 implement the attacks whose rows are already on it.
