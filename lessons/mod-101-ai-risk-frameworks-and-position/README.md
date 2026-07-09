# mod-101 — AI Risk Frameworks and the Risk-Engineer Position on the Governance Ladder

**Estimated effort:** 12 hours
**Position in the track:** first module; prerequisite for every module that follows.

Read NIST AI RMF 1.0, the NIST GenAI Profile, ISO/IEC 42001 / 23894 / 42005 / 25059, and the EU AI Act *as an engineer* — mapping each obligation to a concrete test, threshold, guardrail, monitor, or evidence artifact. Position SR 11-7 / OCC 2011-12 / SR 23-4, the FDA GMLP + PCCP, and the Colorado AI Act as the sector / geography flavours of that shape. Diagram this role's exact scope on the level ladder and write the deferral contract.

## Learning objectives

- Read **NIST AI RMF 1.0** (AI 100-1) as an engineer, using the **NIST AI RMF Playbook** as day-to-day reference. Every MEASURE / MANAGE sub-category translates to a *test, threshold, guardrail, monitor, and evidence artifact*.
- Read the **NIST GenAI Profile** (AI 600-1) as the *12-risk starting checklist* your harm model must clear.
- Read **ISO/IEC 42001** (AIMS — auditable), **ISO/IEC 23894** (AI risk-management guidance), **ISO/IEC 42005** (impact-assessment guidance), and **ISO/IEC 25059** (AI quality model) as the standards a risk-engineering program plugs into and reports against.
- Read **EU AI Act Articles 9, 10, 14, 15, 55, 61, 72** at the level required to translate each obligation into an engineering deliverable.
- Position **SR 11-7 / OCC 2011-12 / SR 23-4** (US FI MRM), **FDA GMLP + PCCP** (health SaMD), and the **Colorado AI Act** as sector / geography flavours of the same shape.
- Diagram this role's exact scope on the level ladder — what you own, what you consume from `ai-governance-analyst` (L15), what you hand off sideways to peer eval / MLOps / security tracks, and what you escalate to `ai-evaluation-engineer` (L35), `agentic-safety-engineer` (L40), `senior-ai-governance-architect` (L50), and `head-of-ai-governance` (L60).

## Chapters

1. [`01-nist-ai-rmf-as-an-engineer.md`](01-nist-ai-rmf-as-an-engineer.md) — reading AI RMF 1.0 as an engineer; the sub-category → test / threshold / guardrail / monitor / evidence translation.
2. [`02-nist-genai-profile-as-harm-checklist.md`](02-nist-genai-profile-as-harm-checklist.md) — the twelve GenAI risks in AI 600-1 as a starting checklist for the harm model.
3. [`03-iso-standards-your-program-plugs-into.md`](03-iso-standards-your-program-plugs-into.md) — ISO/IEC 42001, 23894, 42005, 25059 and how one engineering artifact files against several standards at once.
4. [`04-eu-ai-act-articles-to-deliverables.md`](04-eu-ai-act-articles-to-deliverables.md) — Articles 9, 10, 14, 15, 55, 61, 72 translated to concrete artifacts.
5. [`05-sector-and-geography-flavours.md`](05-sector-and-geography-flavours.md) — SR 11-7 / OCC 2011-12 / SR 23-4, FDA GMLP + PCCP, Colorado AI Act as label-and-routing variants.
6. [`06-role-scope-on-the-level-ladder.md`](06-role-scope-on-the-level-ladder.md) — the deferral contract; what this role owns / consumes / hands-off / escalates / out-of-scopes.

## Exercises

- [`exercises/exercise-01-nist-ai-rmf-obligation-to-engineering-artifact-map.md`](exercises/exercise-01-nist-ai-rmf-obligation-to-engineering-artifact-map.md) — translate a chosen slice of AI RMF sub-categories into engineering artifacts.
- [`exercises/exercise-02-genai-profile-risk-checklist-tour.md`](exercises/exercise-02-genai-profile-risk-checklist-tour.md) — walk the twelve GenAI risks with a real system in scope.
- [`exercises/exercise-03-iso-42001-and-23894-clause-to-artifact-map.md`](exercises/exercise-03-iso-42001-and-23894-clause-to-artifact-map.md) — build the ISO clause → artifact map for the same system.
- [`exercises/exercise-04-eu-ai-act-articles-to-engineering-deliverables.md`](exercises/exercise-04-eu-ai-act-articles-to-engineering-deliverables.md) — produce the EU AI Act article → artifact map for the same system.
- [`exercises/exercise-05-role-scope-and-deferral-contract.md`](exercises/exercise-05-role-scope-and-deferral-contract.md) — write the deferral contract for the same system.

## Labs & quizzes

- [`labs/`](labs/) — a longer hands-on lab is scheduled here; see the lab README for the current status.
- [`quizzes/`](quizzes/) — knowledge-check quiz for the module.

## Resources

See [`resources.md`](resources.md) for the primary sources — every reference is a link to the standards body, regulator, or original publication.

## How to work through the module

1. Read chapters 01-06 in order. Each is short and pins the engineering translation of one standard family.
2. Pick a *single AI system* (real or documented) to be the running example. Every exercise in the module uses it.
3. Do exercises 01-05 in order. They stack: exercise 01 builds the RMF map for the running system; 02 adds the GenAI checklist; 03 adds ISO; 04 adds EU AI Act; 05 closes with the deferral contract.
4. Take the quiz.

By the end of the module you should be able to look at any AI-risk obligation phrased in any of the frameworks above and answer *what would this look like as an engineering artifact* in five sentences (see chapter 01).
