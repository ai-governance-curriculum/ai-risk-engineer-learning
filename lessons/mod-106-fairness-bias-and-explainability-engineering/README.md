# mod-106-fairness-bias-and-explainability-engineering: Fairness, Bias, and Explainability Engineering

> Scaffolded by `aicg org execute-plan`. Lecture chapters and exercise content are authored on subsequent autonomous cycles.

**Estimated effort:** 16 hours

## Learning objectives

- Design and ship group-fairness evaluations (demographic parity, equalised odds, equal opportunity, calibration by group, disparate impact / four-fifths, predictive parity) with Fairlearn and AIF360; know the mathematical incompatibilities that force a metric choice
- Design and ship individual-fairness and causal-fairness diagnostics where the metric family fits the use case; use Holistic AI to sweep multi-dimensional trustworthiness
- Ship a mitigation loop — reweighting, adversarial debiasing, threshold optimisation post-hoc, sampling — and quantify the accuracy / fairness trade-off
- Author an explainability slice with SHAP (global + local), Alibi anchor / counterfactual, and LIME as a compare-point; know when explainability is misleading and when it is load-bearing
- Deliver an NYC Local Law 144 shape bias-audit report and an EEOC adverse-impact evaluation for a hiring / lending / housing model — comfortable running the ForHumanity / BABL AI algorithm-bias-auditor style artifact
- Cite the boundary to `ai-governance-analyst` (level 15) — analyst *reads* the bias report and files it; this role *builds* it

## Structure

- `01-…md` … `0N-…md`: lecture chapters.
- `exercises/`: per-exercise prompts.
- `labs/`: long-form hands-on labs.
- `quizzes/`: knowledge checks.
- `resources.md`: external references.
