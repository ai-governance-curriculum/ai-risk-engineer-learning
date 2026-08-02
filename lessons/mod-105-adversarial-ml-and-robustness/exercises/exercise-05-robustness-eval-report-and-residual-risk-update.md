# Exercise 05 — Robustness-Eval Report and the Residual-Risk Register Update

**Estimated effort:** 3 hours

## Objective

Compose the shippable **robustness-eval report** chapter 05 defines for the running system, pulling attacks / defences / numbers from exercises 02 (evasion), 03 (poisoning), and 04 (API-surface), and file the corresponding **residual-risk update** into the mod-103 register. The report has seven sections — methodology, attack success, transferability, perturbation-budget curves, defence deltas, residual-risk update, boundary and handoffs — every section filled, every scenario tied back to the mod-102 harm model and the mod-103 register, every co-owned control tied to the chapter-06 handoff contract. The register update is what buys the eval its keep: scenarios move from *estimated residual* to *quantified residual*, and control-defeated columns get real numbers.

## Prerequisites

- Chapter 05 read; chapter 06 read for the handoff-contract shape; chapter 01 available for the cross-tag; chapters 02–04 available for source numbers.
- Exercises 01–04 completed. This exercise consumes their outputs; if any prior exercise is incomplete, the corresponding section of the report is blocked and must be flagged, not filled with placeholders.
- Access to the mod-103 register (from mod-103 exercise 05) with write permission to update residual / control-defeated columns and to open new register rows as needed.
- Access to the chapter-06 handoff-contract table for the running system so specifier / implementer / operator / evidence rows can be added or updated for the new controls from exercise 04.
- Access to the mod-102 harm model to verify every scenario in the report ties back; and to file harm-model updates if the eval surfaced a scenario the harm model does not yet name.
- The mod-109 monitor spec repository (or a stand-in) so the monitor entries the report nominates land somewhere.
- The mod-111 runbook repository (or a stand-in) so incident-response threshold entries land somewhere.

## Problem statement

Author the report as a single, versioned artifact for the running system this cycle. The report reads from the four upstream artifacts (cross-tag, evasion, poisoning, API-surface) and writes into three downstream registers (mod-103 residual-risk, chapter-06 handoff contract, mod-109 monitor + mod-111 runbook nominations). The point of the exercise is *round-tripping*: numbers in from the tools, numbers out into the register.

The output is not a slideware summary. Every attack scenario reports a threat model, an attack number with a CI, a transferability rate where measured, a perturbation-budget curve (or the category-equivalent), a defence delta per lever, and a residual-risk-column update. Every defence entry links to a monitor spec and (where applicable) an incident-response runbook branch. Every co-owned control has an owner from the chapter-06 contract.

## Requirements

Produce all nine items below. Missing any is an audit finding on the report.

1. **Report shell** — `risk/mod-105/<system-id>/report/robustness-eval-<version>.md` where `<version>` is a semver-style tag (e.g., `v1.0.0`) plus a date stamp. Sections in order:
   1. *Methodology* — the threat-model table for every attack in scope: attack name, NIST AI 100-2 category, MITRE ATLAS technique id, OWASP ML (and LLM if applicable) Top 10 entry, knowledge / capability / goal axes, norm and budget (evasion) / poison-rate sweep (poisoning) / query-budget sweep (extraction) / shadow-training budget (MIA), runner name and pinned version, clean-eval-set provenance and fingerprint.
   2. *Attack success rates* — per-attack primary numbers with 95% CIs. Evasion: success rate at each ε with the curve embedded. Poisoning: success rate at each poison rate. Extraction: surrogate agreement at each query budget. MIA: advantage plus TPR at 0.1% and 1% FPR with ROC embedded. Inversion (if in scope): fidelity metric.
   3. *Transferability* — the sub-table with source model, target model, attack, success on each, transfer ratio. Minimum one row.
   4. *Perturbation-budget curves* — the primary artifact for evasion; equivalent budget-dimension curves for the other categories (poison-rate curve, agreement-vs-queries curve, ROC-per-defence for MIA). Embed the plots.
   5. *Defence deltas* — the panel table: attack, baseline number, defence, defence-enabled number, delta, clean-accuracy or utility cost, latency / cost delta, notes. Every defence measured against every attack it plausibly addresses, adaptively where applicable.
   6. *Residual-risk update* — the register-facing table: scenario id, inherent (before / after; usually unchanged), residual (before / after), control-defeated variants (populated with quantified attack-success-under-defence-absence numbers), monitors added / adjusted, incident-response thresholds added / adjusted.
   7. *Boundary and handoffs* — the chapter-06 contract row diff for every new / changed control; the mod-107 handoffs (formal DP-accounting, LiRA depth) with intake references; the mod-108 candidate-guardrail nominations (output redaction, STRIP-style detector); the mod-109 monitor specs; the mod-111 incident-response runbook nominations; the sec-L35 rate-limit and signed-artifact deliverables.
2. **Register-update patch** — `risk/mod-105/<system-id>/report/register-update-<version>.md` (or the format the register repo expects). One block per touched register row: row id, inherent (before / after), residual (before / after), control-defeated column update, monitor id linked, runbook id linked, evidence-artifact pointer (which section of which report supplies the number). If a row does not exist and needs to be opened, cite the new row id.
3. **Handoff-contract diff** — the diff / patch against the chapter-06 handoff-contract table for the system: rows added / updated with specifier (this module) / implementer (sec-L35, ML platform, ML team) / operator (on-call rotation) / evidence artifact (report section, monitor id, training-config hash).
4. **Monitor spec nominations** — one file per nominated monitor at `risk/mod-105/<system-id>/report/monitors/<monitor-id>.md`: what the monitor measures, at what cadence, with what alert threshold, and what defence-delta becomes invalid when the monitor alerts. Nominations are consumed by mod-109; the monitor itself is wired there.
5. **Incident-response runbook nominations** — one file per nominated runbook branch at `risk/mod-105/<system-id>/report/runbooks/<runbook-id>.md`: which control-defeated variant it triggers on, the containment steps, and the notification chain. Nominations are consumed by mod-111.
6. **Confidence intervals throughout** — every number in the report carries a CI (or an explicit small-set caveat); single-run point estimates fail acceptance.
7. **Adaptive-attack citations for defence deltas** — every defence-delta row cites the adaptive-attack construction used (from exercise 02 / 03 / 04), following the Carlini et al. "On Evaluating Adversarial Robustness" guidance. A defence delta with no adaptive-attack citation is treated as an inflated number.
8. **Version tag and change log** — the report ships with a version and a change log; regeneration triggers per chapter 05 (new model candidate, new material attack, new defence, quarterly rerun) are enumerated.
9. **Executive brief** — the first section of the report, one page, for the risk council: what changed on the register, which residual-risk columns moved and in which direction, which new monitors are wired, which handoffs are open. The brief is *derived* from the sections below; it does not introduce new numbers.

Cross-cutting rules:

- **Every section is filled or explicitly deferred with a reason.** A missing section is treated as unmeasured, not as "no findings." A deferral must name the block and the owner.
- **Every scenario in the report ties back to a mod-102 harm-model row.** A report scenario with no harm-model row is either a harm-model gap (file the update) or out of scope (remove from the report). The chain is: harm model → cross-tag row → attack scenario in the report → register row updated.
- **Every co-owned control has a row on the chapter-06 handoff contract.** A defence lever specified in the report with no contract row is an orphan-owned control; the report is not shippable.
- **The report is versioned; the register update timestamps to the report version.** "We did the eval once" is not the module's target; "we run the eval; here's the trend" is.
- **Numbers in the executive brief must reconcile with the tables in the sections.** The brief is derived, not authored.

## Starter guidance

- **Draft the residual-risk-update table first, then work backwards.** The primary output of the eval is the register update. Filling that table first shows which numbers you still need from exercises 02–04 and which handoffs are still open. Everything else in the report is *evidence* for the table.
- **A residual number should move because a real defence lever was measured, not because the eval was optimistic.** If a residual goes from Medium (estimated) to Low (quantified) with no defence delta driving the move, the estimator was wrong — not the eval. Argue the move in `notes`.
- **Control-defeated columns are populated with attack-success-under-defence-absence numbers.** They are *not* worst-case estimates. If your eval didn't measure it, mark the column `unmeasured:<reason>` — do not fill it with a plausible-sounding number.
- **Monitor specs are small but load-bearing.** A defence with no monitor is a defence with no ongoing validity claim; the residual number will drift and nobody will know. Nominate a monitor per defence, even if the monitor is "check quarterly that the defence is still enabled" (which is a real monitor).
- **Runbook branches are nominated per control-defeated variant, not per attack.** If PGD-at-ε=8/255 breaks adversarial-training, the runbook is for "adversarial-training-defeated" — not for "PGD detected."
- **The chapter-06 handoff-contract diff is a small artifact but is the highest-leverage output.** Filling in specifier / implementer / operator / evidence per lever is what turns the report from an internal document into an interface document with the sec-L35 peer.
- **Do not oversell the executive brief.** The council is reading it to make decisions; if the residual is still High, say so. Overclaiming here is the reason the register drifts.
- **Regenerate cadence matters.** Chapter 05 names the four triggers (new model candidate, new attack, new defence, quarterly). Wire them into your team's change-management workflow, even as a stub. A one-time report is not the deliverable.

## Acceptance criteria

The submission is accepted when:

- The report exists at the specified path, tagged with a version and a date, and has all seven required sections filled.
- Every attack scenario in the report has a `scenario_id` (from exercise 01) and traces back to a mod-102 harm-model row and forward to a mod-103 register row.
- Every reported number has a 95% CI or a small-set caveat.
- Every defence-delta row cites the adaptive-attack construction used.
- The register-update patch touches at least the register rows whose residual columns the eval quantified; new rows opened during the exercise are cited by id.
- The chapter-06 handoff-contract diff has a row for every co-owned control specified in the report, with specifier / implementer / operator / evidence filled.
- Monitor spec files exist for every nominated monitor and reference the defence-delta they protect.
- Incident-response runbook nominations exist for every control-defeated variant the report populates.
- The executive brief is one page, derived from the tables, and reconciles numerically.
- The change log names the triggers for the next regeneration.
- A reviewer given only the report can, in twenty minutes, answer: which scenarios moved residual and by how much, which defences are shipping and with what monitor, which handoffs are open with which peer, and what would trigger the next regeneration.

## Stretch goals

- Draft the **next-cycle plan** as an appendix: which attacks will be added to the panel next cycle (Auto-PGD, clean-label poisoning, LiRA), which defences to baseline next (input-transform on evasion, Neural Cleanse on poisoning, DP-training at a different ε), and which handoffs to close (mod-107 LiRA depth, mod-108 STRIP catalogue entry). The plan is what makes the report *serial* rather than one-off.
- Author a **red-team-vs-eval reconciliation** — one paragraph mapping how this module's evasion results relate to mod-104's LLM red-team findings on the dual-domain scenarios (safety-classifier evasion that is also a jailbreak). The two artifacts should tell the same story; the reconciliation is the check.
- Produce a **safety-card-derivable summary** — a small block of numbers the external safety card can derive from the internal report, plus the rules for what stays internal (adaptive-attack constructions, specific defence hyperparameters). Chapter 05 is explicit that the report is not the safety card; this stretch draws the derivation line.
- Wire the **quarterly-regeneration trigger** as a scheduled job stub with the change-management workflow. Even as a documented ticket, the trigger prevents the "we did it once" failure mode.
- If the running system's harm model rates privacy risk as material, draft the **mod-107 depth-work handoff spec** — the exact intake artifact mod-107 will need (baseline MIA advantage curve, DP-SGD run reference, membership / non-membership set fingerprint) to run the formal (ε, δ) accounting.

## Deliverable format

A single markdown document `mod-105-ex05-robustness-eval.md` that indexes the artifacts, plus the artifacts themselves. The index document contains:

1. *System and eval cycle* — one paragraph naming the system, the release under eval, the report version, and the cycle trigger (new model / new attack / new defence / periodic).
2. *Executive brief* — the one-page summary from the report, embedded.
3. *Section fill-status* — the seven report sections and their fill state (filled / deferred:<reason>).
4. *Register update summary* — the count of rows touched, the count of new rows opened, and the net residual-column movements (per direction).
5. *Handoff-contract summary* — the rows added / updated on the chapter-06 contract for the system, with the specifier / implementer / operator / evidence filled.
6. *Monitor and runbook nominations* — the nominated ids with a one-line description each.
7. *Traceability index* — every scenario in the report to its harm-model row and register row.
8. *Reproducibility* — pointers to the exercise-02 / 03 / 04 artifacts, tool versions, seeds, and dataset fingerprints.
9. *Open handoffs* — mod-107, mod-108, mod-109, mod-111, and sec-L35 deliverables, each with an owner (this role, peer role, or "unassigned:<reason>").
10. *Change log and next-cycle plan* — regeneration triggers wired.

## Submission

Attach the index markdown, the report, the register-update patch, the handoff-contract diff, and the monitor / runbook nomination files to the module submission channel. If the system is under NDA, redact system identifiers but preserve the section shape, the numbers with CIs, the residual-column movements, the handoff-contract row shape, and the monitor / runbook nominations — a reviewer cannot review a robustness-eval report whose numbers and handoffs are blanked out.

The eval is not shippable without the co-authored chapter-06 contract diff. If the diff is missing, the report is a research artifact, not an engineering deliverable.
