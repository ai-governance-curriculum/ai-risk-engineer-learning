# Exercise 01 — Fairness Metric Choice Drill

**Estimated effort:** 2 hours

## Objective

Turn chapter 01's metric-choice decision table into a *pinned* choice on paper before you touch any tool. For three concrete decision systems you will produce the **metric-choice five-liner** — decision context, protected-attribute partition, primary harm, headline metric, metrics you explicitly do not headline — and defend it against the Kleinberg (2016) and Chouldechova (2016) impossibility results. The output of this exercise is the header block that every Fairlearn / AIF360 evaluation you author in the rest of the module reads from.

## Prerequisites

- Chapter 01 read; the impossibility papers (Kleinberg arXiv:1609.05807, Chouldechova arXiv:1610.07524) at least skimmed.
- Mod-102 harm-model rows for the running system available; you will pull the "primary harm" line straight from those rows.
- Mod-103 register at hand; the metric-choice justification is a register-row field and the row IDs are what you cite.
- A picked **running model** for the rest of mod-106 — one classifier with a real fairness dimension (hiring shortlist, credit approval, housing eligibility, clinical triage). Two of your three scenarios in this drill use the running model; one is a contrast case drawn from a different context so you can feel the decision changing.
- The regulatory anchors from the chapter-01 decision table available for citation (EEOC UGESP §1607.4(D); ECOA / Regulation B; Fair Housing Act; FDA GMLP; jurisdiction-specific text as applicable).

## Problem statement

Author the metric-choice five-liner for **three** decision systems. Two are anchored to *the running model* (one under its current framing, one under an *adversarial re-framing* where the regulator, the affected population, or a different jurisdiction would name a different headline metric); the third is a **contrast case** drawn from a different context (see the picker below) so the drill exercises the table across more than one row.

For each scenario, walk the decision explicitly:

1. Which row of the chapter-01 decision table does the context sit on?
2. Which harm-model row (mod-102) supplies the "primary harm" line?
3. Which regulator (or affected population) has standing to name the headline?
4. What are the base-rate facts about the population, and which impossibility result is engaged by them?
5. Which metrics are you *explicitly not* headlining, and why is that defensible?

The output is a five-liner per scenario, plus a short (≤10-line) *impossibility-collision paragraph* per scenario that names the Kleinberg / Chouldechova / demographic-parity-vs-error-rate collision and states which side of it you have chosen. Anything less than that paragraph leaves the choice indefensible under audit.

### Contrast-case picker

Pick the contrast case from one of the following, and note which you picked in the deliverable:

- **Consumer credit / lending** — a credit-approval score whose downstream is a lender's Reg B adverse-action notice.
- **Tenant screening** — a housing eligibility classifier consumed by a leasing agent under the Fair Housing Act.
- **Clinical risk stratification** — a hospital sepsis / readmission risk score consumed by a triage nurse.
- **Criminal-justice risk score** — a pretrial / recidivism risk score of the COMPAS shape, so you have to walk through the ProPublica / Northpointe collision on your own.
- **Content moderation across demographic groups** — a per-post removal decision consumed by an appeals process.

Do not pick a contrast case from the same row of the decision table as your running model — the exercise is worthless without cross-row spread.

## Requirements

Produce all six items below. Missing any is an audit finding on the drill.

1. **Scenario briefs** — `risk/mod-106/<system-id>/metric-choice/scenarios.md`. One 5–10 line brief per scenario naming: the decision produced, the population it is produced over, the base-rate facts (positive-class rate per protected group in the training / production population, with source), the downstream consumer, and the jurisdiction / regulator.
2. **Five-liners** — the same file, one per scenario, in the exact template from chapter 01 §"Worked example — the metric-choice five-liner". Do not skip a line, do not merge lines, do not paraphrase away the headline metric name.
3. **Impossibility-collision paragraph** — one paragraph per scenario naming which of Kleinberg calibration-vs-balance, Chouldechova predictive-parity-vs-equalised-odds, or demographic-parity-vs-error-rate is engaged by *this* scenario's base rates, which side you chose, and what a reviewer holding the other side would argue. If your paragraph reads "no collision because base rates are equal," you have not looked at the base rates — go back.
4. **Decision-table trace** — for each scenario, a one-line cite of the row of the chapter-01 decision table it maps to (with the row's regulatory anchor), plus one sentence naming any decision-table row you *rejected* and why.
5. **Harm-model + register cross-refs** — for each scenario, the mod-102 harm-row ID that supplies the primary-harm line, and the mod-103 register-row ID whose "headline metric" field this five-liner becomes.
6. **Reviewer-facing summary** — `risk/mod-106/<system-id>/metric-choice/README.md`. Three-paragraph document (one per scenario) written for a reviewer who has read chapter 01 but has *not* read your scenario briefs; the reviewer must be able to reconstruct your choice from the summary alone.

Cross-cutting rules:

- **Metric-choice is pinned before evaluation.** Once the five-liner ships from this exercise, exercises 02–06 read it. Do not renegotiate the metric because a Fairlearn number comes in inconvenient — that is metric roulette. If new evidence emerges, change the five-liner *on paper* first with a dated note explaining why.
- **The primary-harm line is copied, not invented.** It comes from mod-102. If the harm model does not have the harm you need, add the row to mod-102 first, then come back.
- **Regulatory anchors get cited by CFR / statute number, not a summary URL.** "EEOC UGESP §1607.4(D)" rather than "the EEOC website".
- **The three scenarios must land on at least two different rows of the decision table.** Otherwise the drill has not stretched the decision.

## Starter guidance

- **Read the running system's harm-model rows first.** The primary-harm line is upstream of everything; when it is fuzzy, the five-liner will be fuzzy. Sharpen it in mod-102 before starting here.
- **Look up the base-rate numbers, do not estimate.** For the running system these should be knowable from the training / production population; for the contrast case, cite the population source (labour-force participation stats, HMDA lending data, published clinical prevalence). A five-liner grounded in a guessed base rate is a five-liner that will be re-litigated.
- **The adversarial re-framing is where the exercise pays off.** Ask: if a plaintiff-side attorney, an investigative journalist, or a regulator in a different jurisdiction reviewed this decision, which metric would *they* name as the headline? If your first-pass metric survives that framing, defend it; if it does not, either change the metric or write the defence explicitly.
- **Do not conflate a metric with a mitigation.** Metric choice is chapter 01; mitigation is chapter 04 / exercise 04. A five-liner that says "we headline equal opportunity because we can post-process to it" is putting mitigation before metric choice — reverse the order.
- **When impossibility results collide, name the loser explicitly.** "We prioritise predictive parity; we accept that equalised-odds difference will not be zero per Chouldechova. The affected population's downstream harm is the score meaning, not the error balance, so this is the defensible side." That sentence is the scarce artifact.
- **For the criminal-justice contrast case, walk the ProPublica vs Northpointe debate on your own.** ProPublica pointed at FPR disparity; Northpointe pointed at predictive parity; Chouldechova proved both were right, base rates differing. The exercise is deciding which side you are on given the harm model — not repeating that both are right.
- **Intersectional slices are metric decisions too.** If the primary harm attaches to a sex × race/ethnicity intersection (typical of LL144 shape), the five-liner should say so — the protected-attribute partition line is intersectional, not two single-axis lines.

## Acceptance criteria

The submission is accepted when:

- Three scenario briefs exist at the specified path, with base-rate facts cited to a source (not guessed).
- Three five-liners exist in the exact chapter-01 template, one per scenario, each naming a headline metric and metrics explicitly not headlined.
- Three impossibility-collision paragraphs exist, each naming a specific result (Kleinberg, Chouldechova, or the demographic-parity-vs-error-rate collision) that engages the scenario's base rates, and naming which side of the collision you took.
- Each scenario cites the chapter-01 decision-table row it sits on with the regulatory anchor by CFR / statute number, and names at least one row it rejected.
- Each scenario cross-refs a mod-102 harm-row ID and a mod-103 register-row ID.
- The three scenarios span at least two different rows of the decision table.
- The reviewer-facing README reconstructs the choice for a reader who has read chapter 01 but not the briefs.
- Nothing in the deliverable requires the reader to have already seen the Fairlearn or AIF360 numbers — the metric choice must stand *before* the assessment runs.

## Stretch goals

- Add a **fourth scenario** on the *same running system* but under a *different jurisdiction* (Colorado AI Act, EU AI Act Article 10 / 14 slice, UK Equality Act) and observe whether the headline metric changes with the regulator. This is the shape the mod-110 vendor-review conversation needs when the same model ships into multiple markets.
- Write a **counter-brief** for one scenario in the voice of a plaintiff-side attorney or the affected population's representative: which metric would they headline and why? Ship the counter-brief alongside the five-liner. If the counter-brief lands on the same headline, defend the convergence; if it lands elsewhere, name the divergence in the impossibility paragraph.
- Contribute a **new row to the chapter-01 decision-table** for a context not currently covered (public-benefits eligibility, immigration triage, education admissions) — draft the row with the primary harm, headline metric, diagnostic metrics, and regulatory anchor. Cite the anchor by primary source.
- Draft the **model-card fairness section stub** (mod-112 shape) that each five-liner will feed into: named headline metric, named diagnostic metrics, named impossibility trade-off accepted, named policy justification. A stub of ~15 lines per scenario is enough.

## Deliverable format

A single markdown document `mod-106-ex01-metric-choice.md` that indexes the artifacts, plus the `scenarios.md` and `README.md` files. The index document contains:

1. *Scope statement* — which system is the running model; which contrast case was picked and why; which decision-table rows are exercised.
2. *Scenario summaries* — three short paragraphs, one per scenario, each ending with the five-liner in a fenced block for copy-paste into a Fairlearn call header.
3. *Impossibility-collision paragraphs* — three short paragraphs, one per scenario, each naming the specific result engaged.
4. *Cross-refs table* — three rows: scenario | decision-table row + anchor | mod-102 harm-row ID | mod-103 register-row ID.
5. *Open items* — anything for exercises 02–06 (particularly which five-liner header exercise 02's Fairlearn call will consume) or upstream mod-102 / mod-103 (a harm row you had to add, a register field you had to define).

## Submission

Attach the index markdown, `scenarios.md`, and `README.md` to the module submission channel. If the running system is under NDA, redact model / product identifiers but preserve the five-liners in full, the base-rate facts, the impossibility paragraphs, and the decision-table trace — the metric-choice justification cannot be reviewed with the metric name blanked out.
