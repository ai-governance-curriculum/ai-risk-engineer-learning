# Exercise 03 — Model / System / Dataset Card Risk-Slice Authoring

**Estimated effort:** 3 hours

## Objective

Author the **risk-engineering slice** of a model card, a system card, and a dataset card for the running system, using the Hugging Face model-card specification as the operational schema. Every section of the risk slice is a projection of an artefact you already authored in mods 102-111 — the exercise is not fresh authoring, it is *extract and cross-link* discipline. The residual-risk claim is the load-bearing section: testable, tied to named monitors, reversible by pre-registered triggers.

## Prerequisites

- Chapter 03 read closely — the seven-component risk-engineering slice, the Hugging Face schema, and the card-as-projection pattern.
- The Hugging Face model-card documentation skimmed — <https://huggingface.co/docs/hub/model-cards> — so you know the section names and YAML metadata block conventions.
- The Mitchell et al. model-card paper skimmed for the discipline behind the "Bias, Risks and Limitations" and "Uses" sections — <https://arxiv.org/abs/1810.03993>.
- The Gebru et al. datasheet paper skimmed for the dataset-card sections — <https://arxiv.org/abs/1803.09010>.
- Evidence artefacts from mods 104-108 for the running system: red-team run report, adversarial-ML evaluation output, fairness slice measurements with caveats sheet, privacy-risk findings, guardrail effectiveness numbers.
- Monitor spec from mod-109 — the residual-risk claim is invalidated by named monitor thresholds firing; you need the monitor IDs.
- Deployment-tier statement from mod-102 chapter 04 (or from exercise 04 of this module if you are doing them in parallel) — the intended-use section names the tier.

## Problem statement

Produce three cards for the running system:

1. A **model card** for the primary model component (the LLM, the classifier, or the model whose risk-engineering slice this exercise is authored around).
2. A **system card** for the deployed system (the model plus scaffolding, guardrails, monitors, and policy).
3. A **dataset card** for one dataset the system trained or evaluated against (pick the one where the risk-engineering evidence is richest — usually the retention / evaluation corpus, or the fine-tuning dataset if there is one).

All three cards use the Hugging Face model-card template as the *schema*, adapted for the dataset-card section list (Gebru et al. + Data Cards Playbook) where relevant. Where an internal template diverges from Hugging Face, note the divergence explicitly.

## Requirements

Produce three markdown files:

- `mod-112-ex03-model-card.md`
- `mod-112-ex03-system-card.md`
- `mod-112-ex03-dataset-card.md`

Each card must populate all seven components of the risk-engineering slice from chapter 03. Populate from *existing artefacts*, not fresh authoring.

### Common to all three cards

- **YAML metadata block** at the top of the card, following Hugging Face conventions (`license`, `tags`, `datasets`, `metrics`, `model-index` where applicable).
- **Card version and date** in the metadata or header.
- **Card authors** section listing the risk engineer as a co-author.
- **Card contact** section naming the escalation contact for external users.

### Component-by-component requirements

For each of the seven risk-slice components below, populate the card with the fields listed. If evidence is missing, cite `<!-- needs-research: ... -->` — do not fabricate numbers.

1. **Intended and out-of-scope use** — populate `Uses — Direct Use`, `Uses — Downstream Use`, `Uses — Out-of-Scope Use`. Name the deployment tier (per chapter 04 / exercise 04). List *specific* out-of-scope uses ("Do not use for hiring decisions"), not generic disclaimers.
2. **Safety evaluations** — populate `Evaluation` (or a `Safety evaluations` subsection). Cite the red-team run report (mod-104): run scope, attack corpus size, pass / fail thresholds, report reference. Cite guardrail effectiveness numbers (mod-108): per-guardrail F1 (or programme metric), coverage grid, known defeat modes.
3. **Adversarial-robustness delta** — populate `Evaluation` (or a `Robustness` subsection). Cite clean-vs-worst-case accuracy under a named perturbation set (mod-105). Report the delta, the tolerated worst case, and the mitigation that reduces deployment exposure.
4. **Fairness slice measurements** — populate `Evaluation` (or a `Fairness` subsection). Cite subgroup slice definitions (mod-106) alongside per-slice numbers. Populate `Bias, Risks and Limitations` with the caveats sheet's honest limitations of what the fairness measurements *do not* cover.
5. **Privacy findings** — populate `Training Data` (with provenance / DPIA reference) and `Bias, Risks and Limitations` (memorisation / MIA / linkage findings from mod-107). The card summarises what the evaluations *found* and what the deployed system does about it, not the DPIA in full.
6. **Residual-risk claim** — populate a **first-class section** (either `Bias, Risks and Limitations` with a sub-heading or a dedicated `Residual Risk` section). The claim has three parts:
   - The residual level on the named population / cohort / use case.
   - The monitoring conditions under which the claim holds (named monitor IDs from mod-109, named thresholds).
   - The pre-registered invalidation triggers (which conditions revoke the claim; what the system reverts to).
7. **Monitoring plan** — populate `Recommendations` (or a dedicated `Monitoring` subsection). Name the monitors, thresholds, and escalation paths that downstream integrators must retain.

### Card-family-specific requirements

- **Model card**: emphasise the model-level residual claim (training data provenance, memorisation findings, model-scale evaluation numbers). Include the `model-index` YAML block with the safety-eval numbers structured.
- **System card**: emphasise the *system*-level composition — model + scaffolding + guardrails + policy — and the interaction effects. Include a `Safety mitigations` section listing the guardrails and monitors that are part of the deployed composition (not just the model). Include the tier and the escalation-to-council routing.
- **Dataset card**: follow the Gebru et al. section list (Motivation, Composition, Collection Process, Preprocessing, Uses, Distribution, Maintenance) with risk-engineering content in `Composition` (sensitive attributes, subgroup coverage, canary set state) and `Uses` (in-scope / out-of-scope reuse, downstream integrator obligations).

### Cross-card consistency

- All three cards agree on evidence-linked numbers. If the fairness delta cited on the model card differs from the one cited on the system card, one is wrong.
- All three cards cite the same residual-risk claim structure (level, monitors, invalidation triggers), scoped appropriately to the card's unit of description.
- Each card carries a *pointer* to the register row(s) it publishes (the `card_ref` field in the register points back here; the card should point *forward* to the register too).

## Starter guidance

- Do the model card first, the system card second, the dataset card third. The model card's evaluations are the source for the system card's `Evaluation` section; the dataset card's `Composition` section is what the model card's `Training Data` section references.
- On the `Uses — Out-of-Scope` section, be specific to the point of discomfort. Generic disclaimers ("not for high-stakes decisions") are marketing; specific exclusions ("not licensed for hiring, credit, medical, legal, or tax decisions; not licensed for use with minors as end users") are evidence.
- On the residual-risk claim, write the invalidation trigger first and the level second. A residual claim without an invalidation trigger is not a claim, it is a hope.
- On the fairness section, cite the *slice definitions* alongside the numbers. A card that reports fairness deltas without the subgroups defined is not useful evidence to a downstream integrator.
- On the privacy section, do not restate the DPIA. Summarise what the risk-engineering evaluations *found* (canary extraction rate, MIA success rate, linkage findings) and what the deployed system does about it. The DPIA is a linked artefact, not a re-authored section.
- If an evidence artefact does not yet exist upstream (e.g., no fairness slice measurement has been done because the system is too new), do not write a fairness section from scratch. Write `<!-- needs-research: mod-106 fairness slice measurement not yet produced for this system; card section left as gap. -->` and file a follow-up.
- Where your organisation has an internal template that extends the Hugging Face schema, produce the *Hugging Face-schema version* here (the public face); note in a comment where an internal template would carry additional fields (audit-trail IDs, internal deployment IDs, regulator citations).

## Acceptance criteria

The three cards are accepted when:

- All three cards have the YAML metadata block, card-version header, authors, and contact populated.
- All seven risk-slice components are populated on the model card and the system card, with evidence pointers to mod-102 through mod-111 artefacts.
- The residual-risk claim on every card names (a) the level on a specific population / cohort / use case, (b) the monitors that keep the claim honest, (c) the pre-registered invalidation triggers.
- Numbers cited across the three cards agree; slice definitions are cited alongside fairness deltas; caveats are honest.
- Out-of-scope uses are enumerated specifically, not generically.
- The dataset card follows the Gebru et al. section list, with risk content in `Composition` and `Uses`.
- Each card carries a pointer to the register row(s) it publishes.
- A downstream integrator reading only the system card can (i) name the tier and the intended cohort, (ii) name the residual claim and its invalidation triggers, (iii) know which monitors they must retain, (iv) know which off-label uses are excluded.

## Stretch goals

- Draft the **card-refresh checklist** that runs alongside the cards: when the underlying artefact refreshes (a new red-team run, a new guardrail effectiveness report), which card sections update automatically vs. require review. This is the CI shape that keeps cards honest — see mod-108 / mod-109 refresh cadences.
- Produce the **internal-template overlay**: a second version of the model card that adds the internal-only fields (audit-trail IDs, regulator citations, internal deployment IDs). Note in the diff which fields are internal-only and why. This is the pattern chapter 03 describes: external Hugging Face schema face vs. internal authoritative face, with never-disagreeing evidence-linked fields.
- Publish the **card change log** — a versioned record of every material change to the card since its first publication, with rationale, author, and sign-off. This is what makes a card the audit-durable artefact chapter 03 says it should be.
- If the running system publishes to the Hugging Face Hub or an internal model registry, do the actual publishing dry-run: render the card, check the YAML metadata block validates, check the section headings map to the platform's expected structure.
- Where an EU AI Act Article 13 transparency obligation attaches (for a high-risk system), draft the transparency-notice extract from the card — the customer-facing text that satisfies the obligation. This closes the loop between the card and mod-101 chapter 04 obligation citations.

## Deliverable format

Three markdown files in a folder `mod-112-ex03-cards/`:

- `mod-112-ex03-model-card.md`
- `mod-112-ex03-system-card.md`
- `mod-112-ex03-dataset-card.md`
- `README.md` — one paragraph explaining which running system the cards cover, the version of the Hugging Face template used, and any internal-template divergences noted.

## Submission

Commit the folder to the running-system programme repo. Where the card is publishable externally (open-source model release, Hugging Face Hub, or an internal card registry), file a separate publish request through the programme's comms / legal review flow — the card is externally visible evidence and the review is part of the sign-off routing (per chapter 01 decision matrix: "Publish a model / system / dataset card externally" is a co-signed decision).
