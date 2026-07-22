# Fairness Metric Families and the Incompatibilities that Force a Choice

Fairness is not a single number. It is a family of *mutually incompatible* mathematical properties, and picking the one your system will be judged against is a **policy decision that must be made before an evaluation is authored**. This chapter enumerates the group-fairness metrics you will actually ship, states the impossibility results that mean you cannot satisfy them all, and gives you a decision table that turns "which metric" into a routine.

## Motivation

Two failure modes an auditor spots immediately in first-pass fairness work:

- **Metric roulette.** The team runs Fairlearn on a classifier, sees demographic-parity difference at 0.03 and equalised-odds difference at 0.11, and writes the report against demographic parity because the number is smaller. There is no policy justification for the choice. When challenged, the team switches metric mid-review.
- **Metric monism.** The team locks in one metric ("we optimise for demographic parity") without acknowledging that a different metric — the one the regulator or the affected population would name — shows a violation. The report reads fine internally and fails externally.

Both failures come from treating "which fairness metric" as a *technical* question. It is not. It is a policy question: *which mistake are you willing to make against which group?* The engineering job is to make that decision legible, defensible, and pinned to the harm model.

## The group-fairness families you actually ship

Group-fairness metrics compare the classifier's behaviour across a *protected group* partition (race, sex, age, disability, national origin, or a domain-specific analogue). The six that a level-25 evaluation must be able to compute and defend are below. Notation: `A` is the protected attribute (`a=0` unprivileged, `a=1` privileged), `Y` is the true label, `Ŷ` is the classifier's prediction, `R` is a real-valued score.

| Family | Formal statement | What it demands | Where it is the right choice |
|---|---|---|---|
| **Demographic parity (statistical parity)** | `P(Ŷ=1 \| A=0) = P(Ŷ=1 \| A=1)` | Same positive-prediction rate across groups | Allocation of a scarce benefit where the *rate* is the fairness question (targeted advertising, outreach programmes, some lending contexts under fair-housing frames) |
| **Disparate impact / four-fifths rule** | `P(Ŷ=1 \| A=0) / P(Ŷ=1 \| A=1) ≥ 0.8` | Selection rate for the unprivileged group at least 80 % of the privileged group's rate | US employment law shape — EEOC Uniform Guidelines on Employee Selection Procedures; carried into NYC Local Law 144 AEDT reporting |
| **Equalised odds** | `P(Ŷ=1 \| Y=y, A=0) = P(Ŷ=1 \| Y=y, A=1)` for `y ∈ {0,1}` | Equal true-positive *and* equal false-positive rates across groups | Diagnostic-style decisions where both kinds of error matter to the affected party (medical triage, high-stakes eligibility) |
| **Equal opportunity** | `P(Ŷ=1 \| Y=1, A=0) = P(Ŷ=1 \| Y=1, A=1)` | Equal true-positive rate (recall for the positive class) across groups | Access to opportunity where withholding from a qualified individual is the harm you most want to equalise (loan approval for creditworthy applicants, admission for qualified applicants) |
| **Predictive parity (calibration by group)** | `P(Y=1 \| Ŷ=1, A=0) = P(Y=1 \| Ŷ=1, A=1)`; calibration: `P(Y=1 \| R=r, A=a)` equal across `a` for every `r` | Same precision (or same score-conditional risk) across groups | Risk-score deployments where a downstream decision-maker treats the score as a probability (recidivism scoring, credit scoring, clinical risk stratification) |
| **Treatment equality** | Ratio of false negatives to false positives equal across groups | Error *balance* is equalised even if magnitudes differ | Contexts where the *type* of error carries different social weight per group but you cannot equalise magnitudes without harming utility |

Two supporting families that show up as diagnostics rather than headline metrics:

- **False-positive rate parity** on its own (a strict sub-part of equalised odds) — used when the primary harm is being *falsely selected* (arrest, investigation, adverse action) and you want to hold that error equal even if you allow true-positive rate to drift.
- **False-negative rate parity** on its own — used when the primary harm is being *falsely rejected* (denied loan, denied benefit) and you accept some false-positive imbalance in exchange.

Every one of these has a Fairlearn or AIF360 primitive (chapter 02). None of them is universally correct.

## The impossibility results — why you must pick

Three formal results tell you that satisfying more than one of the above families simultaneously is, in most real cases, mathematically impossible.

### Kleinberg-Mullainathan-Raghavan (2016)

Kleinberg et al., "Inherent Trade-Offs in the Determination of Risk Scores" (arXiv:1609.05807), prove that except in the trivial cases of *perfect prediction* or *equal base rates across groups*, a real-valued risk score cannot simultaneously satisfy all three of:

1. **Calibration within groups** — the score means the same thing across groups.
2. **Balance for the positive class** — average score among true positives equal across groups.
3. **Balance for the negative class** — average score among true negatives equal across groups.

Base rates *always* differ in the settings you will actually be audited on. So *at least one* of the three must be violated. Which one you violate is a policy choice.

### Chouldechova (2016)

Chouldechova, "Fair Prediction with Disparate Impact: A Study of Bias in Recidivism Prediction Instruments" (arXiv:1610.07524), gives the corresponding result for binary classifiers with unequal base rates: **predictive parity and equalised odds (equal FPR and equal FNR) cannot both hold simultaneously**. This is the result that ended the ProPublica / Northpointe COMPAS debate as a matter of pure mathematics — both parties were right, they were pointing at different metrics, and no single classifier can satisfy both when base rates differ.

### Base-rate implication of demographic parity

Demographic parity and any error-rate-based metric (equal opportunity, equalised odds, predictive parity) also collide when base rates differ. Forcing equal positive-prediction rates across groups either forces unequal error rates or forces the classifier to ignore the label. This is not a subtle collision — it falls out of the definitions in one line — and it is why "just use demographic parity" is never a complete answer for a decision system.

### What the impossibility results mean for your report

You cannot ship a report that claims "the classifier is fair" without qualifying *along which axis*. The shippable formulation is:

> The classifier satisfies **{named metric}** within tolerance **{ε}** on the partition **{named protected attribute}**. It **does not** satisfy **{named metric 2}** and **{named metric 3}**; the policy justification for prioritising the named metric is **{cite harm model + regulator + affected-population reasoning}**.

That paragraph is the *only* honest fairness conclusion. Everything else is either dishonest or lucky (base rates happened to line up, which they do not in production populations).

## The metric-choice decision table

This table is the artifact you build once per model and revisit whenever the deployment context, protected-attribute definition, or downstream decision changes. It is the input Fairlearn or AIF360 evaluation code reads.

| Decision context | Primary harm you are equalising | Metric to headline | Metric to also report (diagnostic) | Regulatory anchor |
|---|---|---|---|---|
| Employment screening (US) — resume filter, video interview scorer, skills-test scorer | Selection-rate disparity produces adverse-impact liability | **Disparate impact / four-fifths** | Equal opportunity, false-positive rate parity | EEOC Uniform Guidelines on Employee Selection Procedures §1607.4(D); NYC Local Law 144 AEDT |
| Consumer credit / lending | Denial of credit to a creditworthy applicant from a protected class | **Equal opportunity** | Predictive parity, four-fifths | US Equal Credit Opportunity Act (ECOA) / Regulation B; UK Equality Act |
| Housing / tenant screening | Denial of housing to a qualified applicant | **Equal opportunity** + **four-fifths** | False-positive rate parity | US Fair Housing Act; HUD disparate-impact rule |
| Clinical risk stratification / diagnostic assist | Score meaning must be consistent across groups for the downstream physician | **Calibration by group / predictive parity** | Equalised odds | FDA GMLP guiding principles (mod-101) |
| Criminal-justice risk score | Both false positives (wrongful high-risk label) and false negatives (missed high-risk) carry harm | **Equalised odds** | Predictive parity (and be explicit you cannot have both) | State-level jurisprudence; ProPublica / COMPAS precedent literature |
| Content moderation across demographic groups | Over-removal of speech from an identifiable group | **False-positive rate parity** | Demographic parity of removals | Platform policy; sometimes UK Online Safety Act shape (jurisdiction-dependent) |
| Advertising / outreach allocation | Systematic exclusion of a group from a beneficial audience | **Demographic parity** | Four-fifths on selection into audience | HUD Facebook consent-decree shape; sector-specific advertising codes |
| Fraud / abuse detection | False-positive rate skew produces disparate scrutiny burden | **False-positive rate parity** | Equalised odds | Sectoral (e.g., anti-money-laundering supervisor guidance) |

Anything not on this list is not a novel decision — it is a decision that has not yet been anchored to a harm and a regulator. Anchor it before you author code.

## Worked example — the metric-choice five-liner

A shippable metric-choice justification is five lines. Use it as a template.

1. **Decision context.** *The model produces a shortlist for human-recruiter review from a pool of applicants for a US-based software-engineering role.*
2. **Protected-attribute partition.** *Race (per EEO-1 categories) and sex (per EEO-1 categories); partition definitions are pinned to the source data dictionary.*
3. **Primary harm.** *A qualified applicant from a protected class is not surfaced to the recruiter, denying them access to the opportunity.*
4. **Headline metric.** *Disparate impact / four-fifths on shortlist selection rate per group, per the EEOC Uniform Guidelines; equal opportunity as a diagnostic on the subset with known outcomes.*
5. **Metrics we explicitly do not headline.** *Predictive parity — base rates differ across groups in our historical hiring data (a documented bias in the label), so predictive parity would encode that bias into the shortlist criterion. We note this trade-off in the report per Chouldechova.*

That five-liner is what the header of every Fairlearn / AIF360 evaluation you author in mod-106 must carry.

## Interfaces with adjacent modules

- **Mod-101 (frameworks)** — MEASURE 2.11 (fairness evaluated) is discharged by this module; the register cites the audit report as evidence, the audit report cites this decision table.
- **Mod-102 (harm modelling)** — the "primary harm" line in the metric-choice five-liner is pulled directly from the harm-model row for the protected group.
- **Mod-103 (register)** — the metric-choice decision is a register-row field; changing metrics changes the residual score and must go through change control.
- **Mod-109 (monitoring)** — the headline metric selected here is what you monitor on production traffic sliced by protected group; drift on that metric is a first-order alert.
- **Mod-112 (model card)** — the model-card fairness section names the headline metric, the diagnostic metrics, the impossibility trade-off you accepted, and the policy justification. The card cannot be signed without this section.

## Summary

- Group fairness has six families you will ship: demographic parity, disparate impact / four-fifths, equalised odds, equal opportunity, predictive parity / calibration by group, treatment equality.
- Kleinberg (2016) and Chouldechova (2016) prove you cannot satisfy them all simultaneously when base rates differ (which is always).
- Which metric you headline is a **policy decision**, not a technical one; it must be anchored to the harm model and the regulator's shape.
- Ship a metric-choice five-liner per model — decision context, protected-attribute partition, primary harm, headline metric, metrics you explicitly do not headline.
- The metric-choice decision table anchors employment, lending, housing, clinical, criminal-justice, moderation, advertising, and fraud contexts to their respective headline metrics.
- Exercise 01 drills the metric-choice decision on three realistic scenarios.
