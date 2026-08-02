# Exercise 05 — Guardrail Effectiveness: FP / FN + Cost + Latency Report

**Estimated effort:** 3 hours

## Objective

Measure the **composed guardrail stack** shipped in exercises 02, 03, and 04 against the **bounded adversarial and legitimate sets hosted by the mod-104 harmful-payload workflow**. Produce the full **per-slot × per-adversarial-category × per-control-instance measurement table** (TP / FP / TN / FN, precision, recall, cost, latency, pinned versions). Convert it into the **one-page FP / FN report a product manager can read** in the chapter-05 skeleton shape. Back-feed each measurement row into the mod-103 register's residual and control-defeated columns. This exercise turns "we shipped the guardrail" into "we measured the guardrail and the residual number is defensible."

## Prerequisites

- Exercises 01–04 completed. The slot set, the rules, the classifiers with tuned thresholds, and the Presidio pipelines are all shipped and wired.
- Chapter 05 read in full. The measurement contract, the FP / FN report skeleton, and the register-update pattern are the reference shapes.
- Access to the **mod-104-hosted adversarial payload set** and the matching **legitimate payload set** — both at a specific pinned version. If mod-104's workflow does not yet host a legitimate set for some adversarial categories, either escalate back to mod-104 to add one, or bound the measurement to the categories where both sets exist and mark the rest as unmeasured (do not measure recall without precision on a claimed control).
- The exercise-01 slot specification and the exercise-02/03/04 control artefacts. The measurement table's `slot` and `control_instance` columns join back to these.
- The mod-103 register file for the running system with the inherent, residual, and control-defeated columns from mod-103 exercise 01.
- Instrumentation confirmed from exercise 02 (trace format including `request_id`, `slot`, `control_instance`, `config_version`, `latency_ms`), exercise 03 (classifier version + threshold), and exercise 04 (Presidio version + threshold per path).

## Problem statement

Run the composed stack over the mod-104 sets, produce the measurement rows in the chapter-05 contract shape, write the one-page report a product manager reads, and update the register.

### The measurement rows

Chapter 05 §"The measurement contract" defines the row shape. One row per `(slot, adversarial category, control instance)` tuple. Fields:

`slot`, `control_instance`, `category`, `TP`, `FP`, `TN`, `FN`, `precision`, `recall`, `mean_cost_per_call`, `p50_latency_ms`, `p95_latency_ms`, `n_adversarial`, `n_legitimate`, `set_version`, `control_version`, `run_id`.

Every field is load-bearing; `set_version` and `control_version` in particular make the row reproducible.

### The runner

Author a runner that:

- Loads the mod-104 adversarial set and the matching legitimate set at pinned versions. Categorises each payload by the adversarial category label mod-104 supplies.
- For each payload, plays it through the composed stack in the exact request path the running system uses (do not run a stripped-down "just call the classifier" harness — the measurement must include the middleware, rails, and end-to-end path so the numbers reflect production behaviour).
- Records the per-slot verdict for each payload (which slot fired, what was the verdict, was it correct against the payload's known label).
- Instruments cost and latency in the middleware, not on the classifier server side (chapter 05 §"Latency accounting" — wall-clock in the app).
- Emits the run manifest (set version, control versions, run id, git commit, run timestamp, hardware, model runtime versions) alongside the measurement table.

### The FP / FN report

Chapter 05 §"The FP / FN report a product manager reads" and §"Skeleton (per surface)" define the shape. One page per surface. Sections:

- Surface identifier and revision.
- Adversarial and legitimate set versions.
- Top three FPs: category, rate, example legitimate payload that gets blocked, estimated volume per day at production traffic.
- Top three FNs: category, rate, example adversarial payload that passes, harm class from the mod-102 harm model, downstream monitor status (mod-109).
- Coverage summary against OWASP LLM Top 10 and the mod-102 harm model — categories fully covered, partially covered, uncovered.
- Cost roll-up per surface per month.
- Latency roll-up at p50 and p95.
- Trend arrows vs. the previous run (or "first run" if this is the baseline).
- Decisions requested — explicit list.

The report is one page. Fifty rows of numbers is the engineer's view; the product manager gets the shaped one-pager.

### The register back-feed

Chapter 05 §"Per-slot residual-risk update into the register" defines the pattern. For each mod-103 row this measurement updates:

- Residual credit updated to the measured number where the credit was previously an assumption.
- Control-defeated column populated with the "N-th slot down" number, either derived from the "no-Nth-slot" run or from the measured recall of the remaining slots without the Nth on its own.
- Change-log entry naming the run id and the set version.

If any residual number moves above the risk appetite defined for that scenario, the row is flagged for escalation to mod-112.

## Requirements

Produce all nine items:

1. **Runner** — `risk/mod-108/<system-id>/measurement/runner.py`. Loads mod-104 sets at pinned versions; walks each payload through the composed stack in the actual request path; emits the measurement rows.
2. **Measurement table** — `risk/mod-108/<system-id>/measurement/measurements.csv` (or Parquet). One row per `(slot, adversarial category, control instance)` in the chapter-05 contract shape. All 17 fields populated. No unpinned versions.
3. **Cost accounting** — `risk/mod-108/<system-id>/measurement/cost.md`. Per-control mean cost per call at production pricing; per-slot per-day roll-up at production request volume; per-surface per-month roll-up in the same currency as the mod-103 register's financial-impact bands. Chapter 05 §"Cost accounting" is the reference.
4. **Latency accounting** — `risk/mod-108/<system-id>/measurement/latency.md`. p50 and p95 added latency per control per slot, measured in the middleware. Both serial and parallel numbers for slots where classifiers can run in parallel (chapter 05 §"Latency accounting"). Any slot over the exercise-01 latency budget flagged with a treatment plan.
5. **FP / FN report (product manager)** — `risk/mod-108/<system-id>/measurement/fp-fn-report.md`. One page in the chapter-05 skeleton shape. Every top-3 FP and top-3 FN row includes an example payload (redact if the payload contains real content that cannot leave the environment); each includes the volume estimate and the harm-class linkage.
6. **Coverage table update** — the exercise-01 coverage table refreshed with a "measured" cell colour or annotation per slot per OWASP LLM cell. A cell is "measured" only if the measurement table has at least one row for the corresponding slot and category and the recall on that row is above the treatment appetite; otherwise "claimed" or "open." Chapter 01 §"Slot × attack-class coverage table" is the "addressed, not claimed" discipline.
7. **Register updates** — the mod-103 register rows updated:
   - Residual column: measured number citing the run id and set version.
   - Control-defeated column: measured number for each "Nth slot down" case relevant to the row.
   - Change log entry: pointer to the run manifest.
   - Any row whose residual moves above appetite is flagged with a mod-112 escalation queue entry.
8. **Run manifest** — `risk/mod-108/<system-id>/measurement/manifest-<run-id>.json`. Pins the mod-104 set versions (adversarial and legitimate), every control version (Llama Guard checkpoint, ShieldGemma checkpoint + policy set, Presidio version, spaCy version, NeMo GR version, Colang version, Guardrails AI version, custom middleware git commit hash), the runtime (hardware / GPU model / instance type), the run timestamp, and the runner git commit.
9. **Anti-pattern probe report** — `risk/mod-108/<system-id>/measurement/anti-patterns.md`. Four sections, one per chapter-05 anti-pattern:
   - **Adversarial-only measurement.** Confirmed a legitimate set was used alongside every adversarial category; any category where mod-104 does not host a legitimate set is bounded and flagged.
   - **Unpinned versions.** Confirmed every row cites `set_version` and `control_version`; any unpinned control is filed as a fix.
   - **Cost / latency in the wrong place.** Confirmed the cost and latency numbers are from the middleware, not from the classifier server. Include one sanity check comparing middleware p95 against server-side p95 to make the network / dispatch overhead visible.
   - **Report hides the tradeoff.** Confirmed the FP / FN report names example payloads (not just percentages) and includes volume estimates.

Cross-cutting rules:

- **Consume mod-104, do not fork** (chapter 05 §"The bounded adversarial set"). If a category is missing, escalate to mod-104's workflow; do not create a private extension that produces numbers your peer eval track cannot reproduce.
- **Both sets, or the number is meaningless.** Recall without precision is a claim, not a measurement. If mod-104 has adversarial but no legitimate set for a category, either omit the category from the measurement or file the escalation to add a legitimate set.
- **Pin everything.** A measurement row that cannot be reproduced next week is not evidence.
- **The FP / FN report is one page.** If it grows to three pages, the discipline has broken — the product manager will not read three pages. Split by surface if a surface produces enough interesting content to fill its own page; do not aggregate.
- **Do not compute bootstrap CIs manually** (chapter 05 §"Cross-references" and chapter 06 §"Handoff to `ai-eval-engineer`"). Reference the eval track's calibration in the register row rather than duplicating.
- **Fail-closed and error-rate matter.** A classifier that errors at 5% inflates FN if the fail-closed shape is not the reject verdict. Report the classifier error rate alongside FP / FN so the reader knows the pipeline's operational health.

## Starter guidance

- **Pin the mod-104 set version before you touch the runner.** If the set changes mid-run, the numbers are not reproducible. Pin, run, hash the payload set, cite the hash in the manifest.
- **Run a smoke test on 10 adversarial + 10 legitimate first.** Confirm every field of the measurement row is populated end-to-end. If `p95_latency_ms` is `null` because the middleware timer was misplaced, that is a fix before you commit to a 500-payload run.
- **Category taxonomy alignment matters.** The mod-104 categories (`prompt injection — direct`, `prompt injection — indirect`, `disallowed content — self-harm`, `PII exfiltration`, etc.) must map cleanly to your slot's control categories. Author a small mapping table if the labels do not match directly, and cite the mapping in the runner.
- **Cost accounting has three lines** (chapter 05 §"Cost accounting"): compute (GPU-hour amortised or per-call pricing), operational (registry storage, log ingestion, on-call time), third-party licence. Do not report only the compute line; the register wants the fully loaded number.
- **Latency parallel vs. serial matters at the output slot.** Report both numbers if the output slot runs two classifiers; if only serial is reported, the option to parallelise is opaque to the operator.
- **The report's decisions-requested list is where the measurement earns its keep.** A report without explicit decisions is a debrief; a report with decisions is a control on the risk council's calendar. Chapter 05 §"Skeleton" is the reference — cite specific decisions (accept an FP rate, sign off on a spend increase, agree to a latency-budget breach).
- **Do not update the register in a way you cannot roll back.** Every register update names the run id; if a later re-run overturns a number, the change log tracks the movement.
- **Coordinate up front with `ai-eval-engineer` (L30).** Chapter 06 §"Handoff" defines the contract; the L30 eval engineer will re-run against the same sets and add statistical calibration. Share the run manifest and the measurement table; do not duplicate their bootstrap CI work.

## Acceptance criteria

The submission is accepted when:

- The runner exists and runs the composed stack against the mod-104 sets end-to-end.
- Every measurement row has all 17 fields populated with pinned versions.
- Both adversarial and legitimate sets were used; any category where a legitimate set was unavailable is bounded and flagged.
- Cost and latency are measured in the middleware, with per-surface per-month roll-ups.
- The FP / FN report is one page in the chapter-05 skeleton shape, with example payloads, volume estimates, harm-class linkage, and decisions requested.
- The coverage table is refreshed with measured / claimed / open annotations per cell.
- The mod-103 register is updated for every row this measurement touches, with change-log entries citing the run id; any row above appetite is flagged for mod-112 escalation.
- The run manifest pins every version referenced.
- The anti-pattern probe report resolves each chapter-05 anti-pattern.
- A one-paragraph interpretive note names which slot is load-bearing on the top-two harm classes, and what would happen to the residual number if the load-bearing slot went down.

## Stretch goals

- **A "slot-down" run.** Turn one slot off (e.g., disable the retrieval-filter classifier) and re-run the measurement. Compare to the full-stack numbers; the delta is the load-bearing contribution of that slot on that adversarial category. Chapter 05 §"Per-slot residual-risk update" describes how this populates the control-defeated column with a measured (not derived) number.
- **A per-language measurement.** If your system serves multiple languages, run the measurement per language. Report the language deltas. Classifiers routinely degrade on lower-resource languages; the language delta is often the largest cross-cutting weakness.
- **A cost / recall pareto plot.** Plot each control's cost per call against its recall on the categories that matter. Any point strictly dominated (higher cost, lower recall) is a candidate for removal. This visual is what chapter 06's build-vs-buy matrix will lean on.
- **A latency-parallelisation experiment.** For the output slot's dual-classifier composition, measure the serial p95 and the parallel-with-speculative-cancellation p95. Report the cost delta of the speculative shape. Chapter 05 §"Latency accounting" is the reference.
- **A monitor stub that ingests the run manifest.** For mod-109: write a small consumer that reads consecutive run manifests and emits a per-slot per-category recall-drift alarm when a run moves >2 standard deviations from the trailing average. The measurement contract is authored here; the metric plumbing lands in mod-109.
- **A three-run trend report.** Repeat the measurement three times with the same set version and the same controls; report the run-to-run variance. If variance is meaningful, the number the register cites needs a confidence band, not a point value (handoff to `ai-eval-engineer`).

## Deliverable format

A single markdown document `mod-108-ex05-measurement.md` that indexes the artefacts. The index document contains:

1. *System and run summary* — one paragraph naming the running system, the surface(s) measured, and the mod-104 set versions used.
2. *Composed stack summary* — one paragraph naming which controls fired in which slots, cross-referenced to exercises 02–04.
3. *Runner and manifest* — pointers to `runner.py` and the run manifest.
4. *Measurement table* — inline a small excerpt (the top-3 FPs and FNs' underlying rows) and a pointer to the full CSV.
5. *Cost and latency roll-ups* — blocks from `cost.md` and `latency.md`.
6. *FP / FN report* — the one-page product-manager document inlined.
7. *Coverage table refreshed* — the exercise-01 table updated with measured / claimed / open annotations.
8. *Register updates* — the list of mod-103 rows updated with the residual and control-defeated deltas.
9. *Interpretive note* — which slot is load-bearing on the top-two harm classes; what happens if the load-bearing slot goes down.
10. *Anti-pattern probe report* — the four chapter-05 sections.
11. *Open items* — any category where the legitimate set is missing; any slot over budget on cost or latency; any register row flagged for mod-112 escalation; the handoff pointer to `ai-eval-engineer` (L30) for statistical calibration.

## Submission

Attach the index markdown plus the nine artefacts. The measurement table and the run manifest are the load-bearing artefacts for the module's evidence trail; treat them as first-class deliverables to the risk council and to the peer eval track. If the FP / FN report contains real payload examples, redact them per the environment's data-handling policy but preserve the categories, rates, volume estimates, and harm-class linkage — the shape is what the reader needs.
