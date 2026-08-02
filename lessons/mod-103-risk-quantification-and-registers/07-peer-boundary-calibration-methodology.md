# Peer Boundary: What the Risk Engineer Uses vs. What the Model-Evaluation Engineer Derives

Chapters 02 and 05 introduced likelihood anchors that lean on measured attack-success rates, judge-vs-human agreement numbers, and confidence intervals around guardrail effectiveness. Those numbers do work in the register — a likelihood level of "Unlikely" only survives audit if the underlying evidence is calibrated. But the *methodology* behind those numbers — how the confidence interval is computed, whether the LLM-as-a-judge grader agrees with human raters, how much of the observed effect could be noise — is not the level-25 risk engineer's core work. It is the level-30 peer role of `model-evaluation-engineer`, and this chapter pins the boundary so both sides know where the handoff sits.

## Motivation

Two failure modes appear when the boundary is not written down:

- **The risk engineer runs the calibration.** Bootstrap CIs on eval scores, judge-vs-human calibration studies, Cohen's kappa on human-agreement panels — all of it — end up authored by the risk engineer. The risk engineer's MEASURE-side output (register rows, aggregation reports, treatment plans) slips because the calibration work displaces it. Worse, the calibration itself is often shallow because the risk engineer is a *consumer* of statistical methodology, not a specialist in it.
- **The risk engineer takes calibration numbers on faith.** A vendor scorecard reports "97% accuracy on our safety benchmark." The number lands in the register as a residual-likelihood anchor without asking who computed the CI, what the sample size was, or whether the graders were calibrated against humans. The register looks defensible until the auditor asks *how* the number was produced and there is no answer.

Neither extreme is correct. The right shape is: **the risk engineer consumes calibrated numbers with a citation and a version, and refuses to consume uncalibrated numbers.** The `model-evaluation-engineer` (L30) is the peer role that either performs the calibration in-house or reviews vendor-supplied calibration on the risk engineer's behalf.

## What the risk engineer *uses* (in scope for this module)

Every level-25 register row lands calibration numbers in specific slots. What the risk engineer needs to be able to do:

- **Read a confidence interval and know what it constrains.** A bootstrap 95% CI on a guardrail's false-negative rate of `[0.01, 0.05]` means the residual-likelihood credit for that guardrail should be scored against a plausible worst-case of `0.05`, not the point estimate of `0.02`. The register row picks the CI-upper-bound story or the point-estimate story deliberately, and the choice is a documented convention.
- **Read a judge-vs-human calibration report and know when to trust the judge score.** If the LLM-as-a-judge grader agrees with humans at κ = 0.35, the judge score can be used as a *screening* signal but cannot be the sole anchor for a residual-likelihood claim. If κ = 0.85, the judge score can anchor.
- **Ask for the sample size, the observation window, and the version of the eval harness.** A guardrail-effectiveness number without n, without a date, and without a harness version is not a citation.
- **Refuse to fold uncalibrated numbers into aggregation reports.** Chapter 05 said uncalibrated rows drop out of portfolio roll-ups. That drop-out rule is enforced against calibration provenance.
- **Choose whether to score the row against the point estimate, the CI lower bound, or the CI upper bound.** The default in a defensible register is: *use the CI upper bound for likelihood on adverse scenarios, and the CI lower bound for guardrail effectiveness credited on residual.* Both choices reflect a "make the register hard to be surprised by" posture.

That is the level-25 competence. It is *statistical literacy*, not statistical methodology.

## What the model-evaluation engineer *derives* (out of scope, escalates to L30)

The following are the L30 peer's work. If your register needs them and no L30 role exists yet, you are either negotiating one into existence or you are paying a vendor / consultancy for that depth. You are not producing them at level 25.

### 1. Bootstrap confidence intervals on measured effects

Non-parametric bootstrap resampling to produce CIs on:

- Guardrail false-negative and false-positive rates.
- LLM-as-a-judge agreement with human raters.
- Fairness metrics (demographic parity, equalised odds, subgroup accuracy) with a sensible resampling unit.
- Attack-success rates from mod-104 red-team runs.
- Drift metrics from mod-109 monitors.

The design choices — resampling unit, number of iterations, block bootstrap for time-series data, corrections for multiple comparisons — belong to the L30 role. The risk engineer *reads* the resulting CI and cites it.

### 2. Judge-vs-human calibration studies for LLM-as-a-judge graders

LLM-as-a-judge is used across mod-104 through mod-108. Its scores anchor guardrail-effectiveness numbers, red-team-verdict aggregation, and fairness-audit outputs. Calibration requires:

- A representative sample of items graded by both the judge and calibrated human raters.
- Computed agreement metrics (Cohen's κ for two raters, Fleiss' κ or Krippendorff's α for multi-rater panels).
- Bias analysis (does the judge over- or under-flag certain subgroups relative to humans?).
- Recalibration cadence tied to model or prompt updates.

The L30 role owns this study design and execution. The risk engineer consumes the calibration report and refuses to credit judge scores whose calibration is stale.

<!-- needs-research: cite specific published guidance on judge-vs-human calibration for LLM-as-a-judge (Zheng et al. LLM-as-a-Judge; Chiang & Lee; Anthropic / OpenAI evaluation methodology posts) once verified against primary sources. -->

### 3. Inter-rater reliability on human evaluation panels

Where human raters are used (safety-red-team verdicts, fairness case reviews, hallucination scoring), inter-rater reliability metrics — Cohen's κ (two raters), Fleiss' κ (multi-rater, categorical), Krippendorff's α (multi-rater, mixed-scale) — are the L30 role's domain. A κ below common thresholds (≈ 0.4 slight to moderate agreement; ≈ 0.6 substantial; ≈ 0.8 near-perfect — thresholds vary by source and by the cost of disagreement) means the human panel's output cannot anchor a register row on its own.

### 4. Sample-size and power analysis for evaluations

Whether a red-team run of 300 prompts against a guardrail is large enough to distinguish an FNR of 2% from an FNR of 5% at 80% power — that is a sample-size question the L30 role answers before the run. The risk engineer sees the answer as either "the run is powered to detect the effect" or "the run is under-powered and cannot anchor the residual credit."

### 5. Correction for multiple comparisons across subgroups

When fairness is audited across k subgroups, at least some of the observed differences will be spurious. Bonferroni, Holm, or false-discovery-rate corrections apply. The risk engineer reads the corrected p-values and effect sizes rather than the uncorrected ones; the correction methodology is L30.

### 6. Distributional-shift diagnostics

Distinguishing "the guardrail's FNR moved from 2% to 4% because the world changed" from "…because our sample was small" is a change-point / drift-detection question at depth (Kolmogorov-Smirnov, PSI, MMD, etc.). The L30 role designs the diagnostic; the risk engineer consumes the "drift detected: yes / no / uncertain" signal and moves it into monitor logic (mod-109).

### 7. Causal-inference support for control effectiveness

"Adding this guardrail reduced FNR from 5% to 2%" is a causal claim. In a controlled A/B, the L30 role designs the experiment, controls for confounders, and reports the effect with a CI. Observational claims ("we added the guardrail, and monthly incident counts dropped") are much weaker; the L30 role helps you understand *how much* weaker and whether the number can be credited.

## The consumption contract

Each register row that credits a calibrated evidence point carries a small provenance record:

```yaml
credited_controls:
  - id: CTL-CS-RAG-PROV-01
    evidence_type: guardrail-effectiveness-report
    provenance:
      author_role: model-evaluation-engineer-L30       # or vendor + reviewer
      report_ref: eval-reports/2026-q2/prov-fnrate.md
      report_version: v2
      sample_size: 4200
      observation_window: 2026-04-01..2026-06-30
      effect_estimate: 0.02
      ci_95: [0.01, 0.04]
      calibration_notes: "judge κ vs. 3-human panel = 0.82 on stratified 500-item sample"
      staleness_check: "recomputes on model version bump; last check 2026-06-28"
    risk_engineer_credit_choice: ci_upper_bound        # scored at 0.04 for residual
```

Points to note about that shape:

- **Author role is named.** If it is a vendor, the internal reviewer who signed off is named separately.
- **Report version is pinned.** Report v2 says one thing; report v3 may say another. The row cites v2 until it is refreshed.
- **The CI is captured alongside the point estimate.** The register is not tricked by point estimates.
- **The judge calibration is noted.** If κ is missing or too low, the row's credit is not defensible.
- **Staleness check is wired.** A model version bump forces a recompute; without that wire, the credit ages silently.
- **The risk engineer's choice of estimate is documented.** "We scored against the CI upper bound because this is an adverse scenario" is a decision the register makes explicit.

## What happens when the L30 role does not exist

Many level-25 programs run at a maturity where a peer `model-evaluation-engineer` role is aspirational rather than staffed. Three pragmatic postures for the risk engineer in that case:

- **Consume vendor calibration under review.** Vendor-supplied numbers (LLM providers, guardrail vendors, benchmark maintainers) are treated as *inputs to a review*, not as final. The review can be a short read by anyone with statistics literacy in the org (a data scientist, an SRE with a stats background, a research colleague). Name the reviewer on the row.
- **Use CI-upper-bound / lower-bound as a default.** In the absence of custom calibration, the CI-upper-bound-for-adverse / CI-lower-bound-for-credit convention above bakes in caution. This is defensible under audit as "we picked the pessimistic estimate."
- **Escalate the coverage gap.** File the missing L30 role as a program-level coverage gap in your risk register itself. This is what makes the argument that the register's quality is bounded by the depth of upstream calibration; if the org wants tighter numbers, it needs the L30 role.

Do not silently take on the L30 work under time pressure. It shows up as either shallow calibration (worse than the vendor number you started with) or as scope creep that eats your MEASURE-side output.

## What about the L40 boundary? (`agentic-safety-engineer`)

Mod-101 ch 06 and mod-102 ch 06 already pinned the L40 boundary at the ladder and harm-model levels. This chapter is not the place to re-derive it; the register-shaped restatement is short.

- Rows whose *calibration methodology* requires frontier or dangerous-capability evaluation (deceptive-alignment probing, sandbagging detection, situation-aware behaviour, long-horizon autonomous evaluation) do not escalate to L30. They escalate to L40.
- The escalation shape is the same as mod-102 ch 06's `frontier_escalations` block: cite the row, name the reason, name the L40 owner, keep residual in scope on the L25 side.
- The distinction: L30 helps you *measure well* on scenarios your enterprise methodology can already reach. L40 owns *the methodology itself* for scenarios your enterprise methodology cannot reach.

If a row cannot be answered by better calibration alone — no amount of bootstrap CIs on an existing eval will surface the risk — the escalation is L40, not L30.

## Handoff artifacts at the L25 ↔ L30 seam

Two documents keep the seam working:

### 1. The calibration request

When a register row needs a calibration the L30 role has not yet produced, the risk engineer files a *calibration request*:

```yaml
calibration_request:
  request_id: CAL-REQ-2026-Q3-014
  requesting_row_ids: [HR-CS-INJ-01, HR-CS-INJ-02]
  ask: |
    Bootstrap 95% CI on guardrail CTL-CS-RAG-PROV-01 false-negative rate
    against the mod-104 indirect-prompt-injection prompt set v3;
    recompute with each model version bump (>= quarterly).
  needed_by: 2026-09-15
  blocking_treatment_plan: TP-CS-INJ-01
```

This is what turns "we need better numbers" into a piece of tracked work for the L30 role. Without the artifact, the ask is a hallway conversation and it lapses.

### 2. The calibration report acceptance

When the L30 role delivers a report, the risk engineer's acceptance is a documented act:

```yaml
calibration_acceptance:
  report_ref: eval-reports/2026-q2/prov-fnrate.md v2
  reviewer_role: ai-risk-engineer-L25
  reviewer_person: <name>
  accepted_for_rows: [HR-CS-INJ-01, HR-CS-INJ-02]
  credit_choice: ci_upper_bound
  staleness_trigger: model_version_bump
  notes: "Judge κ 0.82 is above internal threshold 0.7 — judge score anchors."
```

Acceptance is not tacit. The register credits the number *because* the risk engineer accepted the report with an explicit note.

## Anti-patterns

- **Point-estimate laundering.** A vendor scorecard reports a single accuracy number with no CI. The risk engineer files it directly into a residual-likelihood claim. The number could be off by an order of magnitude and the register cannot see it.
- **Judge scores anchoring without κ.** LLM-as-a-judge grades are used as the sole anchor with no human calibration study. The auditor asks "how do you know the judge is right?" and the answer is silence.
- **In-house bootstrap by a non-statistician.** A risk engineer writes a bootstrap loop and reports the interval. It is trivial to get the resampling unit wrong (e.g., resampling prompts when the natural unit is *conversations*) and produce a CI that is much too tight. Own the ask; do not own the method.
- **Calibration staleness ignored.** A report from Q1 anchors residual scores in Q3 after a model bump. The register believes an old number. Wire the staleness trigger.
- **Silently absorbing L30 scope.** The peer role isn't staffed; the risk engineer starts running calibration studies. Their MEASURE-side output slips. File the coverage gap; do not silently absorb.

## Interfaces with adjacent modules

- **Mod-104 (LLM red teaming)** produces the attack-success-rate numbers the L30 role calibrates; the risk engineer consumes the calibrated version.
- **Mod-105 (adversarial ML)** produces robustness numbers similarly.
- **Mod-106 (fairness / bias / explainability)** requires subgroup effect calibration and multiple-comparison corrections; the L30 role owns the correction methodology.
- **Mod-108 (guardrails)** produces the guardrail-effectiveness numbers that credit residual scores; L30 calibrates each.
- **Mod-109 (continuous monitoring)** produces the drift signals; L30 supplies the drift-detection methodology.
- **Mod-110 (MRM and supply chain)** requires the register to cite calibrated vendor claims; L30 reviews vendor calibration reports on the risk engineer's behalf.

Whether the peer role is staffed at your organisation or not, the *contract* remains. The register is only as defensible as its evidence, and the evidence is only as defensible as its calibration.

## Summary

- Level-25 competence around calibration is *literacy*, not methodology: read a CI, know what κ means, ask for sample size and window, refuse uncalibrated numbers.
- Bootstrap CIs, judge-vs-human calibration, inter-rater reliability metrics, sample-size and power analysis, multiple-comparison corrections, distributional-shift diagnostics, and causal-inference support for control effectiveness are the L30 peer's work.
- Every credited evidence point on a register row carries a provenance record — author, report version, sample size, window, CI, calibration notes, staleness trigger, and the risk engineer's explicit credit choice.
- If the L30 role is not staffed, consume vendor calibration under review, default to pessimistic-choice conventions (CI upper bound for adverse, lower bound for credit), and file the missing role as a coverage gap.
- Rows whose calibration cannot be answered by better methodology on existing evaluations escalate to L40 (`agentic-safety-engineer`), not to L30.
- Handoffs at the seam are documented: a calibration request when a number is needed, a calibration acceptance when a report is used. Neither is tacit.
