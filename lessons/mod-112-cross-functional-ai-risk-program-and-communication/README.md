# mod-112 — Cross-Functional AI Risk Programme and Communication

**Estimated effort:** 14 hours
**Position in the track:** twelfth and final module; assumes mods 101-111 are done.

This is the integrator module. Mods 101-111 taught the engineering craft one dimension at a time — frameworks, harm modelling, quantification, red-team, adversarial ML, fairness, privacy, guardrails, monitoring, MRM, incident response. Mod-112 is where those dimensions become a *programme*: a cadence, a bill of artefacts, a decision surface, and a sign-off graph that spans product / ML / infra / security / governance / legal at team scope. The chapters cover the operating cadence, the council-facing briefing pack, the external-facing model / system / dataset card, the deployment-tiering contribution, the RMF / ISO 42001 evidence crosswalk, and the certifications portfolio that closes the track.

The programme you assemble here is what the risk council reads on cadence, what the external audit consumes for AIMS conformity, what the release-assurance engineer at level 35 packages for regulator engagement, and what the head of AI governance at level 60 escalates against. Everything you have built in the previous eleven modules lands in one of the artefacts this module names.

## Learning objectives

- Own an AI risk programme at team scope — cadence, artefacts, decisions, sign-off routing — across product / ML / infra / security / governance / legal.
- Brief the AI governance council with a pre-read deck, a one-pager, and a decision-log template that captures the risk decisions the council owns; produce meeting minutes that survive audit.
- Author the risk-engineering slice of a model / system / dataset card — safety evaluations, red-team findings, adversarial-robustness delta, fairness / privacy findings, residual-risk claim, monitoring plan — publishing through the Hugging Face model-card format or an internal template.
- Contribute to a Preparedness / RSP-style deployment tiering — pre-registered thresholds, rollback criteria, and dangerous-capability tripwires — deferring frontier-lab depth to `agentic-safety-engineer` (level 40) and program-level ownership to `head-of-ai-governance` (level 60).
- Reconcile the risk-engineering evidence to NIST AI RMF MEASURE / MANAGE sub-categories and ISO/IEC 42001 controls; hand release-assurance-shape work to `ai-evaluation-engineer` (peer, level 35, Governance family) and control-library architecture to `senior-ai-governance-architect` (level 50).
- Position the certifications portfolio — IAPP AIGP, ForHumanity Independent AI Auditor, BABL AI Algorithm Bias Auditor, ISO/IEC 42001 LI/LA, IAPP CIPP/US or CIPP/E where privacy-adjacent — and know which are load-bearing in enterprise hiring.

## Chapters

1. [`01-risk-program-at-team-scope.md`](01-risk-program-at-team-scope.md) — the four-tier cadence, the bill of artefacts, the decision matrix, the sign-off graph, and the RACI across the six functions the programme spans.
2. [`02-governance-council-briefings.md`](02-governance-council-briefings.md) — the pre-read deck ("one decision per slide"), the one-pager as decision index, the queryable decision log, and audit-durable minutes.
3. [`03-model-card-risk-slice.md`](03-model-card-risk-slice.md) — the seven-component risk-engineering slice of the model / system / dataset card; card-as-projection-of-the-register discipline; Hugging Face schema vs. internal template.
4. [`04-rsp-preparedness-tiering-contribution.md`](04-rsp-preparedness-tiering-contribution.md) — pre-registered thresholds, rollback criteria, and dangerous-capability tripwires at team scope; the L40 and L60 escalation contract in operation.
5. [`05-evidence-crosswalk-nist-rmf-and-iso-42001.md`](05-evidence-crosswalk-nist-rmf-and-iso-42001.md) — the "one artefact, many standards" crosswalk shape; RMF MEASURE / MANAGE and ISO/IEC 42001 Annex A citations; gap-detection discipline and the L35 / L50 hand-off.
6. [`06-certifications-portfolio-and-career-shape.md`](06-certifications-portfolio-and-career-shape.md) — the five credential families relevant at level 25, the which-one-when decision table, and the career hand-offs into L35 / L40 / L50.

## Exercises

- [`exercises/exercise-01-risk-program-charter-for-one-team.md`](exercises/exercise-01-risk-program-charter-for-one-team.md) — draft the programme charter for the running system: scope, cadence, artefact bill, decision matrix, sign-off routing, escalation criteria.
- [`exercises/exercise-02-council-pre-read-and-decision-log-drill.md`](exercises/exercise-02-council-pre-read-and-decision-log-drill.md) — assemble a full council-facing packet (pre-read deck, one-pager, decision-log entries, minutes) for a mock quarterly session.
- [`exercises/exercise-03-model-card-risk-slice-authoring.md`](exercises/exercise-03-model-card-risk-slice-authoring.md) — author the risk-engineering slice of a model / system / dataset card set for the running system in the Hugging Face schema.
- [`exercises/exercise-04-rsp-preparedness-style-tiering-contribution.md`](exercises/exercise-04-rsp-preparedness-style-tiering-contribution.md) — write a T2 → T3 tier-promotion contribution with cited thresholds, pre-registered rollback triggers, and wired tripwires.
- [`exercises/exercise-05-rmf-and-42001-evidence-crosswalk.md`](exercises/exercise-05-rmf-and-42001-evidence-crosswalk.md) — build the RMF MEASURE / MANAGE and ISO/IEC 42001 crosswalk for the running system, disposition every gap, and mark L35 / L50 escalations.
- [`exercises/exercise-06-certifications-portfolio-planner.md`](exercises/exercise-06-certifications-portfolio-planner.md) — draft a defensible three-year certification and career plan matched to a target employer profile.

## Labs & quizzes

- [`labs/`](labs/) — a longer hands-on lab is scheduled here; see the lab README for the current status.
- [`quizzes/`](quizzes/) — knowledge-check quiz for the module.

## Resources

See [`resources.md`](resources.md) for primary sources — every reference points at a standards body, regulator, certification-body, or original publication.

## How to work through the module

1. Carry forward **the same running system** you used in mods 101-111 and the artefacts you built there (harm model, register rows, red-team programme, adversarial-ML evaluation, fairness / privacy findings, guardrail effectiveness reports, monitor spec, MRM / supply-chain evidence, incident log). The six exercises here reuse those artefacts; starting from a fresh system would require re-doing eleven modules of upstream work.
2. Read chapters 01-06 in order. Chapter 01 defines the *programme* shape; chapter 02 walks the *council* interface; chapter 03 walks the *external* face; chapter 04 walks the *deployment-tier* contribution; chapter 05 walks the *framework crosswalk*; chapter 06 closes with the *certifications and career* view.
3. Do exercises 01-06 in order. They stack:
   - 01 stands up the programme charter and decision matrix,
   - 02 puts a full council packet on the table,
   - 03 publishes the external card,
   - 04 pins deployment tiering with rollback and tripwires,
   - 05 reconciles the evidence to RMF and ISO 42001,
   - 06 plans the credential portfolio and next-up move.
4. Take the quiz.

By the end of the module you should be able to hand an auditor and a hiring manager the same programme packet: a charter naming what your team owns and what escalates; a versioned council pre-read with decision-log entries; a published card slice with a testable residual-risk claim; a deployment-tier contract with pre-registered rollback; a crosswalk with no unmarked gaps; and a certifications plan matched to your target employer profile. If any one of those five artefacts is missing, the programme is not yet at level-25 shape.
