# The Role's Scope on the Level Ladder

This chapter draws the *exact* boundary line around the AI Risk Engineer role at level 25 in the AI Governance family. It is the last chapter of module 101 on purpose: everything the module taught you gets *scoped* here.

## Motivation

Governance work spans a very wide surface — policy authoring, intake, harm modelling, evaluation, red-team, monitoring, incident response, audit, regulator engagement, executive briefing, program leadership. You cannot own all of it; if you try, you either burn out or accidentally step on the analyst, the architect, or the peer eval engineer, and the program stops working. The *deferral contract* — what you own, what you consume from below, what you hand off sideways, what you escalate — is what makes a cross-functional program actually functional.

## The ladder, one line each

| Level | Role | Family | One-line scope |
|---|---|---|---|
| 10 | `ai-infra-junior-engineer` | Infra | Engineering-craft prerequisites — Linux, Git, Python, packaging, HTTP APIs |
| 15 | `ai-governance-analyst` | Governance | Operational analyst work — intake, inventory, framework crosswalks, impact-assessment first drafts, filing evidence |
| 20 | `ml-engineer` | ML Engineering | Classical ML / PyTorch / sklearn / MLflow / packaging fundamentals |
| 25 | **`ai-risk-engineer` (this track)** | **Governance** | **Engineering craft of AI risk end-to-end — harm-model authoring through incident RCA at team scope** |
| 25 | `ai-infra-mlops` | Infra | MLOps automation — CI/CD, registry, deployment gates that your risk-eval work plugs into |
| 30 | `ai-eval-engineer` | AI Engineering | Application-layer LLM/agent evaluation-engineering depth — traces, judges, RAG eval, eval-gated CI/CD |
| 30 | `model-evaluation-engineer` | ML Engineering | Benchmark, statistical, judge-vs-human calibration methodology depth |
| 35 | `ai-evaluation-engineer` | Governance | Release-assurance / audit-trail / regulator-facing methodology depth |
| 35 | `security-learning` / `ai-infra-security` | Infra | Deep ML/AI security at platform scale — MLSecOps, judge supply-chain, eval-set exfiltration prevention |
| 40 | `agentic-safety-engineer` | Governance | Frontier-agent red-team methodology and dangerous-capability evaluation depth |
| 50 | `senior-ai-governance-architect` | Governance | Control-library architecture, policy taxonomy, cross-jurisdiction reconciliation |
| 60 | `head-of-ai-governance` | Governance | Program leadership, board reporting, regulator engagement |
| 70 | `chief-ai-officer` | Governance | Executive scope |

## What you own at level 25

Twelve modules of engineering craft, at team scope, for the AI Governance family:

1. Reading AI risk frameworks *as an engineer* (this module).
2. Authoring the **harm model** for a concrete system (mod-102).
3. **Quantifying** inherent, residual, control-defeated risk and populating an **AI risk register** (mod-103).
4. Engineering **LLM / agent red-team programs** (mod-104).
5. Engineering **adversarial-ML** evaluations (mod-105).
6. Engineering **fairness, bias, explainability** evaluations (mod-106).
7. Engineering **privacy-risk** evaluations (mod-107).
8. Engineering **guardrails** and measuring their effectiveness (mod-108).
9. Wiring **monitoring** signal into the risk register + post-market surveillance reports (mod-109).
10. **Model Risk Management** (SR 11-7 / FDA GMLP / PCCP) shape + **AI supply-chain** provenance (mod-110).
11. AI-specific **incident response** and post-mortems, with permanent-regression-fixture back-feed (mod-111).
12. Owning a **cross-functional AI risk program at team scope** (mod-112) — governance-council briefings, model-card risk slice, deployment-tiering contribution.

## What you *consume from below*

From `ai-governance-analyst` (level 15) — you assume these are already done:

- AI-system intake and inventory,
- jurisdictional regulatory tracking,
- first-draft impact-assessment authoring,
- framework-crosswalk legwork,
- reading eval / audit evidence into governance records.

From `ml-engineer` (level 20) / `ai-infra-junior-engineer` (level 10):

- classical ML / DL toolchain fluency,
- packaging, deployment, environment,
- HTTP APIs, Git, Linux, Python.

If any of these are missing on the ground, do not re-teach the analyst — flag it as *unowned* work and route it back. You should not be doing intake or inventory yourself.

## What you *hand off sideways*

From peer `ai-eval-engineer` (level 30, AI Engineering family):

- traces / spans, judge implementations, RAG eval methodology, eval-gated CI/CD, online eval, cost / latency / quality trade-offs.

*Your interface with this peer:* they build the app-side measurement plumbing; you *consume the metrics* and set risk thresholds against them. Mod-108 and mod-109 both include an explicit hand-off contract.

From peer `model-evaluation-engineer` (level 30, ML Engineering family):

- benchmark methodology, statistical methodology, judge-vs-human calibration.

*Your interface:* you *use* their methodology to defend your own risk claims — a fairness delta of X between two groups, an accuracy drop of Y under adversarial perturbation. You are their consumer, not their competitor.

From peer `ai-infra-mlops` (level 25, Infra family):

- registry, deployment gates, canary infrastructure, feature-store integration.

*Your interface:* you author the *risk-gated release contract*; they implement the pipeline.

From peer / next-up `security-learning` / `ai-infra-security` (level 35):

- deep adversarial defence at inference edge, judge supply-chain, eval-set exfiltration prevention, MLSecOps.

*Your interface:* you cover adversarial-ML at *working depth* (mod-105) — enough to attack, enough to defend one deployed system; platform-scale MLSecOps escalates to them.

## What you *escalate upward*

To `ai-evaluation-engineer` (peer / next-up, level 35, Governance):

- Release-assurance sign-off, regulator-facing audit-trail methodology, evidence-packaging discipline for external audit.

*Your interface:* you produce the risk-engineering evidence; that role packages it for release-assurance and for the regulator.

To `agentic-safety-engineer` (level 40, Governance):

- Frontier-agent red-team methodology, dangerous-capability evaluations, RSP-shape deployment tiering *at frontier scale*.

*Your interface:* mod-104 and mod-108 cover enterprise / production-agent depth. Frontier depth escalates.

To `senior-ai-governance-architect` (level 50, Governance):

- Control-library architecture, policy-taxonomy design, cross-jurisdiction reconciliation, SoA authorship.

*Your interface:* you contribute in mod-112; you do not design the org-wide control library.

To `head-of-ai-governance` (level 60) / `chief-ai-officer` (level 70):

- Program leadership, board reporting, regulator engagement, staffing / hiring / budget of the governance function.

*Your interface:* you provide summaries and evidence; you brief on request; you do not run the council.

## What is *out of scope*

Two categories are permanently out-of-scope for this role and should be escalated the moment they appear:

- **Legal opinion.** You do not deliver legal opinion. You produce technical evidence a lawyer can rely on. If someone asks "does this violate GDPR?" the answer is *legal counsel decides*.
- **Security operations / DFIR.** SOC-side incident handling (network compromise, credential theft, endpoint response) is not yours. What *is* yours is AI-specific signal — a jailbreak that succeeded, a memorised training-set exposure, a poisoned-model incident. You feed AI-specific signal to SecOps; SecOps runs the SOC.

## The deferral contract — a template you actually write

The contract you produce is a table with four columns (owned / consumed / handed-off / out-of-scope) and one row per obligation in the harm-model + framework map. Its purpose is to make invisible ownership *visible*, so nothing sits unowned by accident. A short excerpt:

| Obligation | Owner | Deferral relationship | Contract details |
|---|---|---|---|
| Author the harm model | You (mod-102) | — | Deliverable to AI risk council |
| Maintain system inventory | `ai-governance-analyst` (L15) | Consumed | You depend on it monthly; you file discrepancies back |
| Author the release-gate spec | You (mod-108 / mod-109) | — | Deliverable; owned by AI risk council |
| Implement the release-gate pipeline | `ai-infra-mlops` (L25 Infra peer) | Handed off | You specify inputs / thresholds; they implement |
| Run the release-assurance review | `ai-evaluation-engineer` (L35) | Escalated up | You supply signed evidence packet |
| Own the org-wide control library | `senior-ai-governance-architect` (L50) | Escalated up | You contribute inputs |
| Decide "is this GDPR-lawful?" | Legal counsel | Out of scope | You supply technical facts on request |
| Run the SOC | Security operations | Out of scope | You send AI-specific incidents into the SecOps queue |

## An anti-pattern catalogue

Behaviours that break the ladder:

- **Doing analyst legwork.** If you are typing inventory rows into a GRC platform, escalate the ask; you are not the intake analyst.
- **Substituting for release-assurance.** You *supply* evidence. If your organisation lacks a level-35 role, that is a staffing gap; do not silently take on release sign-off you cannot defend.
- **Substituting for the architect.** If a governance decision affects multiple systems and multiple jurisdictions, the answer is not for you to unilaterally decide; loop the senior architect (level 50) in.
- **Substituting for legal.** Any sentence starting "we can/can't do X because $LAW says…" is a legal-counsel sentence. Bring the facts, not the ruling.
- **Silent scope creep.** If you accept an assignment three levels above yours because nobody else picked it up, the outcome will not be attributed to under-staffing. Escalate before accepting.

## The upward path

Two natural next-up tracks from this role:

- **`ai-evaluation-engineer` (level 35, Governance family)** — release-assurance / audit-trail depth. Same family, one level up, natural next step for engineers who want to own the auditor-facing side.
- **`agentic-safety-engineer` (level 40, Governance family)** — frontier / agent-scale red-team depth. Same family, two levels up, for engineers who want to specialise into safety engineering rather than compliance engineering.

Sideways cross-training moves that make you more effective inside this role:

- **`ai-eval-engineer` (peer, level 30, AI Engineering family)** for application-layer eval depth,
- **`model-evaluation-engineer` (peer, level 30, ML Engineering family)** for statistical methodology depth,
- **`security-learning` / `ai-infra-security` (level 35)** for deep ML/AI security depth.

## Summary

- You own the *engineering craft of AI risk end-to-end at team scope* — twelve modules, three projects.
- You *consume* analyst legwork from level 15 and ML fundamentals from level 20.
- You *hand off sideways* to peer eval / MLOps / security tracks per the module-by-module contract.
- You *escalate up* release-assurance to level 35, frontier depth to level 40, architecture to level 50, program leadership to level 60.
- Legal opinion and SOC-side DFIR are out of scope.
- Exercise 05 puts you through writing the deferral contract for a concrete system.
