# Lab 01 — Signal-Contract Row Authoring Drill (Fifteen-Column Shape)

**Duration:** 45 minutes · **Format:** guided practice, markdown authoring on a fixed fabricated LLM application

## Objective

Internalise the chapter-01 **fifteen-column signal contract** — `contract_id`, `register_row_id`, `register_field`, `monitor_id`, `signal_source_owner`, `signal_semantics`, `cadence`, `threshold`, `severity`, `register_effect`, `alert_routing`, `owner_on_call`, `escalation_path`, `evidence_pointer`, `deferral_boundary` — and the chapter-01 five-condition monitor test by authoring the contract for a fixed fabricated running system (`HelpDesk`), with one row per signal family, front-matter, and anti-pattern probe. The lab drills the artefact shape chapters 01-03 pin so that when **exercise 01** produces the contract for your real running system, the engineering budget goes into which register rows need runtime signals, which monitors supply them, and how the thresholds are defended — not into inventing the artefact format.

The chapter-01 anti-patterns the shape forecloses: the **ad-hoc dashboard** credited as a monitor, the **silent Slack-only alert** that only the on-call sees, the **unpinned threshold** with no defended origin, and the **missing deferral clause** that leaves ownership fog.

## Prerequisites

- Chapter 01 read — specifically "What a signal contract is" (the fifteen-column schema), "Why an ad-hoc dashboard does not count as a monitor" (the five conditions), "Cadence and threshold as engineering choices" (the four cadence families; the three threshold families), "The escalation column", and "What the contract is *not*" (the two peer boundaries).
- Chapters 02 and 03 skimmed at the worked-row examples (`SC-CS-DRIFT-01`, `SC-CS-QUAL-01`, `SC-CS-EMB-01`, `SC-CS-INJ-01`, `SC-CS-PII-01`, `SC-CS-NEAR-01`, `SC-CS-COST-01`, `SC-CS-LAT-01`) so the semantics / cadence / threshold shapes are recognisable.
- A terminal where you can create a working directory and author a markdown file.
- No external access or compute required; the system brief, the register row stubs, and the monitor identifiers are fixed below.

## The fixed example system

Use this system throughout the lab — do *not* substitute your own. Carrying a fixed scenario keeps the lab bounded to the contract-row authoring skill. The system is deliberately aligned with the customer-support RAG assistant the chapters use as the running example.

> **`HelpDesk`** — a customer-support RAG assistant for a mid-size B2B SaaS company. Serves English-language customers of a billing-and-provisioning product. Assembles context from a knowledge-base store (product docs + internal runbooks) and from the authenticated caller's own recent ticket history. Backed by an instruction-tuned LLM behind a FastAPI service wrapping a LangGraph agent. Two production tool calls: `escalate_ticket(destination, summary)` and `lookup_account(account_id)`. Structured JSON response with fields `answer`, `citations`, `escalate_ticket`.
>
> Classification under the EU AI Act: **high-risk** (Annex III use case affecting a provider's customers, with account data in scope); Article 61 / 72 post-market monitoring plan applies.
>
> Protected-attribute slice in scope: tenant tier (`standard` vs. `enterprise`), used as a per-slice proxy for coverage / quality parity monitoring.

### Fixed register row stubs (do not add more)

Every contract row cites a `register_row_id` that exists in this stub set. The register-side schema is from mod-103; the fields referenced below are `likelihood_evidence`, `control_status`, `monitor_last_fired`, `residual_state`, `near_miss_last_7d`.

| Register row id | Scope line |
|---|---|
| `HR-CS-DRIFT-02` | Covariate drift on top-20 input features and on prompt-embedding distribution; above-appetite if drift sustained. |
| `HR-CS-QUAL-03` | Quality regression on the composite (faithfulness + coverage + safe-refusal) metric; per-slice sibling at `HR-CS-FAIR-08`. |
| `HR-CS-FAIR-08` | Per-slice quality parity on the `tenant_tier` slice; fairness-tagged row; mod-106 seam. |
| `HR-CS-INJ-01` | Prompt-injection control credit — defence in depth across input filter, retrieval filter, and tool-invocation policy. |
| `HR-CS-PII-04` | PII egress control credit — Presidio at ingest / retrieval / output and per-caller entitlement; absolute-rule row. |
| `HR-CS-NEAR-07` | Near-miss cumulative counter across guardrail fires above severity S; weekly rolling count. |
| `HR-CS-COST-06` | Cost-per-request operating envelope; cost anomaly leads to a residual move on sustained trip. |
| `HR-CS-LAT-05` | P95 latency SLO burn; latency-linked control. |

### Fixed monitor identifiers (do not revise)

Use these monitor identifiers verbatim; they match the chapter 02 and 03 worked examples. Each has a stable identifier, a version, and a signal source owner (peer L25 `ai-infra-mlops`).

| Monitor id | Family | Source | Version pin |
|---|---|---|---|
| `whylogs-ref-vs-prod-daily` | Covariate drift | peer L25 pipeline | `v1.4.0` |
| `alibi-detect-mmd-embeddings-hourly` | Embedding / multivariate drift | peer L25 pipeline | `v0.11.1` |
| `evidently-quality-daily` | Batch quality regression + test suite | peer L25 pipeline | `v0.4.17` |
| `evidently-quality-per-slice-daily` | Per-slice quality regression | peer L25 pipeline | `v0.4.17` |
| `guardrail-injection-firing-rate` | Mod-108 injection classifier event stream | peer L25 pipeline (schema joint with mod-108) | mod-108 `v2.1.3` |
| `pii-egress-detector-firing-rate` | Mod-108 PII detector event stream + labelling queue | peer L25 pipeline (schema joint with mod-108) | mod-108 `v2.1.3` |
| `near-miss-counter-daily` | Trace-store aggregation per `HR-*` row | peer L25 pipeline | `v1.0.4` |
| `cost-per-request-anomaly` | Cost telemetry anomaly detector | peer L25 pipeline / SRE | `v0.9.0` |
| `latency-p95-slo-burn` | SLO burn-rate calculator | peer L25 pipeline / SRE | `v0.9.0` |

### Fixed peer contacts (do not revise)

- `signal_source_owner`: **`ai-infra-mlops-L25` — owner role** (always this value; the contract never re-owns the pipeline).
- `deferral_boundary` cites `ai-infra-mlops` (L25) on all pipeline-plumbing rows and `ai-eval-engineer` (L30) on every row where the eval loop supplies a calibrated label, a judge-calibration feed, or the near-miss false-positive propagation.

## Steps

### Step 1 — Create your working directory and the contract scaffold

```bash
mkdir -p mod109-lab01 && cd mod109-lab01
touch helpdesk-signal-contract.md
```

Open `helpdesk-signal-contract.md` and paste this scaffold. You will fill the sections in order.

```markdown
# HelpDesk — signal contract (lab 01)

System: HelpDesk (customer-support RAG assistant; see lab brief)
Chapter references: mod-109 ch 01 (contract schema; five-condition monitor test; cadence / threshold; peer boundaries); ch 02 (drift / quality worked rows); ch 03 (safety-monitor / near-miss / cost / latency worked rows)
Register rows cited: HR-CS-DRIFT-02, HR-CS-QUAL-03, HR-CS-FAIR-08, HR-CS-INJ-01, HR-CS-PII-04, HR-CS-NEAR-07, HR-CS-COST-06, HR-CS-LAT-05

## 1. Front-matter

(system identifier; contract semantic version; sign-off block; plan version; families-in-scope list)

## 2. Signal-contract rows

(one row per signal family; all fifteen columns populated per chapter 01 §"What a signal contract is")

## 3. Family coverage table

(seven families × rows drafted; blank cells justified in-line)

## 4. Anti-pattern probe

(four short sections per chapter 01: ad-hoc-dashboard, silent-Slack-only, unpinned-threshold, missing-deferral)

## 5. Open items

(what is still draft; what the exercise-01 run picks up)
```

### Step 2 — Fill Section 1 (Front-matter)

Pin the system, the contract version, the sign-off roles, the plan version, and the families in scope. Chapter 01 §"Versioning and change control" and exercise 01 §"The contract front-matter" are the reference shapes.

```markdown
## 1. Front-matter

| Field | Value |
|---|---|
| System identifier | HelpDesk (mod-102 intake `INT-HD-001`) |
| EU AI Act classification | High-risk (Annex III) — Article 61 / 72 plan applies |
| Contract semantic version | v0.1 (initial draft; change-log entry `0.1-initial` dated lab-authoring) |
| Plan version (Article 61 / 72) | `pms-plan-v0.1` (exercise 04 produces the paired plan) |
| Sign-off routing | row author (risk engineer) + system owner (HelpDesk tech lead) + control owner (platform lead) + risk-council chair (high-risk system — per mod-101 ch 04) |
| Families in scope | drift, quality (aggregate + per-slice), safety-monitor (injection), PII egress, near-miss cumulative, cost, latency |
| Families scoped out | jailbreak detector — folded into the injection family on HelpDesk (single input-side classifier covers both); justified in front-matter not per-row (§exercise 01 pattern) |
```

Reading check: five fields; every value is a specific pointer (intake identifier, plan version, named sign-off role) rather than "TBD"; scoped-out family carries a one-line justification.

### Step 3 — Fill Section 2 (Signal-contract rows)

One row per signal family. Each row has all fifteen columns populated per chapter 01. Use the fabricated monitor identifiers and register row stubs verbatim; the engineering choices are in the semantics sentence, the cadence / threshold, the register effect, and the routing.

Format each row as a labelled block, not a horizontal mega-table — the fifteen columns do not fit on one screen and a block per row is easier to review.

```markdown
### Row `SC-HD-DRIFT-01` — daily PSI on top-20 input features

- `contract_id`: `SC-HD-DRIFT-01`
- `register_row_id`: `HR-CS-DRIFT-02`
- `register_field`: `likelihood_evidence`, `residual_state`
- `monitor_id`: `whylogs-ref-vs-prod-daily` (v1.4.0)
- `signal_source_owner`: `ai-infra-mlops-L25`
- `signal_semantics`: PSI computed per feature on the top-20 features from the reference profile taken at the last release, compared to a rolling 24-hour production profile; emitted in the middleware.
- `cadence`: daily batch at 02:00 UTC (chapter 01 §"Cadence and threshold" — daily batch for distributional drift).
- `threshold`: PSI > 0.2 on any single feature *or* PSI > 0.1 on three or more features simultaneously (Siddiqi band + validated against HelpDesk's reference-window study).
- `severity`: S2.
- `register_effect`: `HR-CS-DRIFT-02` residual state → `control-defeated`; treatment flag opens with a 5-day due date; feature list attached to the register-side note.
- `alert_routing`: ML on-call channel (page on trip); risk-engineer digest (daily summary).
- `owner_on_call`: ML on-call.
- `escalation_path`: ML on-call manager at 2h non-ack; system owner on sustained trip > 24h.
- `evidence_pointer`: `evidence/sc-hd-drift-01/<yyyy-mm-dd>/report.json` with reference profile hash + production profile hash.
- `deferral_boundary`: peer L25 `ai-infra-mlops` owns the whylogs pipeline, the profile storage, and the batch scheduler; the signal semantics + threshold + register effect + routing are owned here.
- `status`: draft (exercise 02 ships the detector).

### Row `SC-HD-QUAL-01` — daily aggregate quality regression

- `contract_id`: `SC-HD-QUAL-01`
- `register_row_id`: `HR-CS-QUAL-03`
- `register_field`: `likelihood_evidence`, `residual_state`
- `monitor_id`: `evidently-quality-daily` (v0.4.17)
- `signal_source_owner`: `ai-infra-mlops-L25`
- `signal_semantics`: task-level composite quality (faithfulness + coverage + safe-refusal) computed on the HelpDesk golden set of 500 examples; the golden set is versioned and the version is recorded per run.
- `cadence`: daily batch; also triggered on model / prompt / retriever change.
- `threshold`: aggregate metric delta > 5% absolute vs. rolling 7-day median (Evidently test-suite pass/fail is the vehicle).
- `severity`: S2.
- `register_effect`: `HR-CS-QUAL-03` residual state → `control-defeated` on aggregate breach.
- `alert_routing`: ML on-call; risk-engineer digest daily.
- `owner_on_call`: ML on-call.
- `escalation_path`: ML on-call manager at 2h non-ack; system owner on sustained trip > 24h.
- `evidence_pointer`: `evidence/sc-hd-qual-01/<yyyy-mm-dd>/report.json` with golden-set version + per-metric breakdown.
- `deferral_boundary`: peer L25 `ai-infra-mlops` owns the Evidently pipeline; peer L30 `ai-eval-engineer` owns the LLM-as-judge calibration that keeps faithfulness honest (seam cited explicitly).
- `status`: draft (exercise 02 ships the detector).

### Row `SC-HD-QUAL-02` — per-slice quality regression (tenant_tier)

- `contract_id`: `SC-HD-QUAL-02`
- `register_row_id`: `HR-CS-FAIR-08` (fairness-tagged sibling of `HR-CS-QUAL-03`)
- `register_field`: `likelihood_evidence`, `residual_state`
- `monitor_id`: `evidently-quality-per-slice-daily` (v0.4.17)
- `signal_source_owner`: `ai-infra-mlops-L25`
- `signal_semantics`: per-slice composite quality computed with a group-by on `tenant_tier` (standard vs. enterprise); each slice is compared against its own rolling 7-day median; per-slice trip counts even if aggregate is quiet.
- `cadence`: daily batch.
- `threshold`: per-slice metric delta > 10% absolute for any slice (Evidently per-slice test suite; the per-slice threshold is tighter than aggregate per chapter 02 §"Per-slice monitoring").
- `severity`: S2.
- `register_effect`: `HR-CS-FAIR-08` residual state → `control-defeated`; slice name + delta attached to the register note; `HR-CS-QUAL-03` is additionally flagged on slice breach.
- `alert_routing`: ML on-call + **fairness liaison** on slice breach; risk-engineer digest.
- `owner_on_call`: ML on-call (joint with fairness liaison on slice breach).
- `escalation_path`: fairness liaison manager at 4h non-ack; risk-council chair on sustained slice regression > 72h.
- `evidence_pointer`: `evidence/sc-hd-qual-02/<yyyy-mm-dd>/report.json` with per-slice breakdown.
- `deferral_boundary`: peer L25 `ai-infra-mlops` on the pipeline; peer L30 `ai-eval-engineer` on the judge calibration that keeps per-slice composite honest.
- `status`: draft.

### Row `SC-HD-EMB-01` — hourly MMD on prompt embeddings

- `contract_id`: `SC-HD-EMB-01`
- `register_row_id`: `HR-CS-DRIFT-02`
- `register_field`: `likelihood_evidence`, `residual_state`
- `monitor_id`: `alibi-detect-mmd-embeddings-hourly` (v0.11.1)
- `signal_source_owner`: `ai-infra-mlops-L25`
- `signal_semantics`: MMD (RBF kernel, median-heuristic bandwidth per Gretton et al. 2012) on user-prompt embeddings vs. a reference sample taken at last release; permutation-test p-value emitted per window.
- `cadence`: hourly batch on the last hour's prompts; sub-sampled to a fixed test-set size (recorded on the row).
- `threshold`: permutation-test p-value < 0.01 for three consecutive hours.
- `severity`: S2.
- `register_effect`: `HR-CS-DRIFT-02` residual state → `control-defeated`; drifted-cluster description attached to the register row.
- `alert_routing`: ML on-call; risk-engineer digest.
- `owner_on_call`: ML on-call.
- `escalation_path`: ML on-call manager at 4h non-ack; system owner on sustained > 24h.
- `evidence_pointer`: `evidence/sc-hd-emb-01/<yyyy-mm-dd-hh>/report.json` with sample identifiers + MMD statistic + reference sample hash.
- `deferral_boundary`: peer L25 `ai-infra-mlops` owns the Alibi Detect detector plumbing; the kernel and bandwidth choice are defended here.
- `status`: draft.

### Row `SC-HD-INJ-01` — prompt-injection firing rate (two-sided)

- `contract_id`: `SC-HD-INJ-01`
- `register_row_id`: `HR-CS-INJ-01`
- `register_field`: `control_status`, `monitor_last_fired`
- `monitor_id`: `guardrail-injection-firing-rate` (mod-108 v2.1.3)
- `signal_source_owner`: `ai-infra-mlops-L25`
- `signal_semantics`: 5-minute rolling firing rate of the input-side prompt-injection classifier over evaluated requests; **two-sided deviation** — rate up *and* rate down are both signals (chapter 03 §"Deviation is a two-sided signal").
- `cadence`: streaming with 5-minute evaluation window.
- `threshold`: rate > 3x rolling 24h baseline for ≥ 15 minutes (up) *or* rate < 1/3x rolling 24h baseline while request volume stays within ±20% (down — control-availability case).
- `severity`: S1 (both directions).
- `register_effect`: on rate-up, `HR-CS-INJ-01` control flagged + incident candidate opened + near-miss counter increments on each fire; on rate-down with steady volume, `HR-CS-INJ-01` `control_status` → unavailable + incident candidate opened.
- `alert_routing`: on-call + security + governance-council digest on sustained trip; security + circuit-breaker check on rate-down case.
- `owner_on_call`: ML on-call (joint with security on S1).
- `escalation_path`: ML on-call manager at 15m non-ack; system owner at 1h; security lead if attack pattern is novel; legal + DPO if any PII touch.
- `evidence_pointer`: `evidence/sc-hd-inj-01/<yyyy-mm-dd-hh>/record.json` with payload-hash sample + rate numbers.
- `deferral_boundary`: peer L25 `ai-infra-mlops` on the event stream; mod-108 owns the guardrail's shape; peer L30 `ai-eval-engineer` on the labelling-queue protocol that feeds the false-positive propagation into the near-miss counter.
- `status`: draft.

### Row `SC-HD-PII-01` — PII egress (absolute-rule)

- `contract_id`: `SC-HD-PII-01`
- `register_row_id`: `HR-CS-PII-04`
- `register_field`: `control_status`, `residual_state`
- `monitor_id`: `pii-egress-detector-firing-rate` (mod-108 v2.1.3)
- `signal_source_owner`: `ai-infra-mlops-L25`
- `signal_semantics`: PII-classifier firings on outbound content; per-firing samples routed to a labelling queue; a **confirmed positive** is any firing the labelling queue verifies as real PII (chapter 03 §"PII / sensitive-data leakage — the absolute-rule threshold").
- `cadence`: streaming.
- `threshold`: **any confirmed positive** → S1 (absolute rule); firing-rate deviation > 3x rolling 24h baseline → S2 (secondary row).
- `severity`: S1 (absolute rule) / S2 (deviation).
- `register_effect`: on confirmed positive, `HR-CS-PII-04` residual state → `control-defeated`; incident candidate opened; treatment flag until pathway remediated.
- `alert_routing`: on-call + security + **DPO (by role name)** + governance-council digest.
- `owner_on_call`: ML on-call (joint with security + DPO on S1).
- `escalation_path`: DPO within 15m on S1; legal counsel within 1h on S1 (Article 73 triage); system owner within 1h.
- `evidence_pointer`: `evidence/sc-hd-pii-01/<yyyy-mm-dd-hh>/record.json` with payload-hash + labelling-queue verdict + sample identifier.
- `deferral_boundary`: peer L25 `ai-infra-mlops` on the event stream; mod-108 owns the guardrail; peer L30 `ai-eval-engineer` on the labelling protocol depth; the DPO is the risk-side owner of the privacy pathway.
- `status`: draft.

### Row `SC-HD-NEAR-01` — near-miss weekly rolling count

- `contract_id`: `SC-HD-NEAR-01`
- `register_row_id`: `HR-CS-NEAR-07`
- `register_field`: `near_miss_last_7d`, `likelihood_evidence`
- `monitor_id`: `near-miss-counter-daily` (v1.0.4)
- `signal_source_owner`: `ai-infra-mlops-L25`
- `signal_semantics`: daily count of guardrail hits above severity S mapped to per-`HR-*`-row identifier; **excludes** fires whose `false_positive: true` field is set by the labelling loop (chapter 03 §"The false-positive discipline").
- `cadence`: daily batch (count); weekly rolling window (threshold).
- `threshold`: weekly rolling count > per-row threshold defined on `HR-CS-NEAR-07` (default: 2x the 30-day baseline for the row).
- `severity`: S3.
- `register_effect`: `HR-CS-NEAR-07.near_miss_last_7d` field updated daily; treatment flag opens on weekly-threshold breach; escalates to S2 if two consecutive weeks breach.
- `alert_routing`: risk-engineer weekly review; governance-council digest.
- `owner_on_call`: risk engineer (weekly review owner).
- `escalation_path`: risk-council chair on two consecutive weekly breaches.
- `evidence_pointer`: `evidence/sc-hd-near-01/<yyyy-ww>/count.json` with per-day counts + the labelling-queue state that fed the FP exclusion.
- `deferral_boundary`: peer L25 on the counter plumbing + trace-store retention; peer L30 `ai-eval-engineer` on the labelling protocol depth that drives the FP field.
- `status`: draft.

### Row `SC-HD-COST-01` — cost anomaly

- `contract_id`: `SC-HD-COST-01`
- `register_row_id`: `HR-CS-COST-06`
- `register_field`: `likelihood_evidence`, `residual_state`
- `monitor_id`: `cost-per-request-anomaly` (v0.9.0)
- `signal_source_owner`: `ai-infra-mlops-L25`
- `signal_semantics`: cost per 1k requests vs. rolling 24h median, **volume-adjusted** (denominator is 1k requests, not unit time; chapter 03 §"Cost and latency anomalies").
- `cadence`: hourly batch.
- `threshold`: > 3σ above rolling 24h median, sustained > 4 hours.
- `severity`: S3 (S2 if correlated with `SC-HD-INJ-01` in the same window — see correlation memo in exercise 03).
- `register_effect`: `HR-CS-COST-06` residual state → `control-defeated` on sustained trip; treatment flag with a 3-day due date.
- `alert_routing`: ML on-call / SRE + risk engineer; finance liaison on sustained.
- `owner_on_call`: ML on-call / SRE.
- `escalation_path`: SRE lead at 4h non-ack; system owner + finance on sustained > 12h.
- `evidence_pointer`: `evidence/sc-hd-cost-01/<yyyy-mm-dd-hh>/record.json` with per-hour cost + rolling baseline snapshot.
- `deferral_boundary`: peer L25 on the telemetry pipeline / median baseline recompute cadence.
- `status`: draft.

### Row `SC-HD-LAT-01` — latency SLO burn

- `contract_id`: `SC-HD-LAT-01`
- `register_row_id`: `HR-CS-LAT-05`
- `register_field`: `likelihood_evidence`, `residual_state`
- `monitor_id`: `latency-p95-slo-burn` (v0.9.0)
- `signal_source_owner`: `ai-infra-mlops-L25`
- `signal_semantics`: P95 latency SLO burn rate over a 1-hour window (two-threshold burn rule: 2x burn over 1h; 14x burn over 5m per the Google SRE workbook burn-rate pattern).
- `cadence`: streaming.
- `threshold`: > 2x burn budget over 1h window (S2) *or* > 14x burn over 5m (S1).
- `severity`: S2 / S1.
- `register_effect`: `HR-CS-LAT-05` residual state → `control-defeated` on trip; treatment flag.
- `alert_routing`: on-call + SRE; system owner on sustained; governance-council digest on S1.
- `owner_on_call`: ML on-call / SRE.
- `escalation_path`: SRE lead at 15m on S1; system owner within 1h.
- `evidence_pointer`: `evidence/sc-hd-lat-01/<yyyy-mm-dd-hh>/record.json` with p95 numbers + SLO version.
- `deferral_boundary`: peer L25 on the telemetry pipeline; SRE owns the SLO document.
- `status`: draft.
```

Reading check: nine rows, one per signal family (plus the per-slice sibling and the embedding-drift row). Every row has all fifteen columns. Every `signal_semantics` sentence names the unit, window, and denominator. Every threshold cites the origin (Siddiqi band, Gretton MMD, SLO doc, absolute rule). Every row's `deferral_boundary` names the peer track. S1 rows carry a DPO or security escalation path by role name where chapter 03 requires it.

### Step 4 — Fill Section 3 (Family coverage table)

Seven families from exercise 01's family set. Each cell is either a contract row identifier or a one-sentence justification for scoping out.

```markdown
| Signal family | Contract rows drafted |
|---|---|
| Model / data drift | `SC-HD-DRIFT-01` (daily PSI), `SC-HD-EMB-01` (hourly MMD on embeddings) |
| Quality regression | `SC-HD-QUAL-01` (aggregate), `SC-HD-QUAL-02` (per-slice on `tenant_tier`) |
| Safety-monitor (injection / jailbreak) | `SC-HD-INJ-01` (jailbreak detector folded into injection per front-matter justification) |
| PII / sensitive-data egress | `SC-HD-PII-01` (absolute-rule + secondary deviation row shares the identifier) |
| Near-miss cumulative | `SC-HD-NEAR-01` |
| Cost anomalies | `SC-HD-COST-01` |
| Latency anomalies | `SC-HD-LAT-01` |
```

Reading check: every one of the seven families is represented by at least one row (or justified at the front-matter level). No "TBD" cells; no silent blanks.

### Step 5 — Fill Section 4 (Anti-pattern probe)

Four short sections, one per chapter-01 anti-pattern. Exercise 01 §"Anti-pattern probe" is the reference shape.

```markdown
### Ad-hoc dashboard credited as a monitor

Every row satisfies all five conditions from chapter 01 §"Why an ad-hoc dashboard does not count as a monitor": defined signal (semantics sentence with unit and window), threshold (citation on the row), cadence (batch schedule or streaming window named), routing (named on-call + register-side updater), evidence trail (per-trip record at the `evidence_pointer` path). No row credits a Grafana panel.

### Silent Slack-only alert

Every row carries both a first responder (`owner_on_call`) and a register-side updater (`register_effect`). Chapter 01 §"The escalation column" rule — "no signal reaches *only* the on-call" — holds row-by-row. The `#ml-oncall` channel is one derivative view; the record is the source of truth (chapter 05 property).

### Unpinned threshold

Every threshold cites its origin: Siddiqi PSI band (`SC-HD-DRIFT-01`), Gretton MMD kernel (`SC-HD-EMB-01`), Evidently test-suite pass/fail with aggregate 5% and per-slice 10% cutoffs (`SC-HD-QUAL-01` / `SC-HD-QUAL-02`), rolling-baseline deviation (`SC-HD-INJ-01`, `SC-HD-COST-01`), absolute rule from harm-model row (`SC-HD-PII-01`), SRE-workbook two-threshold burn (`SC-HD-LAT-01`), per-row near-miss baseline (`SC-HD-NEAR-01`). No row carries a bare "> 0.2" without a defended citation.

### Missing deferral clause

Every row's `deferral_boundary` cites `ai-infra-mlops` (L25) on the pipeline-plumbing scope. Rows that depend on labelling / judge calibration (`SC-HD-QUAL-01`, `SC-HD-QUAL-02`, `SC-HD-INJ-01`, `SC-HD-PII-01`, `SC-HD-NEAR-01`) cite `ai-eval-engineer` (L30) explicitly. No row is silent on the ownership split.
```

### Step 6 — Fill Section 5 (Open items)

```markdown
- All nine rows are `status: draft` — the detectors are built in exercise 02 (drift + quality + embedding) and exercise 03 (injection + PII + near-miss + cost + latency). The contract is shippable at draft status per chapter 01 §"Versioning and change control" as long as the missing columns are explicit; here no columns are missing, only the detector implementation is pending.
- The jailbreak family is folded into `SC-HD-INJ-01` per the front-matter justification; if the system later adds a separate jailbreak classifier, a sibling row (`SC-HD-JAIL-01`) is opened with its own event stream.
- The tenant-tier slice is the only protected-attribute slice in scope; if the mod-106 fairness review adds slices, the per-slice row is duplicated (one per slice group) rather than folded.
- Register-side back-link (stretch goal from exercise 01) is not populated here — the HR-CS-* stubs would carry a `signal_contracts: [SC-HD-*]` field when the register file is updated.
- `needs-research` markers from chapter 04 (Commission Article 72 implementing act; FDA SaMD updates) are carried into the paired plan (`pms-plan-v0.1`) at exercise 04 time, not here.
```

### Step 7 — Final checkpoint

Re-read the five sections. Confirm:

- **Front-matter** names system, EU AI Act classification (high-risk), contract version, plan version, sign-off routing including the risk-council chair for a high-risk system, families in scope, and families scoped-out with a one-line justification.
- **Section 2 has one row per signal family**, nine rows total (seven families + the per-slice fairness sibling + the embedding-drift row), each with all fifteen columns populated. No `TBD` cells; `status: draft` is explicit on every row.
- **Every `signal_semantics` sentence is specific** — unit, window, denominator, and the deviation / absolute-rule shape named. "Injection firing rate up" is not acceptable; "5-minute rolling firing rate of the input-side prompt-injection classifier over evaluated requests, computed in the middleware, two-sided deviation" is.
- **Every threshold cites its origin** per chapter 01 §"Cadence and threshold as engineering choices" — statistical band, SLO burn, or absolute rule from the harm model.
- **Every S1 row routes to the named escalation roles** — DPO on PII egress, security + legal on injection + PII, risk-council chair on sustained per-slice regression.
- **Two-sided deviation is on the injection row** per chapter 03 §"Deviation is a two-sided signal" — rate-down with steady volume is explicitly wired as a control-availability case.
- **Per-slice row exists** per chapter 02 §"Per-slice monitoring" — separate contract row, fairness-tagged register row, fairness-liaison routing.
- **`deferral_boundary` cites peer L25 on every row** and peer L30 where the eval loop supplies a calibrated feed.
- **Anti-pattern probe resolves four chapter-01 failures** — no row credits a dashboard, no row is Slack-only, no threshold is unpinned, no row omits the deferral clause.

## Expected output

`mod109-lab01/helpdesk-signal-contract.md` — one populated five-section document; two to three pages. Section 2 (the row set) is the load-bearing artefact; Section 4 (anti-pattern probe) is the one an auditor reads first.

## Checkpoint questions (self-check)

1. A reviewer reads `SC-HD-INJ-01` and asks "the threshold is two-sided, but if the classifier rate drops suddenly to zero when request volume stays constant, what register-side effect fires, and why is that different from the rate-up case?" (*Chapter 03 §"Deviation is a two-sided signal" is explicit. Rate-down with steady volume indicates the control is **unavailable** — the classifier HTTP endpoint is failing open, an upstream dependency is down, or the classifier was silently misconfigured. The register-side effect is `HR-CS-INJ-01.control_status → unavailable` and an incident candidate opens — not `control-defeated` (which is the rate-up residual move). Routing adds the circuit-breaker check to security. The two directions are different events on the same row.*)

2. A reviewer asks why `SC-HD-PII-01` carries two severities (S1 and S2) on the same row. (*Chapter 03 §"PII / sensitive-data leakage — the absolute-rule threshold" pins the shape: any confirmed positive from the labelling queue is **S1 absolute rule**; firing-rate deviation > baseline is **S2** on the same register-row identifier. The two are not alternatives — the S1 branch catches real leaks regardless of rate; the S2 branch measures pressure on the control. Folding them into a single deviation row (chapter-03 §"PII" anti-pattern) would allow one confirmed leak to pass without S1 routing, and a data-protection auditor asks about that exact shape.*)

3. The row `SC-HD-NEAR-01` reads `peer L30 ai-eval-engineer on the labelling protocol depth that drives the FP field`. Why is that boundary on the row, and what breaks on the near-miss counter if the labelling loop is missing? (*Chapter 03 §"The false-positive discipline" and §"The boundary to `ai-eval-engineer`" pin it: near-miss counts are only meaningful if false positives are labelled out of the counter. The counter excludes fires with `false_positive: true` set by the labelling loop. Without the loop, the counter is a guardrail-noise count and the trend is uninformative — the mod-112 digest shows a noisy number the council cannot act on. The **depth** of the labelling review belongs to L30 (LLM-as-judge calibration, human-in-loop protocol, inter-rater reliability); the risk-side hook — the `false_positive` field on the record and the propagation into the counter's read path — is owned at L25 here.*)

4. The `SC-HD-QUAL-02` per-slice row routes to the fairness liaison *and* adds `HR-CS-QUAL-03` to the register effect alongside `HR-CS-FAIR-08`. Why both? (*Chapter 02 §"Per-slice monitoring" says the per-slice row's `register_effect` "points at the fairness-tagged row, not the drift-tagged row, if the affected slice is a protected group" — hence `HR-CS-FAIR-08` as the primary target. The chapter also says a per-slice trip is a leading indicator that the aggregate will move; the row additionally flags `HR-CS-QUAL-03` so the aggregate row's residual is marked "watched" even before the aggregate trips. The fairness liaison is on the routing because a protected-attribute slice regression is a fairness signal, not only a quality signal — the two routings run in parallel.*)

## Why this lab matters

Exercise 01 asks you to produce six artefacts (signal contract file, family coverage table, deferral map, change log, anti-pattern probe, cross-reference note) for your real running system. If the fifteen-column shape is new at exercise-01 authoring time, the budget blows on inventing the row format instead of on which register rows need runtime signals, which monitors supply them, how the thresholds are defended against the harm, and which peer boundaries the auditor will probe. This lab drills the row shape on a fixed fabricated system so that exercise 01's time goes into the engineering judgement that matters. The artefact shape it drills is also what exercises 02 and 03 populate with real detectors, what exercise 04 reduces into the Article 72 report, and what exercise 05 fan-outs through the routing dispatcher. Getting the shape right here cascades through the module.
