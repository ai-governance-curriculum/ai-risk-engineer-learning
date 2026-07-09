# Exercise 05 — Role Scope and Deferral Contract

**Estimated effort:** 2 hours

## Objective

Write the *deferral contract* for the AI Risk Engineer role (level 25, Governance family) against your running system. The contract makes visible what you own, what you consume from below (`ai-governance-analyst`, `ml-engineer`), what you hand off sideways (peer eval / MLOps / security tracks), what you escalate upward (`ai-evaluation-engineer` L35, `agentic-safety-engineer` L40, `senior-ai-governance-architect` L50, `head-of-ai-governance` L60), and what is out of scope (legal counsel, SOC-side DFIR).

You will also identify **role-boundary risks** — the places where your organisation currently *doesn't* have a role at the required level, and something is silently landing on the wrong person.

## Prerequisites

- Chapter 06 (this module) read.
- Exercises 01-04 completed. The running system is the same throughout.
- Familiarity with your own organisation's staffing, at least at the level of "who does the intake?", "who owns the deploy pipeline?", "who signs the release?" — you can produce a deferral contract for a *hypothetical* org if you are not currently inside one.

## Problem statement

Produce a single markdown document `mod-101-ex05-deferral-contract.md` with the following sections:

1. **Running system + org sketch** — one paragraph on the system, one paragraph on the org structure surrounding it (real or hypothetical), one paragraph on the current staffing map by level for the AI-governance surface.
2. **Owned obligations** — a table of every obligation *you* own on this system, drawn from your exercise 01 / 02 / 03 / 04 output. Each row cites the framework, the obligation, the artifact, and the delivery cadence.
3. **Consumed obligations** — a table of every obligation you *depend on but do not own* — analyst intake / inventory, ML fundamentals, prerequisite engineering. Each row cites the level-15 or level-20 role that owns it, the artifact you consume, and what you do if the upstream is late.
4. **Sideways hand-offs** — a table of every hand-off to a peer role (`ai-eval-engineer` L30, `model-evaluation-engineer` L30, `ai-infra-mlops` L25, `security-learning`/`ai-infra-security` L35). Each row cites the interface (what you specify vs. what they implement) and the artifact that closes the interface.
5. **Escalations upward** — a table of every escalation to level 35, 40, 50, 60. Each row cites when the escalation is triggered and what you supply.
6. **Out of scope** — the two categories (legal, SOC-side DFIR) plus any organisation-specific out-of-scope items. Each row cites who owns it.
7. **Role-boundary risks** — the interesting part. For each layer of the ladder, ask: *does my organisation have a role at this level?* If not, name the risk explicitly. Example: "We do not currently staff an `ai-evaluation-engineer` at level 35. Release-assurance sign-off is silently landing on the AI risk council; I flag this as a role-boundary risk in mod-112."
8. **Sign-off** — one paragraph on what an executive would need to accept for this contract to be operative.

## Requirements

- Every row in every table must cite a *specific* artifact (using the paths you used in exercises 01-04). No abstract "I own risk assessment" — always the artifact and its path.
- The role-boundary-risks section must be non-empty. If you claim your organisation has every level fully staffed, name the individuals or teams to justify it; a "no risks" answer is usually a missed observation.
- Every consumption row must include a fallback — what you do when the analyst is unavailable or the ML fundamentals gap surfaces mid-work.
- Every escalation row must include the trigger — the condition under which you escalate. Silent escalation is not a contract.
- Every out-of-scope row must name the actual receiver (which legal team, which SOC).

## Starter guidance

- Read chapter 06's example contract (the four-column excerpt) and reproduce the shape, populated for your system.
- If you are hypothetical, model an org large enough to have `ai-governance-analyst`, `ml-engineer`, `ai-eval-engineer`, and `ai-infra-mlops` roles staffed — many mature enterprises have this and are still short on levels 35+. That is a realistic scenario.
- The role-boundary risks section is often the most valuable part of the contract for your management chain. Do not soften it — being explicit about missing roles is what gets them staffed.
- Do *not* write "TBD" anywhere. If you don't know who owns something, either research or explicitly file it as a role-boundary risk.

## Acceptance criteria

- All eight sections populated.
- Every row cites a specific artifact and — where relevant — a specific individual, team, or role.
- Role-boundary risks section names at least one gap and one mitigation.
- No section says "TBD."
- The contract is readable end-to-end by a technical executive in under 15 minutes.
- The document ends with a *sign-off* paragraph the executive can put their name to (or explicitly not).

## Stretch goals

- Turn the contract into a *RACI* matrix keyed on the framework obligations from exercises 01-04.
- Produce a *staffing gap memo* — one page — that translates the role-boundary risks into a hiring / cross-training ask, targeted at whoever owns the AI governance budget.
- Add a *quarterly review cadence* section that names when the contract is refreshed, who initiates the refresh, and what triggers an out-of-cycle refresh (e.g., "new AI system placed in production," "new regulation goes into force").

## Deliverable format

A single markdown document. Do not include internal-org-chart PDF exports or headcount data if the artifact might leave the organisation; abstract to roles and teams.

## Submission

Attach the markdown file. This closes module 101.
