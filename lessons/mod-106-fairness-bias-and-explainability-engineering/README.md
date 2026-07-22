# mod-106 — Fairness, Bias, and Explainability Engineering

**Estimated effort:** 16 hours
**Position in the track:** sixth module; assumes mod-101 (frameworks & role scope), mod-102 (harm modelling), mod-103 (risk quantification & registers), mod-104 (red-team scope), and mod-105 (adversarial ML) are done.

Mod-101 through mod-105 fixed *what* to file, *what* the harms are, *how much* they cost, and *how* the adversary reaches them. This module produces the fairness and explainability slices of the audit — the two artifacts that discharge NIST AI RMF MEASURE 2.11 (fairness evaluated), EU AI Act Articles 10 (data governance) and 14 (human oversight), the EEOC Uniform Guidelines on Employee Selection Procedures, and — for AEDTs in scope — NYC Local Law 144.

The engineering deliverable is not "fairness." It is a *specific metric*, chosen against a *specific harm*, computed against a *specific protected-attribute partition*, mitigated to a *specific operating point on a Pareto curve*, explained on *specific instances* through the *specific tool* the downstream consumer needs, and packaged into a *report that an independent auditor can sign*. This module is that shape end-to-end.

The role boundary that anchors the whole module: **this role builds the bias-audit report; `ai-governance-analyst` (level 15) files it; an independent third-party auditor (typically a ForHumanity Independent AI Auditor or a BABL AI Algorithm Bias Auditor) signs it.** The three do not collapse — LL144 explicitly requires the auditor be independent.

## Learning objectives

- Design and ship group-fairness evaluations (demographic parity, equalised odds, equal opportunity, calibration by group, disparate impact / four-fifths, predictive parity) with Fairlearn and AIF360; know the Kleinberg / Chouldechova impossibility results that force a metric choice.
- Design and ship individual-fairness and causal-fairness diagnostics where the metric family fits (individual — Dwork et al.; causal — Kusner et al. counterfactual fairness); use Holistic AI to sweep multi-dimensional trustworthiness as a "did we miss an axis" scan.
- Ship a mitigation loop — pre-processing (reweighing, disparate-impact remover, learning fair representations), in-processing (adversarial debiasing, exponentiated gradient reductions), post-processing (threshold optimisation, calibrated equalised odds) — and quantify the accuracy / fairness trade-off as a *curve*, not a *point*.
- Author an explainability slice with SHAP (KernelSHAP + TreeSHAP; global + local), Alibi (anchor, counterfactual), and LIME as a compare-point; know when explainability is load-bearing (adverse-action notice, regulator query, human-oversight surface, fairness debug) and when it is misleading (illusion of understanding, saliency traps, wrong tool for the model).
- Deliver an NYC Local Law 144 shape bias-audit report and an EEOC adverse-impact evaluation for an employment / lending / housing model — comfortable producing the artifact a ForHumanity / BABL AI-credentialed auditor can sign.
- Cite the boundary to `ai-governance-analyst` (level 15) — analyst *reads and files* the bias report; this role *builds* it; an independent third-party *audits* it.

## Chapters

1. [`01-fairness-metric-families-and-incompatibilities.md`](01-fairness-metric-families-and-incompatibilities.md) — the six group-fairness families you ship; Kleinberg / Chouldechova impossibility results; the metric-choice decision table; the metric-choice five-liner.
2. [`02-group-fairness-tooling-fairlearn-aif360.md`](02-group-fairness-tooling-fairlearn-aif360.md) — Fairlearn (assessment + `ThresholdOptimizer` + `ExponentiatedGradient`) and AIF360 (algorithm zoo); the reach-for table; the slice-report shape; release + drift dashboards.
3. [`03-individual-and-causal-fairness.md`](03-individual-and-causal-fairness.md) — individual fairness (Dwork et al.); counterfactual fairness (Kusner et al.); when the metric family fits, when it doesn't; Holistic AI as a breadth sweep; the when-to-use decision table.
4. [`04-mitigation-loop-and-tradeoff.md`](04-mitigation-loop-and-tradeoff.md) — pre-, in-, post-processing catalogues; the "which mitigation lands where" table by metric family; the accuracy / fairness Pareto curve; operating-point justification.
5. [`05-explainability-slice.md`](05-explainability-slice.md) — SHAP variants; Alibi anchors and counterfactuals; LIME as compare-point; when explainability is load-bearing vs misleading; the explainability + fairness cross-cut.
6. [`06-audit-shapes-nyc-ll144-eeoc-forhumanity-babl.md`](06-audit-shapes-nyc-ll144-eeoc-forhumanity-babl.md) — NYC Local Law 144 field list; EEOC Uniform Guidelines four-fifths and validity; ForHumanity IAA and BABL AI Algorithm Bias Auditor credentialed shapes; the L15 / L25 / independent-auditor split.

## Exercises

- [`exercises/exercise-01-fairness-metric-choice-drill.md`](exercises/exercise-01-fairness-metric-choice-drill.md) — walk the metric-choice decision table on three concrete scenarios and produce a metric-choice five-liner for each.
- [`exercises/exercise-02-fairlearn-and-aif360-eval-authoring.md`](exercises/exercise-02-fairlearn-and-aif360-eval-authoring.md) — author a Fairlearn evaluation and an AIF360 evaluation on the same model; produce the shared slice-report artifact.
- [`exercises/exercise-03-individual-and-causal-fairness-diagnostics.md`](exercises/exercise-03-individual-and-causal-fairness-diagnostics.md) — run perturbation-stability and (where the DAG supports it) a counterfactual-fairness diagnostic on the running model.
- [`exercises/exercise-04-mitigation-loop-with-accuracy-fairness-tradeoff.md`](exercises/exercise-04-mitigation-loop-with-accuracy-fairness-tradeoff.md) — sweep pre-, in-, and post-processing mitigations against the chosen metric; ship the Pareto plot and pick an operating point with written justification.
- [`exercises/exercise-05-shap-alibi-lime-explainability-slice.md`](exercises/exercise-05-shap-alibi-lime-explainability-slice.md) — author the explainability slice with SHAP, Alibi anchor + counterfactual, and LIME; produce the cross-tool agreement report.
- [`exercises/exercise-06-nyc-ll144-and-eeoc-audit-report-drill.md`](exercises/exercise-06-nyc-ll144-and-eeoc-audit-report-drill.md) — package the mod-106 outputs into an LL144-shape bias-audit report and an EEOC adverse-impact evaluation.

## Labs & quizzes

- [`labs/`](labs/) — a longer hands-on lab is scheduled here; see the lab README for the current status.
- [`quizzes/`](quizzes/) — knowledge-check quiz for the module.

## Resources

See [`resources.md`](resources.md) for the primary sources — the impossibility papers (Kleinberg, Chouldechova), the mitigation-algorithm reference literature, the explainability-tool project pages, the EEOC / NYC DCWP / Colorado / EU regulation text, and the ForHumanity / BABL AI credentialing bodies.

## How to work through the module

1. Pick **one classifier** (real or documented) that produces a decision with a fairness dimension — a hiring shortlist, a credit-approval score, a housing eligibility screen, a clinical triage classifier — and reuse it across every exercise. The running model from mod-102 / mod-103 is the natural choice.
2. Read chapters 01-06 in order. Each closes one section of the shippable audit report.
3. Do exercises 01-06 in order. They stack:
   - 01 pins the metric-choice five-liner,
   - 02 produces the Fairlearn + AIF360 slice report,
   - 03 adds the individual / causal diagnostics where they fit,
   - 04 runs the mitigation loop and picks an operating point,
   - 05 adds the explainability slice,
   - 06 packages everything into the LL144 + EEOC deliverable.
4. Take the quiz.

By the end of the module you should be able to hand a governance analyst (level 15) a bias-audit report where every number is (a) traceable to a Fairlearn / AIF360 / SHAP / Alibi run with a config hash, (b) anchored to a metric-choice justification and a harm-model row, (c) accompanied by a mitigation Pareto and a chosen operating point, and (d) formatted for an independent auditor (ForHumanity IAA or BABL AI Algorithm Bias Auditor) to sign against NYC Local Law 144 or the EEOC Uniform Guidelines. If any of those four is missing, the report is not shippable.
