# Contributing to RSP / Preparedness-Style Deployment Tiering

Mod-102 chapter 04 introduced the Anthropic Responsible Scaling Policy, the OpenAI Preparedness Framework, and the Google DeepMind Frontier Safety Framework as *shape templates* for enterprise deployment tiering. This chapter is where you *contribute* concrete engineering work into a tiering document your organisation actually operates: pre-registered thresholds, rollback criteria, dangerous-capability tripwires. It is also where the deferral contract from mod-101 chapter 06 becomes explicit — you contribute; you do not own frontier-scale thresholds (that is `agentic-safety-engineer` at level 40), and you do not own the programme-wide tiering policy (that is `head-of-ai-governance` at level 60).

## Motivation

Team-scope tiering fails in three characteristic ways at level 25:

- **Cargo-culted frontier thresholds.** A team lifts CBRN uplift language or autonomous-replication triggers directly from the RSP or Preparedness text into a document for a customer-support assistant. The thresholds are irrelevant and the tiering is not defensible.
- **Post-hoc tiering.** The deployment gets built, the demo goes well, someone asks "what tier is this?" and the team writes the tier definition to fit what already shipped. The tiering has no forcing function.
- **Undocumented rollback.** The tiering names a promotion criterion but no rollback criterion; when a monitor fires, there is no pre-committed action, so the team argues in real time about whether to revert.

The productive contribution is a *pre-registered, testable, reversible* tiering document at team scope, aligned to the frontier-lab shape but sized to your actual system. This chapter breaks down the four contributions the level-25 role owns, the frontier-scale contributions that escalate to level 40, and the programme-level ownership that lives at level 60.

## What the role contributes at level 25

Four pieces of engineering work belong to the level-25 risk engineer inside a tiering document:

### 1. Pre-registered thresholds

The tiering names the deployment tiers (T0 through T4 in the mod-102 ch 04 skeleton), and for each tier names the *thresholds* that must be met to enter it. The risk engineer contributes the threshold values by *reading them off existing artefacts*:

| Threshold source | Threshold form |
|---|---|
| Register (mod-103) | "All rows for this system at or below appetite" (or a specific named exception with dated waiver) |
| Guardrail effectiveness (mod-108) | "Guardrails G-01 through G-N attain ≥ [F1 threshold] on the current retention corpus" |
| Adversarial-ML delta (mod-105) | "Clean-vs-worst-case accuracy delta ≤ [percentage points] on the named perturbation set" |
| Fairness slices (mod-106) | "Per-slice delta ≤ [percentage points] on named slices with n ≥ [minimum]" |
| Privacy findings (mod-107) | "Canary-extraction rate ≤ [percentage] on the current retention canary set" |
| Red-team programme (mod-104) | "Last N red-team runs closed all high-severity findings; regression suite green" |
| Monitor health (mod-109) | "Critical monitors green for [duration]; no muted tripwires" |
| Incident history (mod-111) | "No open severity ≥ [level] incidents; post-mortem regression fixtures green" |

Each threshold cites its source artefact by reference; the tiering document does not *restate* the numbers, it *pins* them by reference. When the source updates, the threshold updates.

### 2. Rollback criteria

Every tier promotion carries a *rollback criterion* — a pre-registered condition under which the system returns to a lower tier without renegotiation. The risk engineer authors these as inverses of the promotion thresholds, with tolerances:

| Rollback criterion form | Example |
|---|---|
| Threshold breach on a critical metric | "`mon-cs-injection-rate` exceeds 0.5 percent for 3 consecutive days -> revert to T2" |
| Regression on retention / regression corpus | "Regression on retention corpus > 5 percent -> hold at current tier; investigate before further promotion" |
| Incident above severity | "Severity ≥ 2 incident on the system -> revert to T2 pending post-mortem" |
| Monitor mute of a tripwire | "Mute of any T3+ tripwire monitor without risk-council approval -> automatic revert to T2" |
| Framework or obligation change | "New EU AI Act delegated act requiring transparency addendum -> hold at current tier until addendum published" |

Rollback is *mechanical* — a config flip in the release pipeline. If revert requires a human decision, the criterion has not been pre-registered properly.

### 3. Dangerous-capability tripwires

Tripwires are the *early-warning signals* that a capability regression or emergence would breach the tiering assumptions. At team scope (not frontier scale), tripwires are typically:

| Tripwire class | Example at team scope |
|---|---|
| Guardrail-defeat spike | Injection-defeat rate rising above the pre-registered rate on live traffic |
| Novel red-team hit | A red-team run finds a jailbreak class not seen in the regression suite |
| Fairness-slice divergence | A previously in-appetite slice moves above appetite due to distribution shift |
| Privacy-canary leak | A canary appears in output logs for the first time |
| Autonomy escalation | An agentic variant executes a tool call the tiering did not authorise |
| Supply-chain change | Underlying model provider ships a version bump; capability delta unknown until re-evaluated |

The risk engineer names the tripwires, wires them into monitoring (mod-109), and pre-registers the escalation action (usually: hold current tier, notify risk council, run investigative evaluations). What the risk engineer does *not* do at team scope is define frontier-scale dangerous-capability categories (CBRN uplift, autonomous replication, R&D acceleration) — those escalate.

### 4. The tiering document itself, at team scope

The document is a compact artefact — typically 5-15 pages — that names the tiers, thresholds, rollback criteria, tripwires, and decision-authority per promotion. A working section list:

- Purpose and scope.
- Systems covered.
- Tier definitions (T0-T4, per mod-102 ch 04 skeleton, sized to the systems).
- Per-tier: pre-triggered safeguards, thresholds, decision authority, evaluation cadence.
- Rollback criteria.
- Tripwire register (name, source, threshold, action).
- Governance interface (which tier promotion is council-approved; sign-off routing per this module ch 02).
- Escalation to `agentic-safety-engineer` (L40) and to `head-of-ai-governance` (L60) — the deferral contract, extended.
- Framework crosswalk pointers (to this module ch 05).
- Change log.

The document is versioned, hashed, referenced from council decision-log entries (this module ch 02), and reviewed at least quarterly.

## A worked short-form tier promotion — T2 to T3

Extract from a working tiering document for the customer-support RAG assistant, illustrating the shape of a single promotion:

```markdown
### T2 -> T3 promotion (customer-support RAG assistant)

**From tier:** T2 — limited external, small external cohort, human-in-the-loop.
**To tier:** T3 — general external customer availability, human-in-the-loop.

**Thresholds (all must hold at the moment of promotion, per source artefacts):**
- Register: 0 rows above appetite for cs-rag-assistant (mod-103 aggregation).
- Guardrails: G-INJ-01 F1 ≥ 0.92 (retention corpus n ≥ 2,000); G-PII-01 F1 ≥ 0.95; G-TOXIC-01 F1 ≥ 0.90.
- Adversarial-ML: paraphrase-set worst-case accuracy drop ≤ 8 percentage points.
- Fairness: EN dialect slices per-slice accuracy delta ≤ 4 percentage points; caveats current.
- Privacy: canary-extraction rate ≤ 0.05 percent on retention canary set.
- Red-team: last run closed all high-severity findings; regression suite green.
- Monitoring: `mon-cs-injection-rate` < 0.3 percent (30-day rolling); no muted tripwires; digest current.
- Incidents: no open severity ≥ 2; post-mortem regression fixtures green.
- Cards: model, system, dataset cards current and published (this module ch 03).

**Rollback triggers (any one triggers automatic revert to T2, config-flip in release pipeline):**
- `mon-cs-injection-rate` > 0.5 percent for 3 consecutive days.
- Severity ≥ 2 customer-visible incident.
- Regression on retention corpus > 5 percent.
- Mute of any T3+ tripwire monitor without risk-council approval.
- Any per-slice fairness delta widening beyond 6 percentage points.

**Tripwires (early warning; monitored per mod-109):**
- Injection-defeat rate rising sharply outside 30-day rolling baseline.
- Novel jailbreak-class hit in a red-team run.
- Distribution shift on the input mix causing fairness-slice divergence.
- Any canary appearance in output logs.
- Supply-chain: underlying model provider version bump.

**Decision authority:** Risk council + `ai-evaluation-engineer` (L35) release-assurance sign-off.
**Evaluation cadence at T3:** Per release + monthly deep-dive on guardrail effectiveness, quarterly council review.
**Escalation exits:**
- Any tripwire indicating autonomy escalation -> `agentic-safety-engineer` (L40).
- Any change to the enterprise tiering policy or the tier definitions -> `head-of-ai-governance` (L60).
```

That is the pattern. Every promotion in the document has the same structure. Numeric thresholds have citations to source artefacts; rollback criteria are mechanically enforceable; tripwires are wired.

<!-- needs-research: the specific numeric thresholds above are illustrative for the running example; real thresholds must be calibrated to the retention corpus size, baseline effectiveness numbers, and the enterprise appetite specific to each programme. -->

## The level-40 boundary — what escalates to `agentic-safety-engineer`

The level-40 role owns the frontier / agent-scale extension of everything in this chapter:

- **Frontier capability categories.** CBRN uplift, autonomous replication, R&D acceleration, self-exfiltration, deceptive alignment — these are not team-scope enterprise thresholds. If a deployment enters territory where any of these categories are plausibly in scope, the tiering escalates.
- **Frontier evaluation harnesses.** RSP / Preparedness / FSF-associated evaluation harnesses (some public, some proprietary; some maintained by the frontier developer, some by AISI-style bodies) are level-40 territory.
- **Multi-model / cross-agent dangerous capability evaluation.** Level 25 sizes the eval to one deployed system; frontier-scale eval spans model families and agent scaffolds.
- **Ruling on frontier-tier deployment consequences.** Halting a frontier release is a governance decision at frontier scale that this role does not make; the escalation contract is unambiguous.

The team-scope tiering document should *name* these escalations explicitly — a section titled "Escalation to `agentic-safety-engineer` (L40)" lists which conditions trigger it and what evidence is packaged for the hand-off.

<!-- needs-research: name specific public frontier-evaluation programmes (UK AISI, US AISI, Frontier Model Forum working outputs) and their engagement models for downstream deployers; these change with each announcement cycle. -->

## The level-60 boundary — what belongs to `head-of-ai-governance`

The level-60 role owns the *tiering policy at programme scale*:

- **The tier definitions themselves.** Whether T0-T4 is the right cardinality, what each tier's canonical safeguard set is, and how tiers are named across the organisation.
- **Cross-programme promotion authority.** Any promotion into T3 or T4 for any system in the organisation lands at the council chaired by this role.
- **External commitments.** Any commitment made to regulators (EU AI Act post-market, sector obligations) or to the public about deployment-tier discipline is programme-level.
- **The audit-facing statement of tiering intent.** ISO/IEC 42001-style AIMS documentation of tiering policy is programme-level.

At level 25 you *contribute* to the tiering document for the systems in your programme's scope; the enterprise-wide tiering policy is above you.

## The frontier-lab documents, revisited from mod-102

Mod-102 chapter 04 walked the four documents (Anthropic RSP, OpenAI Preparedness, Google DeepMind FSF, Frontier Model Forum publications) as shape templates. Two things change when you re-read them from the level-25 tiering contribution angle:

- **Read the mitigation / safeguard commitments closely.** These are the pattern for the pre-triggered safeguards column in your tiering document. Even where you cannot copy the specifics, the *categories* of safeguard (deployment access controls, inference-time monitoring, evaluation gates, external red-team engagement, response-plan pre-registration) are the categories your document should cover at your scale.
- **Read the evaluation-cadence commitments closely.** How often the frontier lab commits to evaluate; who conducts the evaluation; how updates are disclosed. Your enterprise cadence is different in scale but should be commensurate in *discipline*.

The Frontier Model Forum publications, in particular, are where cross-industry vocabulary and current-consensus shape live. When your tiering document names a safeguard, using the FMF-consensus terminology (where it exists) makes the document more portable across teams and vendors.

<!-- needs-research: pin the current FMF publications list at authoring time; the working papers and issue briefs grow across cycles. -->

## Interfaces to earlier modules

- **mod-101** — the deferral contract in ch 06 sets the level-25 / level-40 / level-60 boundary this chapter operationalises for tiering; sector-flavour overlays (SR 11-7, FDA GMLP + PCCP) add jurisdictional constraint that must be threaded through tier promotions.
- **mod-102** — chapter 04 is the shape template; this chapter closes the loop by turning shape into a working team-scope contribution.
- **mod-103** — register aggregation reports are the source of the "above appetite" and "residual state" threshold citations.
- **mod-104** — red-team programme run reports and the regression suite state are direct threshold sources.
- **mod-105** — adversarial-ML delta numbers pin the perturbation-tolerance thresholds.
- **mod-106** — fairness slice measurements pin per-slice thresholds.
- **mod-107** — privacy findings (canary extraction, MIA rates) pin the privacy thresholds.
- **mod-108** — guardrail effectiveness numbers are the most frequently cited threshold source.
- **mod-109** — monitors are how rollback and tripwires actually fire in production; the tiering document names monitor IDs and thresholds directly.
- **mod-110** — MRM tiering (SR 11-7 model-materiality) and supply-chain change tripwires (provider version bumps) integrate with deployment tiers at the same document.
- **mod-111** — incident history and open-severity status are direct threshold inputs; the rollback path is the same path the incident-response playbook uses.

Chapter 05 continues into the framework-crosswalk artefact that consumes all of the above.

## Summary

- Deployment tiering at team scope contributes four concrete artefacts: pre-registered thresholds, rollback criteria, dangerous-capability tripwires, and the tiering document itself.
- Thresholds are *citations* to source artefacts (register, guardrails, adversarial ML, fairness, privacy, red-team, monitoring, incidents), not fresh numbers.
- Rollback is mechanical — a config flip — not a human decision at incident time.
- Tripwires are wired to monitors (mod-109) with pre-registered escalation actions.
- Frontier-scale capability categories, frontier evaluation harnesses, and frontier-tier deployment decisions escalate to `agentic-safety-engineer` (level 40).
- Tier-definition cardinality, cross-programme promotion authority, external commitments, and audit-facing tiering-policy statement live at `head-of-ai-governance` (level 60).
- The frontier-lab documents (RSP, Preparedness, FSF, FMF publications) remain the shape templates; read them again from the contribution angle before writing your document.
- Exercise 04 walks a full T2 -> T3 tier-promotion contribution for the running system, with thresholds cited, rollback criteria pre-registered, and tripwires wired.
