# Exercise 03 — FDA GMLP + PCCP Slice for One Device-Shaped System

**Estimated effort:** 3 hours

## Objective

Draft the **risk-engineering slice** of a Predetermined Change Control Plan (PCCP) for the primary running system, treating it as if it were a Software as a Medical Device (SaMD). The slice covers the *Description of Modifications*, the risk-engineering portion of the *Modification Protocol*, and the *Impact Assessment* — the three PCCP components chapter 03 named. Even if your system is not (and will not be) a medical device, do the exercise: the PCCP shape is a disciplined release-gate specification that transfers to any AI system whose behaviour changes after deployment.

## Prerequisites

- Chapter 03 read in full. The GMLP-principle-to-artefact table, the three PCCP components, the "locked vs adaptive" distinction, and the "narrow-and-specified vs vague-and-broad" rubric for the description of modifications are the reference shapes.
- The primary running system's mod-102 harm model, mod-103 register (three-state), mod-106 fairness evaluation, mod-108 guardrail catalogue, and mod-109 monitor spec. The impact assessment cites all of them.
- Chapter 04 skimmed. The modification protocol's data-management block cites the Data Card / Datasheet discipline; the cybersecurity-assessment block cites chapters 05 and 06.
- If the system is genuinely SaMD or SaMD-adjacent (clinical decision support, diagnostic classifier, triage prioritiser, radiology segmentation), identify a named regulatory-affairs contact — even if they will not sign the drafted slice, they are the reviewer of record for the shape.
- If the system is not SaMD, pick a modification type that is plausible for it (a re-training on additional data drawn from the last month of production traffic; a decision-threshold adjustment triggered by calibration drift; a pre-processing update to accept a new input source) and draft against that. The training value is the shape, not the medical vocabulary.

## Problem statement

Draft the risk-engineering slice of a PCCP for the primary running system, covering at least **two** modification types. The slice sits inside a larger PCCP a regulatory-affairs owner would assemble; you are producing the parts that live on the risk-engineering side of the boundary.

### The slice layout

```
pccp-slice/<system-id>/<slice-version>/
  README.md                                  # scope + locked-vs-adaptive posture
  01-gmlp-principle-map.md                   # ten principles × artefact-that-evidences-it
  02-description-of-modifications.md         # two or more modification types, each narrowly specified
  03-modification-protocol/
    <modification-type-slug>/
      data-management-plan.md
      re-training-plan.md
      performance-evaluation-plan.md         # acceptance criteria per stratification
      update-procedures.md
      cybersecurity-assessment.md            # cites chapters 05-06
      post-modification-monitoring.md        # cites mod-109 monitor set
  04-impact-assessment.md                    # cites mod-102 harm rows, mod-103 register, mod-108 guardrails
  05-out-of-envelope-triggers.md             # what forces a new submission instead of an in-envelope change
```

### The GMLP-principle map

The `01-gmlp-principle-map.md` file is a table with ten rows — one per GMLP principle from chapter 03 — and two columns: *artefact that evidences this principle* and *artefact location*. Every cell must resolve to a real file with an owner and a revision hash, or be flagged as a gap with an owner and target date. The table is what a GMLP-literate reviewer reads first.

### The description of modifications

Chapter 03 §"Description of Modifications" is emphatic that broad wording ("improvements to the model") is rejected. Each modification type in `02-description-of-modifications.md` must include:

- The *class of change* — re-training on additional data, threshold adjustment, pre-processing update, hyperparameter re-search within bounds, or similar.
- The *scope of the change* — which components are affected, which are held fixed, what the change envelope's numeric bounds are (data-size range, threshold range, hyperparameter search bounds).
- The *triggering condition* — what causes the modification to be applied (calibration drift beyond a stated threshold, new input-device compatibility requirement, scheduled quarterly retraining, etc.).
- The *out-of-envelope condition* — the condition that turns an "in-envelope modification" into "a change that requires a new submission." Named explicitly.

### The modification protocol

For each modification type, the six sub-files inside `03-modification-protocol/<modification-type-slug>/` populate the release-gate spec from chapter 03's protocol table. The performance-evaluation plan is the load-bearing one: it names the metrics measured, the *minimum acceptable values* (the acceptance criteria), the stratification (site, subgroup, condition — cite mod-106 for the fairness slice), and the comparator to the previously authorised version.

### The impact assessment

The `04-impact-assessment.md` file argues, for each modification type:

- which mod-102 harm-model rows the modification elevates or introduces;
- which mod-108 controls (existing or added) contain the elevation;
- which mod-109 monitor detects each credited control's failure and within what SLA;
- how the modification protocol's acceptance criteria ensure that the residual risk (per mod-103) remains inside the envelope the original submission established.

## Requirements

Produce all six items:

1. **The slice folder** — every file in the layout above exists with real content. Modification-type sub-folders exist for at least two modification types (three preferred).
2. **The GMLP-principle map** in `01-gmlp-principle-map.md` — every one of the ten principles has an artefact citation with a location; gap cells have owner and target date. No blanks.
3. **The description of modifications** in `02-description-of-modifications.md` — each modification type has the class, scope (with numeric bounds), triggering condition, and out-of-envelope condition explicitly named. A `TBD` on any of the four columns means the modification type is not yet in-envelope.
4. **The modification protocol** — for each modification type, the six sub-files (data-management, re-training, performance-evaluation, update-procedures, cybersecurity-assessment, post-modification-monitoring) exist with real content. The performance-evaluation-plan sub-file must include stratification per mod-106 slice.
5. **The impact assessment** in `04-impact-assessment.md` — one section per modification type, with the four bullets above answered by pointing at specific mod-102 rows, mod-108 controls, mod-109 monitors, and mod-103 register rows. Not "safety monitors will catch it" but "monitor SM-04 with threshold X and alert SLA Y, per the mod-109 signal contract row Z."
6. **The out-of-envelope triggers** in `05-out-of-envelope-triggers.md` — a numbered list of conditions that force a new submission (or, for non-SaMD systems, a full re-validation) instead of an in-envelope change. This list is the *scope guard* for the PCCP.

## Starter guidance

- **Narrow is safe; broad is rejected.** A modification type that reads "re-training on additional data drawn from the last quarter of production traffic, size ≤ 20% of the prior training corpus, architecture and pre-processing held fixed, hyperparameter search restricted to learning rate ∈ [1e-5, 5e-5] and warmup ∈ [500, 2000] steps" is in-envelope. A modification type that reads "improvements to the model" is not.
- **The modification protocol is your release-gate spec in regulator vocabulary.** Read chapter 03's protocol table as an engineer's release-gate checklist. If your team already has a release-gate spec, the drafting is largely translation; if not, the exercise is the forcing function.
- **Acceptance criteria are numbers with a rationale, not "better than the previous version."** For each metric, name the minimum acceptable value and the source of the value (previously authorised performance, statistical non-inferiority margin, absolute clinical or business threshold). "≥ current" is not an acceptance criterion.
- **Stratification is where mod-106 pays off.** The performance-evaluation-plan must not report only aggregate metrics. The clinically relevant subgroups (age, sex, race/ethnicity where scientifically justified, site, device model, clinical condition) each have their own acceptance criterion. If the mod-106 fairness evaluation revealed a stratification the drafters would not have thought of, cite it.
- **Post-modification monitoring is the same mod-109 spec, differently packaged.** The `post-modification-monitoring.md` file must not duplicate the mod-109 monitor spec — it must reference specific monitor IDs and thresholds from the mod-109 signal contract. Duplication drifts; reference does not.
- **The impact assessment argues, it does not describe.** For each risk elevated by a modification, the assessment must argue *why* the credited control contains the elevation. "Guardrail catalogue row G-12 mitigates this" is a description; "Guardrail G-12 fires on 98.4% of the harm-model row H-04 fixtures, monitored at 5-minute cadence by SM-07 with a 15-minute alert SLA, keeping the residual on register row R-03 at Medium-Low even under the modification" is an argument.
- **The out-of-envelope trigger list is the scope guard.** Any modification that crosses a listed trigger is not an in-envelope change. Under FDA review the trigger list is what the reviewer relies on to bound the PCCP; on the engineering side, it is what stops "we'll just adjust a bit" from silently sliding out of scope.
- **Do the exercise even if you are not SaMD.** The change-management stub in the exercise-01 packet either points at a PCCP (for SaMD) or at the FI's model-change policy (for non-SaMD). Even for non-SaMD systems, the PCCP shape is a stronger change-management artefact than "we deploy on Fridays." Adapt the vocabulary if needed — "modification" reads as "release variant" — but keep the discipline.

## Acceptance criteria

The submission is accepted when:

- The slice folder exists with every file populated as real content or a labelled gap.
- The GMLP-principle map cites an artefact per principle with a location.
- At least two modification types are specified with class, scope bounds, triggering condition, and out-of-envelope condition.
- Each modification type has the six modification-protocol sub-files, with the performance-evaluation plan including stratification per mod-106 slices.
- The impact assessment cites specific mod-102 rows, mod-108 controls, mod-109 monitor IDs, and mod-103 register rows — not general statements.
- The out-of-envelope trigger list is present and explicitly names what forces a new submission or full re-validation.
- The exercise-01 packet's `10-change-management.md` file is updated to point at this PCCP slice (or, for non-SaMD, to reference it as the shape the model-change policy follows).

## Stretch goals

- **Locked-vs-adaptive determination memo** — draft a one-page memo classifying the running system as *locked* or *adaptive*, with the evidence. An adaptive system without a PCCP is not permitted in a marketed device; the same memo for a non-SaMD system tells you which change-management shape is required.
- **Adverse-modification dry run** — pick a plausible modification, apply it in a shadow environment, run the modification-protocol's performance-evaluation plan against it, and record the pass / fail against the acceptance criteria. Even a synthetic dry run reveals whether the acceptance criteria are actually operable.
- **Register-side back-link** — in the mod-103 register, add a `pccp_modification_types` field per row listing the modification types (from `02-description-of-modifications.md`) that touch the row. The two-way lookup makes it obvious when a proposed modification will move a row previously assessed as inside appetite.
- **Modification-request template** — draft the `mod-request.md` template that a developer will fill when they want to apply an in-envelope modification. The template asks for: modification type, evidence that the change is within scope of the type, evaluation results against the acceptance criteria, signature. The template is the runtime interface to the PCCP; without it, every modification is a bespoke negotiation.
- **Cross-track review with L50 architect** — send the slice to the `senior-ai-governance-architect` contact (level 50, per chapter 06's level ladder) for review of the taxonomy. Their disagreement is a signal that the modification-type taxonomy needs revision before it is codified.

## Deliverable format

A single markdown document `mod-110-ex03-pccp-slice.md` that indexes the slice folder. The index contains:

1. *System and slice summary* — one paragraph naming the running system, its locked-vs-adaptive posture, the slice's semantic version, and the SaMD / non-SaMD framing.
2. *GMLP principle coverage* — the ten-row table quoted verbatim, with gap cells flagged.
3. *Modification types* — one paragraph per type: class, scope bounds, triggering condition, out-of-envelope condition.
4. *Modification protocol summary* — a table of modification type × six-protocol-file completeness.
5. *Impact-assessment excerpts* — for each modification type, the one-line argument that residual stays inside envelope.
6. *Out-of-envelope trigger list* — the numbered list quoted verbatim.
7. *Cross-references* — pointers to the mod-102 rows, mod-103 rows, mod-108 controls, and mod-109 monitor IDs cited in the slice; pointer to the exercise-01 packet's change-management file that now references this slice.
8. *Open items* — GMLP gaps, missing modification-protocol sub-files, or acceptance-criteria whose numeric values still need defence.

## Submission

Attach the index markdown plus the slice folder. If the running system is not SaMD, mark the slice `context: non-SaMD training drill` in `README.md` and adapt the vocabulary to your setting — but preserve the modification-type discipline, the protocol structure, and the impact-assessment argument shape. The FDA PCCP shape is the tightest change-management shape any regulator has published for AI; treat this exercise as change-management training even when medical-device regulation is not the destination.
