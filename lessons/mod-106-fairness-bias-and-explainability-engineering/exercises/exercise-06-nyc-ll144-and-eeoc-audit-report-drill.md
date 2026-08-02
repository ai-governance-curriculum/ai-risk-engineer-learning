# Exercise 06 — NYC LL144 + EEOC Adverse-Impact Audit Report Drill

**Estimated effort:** 2 hours

## Objective

Package the mod-106 outputs from exercises 01–05 into (a) a **New York City Local Law 144** shape bias-audit report ready for an *independent* auditor to sign, and (b) an **EEOC** adverse-impact / four-fifths evaluation with the validity-file pointer. The exercise is where technical numbers become a *fileable* artefact and where the L15 / L25 / independent-auditor split becomes concrete: this exercise builds the report; the analyst (L15) files it; an independent third party audits it. If your running model is not employment / lending / housing (say it is clinical or moderation), the exercise still applies — you produce the LL144-*shape* report against the analogous regulator and note the mapping.

## Prerequisites

- Chapter 06 read; chapters 01–05 available for cross-refs.
- Exercises 01–05 completed. Each of those exercises' outputs is cited in this report; if any is missing, do it first — an audit report without the pre-mitigation baseline (exercise 02) or without the operating-point justification (exercise 04) is incomplete on its face.
- Access to the primary sources: DCWP AEDT final rule (6 RCNY §5-300 et seq.); EEOC UGESP (29 CFR Part 1607) — cite by section, not by summary.
- If the running system is employment-adjacent, an **audit-window definition** (dates of use covered) and a **data-source declaration** (historical use data vs test data — LL144 accepts either, with different narrative implications). If not employment-adjacent, an equivalent regulator anchor (ECOA / Reg B for lending; FHA / HUD for housing; FDA GMLP for clinical; sectoral for others) and the field list it names.
- Named **independent auditor path** — pick either the ForHumanity IAA route or the BABL AI Algorithm Bias Auditor route (or the analogous credentialed route for the analogous regulator) and note it. You are not the auditor; you are engineering the artefact the auditor consumes.

## Problem statement

Produce two documents plus the underlying evidence bundle.

### 1. LL144-shape bias audit report

Author `risk/mod-106/<system-id>/audit/ll144-report.md` in the chapter-06 skeleton. Fill every numbered section from the mod-106 exercise outputs:

1. Auditor identity and independence attestation (placeholder for the external signer; note the L15 / L25 / independent-auditor split explicitly).
2. Tool identity, version, deployment scope, dates of use covered.
3. Data source — historical use data vs test data; per-category counts from exercise 02.
4. Metric definitions and choices — the exercise-01 five-liner verbatim.
5. Selection rate per category — the exercise-02 single-axis tables per sex and per race/ethnicity per EEO-1 vocabulary (or the analogous vocabulary).
6. Scoring rate per category (if the tool is score-based) — the same, on the score metric.
7. Impact ratios per category vs highest-rate — the four-fifths numbers from exercise 02.
8. Intersectional slice — sex × race/ethnicity table from exercise 02.
9. Statistical significance notes — Wilson CIs from exercise 02 and any Fisher / chi-square tests where small samples make significance testing load-bearing.
10. Practical significance narrative — the paragraph the analyst will wrap, drafted here so the numbers are unambiguous.
11. Validity evidence pointer — reference to the job-analysis / criterion / content / construct validity file the HR or assessment team holds (if adverse impact is present).
12. Reproducibility manifest — tool versions, dataset fingerprint, eval-script commit hash, seeds — combined from exercises 02 and 04.
13. Public summary extract — the numbers and dates the employer will publish under LL144's public-summary requirement. Author this section for public consumption; it must not require the reader to have seen the internal report.

### 2. EEOC adverse-impact evaluation

Author `risk/mod-106/<system-id>/audit/eeoc-adverse-impact.md` in the chapter-06 EEOC field set. Fill every field:

- Definition of the "selection" event — what the tool's pass / decision is.
- Applicant pool per race / sex / ethnicity category — counts.
- Selection rate per category — from exercise 02.
- Four-fifths ratio per category vs highest-rate — direct four-fifths computation.
- Statistical significance — Fisher / chi-square where samples matter.
- Practical significance narrative — the paragraph naming the operational impact.
- Validity evidence — pointer to the validity file if adverse impact is present; a statement that no adverse impact is present (with the numbers) if not.

If the running system is not employment-adjacent, produce the analogous document against the analogous anchor (ECOA / Reg B narrative for lending; FHA / HUD narrative for housing; FDA GMLP fairness slice for clinical) and cite chapter 06 §Adjacent shapes for the mapping.

### 3. Evidence bundle

Assemble `risk/mod-106/<system-id>/audit/evidence/` containing:

- The exercise-01 five-liner file.
- The exercise-02 slice-report and Fairlearn / AIF360 manifests.
- The exercise-03 individual + causal diagnostic report.
- The exercise-04 mitigation report + Pareto plot + operating-point justification.
- The exercise-05 explainability slice.
- The hash manifest for every artefact — filename, SHA-256, tool version that produced it. This is the artefact the external auditor cites when they sign; without hashes the sign-off is not defensible.

## Requirements

Produce all nine items below. Missing any is an audit finding — literally, because it is the audit report.

1. **LL144 report** — `risk/mod-106/<system-id>/audit/ll144-report.md` in the chapter-06 skeleton, every section filled from the exercise outputs, every metric number traceable to the evidence bundle.
2. **EEOC adverse-impact evaluation** — `risk/mod-106/<system-id>/audit/eeoc-adverse-impact.md` (or the analogous document if not employment-adjacent), with the four-fifths computation, significance, and validity pointer.
3. **Public summary extract** — `risk/mod-106/<system-id>/audit/public-summary.md`. The section that would go on the employer's website — LL144 requires public availability of the impact-ratio numbers and the audit date. Author for public consumption.
4. **Evidence bundle + hash manifest** — the directory of artefacts plus `manifest.json` listing each file with its SHA-256, producing tool version, and the exercise it came from.
5. **L15 / L25 / independent-auditor split memo** — `risk/mod-106/<system-id>/audit/roles.md`. One page naming: what L25 (this role) produced; what L15 (analyst) does next (files the DCWP submission, tracks the annual re-audit clock, wraps the public summary); what the independent auditor does (audits the report and the process; signs or rejects); why the three do not collapse and why LL144 requires independence.
6. **Compliance-clock stub** — `risk/mod-106/<system-id>/audit/clock.md`. LL144 requires an audit within one year of the last audit; ECOA / Reg B has adverse-action retention; EU AI Act Article 10 governance is continuous. Name the clocks that apply to this system, the next-due date, and the mod-109 monitor that watches for a drift-triggered early re-audit.
7. **Adjacent-jurisdiction cross-map** — `risk/mod-106/<system-id>/audit/adjacent.md`. For the same running system, name how this report reads under Colorado AI Act (SB 24-205), EU AI Act Articles 10 / 14, and any other jurisdiction the system might land in. This is the mod-110 vendor-review conversation input.
8. **Register update** — `risk/mod-106/<system-id>/audit/register-update.md`. The mod-103 rows that this audit report discharges, updates, or newly populates — the "regulated adverse-impact violation" row's residual, the "unexplainable adverse action" row's residual, and any new rows for "audit stale beyond compliance clock".
9. **Reviewer's checklist** — `risk/mod-106/<system-id>/audit/reviewer-checklist.md`. The 15-line checklist the external auditor will walk down (independence attestation present; five-liner pinned before run; selection rates per category present; four-fifths per category present; intersectional slice present; significance notes present; validity pointer present or "no adverse impact" stated; public summary present; reproducibility manifest complete; hashes in evidence bundle match; explainability slice present; individual / causal diagnostic present; mitigation Pareto and operating-point justification present; compliance-clock note present; adjacent-jurisdiction cross-map present). Reviewer uses this as the go / no-go.

Cross-cutting rules:

- **Independence is not a checkbox.** The independent auditor cannot be materially dependent on the vendor (LL144 anti-pattern in chapter 06). Even if the exercise submission does not have a real external auditor lined up, the report must be *engineered to be signable by one* — that means every claim has a hash-referenced artefact underneath.
- **The metric-choice five-liner is pinned before the audit.** Do not renegotiate the metric after seeing the numbers, do not add a "we now realise we should have headlined X" note. If new evidence warrants a metric change, that is a new audit cycle, not this cycle.
- **Public summary hides nothing.** LL144 requires the impact ratios be public. Do not push the numbers into a linked PDF that is then never uploaded — the anti-pattern in chapter 06.
- **If the four-fifths ratio fails and no validity file is cited, the report is incomplete.** Route to HR / assessment for the validity work; the mod-106 report cannot silently swallow the failure.
- **Do not sign as an independent auditor.** You are not independent; L25 is not L15's or the auditor's role. The signature block is left blank for the external signer with a note explaining why.
- **Hashes match.** Every artefact in the evidence bundle has a SHA-256 in the manifest that matches the file on disk. The auditor will spot-check; a hash mismatch is a rejection.

## Starter guidance

- **Author the LL144 report against the DCWP field list, not a template you found.** Read the rule text; when the wording is ambiguous or the rule has been amended, cite the version-of-record. When wording is unclear, mark with `<!-- needs-research: ... -->` rather than guessing.
- **Section 5–8 (selection / scoring / impact / intersectional) are copy-paste from exercise 02.** If you have to compute them again here, exercise 02 was incomplete — fix upstream, do not re-derive.
- **Section 4 (metric definitions) is the exercise-01 five-liner, verbatim.** No paraphrase, no polish. Consistency with the run header is what proves the metric was pinned before the run.
- **Section 10 (practical significance narrative) is where the auditor's eye lingers.** A ratio of 0.79 is nominally a failure; the narrative on whether the failure is operationally meaningful (many decisions affected, few decisions affected but each with high consequence, boundary case with wide CI) is the paragraph the auditor reads twice. Write it plainly; leave the legal interpretation to the analyst's wrapping.
- **Section 12 (reproducibility manifest) is what the auditor will spot-check.** If the manifest says Fairlearn 0.10.0 and your environment shipped 0.9.1, the audit fails on reproducibility grounds. Verify the manifest by re-running the eval on a clean environment before submitting.
- **Section 13 (public summary) is short.** LL144 does not require you to hide numbers, but it does not require narrative either. A table plus the audit date plus the auditor name is a full public summary.
- **The compliance clock is a real clock.** Set a calendar entry now for the next-due date. LL144's "within one year" is not "sometime next year" — an audit that expires on day 366 is stale.
- **Adjacent-jurisdiction cross-map avoids surprise.** If the same system is deployed in Colorado, the impact-assessment shape differs; if in the EU, Article 10 / 14 discharge differs. The mod-110 vendor conversation and the mod-112 model card both need the cross-map, so producing it here saves work later.
- **Reviewer's checklist is the artefact that answers "is this ready?"** Walk it yourself before submission; every checkbox should be green.

## Acceptance criteria

The submission is accepted when:

- The LL144 report exists with all 13 sections from the chapter-06 skeleton filled from the exercise outputs, and every number is traceable to a hash-referenced artefact in the evidence bundle.
- The EEOC adverse-impact evaluation (or its analogue for non-employment systems) exists with the four-fifths computation per category, significance notes, and a validity pointer or a "no adverse impact" statement.
- The public summary extract exists and is authored for public consumption (impact ratios present, audit date present, auditor name / independence attestation placeholder present).
- The evidence bundle exists and includes the exercise 01–05 artefacts with a hash manifest whose SHA-256s match the files on disk.
- The L15 / L25 / independent-auditor split memo names the three responsibilities and why they do not collapse.
- The compliance-clock stub names the LL144 (and any adjacent) clocks and the next-due date, and cross-refs the mod-109 drift-triggered early re-audit monitor.
- The adjacent-jurisdiction cross-map lists Colorado, EU, and any other applicable jurisdictions with a short mapping paragraph each.
- The register-update names the mod-103 row IDs this audit updates or newly populates.
- The reviewer's checklist is 15 items, walked and green.
- Nothing in the report requires the reader to be the engineer who ran the eval — the artefact is self-contained for an external auditor.

## Stretch goals

- Produce the **Colorado AI Act impact assessment** in full (rather than just the mapping paragraph) for the same running system. Colorado's shape adds impact-assessment structure and consumer-notice requirements not in LL144; walking through it end-to-end reveals which mod-106 outputs cover which fields.
- Produce the **EU AI Act Article 10 + Article 14 discharge statement** — one paragraph per Article naming which mod-106 artefact discharges which sub-clause. This is the shape the EU AI Act conformity assessment (mod-110) will consume.
- Draft the **vendor-facing bias-audit template** at `risk/mod-106/<system-id>/audit/vendor-template.md` — the shape a third-party model vendor's bias-audit report must land in to be consumable in mod-110. Third-party audits arriving in free-form are the top mod-110 rework driver; this template pre-empts it.
- Wire the **drift-triggered early-re-audit stub** for mod-109: a small script that reads the production-drift dashboard headline metrics and, if any drift beyond the threshold in the compliance-clock stub, opens a ticket for early re-audit rather than waiting for the annual clock. The monitor itself lives at mod-109; this is the intake artefact.
- Draft the **model-card update** (mod-112) that this audit cycle bumps — the model-card version bumps on re-audit; the fairness section links to this report; the compliance-clock cite is in the card.

## Deliverable format

A single markdown document `mod-106-ex06-audit.md` that indexes the artefacts, plus the LL144 report, EEOC adverse-impact evaluation, public summary, evidence bundle with manifest, roles memo, clock stub, adjacent map, register update, and reviewer's checklist. The index document contains:

1. *System, audit window, and data source* — one paragraph naming the tool, the deployment scope, the dates covered, the historical vs test data source.
2. *Metric-choice five-liner* — verbatim from exercise 01.
3. *Selection / scoring / impact-ratio tables* — the LL144 core numbers embedded.
4. *Intersectional table* — embedded.
5. *Verdict against four-fifths + practical significance paragraph* — the operational finding.
6. *Explainability + individual / causal diagnostic pointers* — pointers into the evidence bundle for the audit sub-artefacts.
7. *Mitigation pointer* — the exercise-04 operating point cited.
8. *Reproducibility manifest* — the combined manifest with hashes.
9. *Roles / clock / adjacent-jurisdiction pointers* — one line each pointing at the memo / clock / cross-map documents.
10. *Reviewer's checklist result* — the checklist walked and green.
11. *Register update summary* — mod-103 row IDs and their new residuals.
12. *Handoff to L15* — the paragraph that goes on the ticket to the governance analyst — "here is the audit report, here are the artefacts, here is what needs to be filed with DCWP, here is the next-due date, here is the independent auditor path."

## Submission

Attach the index markdown, the LL144 report, the EEOC evaluation, the public summary, the evidence bundle with its hash manifest, and the supporting documents (roles memo, clock stub, adjacent map, register update, reviewer's checklist) to the module submission channel. If the model / dataset is under NDA, redact model / dataset identifiers but preserve the selection / scoring / impact-ratio tables, the intersectional table, the four-fifths verdict, the practical-significance paragraph, and the reproducibility manifest — an audit report cannot be reviewed if the shape numbers are blanked out. The public summary extract is designed to be publishable as-is; do not redact it further.
