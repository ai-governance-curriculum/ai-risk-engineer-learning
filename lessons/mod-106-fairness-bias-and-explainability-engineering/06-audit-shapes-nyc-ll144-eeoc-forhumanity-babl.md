# Audit Shapes — NYC Local Law 144, EEOC Adverse-Impact, and the ForHumanity / BABL AI Credentialed Shape

Chapter 01-05 produced the technical shapes; this chapter produces the *report* — the deliverable a governance analyst files, a regulator receives, or an independent auditor signs. The two anchor shapes are **NYC Local Law 144** (Automated Employment Decision Tools) and the **EEOC** adverse-impact / four-fifths shape carried in the *Uniform Guidelines on Employee Selection Procedures*. The two credentialing bodies whose report templates the industry has converged on are **ForHumanity** (Independent AI Auditor) and **BABL AI** (Algorithm Bias Auditor). This chapter is the L15 / L25 handoff at the report boundary.

## Motivation

Two patterns of failure to name:

- **Technically-correct-but-un-fileable report.** The team runs Fairlearn, produces a beautiful `MetricFrame`, ships a JSON, and hands it to the governance analyst. The analyst cannot file it — LL144 wants specific fields, EEOC-style adverse-impact reporting wants specific ratios, and neither wants JSON. Weeks of rework.
- **Report done in prose without the numbers underneath.** The analyst writes the LL144 narrative first and asks the engineer to backfill numbers to support it. The numbers do not support it (the sweep shows a different picture). Panic. The analyst's report was upstream of the evaluation.

The clean shape is the opposite: the engineer builds the report against the *published field list* of the target regulation and the credentialed template of the target auditor, and the analyst files it with narrative wrapping.

## NYC Local Law 144 — AEDT bias audit

**Statute:** New York City Local Law 144 of 2021, effective 5 July 2023 (enforcement start).
**Rule:** NYC Department of Consumer and Worker Protection (DCWP) final rule, "Automated Employment Decision Tools" — codified at 6 RCNY §5-300 et seq.
**Coverage:** Any *automated employment decision tool* used to *substantially assist or replace* discretionary decision-making for *employment* decisions concerning candidates for employment or employees for promotion, where the position is in New York City.

### What the law requires

Two obligations that translate directly to engineering deliverables:

1. **Independent bias audit.** Before use and within one year of the most recent audit, the tool must have a bias audit performed by an *independent auditor* — one not involved in using, developing, or distributing the tool.
2. **Public summary of the audit results.** A summary must be made publicly available on the employer's website; candidates must be notified at least 10 business days in advance that an AEDT will be used.

### The required audit fields

The DCWP rule prescribes the *shape* of the audit. Per the rule (paraphrased — read the DCWP rule text directly for the authoritative wording):

- **Selection rate** per category (sex categories; race/ethnicity categories per EEO categories; intersectional sex × race/ethnicity categories).
- **Scoring rate** per category (where the tool produces a score rather than a selection).
- **Impact ratio** per category — for selection: the selection rate for a category divided by the selection rate for the most-selected category. For scoring: the scoring rate for a category divided by the scoring rate for the highest-scoring category.
- **Number of applicants** (or count of individuals scored) per category, disaggregated by sex, race/ethnicity, and intersectional.
- **Sources of data used** — historical data of the tool's use, or test data if historical is insufficient; the source must be identified in the summary.
- **Date of the audit** and the *auditor*.

<!-- needs-research: confirm the current published categorical vocabulary the DCWP rule references and whether the 2024 amendments changed any of the field list. -->

The public summary must include the impact-ratio numbers and the audit date. Employers and vendors reading the mod-106 output are the two entities that produce the underlying data feeding this shape.

### Mapping the mod-106 output to the LL144 field list

| LL144 field | Source in mod-106 output |
|---|---|
| Selection rate per category | Fairlearn `MetricFrame` `selection_rate` metric, sliced by the LL144-required categories |
| Scoring rate per category (score-based tools) | Same, on the `mean_score` metric or a threshold-based scoring rate |
| Impact ratio per category | Ratio computation in the mod-106 slice report — the four-fifths ratio surface |
| Number of applicants / individuals per category | The per-cell count column of `MetricFrame.by_group` |
| Intersectional (sex × race/ethnicity) | `sensitive_features=[sex, race_ethnicity]` in the Fairlearn call — one line change per chapter 02 |
| Data source | Data-provenance section of the mod-106 slice report |
| Audit date, auditor | Metadata block; the auditor is the *independent* party — see chapter 06 §5 on the L15 / L25 split |

The mod-106 slice report *is* the input to the LL144 audit; the audit is (LL144 field list) ∩ (mod-106 slice report), plus a narrative summary.

### The intersectional cut is non-optional

LL144 explicitly requires intersectional categories. A report that reports only single-axis (sex, or race, but not sex × race/ethnicity) does not satisfy the rule. This is why chapter 02 insists on the intersectional `MetricFrame` shape.

## EEOC adverse-impact and the four-fifths rule

**Anchor:** US Equal Employment Opportunity Commission (EEOC), *Uniform Guidelines on Employee Selection Procedures* (UGESP), 29 CFR Part 1607 (jointly adopted by EEOC, DOL, DOJ, OPM). Codified in 1978, cited essentially unchanged for automated selection today.

### What UGESP says (engineering read)

- **§1607.4(D) — Adverse impact and the four-fifths rule.** A selection rate for any race, sex, or ethnic group which is *less than four-fifths (4/5) (or eighty percent) of the rate for the group with the highest rate* will generally be regarded by the Federal enforcement agencies as evidence of adverse impact.
- **§1607.4(D) continued.** Greater differences in selection rate may not constitute adverse impact where the differences are based on small numbers and are not statistically significant, or where special recruiting or other programs cause the pool of minority or female candidates to be atypical. Smaller differences may constitute adverse impact where they are significant in both statistical and practical terms or where an employer's actions have discouraged applicants disproportionately.
- **§1607.5 — General standards for validity studies.** Even when a selection procedure produces adverse impact, it may still be justified by *validity* — demonstrated job-relatedness of the procedure. This is where the *evidence of validity* file lives.

### The engineering deliverable

The EEOC-shape deliverable is smaller than the LL144 shape but sits inside it:

| Field | Content |
|---|---|
| Definition of the "selection" event | What is the pass / decision the tool produces? Interview? Advance to next stage? Hire? |
| Applicant pool per race / sex / ethnicity category | Counts, ideally aligned to EEO-1 categorical vocabulary |
| Selection rate per category | Same as LL144 selection rate |
| Four-fifths ratio per category vs highest-rate category | Direct four-fifths computation |
| Statistical significance | Where sample sizes are small, a significance test (Fisher exact / chi-square) alongside the ratio |
| Practical significance narrative | Where numbers are on the boundary, the narrative on whether the difference is practically meaningful |
| Validity evidence (if adverse impact is present) | Citation to the validity file — job analysis, criterion / content / construct validity — held elsewhere |

The four-fifths computation is one line in Fairlearn; the validity file is not a mod-106 deliverable — it is prepared by the HR / assessment team and cited from the mod-106 report.

### When the four-fifths rule alone is insufficient

Two situations named explicitly in UGESP:

- **Small samples.** A per-cell count of, say, 20 can produce a four-fifths ratio outside 0.8 by chance. Report the significance test.
- **Large samples with small effects.** A ratio of 0.82 is nominally passing but with 100 000 applicants a chi-square test may flag a statistically significant disparity. The narrative must address this.

Both situations mean the four-fifths ratio is a *screening* number; the significance test and the practical-significance narrative complete the report.

## The credentialed shapes — ForHumanity and BABL AI

Two bodies whose report templates the professional-audit community has converged on. Neither is a legal requirement in most jurisdictions today; both are increasingly what a *credible* independent audit looks like.

### ForHumanity — Independent AI Auditor (IAA)

ForHumanity (`forhumanity.center`) publishes open certification schemes ("Certification Schemes") for AI systems including EU AI Act conformity, NYC LL144, and other frames. The Independent AI Auditor (IAA) credential is a person-level designation; ForHumanity's certification bodies audit *organisations* against the schemes.

Report shape characteristics:

- **Criteria-based.** The scheme is a numbered set of criteria; the audit report cites each criterion and the evidence that discharges it.
- **Third-party structured.** The auditor is independent (as LL144 requires); the report shape is written so a review board can rescind or uphold conclusions.
- **Evidence-linked.** Every claim in the report has a pointer to an artifact — a Fairlearn slice-report file, a signed evaluation manifest, a policy document.

The mod-106 output must be *citable in this shape*. That means: file path, content hash, tool version, run date, run operator all on the artifact. If the fairness slice report cannot be pointed at with a hash, it cannot be audited.

### BABL AI — Algorithm Bias Auditor

BABL AI (`babl.ai`) publishes the Algorithm Bias Auditor credential and a range of course materials for AI auditors, including specific coverage of NYC LL144, EU AI Act, Colorado AI Act, and adjacent shapes. The reporting template BABL AI trains toward mirrors the LL144 field list and adds narrative structure appropriate for other jurisdictions.

Report shape characteristics:

- **Regulation-aware.** The template varies with the regulation being audited against — the LL144 template differs from the Colorado AI Act template differs from the EU AI Act Article 10 / 14 slice.
- **Numeric core plus narrative.** The impact ratios and per-category counts are the load-bearing numbers; the narrative situates them in the applicable rule.
- **Reproducibility-emphasised.** The template calls for the sample size, the data source, the tool version, and the pre-registration of the metric choice.

BABL AI and ForHumanity are the two credentials most commonly named on LL144-shape audit reports at time of authoring. <!-- needs-research: cross-check current market prevalence of ForHumanity IAA and BABL AI Algorithm Bias Auditor credentials on filed LL144 audits. -->

### Adjacent shapes to know

- **Colorado AI Act (SB 24-205, effective February 2026)** — imposes obligations on developers and deployers of high-risk AI systems, including impact assessments and risk-management requirements. The LL144 shape maps well to the Colorado shape for employment-adjacent systems, with additional requirements around impact assessment and consumer notice.
- **EU AI Act Articles 10 (data governance) and 14 (human oversight)** — the fairness evaluation is a discharge for parts of Article 10 (representative and appropriate data) and the explainability slice is a discharge for parts of Article 14 (human oversight requires the reviewer can understand the model output).
- **Illinois Artificial Intelligence Video Interview Act** — narrower than LL144 (specific to AI in video interviews); named here so you route the specific case correctly.

## The L15 / L25 handoff at the report boundary

The critical role boundary in this module — and the one this whole track is organised around — is:

- **`ai-risk-engineer` (level 25, this role)** *builds* the report. Runs the evaluation, curates the numbers, produces the slice, ships the manifest with hashes and reproducibility.
- **`ai-governance-analyst` (level 15)** *reads and files* the report. Wraps it in the narrative the regulation asks for, files it with the DCWP or the internal governance portal, tracks the annual re-audit clock.
- **Independent third-party auditor (external — LL144 requires *independent*)** *audits* the report. Signs off (or does not). May be a ForHumanity IAA or BABL AI Algorithm Bias Auditor at an accredited firm.

The three responsibilities do not collapse. In particular:

- Level 25 does not sign the audit as an independent auditor — you are not independent.
- Level 15 does not build the fairness evaluation — the technical work is not in the analyst's brief.
- The external auditor does not re-run the evaluation from scratch — they audit the artifact you produced and the process that produced it.

This is why the mod-106 output must be *engineered for external audit consumption* — hashed manifests, versioned tool dependencies, pinned commit of the eval script, per-cell sample sizes, the metric-choice five-liner, the mitigation Pareto plot, and everything traceable back to the harm-model row.

## The report skeleton — LL144 shape

The delivered file for an LL144 audit is short. This skeleton is what the mod-106 output composes into.

```
Title:            Bias Audit — <Tool Name> — <Auditor> — <Date>
1. Auditor identity and independence attestation
2. Tool identity, version, deployment scope, dates of use covered
3. Data source: historical vs test; per-category counts
4. Metric definitions and choices (chapter 01 five-liner)
5. Selection rate per category — table
6. Scoring rate per category — table (if scoring tool)
7. Impact ratios per category vs highest-rate — table
8. Intersectional slice — table
9. Statistical significance notes — where relevant
10. Practical significance narrative
11. Validity evidence pointer (if adverse impact present)
12. Reproducibility manifest — tool versions, dataset hash, script commit hash, seed
13. Public summary (extract for the employer's website)
```

Sections 5, 6, 7, 8 come directly out of a Fairlearn `MetricFrame`; section 4 is the chapter-01 five-liner; section 12 is the manifest chapter 02 insisted on; section 13 is what the employer publishes.

## Anti-patterns

- **Auditor-employed-by-vendor.** The LL144 rule requires independence; a "third-party audit" done by a firm materially dependent on the vendor is not compliant. The engineer must not be the signer.
- **Metric changed after the numbers came in.** The metric-choice five-liner must be pinned *before* the run. Changing metric after seeing results is one of the most-cited audit failure modes.
- **Public summary that hides the numbers.** LL144 requires the impact ratios be public. A summary that pushes the ratios into a linked PDF that is then never uploaded is non-compliant in spirit.
- **Annual re-audit missed.** LL144 requires an audit "within one year." Track the clock as a compliance obligation; a stale audit is the same as no audit.
- **EEOC narrative without validity file when adverse impact exists.** If the four-fifths ratio is failed and the report does not point at a validity file, the report is incomplete. Route to HR / assessment for the validity work.

## Interfaces with adjacent modules

- **Mod-101 (frameworks)** — LL144, EEOC UGESP, Colorado AI Act, EU AI Act Articles 10 / 14 are all named in mod-101; this chapter is where they concretise into a report.
- **Mod-102 (harm modelling)** — the harm model's employment / lending / housing scenario rows are what this report answers to.
- **Mod-103 (register)** — the register row for "regulated adverse-impact violation" is discharged by this audit and monitored on a re-audit cadence.
- **Mod-109 (monitoring)** — production-drift dashboard for the same per-group impact ratios; drift outside band triggers a re-audit before the annual clock runs out.
- **Mod-112 (model card)** — the model card links to the audit report and its summary; the model card version is bumped on re-audit.

## Summary

- NYC Local Law 144 requires an independent bias audit of AEDTs, with a published field list — selection rate, scoring rate, impact ratio, per-category counts, intersectional slice — and a public summary.
- The EEOC four-fifths rule (UGESP §1607.4(D)) is the underlying US-employment shape; NYC LL144 is a specific implementation of it plus the intersectional and public-summary requirements.
- ForHumanity Independent AI Auditor and BABL AI Algorithm Bias Auditor are the credentialed shapes the professional-audit community has converged on.
- The mod-106 output must be engineered for external audit consumption: hashed manifests, pinned tool versions, pinned eval-script commit, pre-registered metric choice, per-cell counts.
- L15 / L25 / independent-auditor split: this role *builds* the report; the analyst *files* it; the independent auditor *audits* it. The three do not collapse.
- Adjacent shapes: Colorado AI Act (impact assessment shape), EU AI Act Articles 10 / 14, Illinois AI Video Interview Act.
- Anti-patterns: non-independent auditor; metric changed after the run; hidden public summary; missed annual re-audit; missing validity file when four-fifths fails.
- Exercise 06 drills the LL144 + EEOC report end-to-end on the running model.
