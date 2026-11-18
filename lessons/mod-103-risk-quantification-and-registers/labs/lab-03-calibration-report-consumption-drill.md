# Lab 03 — Calibration Report Consumption Drill (L25 ↔ L30 Seam)

**Duration:** 45 minutes · **Format:** guided practice, markdown authoring

## Objective

Internalise the chapter-07 contract at the level-25 ↔ level-30 seam: *consume* calibrated numbers with a provenance record and a documented credit choice; *refuse* uncalibrated numbers; *file* a calibration request when the number is missing. You will walk three fixed calibration-report shapes — one well-formed, one borderline, one disqualifying — and produce the correct register-side response for each: acceptance, acceptance under a pessimistic choice, or refusal with a calibration request. The lab drills the atomic skill chapter 07 asks for: *statistical literacy, not statistical methodology*.

## Prerequisites

- Chapter 07 read. Chapters 01 and 02 skimmed is helpful for the credit-on-residual pattern.
- A terminal where you can create a working directory and author markdown files.
- No external access required. You will *not* compute a CI or run a bootstrap in this lab — chapter 07 is explicit that computing them is the L30 role's work, and the "in-house bootstrap by a non-statistician" anti-pattern is called out by name.

## The fixed working row

Use `HR-PP-RTV-01` on PolicyPal carried forward from labs 01 and 02. You are scoring the row's **residual state**, which credits `CTL-PP-PROV-01` (retrieval provenance filter) among other controls. The number you are consuming calibrates `CTL-PP-PROV-01`'s false-negative rate — the probability that an un-provenanced chunk slips past the filter.

Chapter 07's convention for adverse likelihood: score against the **CI upper bound** (pessimistic for an adverse scenario) rather than the point estimate. Chapter 07's convention for credited guardrail effectiveness: score against the **CI lower bound** (pessimistic for a credit).

## The three fixed calibration reports

You will not read full reports — only the metadata shape chapter 07 names in the provenance record. Each block below is the entire "report summary" you have to go on; your job is to decide how (or whether) to consume it.

### Report A — well-formed

```yaml
report_id: eval-reports/2026-q2/prov-fnrate.md
report_version: v2
author_role: model-evaluation-engineer-L30
sample_size: 4200
observation_window: 2026-04-01..2026-06-30
effect_estimate: 0.03
ci_95: [0.02, 0.05]
calibration_notes: "judge κ vs. 3-human panel = 0.82 on stratified 500-item sample"
staleness_check: "recomputes on model version bump; last check 2026-06-28"
```

### Report B — borderline

```yaml
report_id: eval-reports/2026-q1/prov-fnrate.md
report_version: v1
author_role: vendor (CloudGuard Inc.) — internal reviewer not yet named
sample_size: 500
observation_window: 2026-01-15..2026-01-22
effect_estimate: 0.02
ci_95: [0.004, 0.09]
calibration_notes: "LLM-as-a-judge used; no judge-vs-human calibration study attached; vendor states κ not measured"
staleness_check: "none stated; report has no recompute trigger"
```

### Report C — disqualifying

```yaml
report_id: vendor-scorecard-2026.pdf
report_version: 1.0
author_role: vendor (unnamed) — marketing scorecard
sample_size: not stated
observation_window: not stated
effect_estimate: "97% accurate at blocking prompt injections"
ci_95: not stated
calibration_notes: not stated
staleness_check: not stated
```

## Steps

### Step 1 — Create your working directory and the template

```bash
mkdir -p mod103-lab03 && cd mod103-lab03
touch calibration-consumption-HR-PP-RTV-01.md
```

Open `calibration-consumption-HR-PP-RTV-01.md` and paste the following template.

```markdown
# HR-PP-RTV-01 — calibration consumption (lab 03)

System: PolicyPal (see lab brief)
Row: HR-PP-RTV-01 — indirect prompt injection via HR handbook content
Control calibrated: CTL-PP-PROV-01 — retrieval provenance filter (false-negative rate)
Chapter reference: mod-103 ch 07 (peer boundary with model-evaluation-engineer L30)

## Report A — decision
- Decision: accept / accept-with-reservation / refuse
- Reasoning (two to three sentences):
- If accepted, provenance block to attach to the row:
  <yaml block in the chapter-07 shape>
- Risk-engineer credit choice: point-estimate / ci-upper-bound / ci-lower-bound
- Credit choice reasoning (one sentence citing chapter-07 convention):

## Report B — decision
- Decision: accept / accept-with-reservation / refuse
- Reasoning:
- If accepted, provenance block:
- Risk-engineer credit choice:
- Credit choice reasoning:
- If refused, calibration-request block to open against L30:
  <yaml block in the chapter-07 shape>

## Report C — decision
- Decision: accept / accept-with-reservation / refuse
- Reasoning:
- If refused, calibration-request block:
- Treatment of the row in portfolio aggregation (chapter 05 quality gate):
  <uncalibrated row behaviour>
```

### Step 2 — Decide Report A

Walk the chapter-07 checklist:

- *Sample size present?* 4200. Yes.
- *Observation window present?* 2026-04-01..2026-06-30. Yes.
- *CI present alongside point estimate?* `[0.02, 0.05]`. Yes.
- *Judge-vs-human calibration?* κ = 0.82 against a 3-human panel on a stratified 500-item sample. Chapter 07 names "κ = 0.85" as the near-perfect threshold and "κ = 0.35" as the screening-signal-only threshold; 0.82 is above most "substantial / near-perfect" thresholds cited in the chapter.
- *Staleness check wired?* Recomputes on model version bump; last check within the quarter. Yes.
- *Author role named?* `model-evaluation-engineer-L30`. Yes.

Decision: **accept.**

Credit choice: the `CTL-PP-PROV-01` credit is a *guardrail effectiveness number* credited on residual. Chapter 07's convention for credited effects is the **CI lower bound** — the pessimistic choice for a credit. FN rate CI is `[0.02, 0.05]`; the FN rate's *upper* bound (0.05) is the pessimistic case for the *adverse story* (likelihood). For the FN-rate-as-adverse-likelihood story, score against 0.05 (CI upper bound). This is the chapter-07 "make the register hard to be surprised by" posture.

Fill the provenance block in the exact chapter-07 shape:

```yaml
credited_controls:
  - id: CTL-PP-PROV-01
    evidence_type: guardrail-effectiveness-report
    provenance:
      author_role: model-evaluation-engineer-L30
      report_ref: eval-reports/2026-q2/prov-fnrate.md
      report_version: v2
      sample_size: 4200
      observation_window: 2026-04-01..2026-06-30
      effect_estimate: 0.03
      ci_95: [0.02, 0.05]
      calibration_notes: "judge κ vs. 3-human panel = 0.82 on stratified 500-item sample"
      staleness_check: "recomputes on model version bump; last check 2026-06-28"
    risk_engineer_credit_choice: ci_upper_bound    # scored residual likelihood against 0.05
```

### Step 3 — Decide Report B

Walk the checklist again:

- *Sample size present?* 500. Small for the question but present. Chapter 07 names sample-size and power analysis as an L30 responsibility — you cannot evaluate the power yourself, but you can see the sample is small for a two-decimal-place FN-rate claim.
- *Observation window present?* Yes, but it is one week (2026-01-15..2026-01-22). Short window; not necessarily a problem, but worth noting on the provenance.
- *CI present?* `[0.004, 0.09]`. Present but very wide — the point estimate 0.02 has a CI upper bound 4.5× larger. This is a signal that the sample is under-powered for the effect size.
- *Judge-vs-human calibration?* κ not measured. Chapter 07 is explicit: "Judge scores anchoring without κ" is an anti-pattern. Without a κ, the judge score is a screening signal at best.
- *Staleness check wired?* No recompute trigger stated.
- *Author role named?* Vendor, no named internal reviewer.

Chapter 07's "What happens when the L30 role does not exist" posture: *consume vendor calibration under review* — a short read by someone in the org with statistical literacy — and name the reviewer on the row. For Report B, the missing κ is a harder block: the judge score cannot anchor residual-likelihood on its own. The wide CI compounds it.

Decision: **refuse to accept as-is.** Two acceptable subsidiary moves:

- *Refuse outright* — the row's credit for `CTL-PP-PROV-01` lapses pending a better report, which means the row's residual likelihood re-scores against inherent + the backstop controls only. File a calibration request.
- *Accept-with-reservation under pessimistic choice* — score the residual against the CI upper bound (0.09), which likely pushes residual likelihood from Unlikely to Possible, and name an internal reviewer on the row. File a calibration request in parallel.

Pick one. Either is defensible; the "accept-with-reservation" posture is the chapter-07 fallback when no L30 role is staffed. The "refuse outright" posture is cleaner when an L30 role exists and a better report is in flight.

For the lab deliverable, pick *refuse outright* and file the calibration request. The calibration request block:

```yaml
calibration_request:
  request_id: CAL-REQ-2026-Q3-001
  requesting_row_ids: [HR-PP-RTV-01]
  ask: |
    Judge-vs-human calibration study for the LLM-as-a-judge used in
    CTL-PP-PROV-01's false-negative-rate measurement. Minimum: κ against a
    multi-rater human panel on a stratified sample. Also: a re-run with
    larger sample (target: narrow CI to within ±0.02 of the point estimate)
    and a stated staleness trigger tied to model version bumps.
  needed_by: 2026-09-15
  blocking_treatment_plan: TP-PP-RTV-01
```

Note that chapter 07 calls out the *in-house bootstrap by a non-statistician* anti-pattern — you may not take this on and compute a tighter CI yourself. The ask goes to the L30 role.

### Step 4 — Decide Report C

Walk the checklist a third time:

- *Sample size?* Not stated.
- *Observation window?* Not stated.
- *CI?* Not stated.
- *Judge-vs-human calibration?* Not stated.
- *Staleness check?* Not stated.
- *Author role?* Unnamed vendor marketing.

This is the chapter-07 "point-estimate laundering" anti-pattern — a vendor scorecard with a single accuracy number and no provenance. The number could be off by an order of magnitude and the register could not see it.

Decision: **refuse.** No "accept-with-reservation" option here — the report is below the floor that a pessimistic-choice convention can rescue. The row's credit for `CTL-PP-PROV-01` cannot be anchored on this.

File the calibration request (first block). Then: per chapter 05, uncalibrated rows **drop out of portfolio roll-ups** — the register still carries the row, but the row carries a `staleness: uncalibrated` flag and the aggregation reports exclude it. For the lab, note this explicitly as the treatment of the row in portfolio aggregation.

Example calibration request for Report C:

```yaml
calibration_request:
  request_id: CAL-REQ-2026-Q3-002
  requesting_row_ids: [HR-PP-RTV-01]
  ask: |
    Full provenance for the "97% block rate" claim on CTL-PP-PROV-01:
    sample size, observation window, 95% CI, LLM-as-a-judge vs. human
    calibration (κ), staleness trigger, and a named author role. Until
    delivered, CTL-PP-PROV-01 is not credited on residual.
  needed_by: 2026-09-15
  blocking_treatment_plan: TP-PP-RTV-01
```

Portfolio-aggregation treatment:

```markdown
Row HR-PP-RTV-01 carries `staleness: uncalibrated` while the credit for
CTL-PP-PROV-01 is unsupported. Per chapter 05, the row is excluded from
portfolio roll-ups (above-appetite exception list, control-defeated
priority list, shared-component report) and appears instead on the
"uncalibrated rows excluded from roll-up" list. Program-quality signal:
rows falling onto this list are the first action, not better reporting.
```

### Step 5 — Author the calibration-acceptance record for Report A

Chapter 07: *acceptance is not tacit*. Even for the accepted Report A, write the acceptance down.

```yaml
calibration_acceptance:
  report_ref: eval-reports/2026-q2/prov-fnrate.md v2
  reviewer_role: ai-risk-engineer-L25
  reviewer_person: <your name / handle>
  accepted_for_rows: [HR-PP-RTV-01]
  credit_choice: ci_upper_bound     # for the adverse likelihood story on residual
  staleness_trigger: model_version_bump
  notes: "Judge κ 0.82 is above the near-perfect threshold cited in chapter 07; staleness wiring is in place."
```

This record is *what the auditor reads when asking whether the credited number was actually accepted*. Silent acceptance is a chapter-07 anti-pattern.

### Step 6 — Run the chapter-07 anti-pattern self-check

Confirm you did not commit any of the five anti-patterns from chapter 07:

- *Point-estimate laundering?* You did not file Report C's 97% as a residual-likelihood anchor. The row carries `staleness: uncalibrated` instead. ✓
- *Judge scores anchoring without κ?* You did not accept Report B as-is. The missing κ was named as the block. ✓
- *In-house bootstrap by a non-statistician?* You did not compute a CI yourself anywhere in this lab. ✓
- *Calibration staleness ignored?* Report A's staleness trigger was recorded on the provenance; Reports B and C carried missing-staleness as part of the refusal reasoning. ✓
- *Silently absorbing L30 scope?* The missing calibration for Reports B and C was filed as a calibration request, not run in-house. ✓

### Step 7 — Final checkpoint

Verify:

- Each of the three reports has a decision (accept / accept-with-reservation / refuse) with two-to-three sentences of reasoning.
- The accepted report (A) has a complete provenance block in the chapter-07 shape, including the explicit `risk_engineer_credit_choice`.
- The refused reports (B and C) each have a calibration request filed, with a `request_id`, `ask` specific to what is missing, a `needed_by`, and a `blocking_treatment_plan` pointer.
- The accepted report also has a calibration-acceptance record — acceptance is not tacit.
- Report C is additionally handled in the portfolio-aggregation report with a `staleness: uncalibrated` flag and an explicit note about exclusion from the roll-up.
- No in-house bootstrap, no κ computation, no sample-size calculation appears in your deliverable — all of that is the L30 role's work.

## Expected output

`mod103-lab03/calibration-consumption-HR-PP-RTV-01.md` — three decision blocks (one per report), one provenance block, one calibration-acceptance record, two calibration-request blocks, one portfolio-aggregation treatment note, roughly one and a half pages.

## Checkpoint questions (self-check)

1. Report A credits `CTL-PP-PROV-01` on residual at FN rate 0.03. Chapter 07 says the register engineer should choose explicitly between point-estimate, CI-lower-bound, and CI-upper-bound. Which choice *maximises defensibility against an auditor who argues the residual is too optimistic*, and which choice *maximises defensibility against an auditor who argues the residual is too pessimistic*? (CI upper bound (0.05) is the pessimistic adverse-story choice and is what chapter 07 recommends for an adverse-scenario likelihood. CI lower bound (0.02) is the pessimistic credited-effect choice — appropriate when the register is crediting the guardrail's *effectiveness* rather than reasoning about the adverse story. Both are chapter-07 patterns; the key is that the choice is *documented*.)
2. Report B's CI is `[0.004, 0.09]`. If the register scored residual likelihood against 0.004 (CI lower bound), what would land, and why is it the wrong call? (The row would score as very low residual likelihood — likely Unlikely or even Rare — and the aggregate register would under-state exposure to the attack. Chapter 07: for an adverse-scenario likelihood, the CI *upper* bound is the defensible pessimistic choice. 0.004 would be the credited-effect choice, not the adverse-story choice.)
3. If no L30 role is staffed and Report B is the only thing you have, what are the two chapter-07 fallbacks you may pick between? (Consume the vendor calibration under a named internal review with the pessimistic-choice convention baked in, *or* file the missing L30 role as a program-level coverage gap in the register itself and refuse to credit. Both are defensible; the second escalates the structural shortfall rather than silently working around it.)
4. Why does chapter 07 route *frontier / dangerous-capability calibration questions* to L40 (`agentic-safety-engineer`) rather than to L30 even when the L30 role is well staffed? (L30 helps measure well on scenarios your enterprise methodology can already reach. L40 owns the methodology for scenarios your enterprise methodology cannot reach — deceptive alignment, sandbagging, situation-aware behaviour, long-horizon autonomous evaluation. A better bootstrap CI does not surface those risks; the methodology itself is the problem, and that is the L40 boundary.)

## Why this lab matters

Exercise 05 asks you to stand up a register where every credited control has an evidence pointer that *resolves*, and chapter 07 says those pointers must carry the provenance record above. If the register ships without the provenance record, the L25 role has quietly absorbed either the L30 methodology work or the "point-estimate laundering" risk — both are the anti-patterns the chapter calls out by name. This lab drills the atomic consumption skill — three decisions, two refusals, one acceptance, one portfolio-exclusion note — so the register you build in exercise 05 does not quietly slip into one of those two failure modes. The seam at L25 ↔ L30 is what makes the register's numbers defensible; the lab is the smallest possible practice run of operating it.
