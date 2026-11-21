# Lab 01 — Metric-Choice Five-Liner and Impossibility-Collision Drill

**Duration:** 45 minutes · **Format:** guided practice, markdown authoring on a fixed fabricated scenario

## Objective

Internalise chapter 01's **metric-choice five-liner** as a pinned, policy-first artefact — authored *before* a Fairlearn or AIF360 call is made — plus the short **impossibility-collision paragraph** that names which Kleinberg / Chouldechova result is engaged by the scenario's base rates and which side of the collision you have chosen. The lab walks a single fabricated hiring-shortlist scenario end-to-end so that when exercise 01 asks for three scenarios on the real running model, the five-liner and the impossibility paragraph are shapes you compose from memory rather than re-derive.

Metric roulette (picking after the numbers come in) and metric monism (headlining one metric without naming what you did not headline) are the two first-pass defects chapter 01 names. This lab drills the shape that forecloses both.

## Prerequisites

- Chapter 01 read — specifically "The group-fairness families you actually ship", "The impossibility results — why you must pick", "The metric-choice decision table", and "Worked example — the metric-choice five-liner".
- Chapter 06 skimmed enough to recognise the NYC Local Law 144 / EEOC UGESP anchors by section number.
- A terminal where you can create a working directory and author a markdown file.
- No external access or compute required; the scenario and the base-rate facts are fixed below.

## The fixed example system

Use this system throughout the lab — do *not* substitute your own. Carrying a fixed scenario keeps the lab bounded to the five-liner authoring skill.

> **`ShortList`** — a resume-pre-screening classifier that produces a per-applicant **shortlist / no-shortlist** decision for a software-engineering role posted in New York City. BERT-base fine-tuned on a labelled corpus of ~85k historical applications and hiring outcomes at the employer; the shortlist feeds a human recruiter who schedules phone screens from the top of the queue. The tool substantially assists discretionary decision-making and is in scope for NYC Local Law 144.
>
> Decision produced: binary `shortlist=1` / `no-shortlist=0`.
> Downstream consumer: a human recruiter who reads shortlisted resumes and schedules phone screens.
> Population: applicants to the posted role over the audit window.

### Base-rate facts (fabricated, authoritative for this lab — do not revise)

Copy these as given. They are the facts the five-liner and the impossibility paragraph argue over.

| Group | Applicant count in audit window | Historical positive-class rate (`hire=1` in the training labels) |
|---|---|---|
| sex=Male | 4 820 | 18.1 % |
| sex=Female | 1 870 | 11.4 % |
| race/ethnicity=White | 3 560 | 19.4 % |
| race/ethnicity=Asian | 1 940 | 16.2 % |
| race/ethnicity=Hispanic or Latino | 620 | 10.1 % |
| race/ethnicity=Black or African American | 480 | 9.3 % |
| race/ethnicity=Two or More | 90 | 12.2 % |

Harm-model row stubs (fixed — do not add more):

| Harm-row id | One-line harm |
|---|---|
| `HM-SL-01` | A qualified applicant from a protected class is not shortlisted and is therefore denied access to the opportunity to interview. |
| `HM-SL-02` | The intersectional cell (sex × race/ethnicity) carries a disparity that single-axis reporting hides from the recruiter and from the LL144 auditor. |
| `HM-SL-03` | A historically-biased label (past hiring decisions) is encoded as the target variable, so predictive parity on that label would carry the historical bias forward. |

Register row stubs (fixed — do not add more):

| Register-row id | Scope line |
|---|---|
| `REG-SL-DISPIMP-01` | Disparate-impact / four-fifths violation on the shortlist decision |
| `REG-SL-EO-01` | Equal-opportunity gap on the shortlist decision among applicants with ground-truth positive labels |

## Steps

### Step 1 — Create your working directory and the scaffold

```bash
mkdir -p mod106-lab01 && cd mod106-lab01
touch shortlist-metric-choice.md
```

Open `shortlist-metric-choice.md` and paste this scaffold. You will fill five sections in order.

```markdown
# ShortList — metric-choice five-liner (lab 01 slice)

System: ShortList (BERT-base shortlist classifier for a NYC software-engineering role; see lab brief)
Chapter references: mod-106 ch 01 (metric families, impossibility results, decision table, five-liner); ch 06 (LL144 / EEOC anchors)
Harm-model rows cited: HM-SL-01, HM-SL-02, HM-SL-03
Register rows cited: REG-SL-DISPIMP-01, REG-SL-EO-01

## 1. Scenario brief

(5–10 lines — decision produced, population, base-rate facts with source, downstream consumer, jurisdiction / regulator)

## 2. Decision-table trace

(one-line cite of the chapter-01 decision-table row with regulatory anchor by CFR / statute number; one sentence naming a rejected row and why)

## 3. Five-liner

(the chapter-01 template, five lines, one per labelled item)

## 4. Impossibility-collision paragraph

(≤10 lines — which Kleinberg / Chouldechova / DP-vs-error-rate collision is engaged by the base rates; which side you chose; what a reviewer holding the other side would argue)

## 5. Cross-refs

(mod-102 harm-row id → "primary harm" line; mod-103 register-row id → "headline metric" field)
```

### Step 2 — Fill Section 1 (Scenario brief)

The brief is the fact-pattern the five-liner argues *about*. Fill it with the material from the fixed example and the base-rate table. Do not paraphrase; name every field.

```markdown
ShortList produces a binary `shortlist=1` / `no-shortlist=0` decision on each applicant to a
software-engineering role posted in New York City. The decision substantially assists a human
recruiter who schedules phone screens from the top of the shortlist queue; the tool is in scope
for NYC Local Law 144. The audit window covers 6 690 applicants (4 820 men, 1 870 women; the
race/ethnicity breakdown per EEO-1 vocabulary is in the lab's base-rate table). Historical
positive-class rates in the training labels differ across groups — 18.1 % for men vs 11.4 % for
women; 19.4 % White, 16.2 % Asian, 10.1 % Hispanic or Latino, 9.3 % Black or African American,
12.2 % Two or More — a documented base-rate gap in the historical hiring data. The downstream
consumer is the recruiter; the jurisdictional regulator for the headline obligation is the NYC
Department of Consumer and Worker Protection (DCWP) under 6 RCNY §5-300 et seq., with the US
EEOC Uniform Guidelines on Employee Selection Procedures (29 CFR Part 1607, §1607.4(D)) as the
underlying US-employment adverse-impact shape.
```

Reading check: the brief names the decision, the population size per group, the base-rate gap, the consumer, and the jurisdiction — nothing is hand-waved. A reviewer should be able to read Section 1 alone and see what the five-liner is going to anchor to.

### Step 3 — Fill Section 2 (Decision-table trace)

Chapter 01's decision table puts "US employment screening" on a specific row. Trace the match with the regulatory anchor cited by CFR / statute number, not by a summary URL, then name one row you *rejected* with a sentence of reason.

```markdown
**Row matched.** Employment screening (US) — resume filter, video interview scorer, skills-test
scorer; primary harm is selection-rate disparity producing adverse-impact liability. Headline
metric: disparate impact / four-fifths. Diagnostic metrics: equal opportunity, false-positive
rate parity. Regulatory anchor: EEOC Uniform Guidelines on Employee Selection Procedures
§1607.4(D) (29 CFR Part 1607); NYC Local Law 144 (6 RCNY §5-300 et seq.).

**Row rejected.** Clinical risk stratification — rejected because the decision is not a score
consumed by a physician but a binary shortlist consumed by a recruiter; the primary harm is
denial of opportunity, not inconsistent score meaning across groups. Calibration by group is
therefore the wrong headline family for this decision.
```

Reading check: the row matched carries the CFR / statute number; the rejected row has a one-sentence reason that names *why the headline family does not fit*, not a generic "not applicable."

### Step 4 — Fill Section 3 (Five-liner)

This is the load-bearing artefact. Use the chapter-01 template verbatim — five lines, one per labelled item. Do not merge lines, do not paraphrase the metric name, do not fold the "metrics we explicitly do not headline" line into a general note.

```markdown
1. **Decision context.** ShortList produces a binary shortlist / no-shortlist decision on
   applicants to a NYC software-engineering role; the tool substantially assists a human
   recruiter and is in scope for NYC Local Law 144.
2. **Protected-attribute partition.** Sex (EEO-1 categories) and race/ethnicity (EEO-1
   categories), plus the intersectional sex × race/ethnicity partition required by LL144.
3. **Primary harm.** A qualified applicant from a protected class is not shortlisted, denying
   them access to the opportunity to interview (HM-SL-01), with a specific intersectional cut
   where single-axis reporting hides the disparity (HM-SL-02).
4. **Headline metric.** Disparate impact / four-fifths on shortlist selection rate per EEO-1
   category *and* per intersectional cell, per the EEOC Uniform Guidelines §1607.4(D) and the
   DCWP LL144 impact-ratio field; equal opportunity reported as a diagnostic on the subset of
   applicants with ground-truth positive labels.
5. **Metrics we explicitly do not headline.** Predictive parity — base rates differ across
   groups in our historical hiring labels (see the lab's base-rate table), and the labels
   themselves are a documented product of past hiring decisions (HM-SL-03), so headlining
   predictive parity would encode that historical bias forward. We note this trade-off per
   Chouldechova (2016).
```

Reading check: the five-liner is five discrete lines, each labelled; the headline metric names a specific family *and* names the two regulators' fields; the "do not headline" line names *which* metric you are not headlining *and why that is defensible* — not a general shrug.

### Step 5 — Fill Section 4 (Impossibility-collision paragraph)

Chapter 01 names three impossibility results: Kleinberg calibration-vs-balance, Chouldechova predictive-parity-vs-equalised-odds, and demographic-parity-vs-error-rate. The paragraph should name *which* one engages the base rates in this scenario, *which side* you chose, and *what a reviewer holding the other side would argue*.

```markdown
Base rates differ across groups (men 18.1 %, women 11.4 %; White 19.4 %, Black 9.3 %, Hispanic
10.1 %, Asian 16.2 %), so **Chouldechova (2016)** is engaged: predictive parity and equalised
odds cannot both hold on the shortlist decision. Headlining disparate impact / four-fifths plus
equal opportunity means we are prioritising *access to opportunity for qualified applicants in
the protected class* — the equal-opportunity side of the Chouldechova trade-off. A reviewer
holding the predictive-parity side would argue that we should equalise precision across groups
so that `shortlist=1` means the same thing (same conditional probability of being hired given
the label) irrespective of group — the ProPublica-style objection in reverse. We disagree in
this context because the primary harm (HM-SL-01) is denial of the interview *to a qualified
applicant*, which equal opportunity targets directly, while predictive parity would carry the
documented label bias (HM-SL-03) forward. We also note the **demographic-parity-vs-error-rate
collision**: four-fifths on selection rate will not satisfy equalised odds at these base rates
either, which is why we report equal opportunity as a diagnostic — not demographic parity — on
the ground-truth-positive subset.
```

Reading check: the paragraph names (a) the specific result engaged (Chouldechova), (b) which side you took (equal-opportunity side), (c) what the opposite-side reviewer would argue, and (d) the ancillary demographic-parity-vs-error-rate collision that explains why the diagnostic metric is equal opportunity rather than demographic parity. A paragraph that reads "no collision because base rates are equal" would be wrong on the numbers; a paragraph that names the result but not the sides is incomplete.

### Step 6 — Fill Section 5 (Cross-refs)

The five-liner is only auditable if its "primary harm" line is traceable to a mod-102 row and its "headline metric" field is traceable to a mod-103 register row. Fill the cross-refs explicitly.

```markdown
| Five-liner line | mod-102 harm-row | mod-103 register-row |
|---|---|---|
| Primary harm — "qualified applicant not shortlisted" | HM-SL-01 | REG-SL-DISPIMP-01, REG-SL-EO-01 |
| Primary harm — intersectional single-axis hiding | HM-SL-02 | REG-SL-DISPIMP-01 (intersectional cell row) |
| "Do not headline predictive parity" justification | HM-SL-03 (historically biased labels) | — (noted on both register rows as a trade-off accepted) |
```

Reading check: every substantive line in the five-liner has at least one harm-row and (where appropriate) one register-row behind it. A five-liner with no cross-refs is a five-liner whose "primary harm" line was invented rather than copied from mod-102 — the exact anti-pattern chapter 01 names.

### Step 7 — Final checkpoint

Re-read the five sections. Confirm:

- **The brief carries the base-rate facts with a source for each number.** Base rates are not estimates in this lab; they are from the fixed table.
- **The decision-table trace cites the CFR / statute number, not a URL or a summary.** "EEOC UGESP §1607.4(D)" and "NYC LL144 (6 RCNY §5-300 et seq.)" — not "the EEOC website."
- **The five-liner is five discrete labelled lines.** Not a paragraph, not four lines, not six.
- **The impossibility paragraph names a specific result, the side taken, and the opposite-side argument.** Not generic.
- **Cross-refs land on a harm-row id *and* a register-row id for every substantive five-liner line.** Not blank.
- **The intersectional partition is in the protected-attribute line.** Sex × race/ethnicity, not two single-axis lines.

## Expected output

`mod106-lab01/shortlist-metric-choice.md` — one populated five-section document (scenario brief, decision-table trace, five-liner, impossibility paragraph, cross-refs); half a page to one page. The five-liner is in the exact chapter-01 template, the impossibility paragraph names Chouldechova and the equal-opportunity side taken, and the cross-refs land on `HM-SL-01 / HM-SL-02 / HM-SL-03` and `REG-SL-DISPIMP-01 / REG-SL-EO-01`.

## Checkpoint questions (self-check)

1. The lab's five-liner headlines disparate impact / four-fifths and reports equal opportunity as a diagnostic. A reviewer argues the headline should be demographic parity "because it is the simplest metric and the number in Fairlearn comes back smallest." Per chapter 01, what are the two specific objections to that argument? (First, metric choice is a *policy decision* anchored to the harm model and the regulator, not a technical decision driven by whichever number flatters the model — picking demographic parity because its number is smallest is **metric roulette** by name. Second, the chapter-01 decision table pins US employment screening to disparate impact / four-fifths per the EEOC Uniform Guidelines §1607.4(D) and the LL144 impact-ratio field — demographic parity is listed as the headline for *advertising / outreach allocation*, not for employment screening where the primary harm is denial of opportunity.)
2. The impossibility paragraph names Chouldechova as the engaged result. A reviewer suggests the demographic-parity-vs-error-rate collision should be the one named instead "because demographic parity is involved." Per chapter 01, what is wrong with that substitution? (Demographic parity is *not* the headline in this five-liner; it is not involved in the decision. The engaged result is between the two metrics actually being compared — equal opportunity (headline diagnostic) and predictive parity (the metric the reviewer-counter-argument would headline) — which is the **Chouldechova (2016)** result at unequal base rates. The demographic-parity-vs-error-rate collision explains an ancillary point (why we report equal opportunity rather than demographic parity on the ground-truth-positive subset) but is not the primary impossibility engaged by the five-liner's choice.)
3. The five-liner's "do not headline predictive parity" line cites `HM-SL-03` (historically biased labels) as the justification. A junior engineer suggests dropping this line "since predictive parity is just one of many metrics we are not headlining and we cannot list them all." Per chapter 01, why is the explicit "do not headline" line required, and why is predictive parity specifically the one to name? (The explicit line is required because chapter 01's shippable formulation is: "satisfies named metric within ε; does *not* satisfy named metric 2 and named metric 3; justification is ..." — the metrics-not-headlined line is part of the formulation that makes the choice honest. The specific metric to name is the one a *credible opposite-side reviewer* would headline — here, a predictive-parity advocate would argue that `shortlist=1` should mean the same thing across groups. Naming the metric and the reason (`HM-SL-03` — the label itself carries bias, so predictive parity would propagate it) forecloses the mid-review metric-switch anti-pattern chapter 01 names.)
4. The lab's base-rate table has `race/ethnicity=Two or More` at 90 applicants. A reviewer argues this cell should be dropped from the five-liner's partition "because the count is below 30 and the number will be noisy." Per chapter 01 and chapter 02's anti-patterns, what is wrong with that proposal? (Dropping a low-count cell is a *reporting* decision taken at the slice-report stage (chapter 02 — flag as `low-power`, carry a Wilson CI, do not delete), not a *partition* decision taken at the five-liner stage. The protected-attribute partition line in the five-liner names the EEO-1 vocabulary; dropping a category from the partition at authoring time hides the category from the entire downstream report, including the LL144 field list which expects the full EEO-1 breakdown. The correct treatment is: keep the category in the partition, flag the cell as low-power in the slice report, note the caveat on the metric value, and do not delete the row from any table.)

## Why this lab matters

Exercise 01 asks you to author three metric-choice five-liners on the real running system in two hours, across different decision-table rows, with impossibility-collision paragraphs and cross-refs for each. If the five-liner-authoring step consumes most of the budget because you are re-deriving the template, the impossibility-result choice, and the cross-ref shape from scratch, the exercise blows its budget before the third scenario is reached. This lab drills the atomic skill — one fabricated scenario authored end-to-end through all five sections — until the shape is routine, so exercise 01 is *picking the decision-table row and defending the choice* rather than *inventing the output format*.
