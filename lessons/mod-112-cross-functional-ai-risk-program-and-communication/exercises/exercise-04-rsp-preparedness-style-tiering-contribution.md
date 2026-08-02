# Exercise 04 — RSP / Preparedness-Style Tiering Contribution

**Estimated effort:** 2 hours

## Objective

Write the **level-25 contribution** into a Preparedness / RSP-style deployment-tiering document for the running system: pre-registered thresholds cited to source artefacts, rollback criteria that fire mechanically, and dangerous-capability tripwires wired into monitoring. The output is one full tier-promotion block (T2 → T3 for the running system, or the tier boundary that fits your system), plus the escalation contract that hands frontier-scale work to `agentic-safety-engineer` (L40) and programme-scale ownership to `head-of-ai-governance` (L60).

## Prerequisites

- Chapter 04 read closely — the four level-25 contributions (thresholds, rollback, tripwires, the tiering document itself), the level-40 boundary, and the level-60 boundary.
- Mod-102 chapter 04 re-skimmed — the frontier-lab shape templates (Anthropic RSP, OpenAI Preparedness, Google DeepMind FSF) and the T0-T4 skeleton.
- Register aggregation reports from mod-103 chapter 05 — you need the current above-appetite state to set the "0 rows above appetite" threshold defensibly.
- Guardrail effectiveness numbers (mod-108), adversarial-ML delta (mod-105), fairness slice deltas (mod-106), privacy findings (mod-107), red-team run outcomes (mod-104), monitor spec (mod-109), incident history (mod-111) — all direct threshold sources.
- Current published versions of at least one frontier-lab document (RSP, Preparedness, or FSF) skimmed for the *shape* of pre-registered safeguards and evaluation cadences; the version numbers evolve so pin the version you read.

## Problem statement

Produce a tier-promotion block for the running system covering **one specific tier promotion** — pick T2 → T3 unless your system's real deployment stage makes a different boundary more meaningful (T0 → T1 for a system still in internal-only mode; T3 → T4 only if your system is genuinely operating at that scale, which for team-scope work at level 25 is uncommon).

The block is a section inside a larger tiering document. For the exercise you author *just this block*, plus the surrounding structural elements chapter 04 lists (short scope statement, escalation contract to L40 / L60, and a placeholder cross-reference to the framework crosswalk from exercise 05).

## Requirements

Produce a single markdown document `mod-112-ex04-tier-promotion.md` with these sections:

### 1. Purpose and scope

- One paragraph: which system(s) the tiering document covers, which tier boundary this block addresses, and the current tier of the system.
- Named systems and current tiers (one line each).
- Explicit note: this is a level-25 contribution, not the enterprise tiering policy (that is L60).

### 2. Tier definitions (short)

Populate the T0-T4 skeleton for the systems in scope. Keep this brief (three lines per tier). Use the mod-102 chapter 04 skeleton as the template. Where a tier does not apply to your system (e.g., your system will never operate at T4 scale), say so explicitly.

### 3. The tier-promotion block

This is the main artefact. Use chapter 04's worked example as the template:

```markdown
### T? -> T? promotion (<system name>)

**From tier:** T?  — one-line description.
**To tier:** T?  — one-line description.

**Thresholds (all must hold at the moment of promotion, per source artefacts):**
- Register: ...
- Guardrails: ...
- Adversarial-ML: ...
- Fairness: ...
- Privacy: ...
- Red-team: ...
- Monitoring: ...
- Incidents: ...
- Cards: ...

**Rollback triggers (any one triggers automatic revert to the lower tier, config-flip in release pipeline):**
- ...
- ...

**Tripwires (early warning; monitored per mod-109):**
- ...
- ...

**Decision authority:** ...
**Evaluation cadence at the higher tier:** ...
**Escalation exits:**
- ...
```

Every threshold must **cite a source artefact** (register aggregation report ID, guardrail effectiveness report ID, red-team run report ID, monitor ID). No fresh numbers.

Every rollback trigger must be **mechanically enforceable** — a config flip in the release pipeline. If the trigger requires a human decision at incident time, it is not a rollback trigger, it is an escalation.

Every tripwire must **name the monitor** it wires into (mod-109 monitor ID) and the **pre-registered escalation action** (usually: hold current tier, notify risk council, run investigative evaluations).

### 4. Escalation contract

A section listing the escalation exits from this tiering block:

- **To `agentic-safety-engineer` (L40)**: which conditions trigger the escalation, what evidence is packaged for the hand-off. Cover at least: (a) any indication of autonomy escalation beyond the tier's authorised tool calls, (b) any indication that a frontier capability category (CBRN uplift, autonomous replication, R&D acceleration) is plausibly in scope for the system.
- **To `head-of-ai-governance` (L60)**: which conditions trigger the escalation. Cover at least: (a) any change to the enterprise tiering policy or the tier definitions, (b) any external commitment (regulator, customer, public) about deployment-tier discipline.

Each escalation exit is a paragraph, not a one-liner — the L40 and L60 partners need to know what they are inheriting.

### 5. Framework crosswalk pointer

A short section that says which RMF sub-categories and ISO/IEC 42001 controls this tiering block satisfies (a forward-reference to exercise 05). At minimum: RMF MEASURE-1.3 (methodology selection at the tiering level), MANAGE-1.2 (prioritisation of risk response), MANAGE-2.1 (risk-response strategy). Real citations get pinned in exercise 05's crosswalk.

### 6. Change log

A header table with columns: `version`, `date`, `author`, `change summary`, `sign-off`. Populate with the first row for this authoring cycle. Every subsequent material change updates the log.

## Starter guidance

- Pull thresholds from the *actual current state* of the source artefacts, not from an aspirational future state. If the current guardrail F1 on the retention corpus is 0.88, the threshold for T3 promotion is not "≥ 0.92" unless you have a treatment plan that gets there. Pre-registering thresholds you cannot meet is worse than pre-registering conservative thresholds you can.
- Rollback triggers are inverses of thresholds, with tolerance. If the threshold is "monitor rate < 0.3 percent," the rollback is "monitor rate > 0.5 percent for 3 consecutive days." The 3-day guard prevents single-day noise from flipping the tier.
- On tripwires, distinguish the *early-warning signal* from the *rollback trigger*. Tripwires escalate to investigation; rollback triggers revert the system. A rate rising sharply is a tripwire; a rate exceeding threshold for the pre-registered duration is a rollback.
- Do not invent frontier-scale tripwires (CBRN uplift, autonomous replication) for a customer-support system. Those escalate to L40 and belong in a different document. Chapter 04's "cargo-culted frontier thresholds" failure mode is exactly this.
- On the L40 escalation exit, name the *evidence bundle* that goes with the hand-off — not just "we escalate." The bundle is at minimum: the tiering block itself, the tripwire that fired, the register rows affected, the last red-team run report, the monitor extract. This is what an L40 partner reads on day one of the hand-off.
- If your system has no plausible L40 escalation (small customer-support LLM with no agentic surface, no dangerous-capability category in reach), still write the section — the *absence* of an escalation trigger is what a real tiering document records. "No conditions currently in scope; re-evaluate on capability change" is a defensible entry.

## Acceptance criteria

The tier-promotion block is accepted when:

- Sections 1-6 are all populated for the actual running system, with no template placeholders.
- Every threshold cites a source artefact (register report ID, evaluation report ID, monitor ID); no fresh numbers.
- Every rollback trigger is mechanically enforceable and specifies the pre-registered duration / condition; no human-decision-at-incident-time triggers.
- Every tripwire cites the monitor ID it wires into and the pre-registered escalation action.
- The decision-authority line names the roles that sign off on the promotion (per chapter 01's decision matrix and sign-off routing).
- The escalation contract sections to L40 and L60 name the trigger conditions and the evidence bundle for the hand-off; they are not one-liners.
- The change log has the first entry for the authoring cycle.
- A release-engineer reading the block can wire the rollback triggers into the release pipeline without further specification.
- A council member reading the block can vote on the promotion decision inside the meeting.

## Stretch goals

- Draft a **second tier-promotion block** for the reverse direction — the T3 → T2 *demotion* path. Real programmes have demotion authority as well as promotion authority, and the demotion block is often what turns out to be missing when an incident forces the discussion. What thresholds trigger a proposed demotion; what evidence is required to defend against it; who signs off.
- Author the **tripwire register** in a separate artefact: one row per tripwire with name, source (which monitor / evaluation), threshold, escalation action, owner, last-refresh date. This is what mod-109 chapter integrates with; the tiering block cites the register rather than restating each tripwire.
- Do a **read-across against one frontier-lab document**. Pick the current published version of the RSP, Preparedness, or FSF; for each safeguard category the frontier document names at the frontier scale, note whether your team-scope block has an equivalent, and if not, whether the absence is defensible (system is not at frontier scale) or a gap. This is chapter 04's "read the frontier-lab documents from the level-25 contribution angle" exercise made concrete.
- If your system has a **supply-chain dimension** (relies on a third-party base model whose version bump is a tripwire), draft the specific tripwire wiring: which provider, which version-bump signal (release notes RSS feed, provider API version field, whatever your programme wires), what the pre-registered action is (typically: hold current tier, re-evaluate the model on the retention corpus before promotion resumes). This closes the loop with mod-110 shared-component review.

## Deliverable format

A single markdown document with the six sections above, in order.

## Submission

Commit the block to the programme's tiering-document repo (or the equivalent workspace). If the tiering block is the *first* addition to a tiering document your programme does not yet have, note that in the commit and file a follow-up to draft the surrounding structural elements (system inventory, cross-block dependencies, whole-document change log). A single tier-promotion block is a shippable artefact; a whole tiering document is a level-25-plus-analyst-support authoring effort that a single exercise does not cover.
