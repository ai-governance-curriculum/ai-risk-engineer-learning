# Safety-Monitor Signal, Near-Miss Data, and Cost / Latency Anomalies

Drift and quality (chapter 02) tell you the model has *moved*. Safety-monitor signal tells you the model was *attacked*, or *misused*, or *leaked something it should not have*. Near-miss data tells you a control caught something that would otherwise have been an incident. Cost and latency anomalies tell you the operating envelope shifted in a way that produces its own harm. This chapter wires each of these signal families into the register.

## Motivation

Two failure modes recur in the safety-signal family:

- **Guardrail firing is treated as a service metric, not a risk signal.** The team knows the injection detector fired 400 times yesterday. Nobody knows what happened after — whether the register learned, whether the trend is stable, whether the fires were true positives, whether any near-miss became an incident.
- **Near-miss data lives in the trace store and nowhere else.** A guardrail fires, the request is refused, the user retries in a way that succeeds, and the near-miss is never counted. The rate of near-misses is the leading indicator for the class of harm; if you do not count them, you cannot see the trend.

The engineering fix is the same as chapter 02's: name the signal, name the threshold, name the register field, name the routing. The signal families in this chapter are the ones that turn mod-108 (guardrails) and the trace-store contents into a live risk signal.

## The signal families this chapter covers

| Family | What it measures | Typical source | Where the value falls |
|---|---|---|---|
| **Guardrail firing rate** | Rate at which any guardrail refuses / rewrites / redacts output | mod-108 guardrails (input filter, output validator, safety classifier, policy engine) | Rising rate = new attack shape or drift into the guardrail; falling rate = guardrail weakened or attack shifted |
| **Prompt-injection detection** | Rate and confidence of prompt-injection classifier firings | mod-108 input-side classifier (or a downstream detector on tool-call arguments) | Sudden spikes = campaign; sustained rise = attacker learning the surface |
| **Jailbreak detection** | Rate at which known-jailbreak patterns are matched, or a jailbreak-classifier fires | mod-108 classifier + fixture-based pattern match | Rising rate = new jailbreak in the wild; watch for classifier drift |
| **PII / sensitive-data leakage detection** | Rate at which the PII-egress detector fires on outbound content | mod-108 output validator + PII classifier | Any confirmed egress is S1; rate signals the *pressure* on the control |
| **Near-miss data** | Count of guardrail hits above severity S that would have been an incident absent the guardrail | Trace store aggregation | The leading indicator for the harm the guardrail is meant to catch |
| **Cost anomaly** | Deviation of cost per request / per session vs. rolling baseline | Cost / usage telemetry | Signals model-thrash, retry storms, prompt bloat, or attacker-driven cost inflation |
| **Latency anomaly** | Deviation of latency percentiles vs. baseline, SLO burn rate | Application / infrastructure telemetry | Latency spikes correlate with quality regressions and with abuse |

Each family has its own signal contract rows, its own threshold logic, and its own escalation path.

## Guardrail firing rate as a risk signal

Every guardrail from mod-108 emits at least three raw counters per unit time: *evaluated*, *flagged*, *blocked*. From those you derive:

- **Firing rate** — `flagged / evaluated`, over a chosen window.
- **Block rate** — `blocked / evaluated`, over a chosen window.
- **True-positive rate** — where a downstream labelling process (spot review, offline judge, user report) confirms the flag was justified. This one is not always available in real time; it feeds the near-miss and calibration loops rather than the streaming threshold.

The signal contract usually monitors *deviation* against a baseline rather than a fixed rate. A guardrail that fires on 2% of requests baseline is not an alarm at 2%; it is an alarm at 6% for a sustained window. The threshold column reflects that:

> Threshold: rolling 1-hour firing rate > 3x rolling 24-hour baseline for at least 15 minutes.

The `signal_semantics` sentence has to name the *guardrail* explicitly. "Guardrail firing rate up" is not a signal; "prompt-injection input classifier firing rate up 3x for 15 minutes" is.

### Deviation is a two-sided signal

A *drop* in firing rate is also a signal — the guardrail may have been silenced, its dependency may have failed, or the attacker may have learned to evade it. Wire deviation in both directions:

| Direction | Interpretation | Contract routing |
|---|---|---|
| Rate up | New attack shape or user population; false-positive drift | ML on-call + security review of pattern |
| Rate down while overall volume steady | Guardrail failure or evasion | ML on-call + security + circuit-breaker check |
| Both directions on a control that itself credits residual on a register row | Control availability lapse | Register-side flag on `control_status`; residual state recomputed |

## Jailbreak and prompt-injection detection specifically

Jailbreak and prompt-injection detection sit at the border between drift signals and outright attack signals. Two things make them worth their own row:

1. **Their true-positive rate is inspectable.** Every fire produces a request payload you can sample and label. A calibration loop (weekly review of a sampled slice) keeps the detector from silently drifting into either noise or blindness.
2. **They feed the mod-108 fixture library.** Every confirmed injection is a candidate red-team fixture (mod-104) and a candidate golden-set entry (chapter 02). The wire is: detector fire -> sample review -> confirmed positive -> fixture -> regression check on next release.

For an agentic system with tool calls (mod-102 chapter 01's capability-surface list), an additional detector runs on the *tool-call arguments* — an injection may pass the input filter but still land on an out-of-scope endpoint. Wire that as a separate detector with its own signal contract row; do not fold it into the input-side rate.

## PII / sensitive-data leakage — the absolute-rule threshold

PII / sensitive-data egress is an *absolute-rule* threshold: any confirmed egress is S1, regardless of rate. The signal contract row looks different from the deviation rows above.

- `signal_semantics`: rate of PII-classifier firings on outbound content, with per-firing samples routed to a labelling queue; a confirmed positive is any firing where the sample is verified as real PII.
- `threshold`: any confirmed positive; secondary alert on firing-rate deviation > baseline (as with other guardrails).
- `register_effect`: on any confirmed positive, the register row is flagged, an incident candidate is opened (mod-111), and the residual state moves to `control-defeated` until the pathway is remediated.
- `alert_routing`: on-call + security + DPO (or the equivalent privacy-officer role) + governance-council digest. This is the row where the routing must include the privacy owner by name.

The engineering discipline here matches what a data-protection auditor asks about: *how did you know the leak happened; how quickly did you know; who did you tell; what did you do*. Every column of the signal contract answers one part of that.

## Near-miss data — counting the harm that did not happen

A near-miss is a request that a guardrail refused / rewrote / redacted, where absent the guardrail the request would have produced a harm on the harm model (mod-102). Near-miss data is one of the two most-cited leading indicators for high-risk system deterioration (the other being the drift family from chapter 02).

Wiring near-miss data:

- **Emit at the guardrail level.** Every guardrail fire that meets a severity threshold is emitted to a near-miss counter with the risk-register row identifier it maps to. The guardrail *knows* which row it credits (mod-108 chapter 06); the near-miss carries the same identifier.
- **Aggregate at the trace-store level.** Sum per-day and per-week; compute rolling counts per row.
- **Feed the register on cadence.** The row's `near_miss_last_7d` field is updated daily; when the count breaches a per-row threshold, a treatment flag opens.
- **Feed the governance-council digest.** The weekly digest (mod-112) includes the top-K near-miss counts and their trend.

Near-miss data is also what turns EU AI Act Article 72 post-market surveillance from a paper artifact into a live record. Chapter 04 uses it directly.

### The false-positive discipline

Near-miss counts are only useful if false positives on the guardrail do not swamp them. A weekly labelling review — sample the fires, label true / false positive, feed the calibration loop and the near-miss adjustment — is not optional. Without it, the near-miss count is a guardrail-noise count, and the trend is uninformative.

The labelling review's *depth* (LLM-as-judge calibration, human-in-loop protocol, inter-rater reliability) belongs to `ai-eval-engineer` at L30. The risk-side hook — that near-miss counts feeding the register are calibrated — is yours.

## Cost and latency anomalies

Cost and latency are the operating-envelope signals that produce their own harm shape:

- **Cost inflation** — retry storms, prompt bloat, model-thrash between vendors, attacker-driven cost inflation (indirect-prompt-injection making the model call an expensive tool repeatedly). The harm is direct budget impact and, at scale, service degradation.
- **Latency degradation** — SLO burn, user-facing timeout, batch throughput drop. The harm is user-facing quality and, in medical / operational systems, safety.

Signal contract rows for these follow the deviation pattern from the guardrail-firing rows:

- `signal_semantics`: cost per 1k requests vs. rolling 24h median (with volume adjustment); or P95 latency SLO burn rate.
- `threshold`: > 3 sigma above rolling median, sustained > N hours; or > 2x burn budget over 1h window.
- `register_effect`: on sustained trip, the corresponding register row (cost-linked or latency-linked) moves residual state and opens a treatment flag.
- `alert_routing`: ML on-call / SRE + risk engineer; finance liaison for sustained cost anomalies; on-call is a system-owner obligation for latency SLO burn.

The reason cost / latency belong in the risk register (and not only in the SRE dashboard) is that both are *correlated* with the other signal families. A cost anomaly and a prompt-injection detection spike happening together is not two independent incidents; it is one incident.

## The boundary to `ai-eval-engineer` on the app-eval online loop

Application-layer online evaluation — session-level scoring, LLM-as-judge harnesses, continuous quality experiments — sits with `ai-eval-engineer` at L30. Several of the signals in this chapter have deep versions of themselves inside the eval-engineer's remit:

| Chapter-03 signal | Deep counterpart at L30 |
|---|---|
| Guardrail firing rate | Continuous quality regression of the guardrail model itself; calibration loop of the guardrail's judge |
| Jailbreak / injection detection | Adaptive-attacker experimentation, red-team-in-the-loop feedback |
| Near-miss labelling | Full session-scoring harness, human-in-loop protocol, inter-rater reliability |
| Quality regression at session level | Multi-turn / task-completion evaluation, longitudinal user-outcome tracking |

Cite the boundary explicitly in every affected signal contract row. The pattern is: *"signal `X` is sourced from the app-eval online loop owned by `ai-eval-engineer` (L30); the risk-side effect on the register is owned here"*.

## Worked contract rows — safety and application signal

### Row `SC-CS-INJ-01` — prompt-injection firing rate

- `monitor_id`: `guardrail-injection-firing-rate`
- `signal_semantics`: 5-minute rolling firing rate of the prompt-injection input classifier over evaluated requests.
- `cadence`: streaming with 5-minute evaluation window.
- `threshold`: rate > 3x rolling 24-hour baseline for at least 15 minutes; or absolute rate > pre-defined ceiling on the row.
- `register_effect`: `HR-CS-INJ-01` control status flagged; incident candidate opened (mod-111) if sustained; near-miss counter incremented per fire.
- `alert_routing`: on-call + security + governance-council digest on sustained trip.

### Row `SC-CS-PII-01` — PII egress detection

- `monitor_id`: `pii-egress-detector-firing-rate`
- `signal_semantics`: rate of PII-classifier firings on outbound content; per-firing samples land in a labelling queue for verification.
- `cadence`: streaming.
- `threshold`: any *confirmed* positive is S1; firing-rate deviation > baseline is S2.
- `register_effect`: on confirmed positive, `HR-CS-PII-04` residual state -> `control-defeated`; incident candidate opened; treatment flag until pathway remediated.
- `alert_routing`: on-call + security + DPO + governance-council digest.

### Row `SC-CS-NEAR-01` — near-miss weekly rolling count

- `monitor_id`: `near-miss-counter-daily`
- `signal_semantics`: daily count of guardrail hits above severity S mapped to a specific register row identifier.
- `cadence`: daily batch.
- `threshold`: weekly rolling count > per-row threshold defined on `HR-CS-NEAR-07`.
- `register_effect`: `near_miss_last_7d` field updated daily; treatment flag opens on threshold breach.
- `alert_routing`: risk-engineer weekly review; governance-council digest.

### Row `SC-CS-COST-01` — cost anomaly

- `monitor_id`: `cost-per-request-anomaly`
- `signal_semantics`: cost per 1k requests vs. rolling 24h median, volume-adjusted.
- `cadence`: hourly.
- `threshold`: > 3 sigma above rolling median, sustained > 4 hours.
- `register_effect`: `HR-CS-COST-06` residual state -> `control-defeated` on sustained trip; treatment flag with a 3-day due date.
- `alert_routing`: ML on-call / SRE + risk engineer; finance liaison on sustained.

### Row `SC-CS-LAT-01` — latency SLO burn

- `monitor_id`: `latency-p95-slo-burn`
- `signal_semantics`: P95 latency SLO burn rate over a 1-hour window.
- `cadence`: streaming.
- `threshold`: > 2x burn budget over 1h window.
- `register_effect`: `HR-CS-LAT-05` residual state -> `control-defeated`; treatment flag.
- `alert_routing`: on-call + SRE; system owner on sustained.

## Interfaces with adjacent modules

- **mod-101 ch 01** — MEASURE 2.7 (security / resilience), MEASURE 3.2 (unforeseen impacts) and MANAGE 4.1 (post-deployment monitoring) close in this chapter's rows.
- **mod-102 ch 01** — the harm-model persona / capability-surface mapping defines *which* guardrails need signal-contract rows.
- **mod-103 ch 03 and 06** — register rows for injection, PII, near-miss, cost, and latency.
- **mod-108** — the guardrails whose fires this chapter turns into signals; every guardrail carries the register-row identifier it credits.
- **mod-111** — S1 trips open incident candidates.
- **mod-112** — governance-council digest consumes the near-miss and safety-signal trend.

## Summary

- Safety-monitor signal, near-miss data, and cost / latency anomalies are the second family of signal-contract rows.
- Guardrail firing rate is a two-sided deviation signal — up *and* down both matter — and the deviation is against a baseline, not an absolute rate.
- PII egress is an absolute-rule threshold; any confirmed positive is S1 with DPO routing.
- Near-miss data is the leading indicator for the harm the guardrail catches; it is only useful with a labelling / calibration loop.
- Cost / latency belong on the register because they are correlated with the other signal families; treating them as SRE-only misses that correlation.
- The application-layer online-eval loop belongs to `ai-eval-engineer` (L30); cite the boundary in every affected row.
- Exercise 03 wires the safety-monitor family into the register for the running system.
