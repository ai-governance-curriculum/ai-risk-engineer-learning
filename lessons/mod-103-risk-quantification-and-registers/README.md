# mod-103 — Risk Quantification, Scoring, and Enterprise AI Risk Registers

**Estimated effort:** 12 hours
**Position in the track:** third module; assumes mod-101 (frameworks & role scope) and mod-102 (harm modelling) are done.

Mod-102 gave you a harm model — the set of scenario rows the register will score. This module teaches you to *score them defensibly*. You author a scoring scheme (impact × likelihood × exposure) that separates inherent, residual, and control-defeated states; you plug the AI risk register into ISO/IEC 27005 so it composes with the enterprise ERM without a schema fight; you reuse ISO/IEC 25059 and NIST AI RMF MEASURE as the axes along which a system is scored; you aggregate rows into portfolio reports that a risk council can read; you operate a working register (fields, cadence, ownership, sign-off) in a GRC-for-AI platform (Credo AI, Holistic AI, ModelOp, Monitaur) or the spreadsheet fallback; and you pin the peer boundary with `model-evaluation-engineer` (level 30) on statistical calibration methodology so the register consumes calibrated numbers rather than authoring them.

The register you produce here is what mod-108 credits controls into, what mod-109 monitors close the loop on, what mod-110 aggregates for supply-chain review, what mod-111 reads during incident response, and what mod-112 packages for cross-functional reporting. If the scoring is soft or the register is stale here, downstream work has nothing to defend against.

## Learning objectives

- Design a scoring scheme for AI risk (impact × likelihood × exposure) that is defensible under audit — inherent vs. residual vs. control-defeated distinctions, calibration against enterprise risk appetite, and aggregation across systems / portfolios.
- Reuse ISO/IEC 27005 information-security risk shape as a compatible parent framework so AI risk plugs into the enterprise ERM without a schema fight.
- Reuse ISO/IEC 25059 quality dimensions and NIST AI RMF MEASURE sub-categories as the axes on which a system is scored.
- Populate a working AI risk register — fields, cadence, ownership, sign-off routing, aggregation reports — in a GRC-for-AI platform (Credo AI, Holistic AI, ModelOp, Monitaur) and the spreadsheet-shaped fallback.
- Cite the boundary to `model-evaluation-engineer` (peer, level 30) on statistical calibration methodology (bootstrap CIs, judge-vs-human calibration, Cohen's kappa) — the risk engineer uses the methodology, but full derivation and methodology-of-methodology depth is linked out.

## Chapters

1. [`01-inherent-residual-and-control-defeated.md`](01-inherent-residual-and-control-defeated.md) — the three states every risk row carries, the two deltas that argue with the auditor, and why AI controls need a control-defeated column.
2. [`02-impact-likelihood-exposure-axes.md`](02-impact-likelihood-exposure-axes.md) — the scoring axes, rubrics you can publish, and the choice between ordinal-matrix and anchored-numeric combination.
3. [`03-iso-27005-as-parent-frame.md`](03-iso-27005-as-parent-frame.md) — reuse the 27005 process shape so AI risk plugs into the enterprise ERM; use ISO/IEC 23894 as the AI-specific content overlay.
4. [`04-iso-25059-and-nist-measure-axes.md`](04-iso-25059-and-nist-measure-axes.md) — ISO/IEC 25059 quality characteristics × NIST AI RMF MEASURE sub-categories as the dimensions a system is scored *along*.
5. [`05-aggregation-and-appetite.md`](05-aggregation-and-appetite.md) — system-level and portfolio-level aggregation rules, calibration against enterprise risk appetite, and the six reports the risk council actually reads.
6. [`06-risk-register-operations.md`](06-risk-register-operations.md) — operational fields (ownership, cadence, sign-off, evidence trail); GRC-for-AI platform patterns and the spreadsheet-shaped fallback; register hygiene checks in CI.
7. [`07-peer-boundary-calibration-methodology.md`](07-peer-boundary-calibration-methodology.md) — what you *use* (CIs, κ, calibrated numbers) vs. what `model-evaluation-engineer` (L30) *derives*; the calibration-request and calibration-acceptance artifacts at the seam.

## Exercises

- [`exercises/exercise-01-inherent-vs-residual-vs-control-defeated-drill.md`](exercises/exercise-01-inherent-vs-residual-vs-control-defeated-drill.md) — score three mod-102 rows across all three states and expose the two deltas.
- [`exercises/exercise-02-impact-likelihood-exposure-scoring-scheme.md`](exercises/exercise-02-impact-likelihood-exposure-scoring-scheme.md) — author, publish, and version a scoring rubric + combination matrix; demonstrate on real rows.
- [`exercises/exercise-03-iso-27005-parent-frame-alignment.md`](exercises/exercise-03-iso-27005-parent-frame-alignment.md) — file your rows against the seven 27005 process steps and reconcile with the security-side register.
- [`exercises/exercise-04-portfolio-aggregation-drill.md`](exercises/exercise-04-portfolio-aggregation-drill.md) — take a multi-system portfolio to the six risk-council reports; enforce dedup and appetite decision rules.
- [`exercises/exercise-05-risk-register-population-in-grc-platform.md`](exercises/exercise-05-risk-register-population-in-grc-platform.md) — stand up the register in a GRC-for-AI platform or spreadsheet-shaped fallback, wire cadence and sign-off, and export the aggregation reports.

## Labs & quizzes

- [`labs/`](labs/) — a longer hands-on lab is scheduled here; see the lab README for the current status.
- [`quizzes/`](quizzes/) — knowledge-check quiz for the module.

## Resources

See [`resources.md`](resources.md) for primary sources — every reference points at a standards body, regulator, framework maintainer, incident-corpus author, or the vendor's own published product documentation.

## How to work through the module

1. Carry forward **the same running system** you used in mod-101 and mod-102, and the harm-catalog rows you built in mod-102 exercises 02-04. The five exercises here score, align, aggregate, and operate those rows — starting from a fresh system would require re-doing the harm model.
2. Read chapters 01-07 in order. Chapters 01 and 02 define *what* a scored row is; chapters 03 and 04 pin the *standards* the row plugs into; chapter 05 walks *aggregation and appetite*; chapter 06 walks *operations*; chapter 07 pins the *peer boundary* with the L30 model-evaluation engineer.
3. Do exercises 01-05 in order. They stack:
   - 01 splits mod-102 rows into inherent / residual / control-defeated,
   - 02 publishes the scoring rubric and combination matrix and re-scores the rows against it,
   - 03 files the rows against the 27005 process and reconciles with the security register,
   - 04 aggregates a multi-system portfolio into the six risk-council reports,
   - 05 stands the register up as a live operation in a platform or fallback, with cadence and sign-off wired.
4. Take the quiz.

By the end of the module you should be able to hand an auditor a register where every row (a) scores inherent, residual, and control-defeated with cited rubric levels and evidence anchors, (b) tags at least one ISO/IEC 25059 characteristic and one NIST AI RMF MEASURE sub-category, (c) files against the 27005 process steps with a security-side reconciliation, (d) rolls into the six standard portfolio reports without silent double-counting, and (e) carries operational fields — owner, cadence, sign-off history, evidence manifest, calibration provenance — that let the register be read as a live artifact rather than a snapshot. If any row fails one of those five checks, the register is not shippable.
