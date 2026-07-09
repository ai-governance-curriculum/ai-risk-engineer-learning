# Exercise 05 — Frontier-Lab RSP / Preparedness / FSF Comparative Read

**Estimated effort:** 3 hours

## Objective

Produce a side-by-side comparative read of Anthropic's Responsible Scaling Policy, OpenAI's Preparedness Framework, and Google DeepMind's Frontier Safety Framework, and use the extracted *shape* to write a first-draft **enterprise deployment-tiering table** for your running system. The output is what a level-25 harm-modeller hands to the risk council for the tiering conversation.

## Prerequisites

- Chapter 04 (this module) read.
- Exercises 01-04 completed — the taxonomy-tagged harm catalog is the input.
- Access to the current published versions of:
  - Anthropic Responsible Scaling Policy — <https://www.anthropic.com/rsp>
  - OpenAI Preparedness Framework — <https://openai.com/safety/preparedness/>
  - Google DeepMind Frontier Safety Framework — <https://deepmind.google/discover/blog/introducing-the-frontier-safety-framework/>
  - Frontier Model Forum publications — <https://www.frontiermodelforum.org/>
- Note the version and publication date of each document you consult.

## Problem statement

Produce two artifacts:

1. A **comparative-read table** covering RSP, Preparedness, and FSF along the six-question axis from chapter 04:
   - What capability categories does it track?
   - How are the thresholds defined?
   - What safeguards attach at each threshold?
   - Who decides that a threshold has been crossed?
   - What is the deployment consequence of crossing a threshold?
   - How is the framework itself updated?
2. A **first-draft enterprise deployment-tiering table** for your running system that lifts the shape (not the thresholds). At minimum name five tiers (POC → high-stakes), and for each tier name:
   - The system state that defines the tier.
   - The pre-triggered safeguards required to hold the tier.
   - The decision authority who signs promotion into the tier.
   - The evaluation cadence and scope required to keep the tier.

Additionally, produce a **residual-map note** identifying the two-to-five scenarios from your harm catalog whose depth requires escalation to `agentic-safety-engineer` (level 40) — the L25 → L40 handoff from chapter 06.

## Requirements

- Output as `mod-102-ex05-frontier-shape-comparative.md`.
- The comparative-read table has three rows (one per document) and six columns (the six questions above). Cite the version of each document used.
- The enterprise tiering table has at least five tiers × four columns. Every safeguard-per-tier cell references at least one harm-catalog row-ID from exercises 02-04 or an explicit *pre-requisite* obligation from mod-101 (a NIST RMF sub-category or an EU AI Act article number).
- Include a short *what we didn't copy* section (three-to-five bullets) explaining which specific frontier thresholds and evaluation harnesses you deliberately did *not* import, and why. This is the anti-cargo-cult section from chapter 04.
- Include a *residual-map* section listing scenarios where L40 handoff is required, with a one-line escalation reason each and a residual-owner assignment on your side.
- Cross-reference the tiering table to sector obligations if applicable — SR 11-7 model-materiality tiering (mod-110), FDA GMLP predetermined change control (mod-101 ch 05), EU AI Act deployer obligations (mod-101 ch 04).

## Starter guidance

- Read all three risk-tier documents *end to end once* before you open a table. The vocabulary is precise; skimming will make you confuse ASL, Preparedness threshold levels, and FSF CCLs with each other.
- Fill the comparative table one column at a time (not one row at a time). Answering "how are thresholds defined?" across all three documents at once forces you to see the differences.
- For the tiering table, start with tier T2 or T3 (limited external / general external) and work outward. T0 (POC) is almost always thin; T4 (high-stakes) is what you have to justify most heavily to the risk council.
- For each tier, walk your harm catalog and ask: *which rows must be closed before promotion into this tier?* Then check the reverse: *which rows must be closed before promotion into the next tier?* The difference set is what defines the tier.
- If your running system is a purely internal tool, your tiering may only need three tiers, not five. That is acceptable; document why the higher tiers do not apply.
- If you find that your enterprise thresholds resemble frontier thresholds too closely, you have probably imported too much — reset and start from your system's specific decision points (advisory-vs.-decision, human-in-the-loop-vs.-autonomous, internal-vs.-external, sector-regulated-vs.-general).
- The residual-map is small on purpose. If more than five scenarios feel like L40 escalation, either your system is genuinely frontier-scale (escalate the *program*, not the map) or your L25 scope has been mis-set (revisit chapter 06).

## Acceptance criteria

The submission is accepted when:

- The comparative-read table has three rows and six columns filled, with version citations for each document.
- The tiering table has ≥ 5 tiers × 4 columns filled, every safeguard-per-tier cell cites a harm-catalog row-ID or a mod-101 obligation.
- The *what we didn't copy* section is present with concrete examples of specifically-declined thresholds.
- The residual-map lists 2-5 scenarios with escalation reasons and residual-owner assignments.
- The document is ≤ 6 pages printed — the tiering conversation is not helped by longer artifacts.

## Stretch goals

- Add a *composability* section that shows how your enterprise tiers compose with:
  - EU AI Act deployer obligations (mod-101 ch 04).
  - SR 11-7 model-risk tiering (mod-110 preview).
  - FDA PCCP change-control (mod-101 ch 05).
- Draft *one* threshold-crossing scenario — walk through what happens the moment a system operating at Tier 2 exhibits a capability that would require Tier 3 safeguards. Who is notified? What is halted? What evidence is filed?
- Sketch the *update mechanism* for your enterprise tiering: how the tiering table itself is versioned, who reviews changes, how transparency to internal stakeholders is maintained. Mirror the mechanism you noted in the comparative read.

## Deliverable format

A single markdown document with the following sections:

1. *Document versions consulted* — RSP, Preparedness, FSF, FMF publications; dates and URLs.
2. *Comparative-read table* — three rows × six columns.
3. *What we did *not* copy* — three-to-five bullets on declined imports.
4. *Enterprise deployment-tiering table* — five tiers × four columns with harm-catalog / mod-101 references.
5. *Residual-map* — 2-5 scenarios escalated to L40 with reasons and residual owners.
6. *Composability notes* (stretch) — sector obligation reconciliation.
7. *Update mechanism* (stretch) — how the tiering itself is maintained.

## Submission

Attach the markdown to the module submission channel. This is the artifact you take into the risk-council conversation about deployment tiering; treat the review as the first read of it in that setting.
