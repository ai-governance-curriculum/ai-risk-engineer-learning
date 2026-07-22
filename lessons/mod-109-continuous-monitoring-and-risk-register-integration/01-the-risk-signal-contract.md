# The Risk-Signal Contract

A monitor that no register row depends on is decoration. A register row whose residual score is not tied to a live signal is a paper claim. The join between the two — *which monitor updates which register field, at what cadence, at what threshold, escalating to which owner* — is the artifact this chapter defines. Every downstream chapter in this module either populates the contract or consumes it.

## Motivation

Two failure modes are common enough that an auditor asks about them by name:

- **The ad-hoc dashboard.** The team has a Grafana panel showing model accuracy over time. Nobody looks at it unless something is already broken. It has no threshold, no alert, no owner. When a residual score on the register drops below appetite, nobody can point to the moment the signal moved.
- **The Slack alert that only the on-call sees.** A monitor fires, the on-call acknowledges it, a rollback happens, and the register never learns. Six weeks later the same class of alert has fired seven times, the residual has silently changed, and the governance council reads a scorecard that no longer reflects the system.

The risk-signal contract closes both. It says, for every risk-register row that depends on a runtime signal, exactly which monitor supplies the signal, exactly which register field the signal updates, and exactly what happens when the signal crosses a threshold. The register is downstream of the contract; the monitor is upstream. The contract is the join.

## What a signal contract is

A signal contract is a versioned artifact — one row per (monitor, register-field) pair — with the following required columns:

| Column | What it holds |
|---|---|
| `contract_id` | Stable identifier used by both the monitor spec and the register row |
| `register_row_id` | The `id` of the risk-register row (mod-103) this signal feeds |
| `register_field` | The field on the row that the signal updates (`likelihood_evidence`, `control_status`, `monitor_last_fired`, `residual_state`) |
| `monitor_id` | The monitor spec identifier (whylogs profile, Evidently report, Alibi Detect detector, guardrail-firing counter, cost / latency SLO burn) |
| `signal_source_owner` | Team that runs the monitor pipeline (typically `ai-infra-mlops` — peer L25) |
| `signal_semantics` | One sentence naming *what the number means* (e.g., "PSI on the top-20 features vs. reference window") |
| `cadence` | Batch cadence (hourly, daily) or streaming (with evaluation window) |
| `threshold` | The numeric or categorical rule that trips the alert |
| `severity` | Ordinal severity used to route the alert (S1/S2/S3 or equivalent) |
| `register_effect` | Deterministic effect on the row when tripped (residual state changes to `control-defeated`; treatment field flagged; near-miss counter increments) |
| `alert_routing` | On-call, security, governance-council routing (chapter 05) |
| `owner_on_call` | Named role that owns first response |
| `escalation_path` | Named roles for S1 escalation; ties into mod-111 incident response |
| `evidence_pointer` | Where the tripped-signal record is filed for later audit |
| `deferral_boundary` | If the monitor pipeline itself is out of scope for this role, cite the peer track and where the boundary sits |

Rows without every column filled are *drafts*. A draft contract is not shippable.

## The signal → register-field mapping

The contract is easier to reason about with concrete rows. Below is the mapping shape for a customer-support RAG assistant (the running example carried across mods 102, 103, and 108). Each row is one signal contract. Read across.

| contract_id | register_row_id | register_field | monitor_id | signal_semantics | cadence | threshold | severity | register_effect | alert_routing |
|---|---|---|---|---|---|---|---|---|---|
| `SC-CS-DRIFT-01` | `HR-CS-DRIFT-02` | `likelihood_evidence` + `residual_state` | `whylogs-ref-vs-prod-daily` | PSI on top-20 input features vs. baseline window | daily | PSI > 0.2 on any monitored feature | S2 | Residual state -> `control-defeated`; treatment flag on | on-call ML + risk engineer digest |
| `SC-CS-QUAL-01` | `HR-CS-QUAL-03` | `likelihood_evidence` | `evidently-quality-daily` | Regression on task-level quality metric vs. golden-set eval | daily | Quality delta > 5% absolute vs. rolling 7-day median | S2 | Residual state -> `control-defeated`; near-miss counter++ | on-call ML + risk engineer digest |
| `SC-CS-INJ-01` | `HR-CS-INJ-01` | `control_status` + `monitor_last_fired` | `guardrail-injection-firing-rate` | Rate of prompt-injection detector firings per 1k requests | 5-min window | Rate > 3x rolling baseline for 15 min | S1 | Row flagged; incident candidate opened | on-call + security + governance-council digest |
| `SC-CS-PII-01` | `HR-CS-PII-04` | `control_status` | `pii-egress-detector-firing-rate` | Rate of PII-detector firings on outbound content | 5-min window | Any confirmed egress; rate > 0 | S1 | Row flagged; incident candidate opened | on-call + security + DPO + governance-council digest |
| `SC-CS-COST-01` | `HR-CS-COST-06` | `likelihood_evidence` | `cost-per-request-anomaly` | Cost per 1k requests vs. rolling 24h median | hourly | > 3 sigma above rolling median | S3 | Near-miss counter++; treatment flag if sustained > 4h | on-call ML + risk engineer digest |
| `SC-CS-LAT-01` | `HR-CS-LAT-05` | `likelihood_evidence` | `latency-p95-slo-burn` | P95 latency SLO burn rate | streaming | 2x burn budget over 1h window | S2 | Residual state -> `control-defeated` on latency-linked control | on-call ML + risk engineer digest |
| `SC-CS-NEAR-01` | `HR-CS-NEAR-07` | `likelihood_evidence` | `near-miss-counter-daily` | Count of guardrail hits above severity S in trace store | daily | Weekly rolling count > threshold set on row | S3 | Row's `near_miss_last_7d` field updated; treatment flag on threshold | risk engineer weekly digest |

Every cell in the table is a decision you defend. The threshold column in particular is the one auditors ask about — *"why 0.2 PSI and not 0.1?"* The answer is "because the reference window study on this system's features showed 0.2 as the noise floor and 0.35 as the point where downstream quality regressed," documented in the row's evidence pointer.

## Why an ad-hoc dashboard does not count as a monitor

A monitor, for the purpose of the contract, has to satisfy all five of the following:

1. **A defined signal.** The number the monitor emits has a name, a unit, and a semantics sentence.
2. **A threshold.** A rule for when the signal is a signal — not a chart the eye interprets.
3. **A cadence.** A batch schedule or a streaming window. The monitor runs whether or not anyone is looking.
4. **A routing.** When the threshold is tripped, the alert reaches a named on-call by a named channel; the register row is updated deterministically.
5. **An evidence trail.** Every tripped signal is filed with a timestamp, the raw value, and the context needed for post-hoc review.

If any of the five is missing, the artifact is a dashboard, not a monitor. Dashboards are useful for exploration; they do not carry regulatory or register load. Do not credit them on register rows.

## Cadence and threshold as engineering choices

The temptation is to pick a cadence and a threshold once and forget. Both have to be defended, and both are revisited on the events listed in mod-103 chapter 06 (system change, model change, control change, incident, framework revision, base-rate shift, red-team finding).

Cadence choices, in decreasing order of latency budget:

- **Streaming with an evaluation window** — for signals where a delay is itself a harm (jailbreak firing rate, PII egress detection, latency SLO burn). Window is a design choice: short windows catch bursts but produce noise; long windows smooth noise but delay response.
- **Sub-hourly batch** — for signals where the sensor cost is high but the harm can wait tens of minutes (cost anomaly, some quality regressions).
- **Daily batch** — for drift-family signals where the underlying distribution needs a reasonable sample size to compute meaningfully (PSI, KS, MMD on tabular inputs).
- **Weekly rolling** — for near-miss and cumulative-count signals where the weekly cadence matches the governance-council digest.

Threshold choices come in three families:

- **Statistical thresholds** — drawn from the drift-metric literature (PSI band conventions, KS at a chosen alpha, MMD at a chosen kernel and test-set size); see chapter 02.
- **SLO / budget thresholds** — burn rates, budget exhaustion, latency percentiles.
- **Absolute-rule thresholds** — any PII egress, any confirmed jailbreak, any tool call to an out-of-scope endpoint.

An absolute-rule threshold sits above the statistical thresholds in severity because the tripping event *is* the harm, not evidence of a shifting probability.

## The escalation column — who owns what when the signal fires

Every contract row names one on-call owner and one escalation path. The floor of who is on the routing:

| Signal family | First on-call | Additional routing on S1 |
|---|---|---|
| Drift / quality regression | ML on-call | Risk engineer digest; system owner on sustained trip |
| Safety-monitor / jailbreak / PII | Application on-call + security | Governance council digest; DPO if PII-linked; incident response (mod-111) |
| Cost / latency anomaly | ML on-call / SRE | Risk engineer digest; finance liaison on sustained cost anomaly |
| Near-miss cumulative | Risk engineer weekly review | Governance council digest on threshold breach |

The important discipline: no signal reaches *only* the on-call. Every signal has both a first responder and a register-side updater, and both are named on the row.

## Interfaces with adjacent modules

- **mod-101 ch 01** — the MEASURE and MANAGE sub-categories translated to test / threshold / guardrail / monitor / evidence; this contract *is* the monitor and evidence columns.
- **mod-103 ch 03 and ch 06** — the register row schema and operational fields; the contract's `register_field` column points at fields defined there.
- **mod-108** — guardrails are the source of the safety-monitor family of signals (chapter 03 of this module elaborates).
- **mod-111** — S1 alerts open incident candidates; the contract's `escalation_path` column is the wire to incident response.
- **mod-112** — the governance-council digest aggregates the register-effect column across contracts.

## What the contract is *not*

- **Not a full observability spec.** The engineering-side spec of the whylogs profile, the Evidently report configuration, the Alibi Detect detector graph — that lives with the MLOps pipeline (peer L25). The contract cites the monitor identifier; it does not re-specify the pipeline.
- **Not the guardrail spec.** The guardrail's shape and effectiveness measurement lives in mod-108. The contract consumes the guardrail's firing rate as a signal.
- **Not the incident-response playbook.** Incident triage, containment, and post-mortem live in mod-111. The contract *hands over* to the playbook when severity is S1.
- **Not the app-eval online loop.** Continuous eval of the application layer (user-facing task quality, LLM-as-judge scoring, session-level metrics) belongs to `ai-eval-engineer` at L30. The contract cites its output as a signal source; it does not own the eval loop.

The last two boundaries are the ones this role's audit trail most commonly muddles. Cite them in the `deferral_boundary` column so the auditor sees them.

## Versioning and change control

The contract is versioned in the risk-engineering repo alongside the register. Every row has a `sign_off` block that mirrors mod-103 chapter 06's routing (row author + system owner + control owner). Every threshold change is a diff and a reviewer. Rubric-scale changes (moving the whole family of drift thresholds when the reference window is regenerated) bump the version of the contract set as a whole.

## Summary

- The signal contract is the join between monitors and register rows. Neither the monitor nor the register is complete without it.
- Every contract row has fifteen required columns; a row missing any of them is a draft.
- An ad-hoc dashboard is not a monitor. A monitor has a signal, a threshold, a cadence, a routing, and an evidence trail.
- Cadence and threshold are engineering choices you defend; they are revisited on the same events that drive register review.
- Every signal has both a first responder and a register-side updater; no signal reaches only the on-call.
- The contract cites boundaries to peer L25 (`ai-infra-mlops`, monitoring pipeline) and peer L30 (`ai-eval-engineer`, app-eval online loop) rather than re-owning them.
- Exercise 01 walks the contract-drill for the running system; every later exercise in the module fills one family of rows.
