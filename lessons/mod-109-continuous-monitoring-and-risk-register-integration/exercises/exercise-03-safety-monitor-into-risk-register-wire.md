# Exercise 03 — Safety-Monitor, Near-Miss, and Cost / Latency into the Register

**Estimated effort:** 2 hours

## Objective

Fill in the safety-monitor, near-miss, and cost / latency rows of the signal contract. Turn the **mod-108 guardrail firing events** into signal contract rows with deterministic register-side effects. Wire the **near-miss counter** off the trace store, the **PII egress absolute-rule row** to the DPO on the routing, and the **cost / latency deviation** rows into the operating-envelope register rows. This exercise is the second half of the "signals into the register" work started in exercise 02; together, exercises 02 and 03 close the register's runtime evidence.

## Prerequisites

- Chapter 03 read in full. The seven signal families table, the two-sided deviation rule for guardrail firing, the absolute-rule threshold for PII, the near-miss labelling loop, and the peer-L30 boundary on the online-eval loop are the reference shapes.
- Exercises 01 and 02 completed. The signal contract file exists; drift and quality rows are shipped; the tripped-signal record schema and the register-effect wire are in place.
- The **mod-108 guardrail inventory** (from mod-108 exercises 01 through 05). Every guardrail carries a stable identifier, a version, a claim about the register rows it credits (mod-108 chapter 06), and an event schema (`evaluated`, `flagged`, `blocked`, `severity`). This exercise consumes those events; if any is missing, escalate back to mod-108 rather than inventing the schema here.
- A **trace store** with per-request records containing guardrail-fire tags. If your running system does not yet have a trace store, an in-memory jsonl file scoped to the exercise is acceptable — but the near-miss counter is only meaningful when it reads the actual trace store, so call the substitution out on the row.
- Cost telemetry and latency telemetry from the running system. Any format is fine (Prometheus, log-derived, warehouse table); the counter reads the source directly.
- The mod-102 harm-model persona / capability-surface list. It is what decides which guardrails need signal contract rows and, for the near-miss row, which fires clear the severity threshold that counts as a near-miss.

## Problem statement

Ship the safety-monitor family of tripped-signal detectors so each row emits a record and updates the register field deterministically. The exercise covers at minimum the five worked rows from chapter 03 (`SC-CS-INJ-01`, `SC-CS-PII-01`, `SC-CS-NEAR-01`, `SC-CS-COST-01`, `SC-CS-LAT-01`) plus any additional rows the running system's guardrail set requires per exercise 01's family coverage table.

### The five detector shapes

| Row family | Signal source | Detector behaviour |
|---|---|---|
| Guardrail firing rate (injection, jailbreak, other classifiers) | Mod-108 guardrail events | Rolling window rate vs. rolling baseline; two-sided (up and down); emit trip record on deviation > row's threshold for sustained window |
| PII egress absolute-rule | Mod-108 PII detector events + confirmed-positive labelling queue | On any confirmed positive: trip record with S1 severity, register row → `control-defeated`, incident candidate opened, DPO on routing |
| Near-miss cumulative | Trace store aggregation on guardrail fires above severity S | Daily count per register-row identifier; rolling weekly count vs. per-row threshold; emit trip record on breach |
| Cost anomaly | Cost telemetry (per 1k requests, per session) | Rolling median + sigma envelope; emit trip record when > 3σ sustained > row's window |
| Latency SLO burn | Latency telemetry (p95 or per SLO) | SLO burn-rate calculator; emit trip record when burn > row's threshold over row's window |

Two additional shapes to consider if the running system has them:

- **Out-of-scope tool call** on agentic systems — a detector on tool-call arguments that flags calls outside the allow-list (chapter 03 §"Jailbreak and prompt-injection detection specifically").
- **Guardrail down-rate** — the "rate down while overall volume steady" branch of chapter 03 §"Deviation is a two-sided signal." A drop is also a signal; wire it on the same row or as a sibling.

### The near-miss labelling loop

Chapter 03 §"The false-positive discipline" pins the discipline: near-miss counts are only meaningful if a labelling / calibration loop keeps false positives out of the count. The loop's depth (protocol, inter-rater reliability, judge calibration) is `ai-eval-engineer` (L30); this exercise's job is the risk-side hook — the tripped-signal record has a `false_positive` field, the labelling loop writes to it, and the near-miss counter reads it.

## Requirements

Produce all seven items:

1. **Guardrail-firing-rate detector** — `risk/mod-109/<system-id>/detectors/guardrail_firing_rate.py`. Reads the mod-108 event stream (or the trace store's guardrail-fire tags), computes rolling firing rate over the row's window, compares to the rolling baseline window, applies the row's deviation threshold, emits a tripped-signal record on trip. Two-sided by default (up and down); the row toggles one-sided if the guardrail's semantics justify it.
2. **PII egress detector wire** — `risk/mod-109/<system-id>/detectors/pii_egress_absolute.py`. Reads PII-classifier events and the confirmed-positive labelling queue; on any confirmed positive, emits an S1 tripped-signal record and opens an incident candidate (the incident-candidate queue's writer is exercise 05, but this detector emits the candidate payload).
3. **Near-miss counter** — `risk/mod-109/<system-id>/detectors/near_miss_counter.py`. Aggregates guardrail fires above severity S per register-row identifier per day; maintains rolling weekly counts; excludes fires whose `false_positive: true` flag is set by the calibration loop; emits a tripped-signal record on breach of the per-row weekly threshold.
4. **Cost anomaly detector** — `risk/mod-109/<system-id>/detectors/cost_anomaly.py`. Reads cost telemetry, computes rolling median + sigma envelope over the row's window, emits a tripped-signal record on > 3σ sustained > row's window.
5. **Latency SLO burn detector** — `risk/mod-109/<system-id>/detectors/latency_slo_burn.py`. Reads latency telemetry, applies the row's SLO burn calculator (Google SRE workbook shape or equivalent), emits a tripped-signal record on burn > row's threshold.
6. **Correlation memo** — `risk/mod-109/<system-id>/correlation-memo.md`. Chapter 03 §"Cost and latency anomalies" pins the discipline: cost and latency are correlated with the other families. Author a short memo listing, per row, which sibling rows share correlation signals and how the runbook (exercise 05) uses the correlation. Example: `SC-CS-COST-01` correlates with `SC-CS-INJ-01` because injection-driven expensive tool calls inflate cost; a joint trip is one incident, not two.
7. **Labelling-loop stub** — `risk/mod-109/<system-id>/labelling/README.md`. Names the labelling protocol at the risk-side hook depth: the sampling schedule off the tripped-signal records, the reviewer role, the field written back (`false_positive: true|false`), the propagation into the near-miss counter's read path, and the citation to `ai-eval-engineer` (L30) for the deep-protocol version. This is the seam chapter 03 §"The boundary to `ai-eval-engineer`" pins.

Cross-cutting rules:

- **Baseline is defended per row.** "Rolling 24-hour baseline" is the default; a longer baseline is fine if the guardrail's traffic pattern varies by day-of-week (weekly seasonality). The choice is on the row, not in the code.
- **Sustained-window matters.** A single 5-minute burst is not a trip; the row's sustained-window rule (chapter 03's "for at least 15 minutes" pattern) is what filters noise. The rule is on the row.
- **PII rows have absolute-rule severity.** Chapter 03 §"PII / sensitive-data leakage" pins the pattern. Do not fold the PII row into a shared-deviation row. Any confirmed positive is S1; the rate deviation is a *secondary* S2 row on the same register-row identifier.
- **Near-miss counts are per-register-row.** The counter is not a global counter; it is per `HR-*` row. Every guardrail that credits a row (mod-108 chapter 06) contributes its fires to that row's near-miss count. This is what makes the near-miss trend on the mod-112 digest actionable.
- **The correlation memo is a runbook input, not a detector output.** The detectors emit their trips independently. The correlation is what the on-call reads in the runbook to decide "one incident or two." Chapter 03 §"Cost and latency anomalies" and chapter 05 §"Runbooks" are the reference shapes.
- **Cite the L30 boundary on every row that depends on the eval loop.** Guardrail true-positive-rate calibration, near-miss label calibration, session-scoring quality — all sit at L30. The `deferral_boundary` column on each affected row names it.

## Starter guidance

- **Start with the guardrail that fires most.** The signal-to-noise pattern on the highest-volume guardrail is the hardest one to get right; if the detector works there, the others are easier. Read the guardrail's chapter-05 measurement (mod-108 exercise 05) for the baseline volume.
- **A "rate down while volume steady" trip is easy to miss.** Chapter 03 §"Deviation is a two-sided signal" is the reference. The detector's two-sided default catches guardrail-silencing bugs — for example, an upstream classifier failing open, or a rollout that regressed the classifier. Do not disable the down-rate branch without a written justification on the row.
- **Sustained windows are a rolling-time counter, not a batch job.** The 15-minute rule means "at some point in the last 15 minutes, the rate has been above the threshold continuously." Implement with a rolling counter over the row's evaluation window; do not run the detector every 15 minutes on the last 15 minutes' worth of data, or the first-minute-of-the-hour boundary makes the row noisy.
- **PII egress needs a labelling queue.** A PII-classifier fire without confirmation is a candidate positive; the labelling queue is what converts it to a confirmed positive. Author the queue writer in this exercise (the reviewer role, cadence, and SLA); the labelling protocol itself is the L30 handoff.
- **Near-miss severity threshold is on the guardrail.** Every guardrail has a per-fire severity (`low`, `medium`, `high`, `critical` or numeric). The near-miss counter only counts fires above the row's severity threshold; the threshold is on the row.
- **The cost detector wants a volume adjustment.** Cost per 1k requests, not cost per unit time. Cost per unit time inflates with traffic and produces spurious trips. The row's `signal_semantics` sentence names the denominator.
- **SLO burn calculators are cheap.** The Google SRE workbook chapter on burn-rate alerting is the reference for the calculator; a simple two-threshold burn-rate rule ("2x burn over 1h; 14x burn over 5m") is the default. If your SLO does not exist yet, name the SLO on the row as `blocked_on: SRE_SLO` rather than picking a threshold that no one owns.
- **The correlation memo is a two-page document.** Do not turn it into a per-row runbook. It is the input to exercise 05's runbook writing; keep it concise.
- **Test the S1 wire on a controlled fire.** For the PII row, feed a synthetic PII payload that the classifier catches; confirm the tripped-signal record fires with S1 severity, the incident candidate opens, and the DPO would be on the routing (exercise 05 wires the routing itself). If the wire only runs in production incidents, it is untested — chapter 04 §"The Article 73 wire that only runs in fire drills" is the parallel anti-pattern.

## Acceptance criteria

The submission is accepted when:

- Every safety-monitor row in the signal contract moves to `status: shipped` or `status: blocked`; no silent draft rows remain.
- Guardrail firing rate detectors are two-sided by default; one-sided rows carry a justification.
- The PII detector has an absolute-rule S1 branch with a labelling queue writer and a candidate-incident emitter; a controlled synthetic fire exercises the wire end-to-end.
- The near-miss counter reads per-register-row, respects the row's severity threshold, and excludes labelled false positives.
- Cost and latency detectors read from the real telemetry and use the row's window and threshold; no hard-coded values.
- The correlation memo lists at least the injection ↔ cost and injection ↔ latency correlations named in chapter 03.
- The labelling-loop stub names the risk-side hook, cites the L30 deep-protocol boundary, and lists the propagation path back into the near-miss counter's read.
- Every affected row's `deferral_boundary` column cites `ai-eval-engineer` (L30) where applicable and `ai-infra-mlops` (L25) where the pipeline is upstream.

## Stretch goals

- **Out-of-scope tool-call detector.** For agentic systems, ship the tool-call-argument detector as its own row and its own detector script. Chapter 03 §"Jailbreak and prompt-injection detection specifically" pins this as a separate row.
- **Guardrail-down control-availability signal.** When a guardrail's evaluation rate drops to zero (the guardrail is not running), emit a control-availability tripped-signal record on the register row that credits the guardrail. This closes chapter 06 §"Two shared surfaces" — pipeline-outage-as-register-event.
- **Correlation-aware alert de-duplication.** In the routing (exercise 05), an alert that shares a correlation window with another alert on the correlation memo's list is annotated with the sibling identifier. This turns "two incidents" into "one incident with two signals" for the on-call.
- **Baseline-recompute cadence.** Rolling baselines can be tricked by slow drift. Author a per-row recompute cadence (weekly or per-release), record the baseline snapshot on each tripped-signal record, and cross-check against the historical residual overlay. Chapter 06 §"Two shared surfaces" argues for treating baseline management as MLOps-side plumbing with a risk-side sign-off; wire the sign-off here.
- **Trace-store retention SLO.** The near-miss counter reads history; if the trace store's retention is shorter than the row's rolling-window, the counter is silently incomplete. Author a `retention SLO` block in the labelling-loop stub and file the mismatch back to the peer L25 as a pipeline requirement.
- **Cost / latency joint-anomaly rule.** A joint anomaly (cost up + latency up + injection up) is one incident. Author a simple rule that emits a single joint-anomaly tripped-signal record when all three fire in the same window; the record cites the three underlying identifiers. This makes the correlation memo executable rather than descriptive.

## Deliverable format

A single markdown document `mod-109-ex03-safety-signals-into-register.md` that indexes the artifacts. The index contains:

1. *Contract rows shipped* — table of safety-monitor, near-miss, cost, and latency rows moved from `draft` to `shipped`.
2. *Detectors* — pointers to the five detector modules with the parameters they consume per row.
3. *PII synthetic-fire drill* — the record produced, the candidate incident payload, the routing (as it would fire in exercise 05).
4. *Near-miss counter output excerpt* — for one representative `HR-*` row, the current weekly count and the trailing history.
5. *Correlation memo* — inlined verbatim.
6. *Labelling-loop stub* — inlined verbatim, with the L30 handoff clause quoted.
7. *Register-effect trace* — for one guardrail-firing trip and one PII trip, the before/after register field with the change-log entry.
8. *Anti-pattern probe* — one sentence per chapter-03 failure mode confirming resolution ("firing rate is a two-sided deviation"; "PII is absolute-rule S1 with DPO routing"; "near-miss excludes labelled false positives"; "cost/latency correlated with sibling rows").
9. *Open items* — rows still `blocked`, MLOps or L30 handoffs pending, trace-store retention gaps.

## Submission

Attach the index markdown plus the seven artifacts. Together with exercise 02, this closes the "signals into the register" work for the module. Exercise 04 will read the register + the tripped-signal records to generate the Article 72 report; exercise 05 will read the same records to build the routing. Everything downstream depends on the rows shipped here being real, tested, and cited.
