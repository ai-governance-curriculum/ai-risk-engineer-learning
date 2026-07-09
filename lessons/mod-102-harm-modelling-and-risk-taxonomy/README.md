# mod-102 — Harm Modelling and Risk Taxonomy Engineering

**Estimated effort:** 14 hours
**Position in the track:** second module; assumes mod-101 (frameworks & role scope) is done.

The frameworks in mod-101 told you *what* to file; this module teaches you *how to fill it in defensibly*. You author a **harm model** for a concrete AI system — misuse personas, capability surfaces, immediate harms, secondary harms, cascading and systemic effects — grounded in real incidents (AIID, OECD.AI, MIT AI Risk Repository) rather than speculation. You then overlay the standardised taxonomies (NIST GenAI Profile, ISO/IEC 23894, AIRO, AIR-Bench 2024) so the harm model is *defensibly complete* under review. Finally you read the frontier-lab risk-tier frameworks (Anthropic RSP, OpenAI Preparedness, DeepMind FSF, Frontier Model Forum publications) as shape templates for enterprise deployment tiering and mark the boundary with `agentic-safety-engineer` (level 40).

The harm model you produce here is the input to every later module in this track: risk quantification (mod-103), red-team scope (mod-104), adversarial-ML scope (mod-105), fairness / bias / explainability targeting (mod-106), privacy-risk (mod-107), guardrail selection (mod-108), monitoring signal (mod-109), MRM / supply-chain (mod-110), and incident response (mod-111). If the harm model is wrong or thin here, downstream work is wrong or thin too.

## Learning objectives

- Author a harm model for a concrete AI system — misuse personas (adversarial user, insider, upstream data supplier, downstream integrator), capability surfaces, immediate and secondary harms, cascading and systemic effects — grounded in AIID / OECD.AI incidents / MIT AI Risk Repository as evidence.
- Layer NIST AI RMF Generative AI Profile risks, ISO/IEC 23894 risk sources, AIRO ontology categories, and AIR-Bench 2024 policy-derived risk categories onto the harm model to make it defensibly complete.
- Reason about frontier-lab risk-tiered frameworks — Anthropic Responsible Scaling Policy, OpenAI Preparedness, Google DeepMind Frontier Safety Framework, Frontier Model Forum publications — as shape templates for enterprise deployment tiering.
- Distinguish operator-caused, model-caused, data-caused, and integration-caused harms so the mitigation contract routes to the right owner.
- Coordinate with `ai-governance-analyst` (level 15) on the intake-shape data this role feeds into, and with `agentic-safety-engineer` (level 40) on the frontier-agent depth that lives above this level.

## Chapters

1. [`01-harm-model-shape.md`](01-harm-model-shape.md) — what a harm model *is* as an engineering artifact; personas, capability surfaces, and the immediate → secondary → cascading → systemic harm chain.
2. [`02-incident-corpora-as-evidence.md`](02-incident-corpora-as-evidence.md) — grounding every claim in AIID, OECD.AI Incidents Monitor, and the MIT AI Risk Repository so the harm model survives review.
3. [`03-taxonomy-overlays.md`](03-taxonomy-overlays.md) — layering NIST GenAI Profile, ISO/IEC 23894 risk sources, AIRO ontology categories, and AIR-Bench 2024 policy-derived categories onto the harm model for defensible completeness.
4. [`04-frontier-lab-rsp-shape.md`](04-frontier-lab-rsp-shape.md) — reading Anthropic RSP, OpenAI Preparedness, Google DeepMind FSF, and Frontier Model Forum publications as *shape templates* for enterprise deployment tiering.
5. [`05-causal-attribution.md`](05-causal-attribution.md) — operator-, model-, data-, integration-caused harm buckets and how they route the mitigation contract to the right owner.
6. [`06-level-ladder-handoffs.md`](06-level-ladder-handoffs.md) — the intake-shape you consume from `ai-governance-analyst` (L15) and the frontier-agent depth you hand off to `agentic-safety-engineer` (L40).

## Exercises

- [`exercises/exercise-01-misuse-persona-and-capability-surface-mapping.md`](exercises/exercise-01-misuse-persona-and-capability-surface-mapping.md) — enumerate misuse personas and capability surfaces for the running system.
- [`exercises/exercise-02-harm-catalog-from-incident-database.md`](exercises/exercise-02-harm-catalog-from-incident-database.md) — populate a harm catalog from AIID + OECD.AI evidence.
- [`exercises/exercise-03-mit-ai-risk-repository-taxonomy-overlay.md`](exercises/exercise-03-mit-ai-risk-repository-taxonomy-overlay.md) — cross-walk the harm catalog to the MIT AI Risk Repository causal and domain taxonomies.
- [`exercises/exercise-04-airo-and-nist-genai-profile-cross-tag.md`](exercises/exercise-04-airo-and-nist-genai-profile-cross-tag.md) — tag each harm with AIRO categories and NIST GenAI Profile risks; run the applicability sheet.
- [`exercises/exercise-05-frontier-lab-rsp-preparedness-comparative-read.md`](exercises/exercise-05-frontier-lab-rsp-preparedness-comparative-read.md) — read Anthropic RSP, OpenAI Preparedness, and DeepMind FSF side-by-side; borrow the shape for enterprise tiering.

## Labs & quizzes

- [`labs/`](labs/) — a longer hands-on lab is scheduled here; see the lab README for the current status.
- [`quizzes/`](quizzes/) — knowledge-check quiz for the module.

## Resources

See [`resources.md`](resources.md) for primary sources — every reference points at a standards body, regulator, incident-database maintainer, or the original publication of a frontier-lab policy.

## How to work through the module

1. Pick **one AI system** (the same running system you used for mod-101, or a fresh one). The five exercises are cumulative — every one uses this system.
2. Read chapters 01-06 in order. Each is short and pins one dimension of the harm-model artifact you are building.
3. Do exercises 01-05 in order. They stack:
   - 01 produces the persona / capability-surface sheet,
   - 02 turns it into an evidence-linked harm catalog,
   - 03 overlays the MIT AI Risk Repository taxonomies,
   - 04 cross-tags AIRO + NIST GenAI Profile applicability,
   - 05 reads the frontier-lab risk-tier documents and lifts the shape into your enterprise tiering.
4. Take the quiz.

By the end of the module you should be able to hand an auditor a harm-model artifact where every harm cell links to (a) at least one incident-corpus record as evidence, (b) at least one standardised taxonomy category, (c) a named cause-bucket (operator / model / data / integration), and (d) a downstream mitigation owner. If any cell fails one of those four checks, the harm model is not shippable.
