# Exercise 02 — Drift and Quality Monitoring with whylogs and Evidently

**Estimated effort:** 3 hours

## Objective

Fill in the drift-and-quality rows of the signal contract drafted in exercise 01. Wire **whylogs profiles**, **Evidently reports and test suites**, and (where the row calls for it) **Alibi Detect** detectors so each row emits a tripped-signal record that updates a mod-103 register field deterministically. Defend the metric, cadence, and threshold choices per row against the drift-metric literature.

## Prerequisites

- Chapter 02 read in full. The drift-type × metric matrix, the golden-set-aging discipline, the per-slice-monitoring pattern, the working-toolset rule of thumb, and the peer-boundary text are the reference shapes.
- Exercise 01 completed. The signal contract file exists and has draft rows for drift and quality regression. This exercise moves each of those rows from `status: draft` to `status: shipped` (or `status: blocked` with a pipeline dependency).
- The mod-103 register file with the drift-tagged and quality-tagged rows. Every signal contract row this exercise ships names a `register_row_id` that must exist there.
- A working environment with `whylogs`, `evidently`, and `alibi-detect` installed. Any environment (Colab, a workstation, or the peer L25 pipeline sandbox) is fine — the point is to run the code, not to prescribe the platform.
- A dataset representative of the running system's inputs, at two windows: a **reference window** (training snapshot or a stable production window) and a **current window** (a recent production window). If the production window is not available yet, a synthetic-shift generator on the reference window is acceptable for the exercise; call the synthetic shift out in the evidence file.
- A **golden set** for quality regression (500+ examples covering aggregate quality and at least one protected slice). If the running system does not have a golden set, an interim seed is acceptable — but the interim seed must be checked in with a version and a `needs-expansion` note per chapter 02's "the golden set has to age well" discipline.

## Problem statement

Ship a working detector, per row, that:

- Reads the correct reference and current windows on the correct cadence.
- Computes the metric named in the row's `signal_semantics` with the parameters recorded (kernel, bandwidth, bin count, alpha, test-set size, per-slice grouping).
- Emits a tripped-signal record when the threshold is met, with the row's `contract_id`, the raw metric value, the reference-window and current-window identifiers, and the payload hash needed for evidence.
- Files the tripped-signal record at the row's `evidence_pointer` location.
- Updates the register-side field named in the row's `register_effect` column deterministically.

The exercise covers at minimum the three worked rows from chapter 02 (`SC-CS-DRIFT-01`, `SC-CS-QUAL-01`, `SC-CS-EMB-01`) *plus* any per-slice row the running system requires per chapter 02 §"Per-slice monitoring."

### The three detector shapes

| Row family | Working tool | What the code does |
|---|---|---|
| Per-feature covariate drift (PSI on top-K, or KS at chosen alpha) | `whylogs` for the reference + current profiles, plus a metric comparison over the pair | Log reference profile once; log current profile per batch; compare with the drift metric; emit trip record if threshold met |
| Batch quality regression against a golden set | `evidently` reports + test suites | Run the report per cadence; the test-suite pass/fail is the threshold; emit trip record on test failure |
| Multivariate / embedding drift on prompts or outputs | `alibi-detect` (MMD with RBF kernel, median-heuristic bandwidth, permutation p-value) | Fit the detector on a reference embedding sample; run per-window on a fresh embedding sample; emit trip record if p-value < row's alpha for N consecutive windows |

Per-slice rows are the same shapes with a group-by on the slice column.

### The register-effect wire

Every tripped-signal record has to move the register-side field named in the row's `register_effect` column. In a repository-stored register (mod-103's spreadsheet-fallback shape), the wire is a script that reads the tripped-signal record and rewrites the field with a change-log entry. In a GRC-for-AI platform, the wire is the platform's API call. Either way, chapter 01 §"What a signal contract is" pins the property: the effect is deterministic — no manual re-interpretation between detector and register.

## Requirements

Produce all seven items:

1. **whylogs detector** — `risk/mod-109/<system-id>/detectors/whylogs_drift.py`. Logs the reference profile, logs the current-window profile, computes the drift metric (PSI on top-K by default, KS as an alternate for a specific row), applies the row's threshold, emits the tripped-signal record. Parameters (feature list, bin count, cadence, threshold) are read from the signal contract row, not hard-coded.
2. **Evidently quality regression** — `risk/mod-109/<system-id>/detectors/evidently_quality.py`. Runs the report on the golden set on the row's cadence, applies the test-suite pass/fail as the threshold, emits the tripped-signal record. Golden-set version is pinned per run.
3. **Alibi Detect embedding drift detector** — `risk/mod-109/<system-id>/detectors/alibi_mmd_embeddings.py`. Fits MMD (RBF kernel, median-heuristic bandwidth) on a reference embedding sample, runs per-window on a fresh embedding sample, applies the row's alpha and consecutive-window rule, emits the tripped-signal record.
4. **Per-slice detector** — `risk/mod-109/<system-id>/detectors/per_slice_drift.py`. Wraps the whylogs or Evidently detector with a group-by on the slice column. Emits per-slice tripped-signal records that route to the fairness-tagged row (mod-106 seam), with the fairness liaison on the routing.
5. **Tripped-signal record schema and store** — `risk/mod-109/<system-id>/records/` (directory) and `risk/mod-109/<system-id>/records/schema.md` (schema doc). Records are JSON with `contract_id`, `run_id`, `timestamp`, `metric_name`, `raw_value`, `threshold`, `reference_window_id`, `current_window_id`, `slice` (if per-slice), `sample_ids`, `payload_hash`, `severity`, and `false_positive` (defaults `null`, updatable by the calibration loop).
6. **Register-effect wire** — `risk/mod-109/<system-id>/register_wire/` (module) and a top-level `apply_effect.py` that reads a tripped-signal record and applies the deterministic effect to the mod-103 register row with a change-log entry. Idempotent: re-running with the same `run_id` does not double-apply.
7. **Threshold defence memo** — `risk/mod-109/<system-id>/threshold-defence.md`. Per contract row, three to five sentences citing (a) the metric family from chapter 02, (b) the numeric threshold, and (c) the origin of the number (Siddiqi PSI band + validated against reference-window study; KS alpha per Massey; MMD alpha + kernel per Gretton; SLO burn per SLO doc; harm-model row for absolute-rule thresholds). This is what the auditor reads when asking *"why 0.2 and not 0.1."*

Cross-cutting rules:

- **Metric parameters are on the row, not in the code.** The detector reads parameters from the contract row (or a config file bound to the row's identifier). Changing a threshold is a row diff, not a code diff. This is the chapter 01 change-control discipline.
- **The reference window is versioned.** Every whylogs reference profile and every embedding reference sample is hashed and the hash is on the record. Regenerating the reference is a contract change (chapter 02 §"The golden set has to age well" and mod-103 chapter 06 event list).
- **Per-slice discipline is enforced by row shape.** If the aggregate row has a per-slice sibling, both run; if only the aggregate runs, the sibling is `status: blocked` and the reason is written down.
- **Boundary citation is per-run.** Each tripped-signal record includes a `signal_source_owner` field that reflects the row's `signal_source_owner`. When the peer L25 pipeline emits the profile / detector output directly, the record cites the pipeline identifier and version.
- **Do not fold quality regression signals into a single "quality" metric.** Chapter 02 §"Quality regression" pins this. Faithfulness, coverage, safe-refusal, and per-slice quality are separate signals with separate contract rows. The Evidently detector emits per-metric records, not a bundled record.

## Starter guidance

- **Whylogs profiles are cheap; log more than you need up front.** The library is designed so per-batch profiles are inexpensive to store; do not pre-filter to a top-20 features until you have a reference window worth of profiles to inspect. Chapter 02 §"WhyLabs / whylogs" is the reference shape.
- **Evidently test suites are the threshold vehicle.** A report on its own is a chart, not a monitor. The test suite is what turns a report into a pass/fail signal. Every quality regression row calls a test suite; the suite's pass/fail is what emits the trip record.
- **Alibi Detect MMD requires a fixed test-set size.** The permutation test's p-value distribution is only meaningful if the sample size is fixed. Pick the size once, record it on the row, and sub-sample the current window to that size on every run.
- **Median-heuristic bandwidth is the reasonable default for MMD RBF.** Cite Gretton et al. 2012 as the source; if you deviate, defend it on the row.
- **Golden-set aging is not a stretch goal.** Chapter 02 §"Quality regression" pins the aging discipline. Bake it into the exercise: the golden set has a version; the version has a "date frozen"; a `stale` flag turns on when the freeze date crosses a per-row threshold; the freeze date itself is on the tripped-signal record.
- **Faithfulness needs the retrieved context.** For a RAG system, the golden-set entry contains the reference answer *and* the retrieval that produced it; the faithfulness metric measures the answer against that context, not against the model's memory. This is chapter 02 §"The metric matches the harm."
- **Per-slice drift on top of PSI is a group-by, not a rewrite.** whylogs supports segmenting a profile. Use the segmentation feature to produce per-slice profiles from the same batch; do not run the detector twice with different filters.
- **The register-effect wire is where the tests live.** Unit-test the `apply_effect` module against the tripped-signal-record schema; a malformed record should refuse to update the register, not fail silently. The idempotency test is not optional — re-applying a record is a common resume-from-crash path.
- **The detector output is the signal, not the dashboard.** If Evidently produces an HTML report, keep it as an artifact under the `evidence_pointer`; the tripped-signal record is what routes and updates. Do not build a Grafana panel and call the row shipped.

## Acceptance criteria

The submission is accepted when:

- Every drift row and every quality regression row in the signal contract moves from `status: draft` to `status: shipped` or `status: blocked` (with a named dependency) — no silent draft rows remain.
- Every detector reads its parameters from the contract row (or a per-row config); no thresholds are hard-coded.
- Every tripped-signal record has the schema fields populated and files at the row's `evidence_pointer`.
- Applying a tripped-signal record deterministically updates the mod-103 register row's residual state and treatment-flag columns, with a change-log entry citing the `run_id` and the reference / current window versions.
- Re-applying a tripped-signal record is idempotent.
- The threshold-defence memo cites the drift-metric literature for each row (KS via Massey, PSI via Siddiqi, MMD via Gretton, JS via Lin) and records the harm-model or SLO origin for absolute-rule and SLO thresholds.
- Every per-slice row exists; if the running system has no protected slices, the justification is on the front-matter of the contract.
- The peer-L25 boundary is cited on every row where the detector pipeline is owned upstream.

## Stretch goals

- **Reference-window regeneration runbook.** Chapter 02 §"The golden set has to age well" applies to the reference window too. Author a runbook (`risk/mod-109/<system-id>/runbooks/refresh-reference-window.md`) that says when to regenerate (mod-103 chapter 06 event list), how to hash the new profile, and how to open the contract diff for the new reference identifier.
- **Backfill the last 30 days.** Run the whylogs and Evidently detectors over the last 30 days of production data (if available). Emit the tripped-signal records into a `backfill` sub-directory. Confirm that (a) the register-effect wire is idempotent and (b) the resulting residual-state history matches the register's operational log. Discrepancies are bugs — file them.
- **Per-slice fairness escalation drill.** Deliberately introduce a per-slice quality regression on the golden set (either by re-labelling a subset or by fixture); confirm the detector fires, the fairness liaison is on the routing per chapter 02 §"Per-slice monitoring," and the mod-106 fairness-tagged register row picks up the effect. Chapter 03 §"PII / sensitive-data leakage" has the DPO-on-routing parallel.
- **Detector-monitor SLO.** Wire a simple detector-uptime signal (last successful run timestamp per contract row). When the detector itself has not run in > 2× cadence, emit a control-availability tripped-signal record against the row's control. This is the chapter 06 §"Two shared surfaces" property — pipeline SLO breaches are register-side events.
- **Handoff dry run with `ai-infra-mlops`.** Package the three detector scripts as a contract to hand the pipeline owner (peer L25). Include the parameters, the reference / current window schema, the tripped-signal record schema, and the SLO the pipeline needs to sustain. The dry run tells you whether the boundary text on the contract row is truthful; if the pipeline owner cannot pick it up as-is, the boundary is muddled and you fix it before shipping.
- **LangKit for LLM signal profiling.** For LLM-specific rows (prompt-embedding drift, response-embedding drift, response-length distribution, refusal-rate drift), use WhyLabs' LangKit atop whylogs. Cite LangKit on the row; the reference is in `resources.md`.

## Deliverable format

A single markdown document `mod-109-ex02-drift-quality-detectors.md` that indexes the artifacts. The index contains:

1. *Contract rows shipped* — a table of the drift and quality rows moved from `draft` to `shipped`, with a pointer to the row-level diffs in the contract file.
2. *Detectors* — pointers to the whylogs, Evidently, Alibi Detect, and per-slice detector modules with the parameters they consume per row.
3. *Threshold defence memo* — quoted verbatim, at least the S1 / S2 rows.
4. *Tripped-signal records excerpt* — three example records (one per detector shape) inlined verbatim.
5. *Register-effect trace* — for one representative row, the before/after of the register field with the change-log entry.
6. *Per-slice coverage* — per protected slice, whether a per-slice row is shipped, blocked, or n/a.
7. *Reference / golden-set versions* — hashes and freeze dates.
8. *Anti-pattern probe* — one sentence per chapter-02 failure mode confirming resolution ("test suite drives the trip"; "reference window versioned"; "per-slice runs separately"; "boundary cited per row").
9. *Open items* — rows still `blocked`, MLOps handoffs pending, LangKit adoption decisions.

## Submission

Attach the index markdown plus the seven artifacts. The detectors and the register-effect wire are load-bearing; the threshold-defence memo is what earns the row the sign-off routing from chapter 01. Every tripped-signal record produced in the exercise is evidence that survives into exercise 04's Article 72 report.
