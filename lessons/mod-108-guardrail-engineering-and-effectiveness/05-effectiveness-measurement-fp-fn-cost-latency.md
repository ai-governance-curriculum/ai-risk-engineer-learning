# Effectiveness Measurement — TP / FP / TN / FN with Cost and Latency Accounting

Every prior chapter shipped controls. This chapter *measures* them. The output is a per-slot, per-adversarial-category table of true positive, false positive, true negative, and false negative counts on a bounded adversarial set — with cost and latency accounting per call and a false-positive / false-negative report a product manager can read. The residual numbers in the mod-103 register cannot credit any control that has not been measured this way.

## Motivation

Two failure modes an auditor spots:

- **"We shipped the guardrail" without a measurement.** The residual score in the register credits the guardrail, but the row cannot answer "what's the false negative rate on the injection payload set?" The credit is unsupported; the residual number is unsafe to defend.
- **Measurement without cost.** The classifier layer is measured — TP / FP / TN / FN are reported. But nobody has measured the cost per call or the p95 latency added to the request path. Six months later, the classifier is a large fraction of the cloud bill and the p95 latency is over budget and the product team is asking "why is this here?" without an answer they can weigh.

Both fail because the measurement contract was not written. This chapter writes it.

## The measurement contract

The measurement produces one row per (slot, adversarial category, control instance). Each row carries:

| Field | What it holds |
|---|---|
| `slot` | Input / retrieval / tool / output / refusal-shaping |
| `control_instance` | Named control (e.g., "Llama Guard 3 output, threshold-taxonomy-A", "Presidio credit-card recogniser, retrieval path, threshold 0.65") |
| `category` | Adversarial category from the mod-104 harmful-payload workflow (e.g., "prompt injection — direct", "prompt injection — indirect", "PII exfiltration", "disallowed content — self-harm") |
| `TP` | Adversarial payload correctly blocked / flagged |
| `FP` | Legitimate payload incorrectly blocked / flagged |
| `TN` | Legitimate payload correctly allowed |
| `FN` | Adversarial payload incorrectly allowed |
| `precision` | TP / (TP + FP) |
| `recall` | TP / (TP + FN) |
| `mean_cost_per_call` | Currency per call at production pricing (self-hosted GPU-hour amortised, or hosted-API price per call) |
| `p50_latency_ms` | Median added latency per call |
| `p95_latency_ms` | 95th-percentile added latency per call |
| `n_adversarial` | Number of adversarial payloads in the set for this category |
| `n_legitimate` | Number of legitimate payloads in the set for this category |
| `set_version` | Version identifier of the mod-104-hosted adversarial set + the legitimate-payload set |
| `control_version` | Pinned version of the control (model checkpoint, threshold, recogniser config) |
| `run_id` | Run identifier for evidence / reproducibility |

Every field is load-bearing. `set_version` and `control_version` in particular make the row reproducible; without them the number is not defensible against a later "we can't reproduce this" audit finding.

## The bounded adversarial set

The adversarial set is *not* built inside this chapter. It is hosted by the mod-104 harmful-payload workflow — one canonical, versioned adversarial-payload corpus per adversarial category, shared across the track. Rules for consuming it:

- **Pin the version.** Every measurement cites the set version. When the set updates, re-run.
- **Do not extend it privately.** Extensions go back to the mod-104 workflow so every module in the track measures against the same corpus. Silent private extensions produce numbers that cannot be compared to the peer eval track's numbers.
- **Legitimate set matches.** For every adversarial category, the mod-104 workflow hosts a *legitimate* payload set of comparable size and shape. Measuring only against adversarial payloads produces recall without precision — you cannot report FP without the legitimate set.
- **Bounded.** The set is designed to be run end-to-end in a bounded time budget (single-digit minutes on the classifier layer, single-digit hours on the full stack including model calls). If it grows beyond that budget, the mod-104 workflow prunes; do not fork.

For the residual-risk update, the numbers that go into the register come from a *specific* run against a *specific* set version and *specific* control versions. Store the run manifest alongside the register row.

## Cost accounting

Cost per call must be reported in the same currency as the register's financial-impact bands (mod-103) so the risk council can weigh guardrail spend against the risk reduction it buys.

The three cost components:

- **Compute cost.** Amortised GPU-hour cost for a self-hosted classifier; per-token or per-call pricing for a hosted classifier; CPU cost for regex / Presidio.
- **Operational cost.** Model registry storage; log ingestion for the trace signal; on-call time for the classifier drift monitoring runbook. Amortised across the volume the classifier serves.
- **Third-party licence.** Where a vendor's licence terms include per-seat, per-endpoint, or per-throughput pricing separately from compute.

For each control, produce mean cost per call. Roll up per slot per day and per month. That per-month roll-up is the number chapter 06's build-vs-buy matrix uses.

## Latency accounting

Latency added by a control is measured *in the request path* — not on the classifier's own timing. Two disciplines:

- **Wall-clock measurement in the app.** Wrap the control invocation in a timer in the middleware; log the timer. p50 and p95 are computed on this log, not on the classifier's server-side timing.
- **Parallelism where safe.** Input filters that need not sequence with each other run in parallel (Llama Guard input classification and Presidio input scan can run in parallel with the model dispatch if the model call is speculatively started and cancelled on rejection — but do this only if the compute cost of speculative cancellation is acceptable). Report both the *serial* and *parallel* p95 numbers so the operator can pick the shape.

The latency budget per slot is set by the application team. Slots that exceed budget need a smaller model, a threshold move that lets the cheaper check short-circuit more traffic, or a re-architecture. Do not silently accept over-budget latency; escalate it as a treatment plan.

## The FP / FN report a product manager reads

The measurement rows are for engineers. The product manager gets a shaped report. The shape:

- **One page.** Not fifty rows of numbers. One page per surface (customer-support assistant, developer copilot, HR bot, etc.) if the org has several.
- **Top three false-positive categories.** With example legitimate payloads that get blocked, and the estimated volume per day that hits this false-positive.
- **Top three false-negative categories.** With example adversarial payloads that pass, the harm class from the harm model this represents, and whether any monitor (mod-109) catches downstream.
- **Coverage summary against OWASP LLM Top 10 and the harm model.** Categories fully covered (residual inside appetite), partially covered (residual outside appetite but treatment planned), uncovered (open work).
- **Cost roll-up per surface per month.** Compute + operational + licence.
- **Latency roll-up per surface at p50 and p95.**
- **Trend arrows.** Compared to the previous run: better / worse / unchanged, with confidence intervals or "not significant."
- **Decisions requested.** Explicit list of decisions the product manager must make — accept an FP rate, accept an FN rate, sign off on a spend increase, agree to a latency budget breach, agree to a scope reduction.

The report is versioned. It is filed as evidence for MEASURE 2.3 / MEASURE 2.7 (mod-101 ch 01) and as an ISO/IEC 42001 management-review input.

### Skeleton (per surface)

- **Surface:** Customer-support RAG assistant, revision `2026.03.14`.
- **Adversarial set:** `mod-104/adversarial-v0.9.2`; legitimate set `mod-104/legit-v0.9.2`.
- **Top three FPs.**
  - Category "PII — internal ticket ID": 8.2% FP; example legitimate payload "customer referenced ticket TKT-01234567 in the transcript"; ~150 events/day estimated.
  - Category "prompt injection — direct": 3.1% FP; example legitimate payload "the user asked us to ignore the previous invoice line because it was billed in error"; ~40 events/day estimated.
  - Category "disallowed content — self-harm": 2.4% FP; example legitimate payload "the user described a wellness question about sleep aids"; ~15 events/day estimated.
- **Top three FNs.**
  - Category "prompt injection — indirect": 4.7% FN; example adversarial payload "instructions embedded in retrieved KB article"; harm class `HR-CS-INJ-01`; monitor caught downstream at tool-invocation slot in the last two audited incidents.
  - Category "PII — customer account number": 1.9% FN; example missed pattern "spaces between digit groups the current regex does not match"; harm class `HR-CS-PII-04`; no downstream monitor; treatment: add pattern variant.
  - Category "disallowed content — legal advice out of scope": 6.1% FN; harm class `HR-CS-OOS-02`; treatment: add ShieldGemma custom policy.
- **Coverage.** OWASP LLM01, LLM06, LLM10 covered inside appetite; LLM04 (data poisoning) partially covered (retrieval-side scan is not designed for this — mod-110 owns the supply-chain shape); LLM09 (misinformation) uncovered — open work escalated to risk council.
- **Cost.** $18,400 / month (compute $12,200; operational $4,600; licence $1,600).
- **Latency.** Input p50 82 ms, p95 210 ms; output p50 140 ms, p95 460 ms. Output p95 is over the 400 ms budget; treatment: parallelise Llama Guard and ShieldGemma output checks (spec change).
- **Trends.** FN on indirect injection down from 7.3% to 4.7% (retrieval-side ShieldGemma policy added last run); FP on internal ticket ID up from 5.6% to 8.2% (new template introduced in customer transcripts confuses the recogniser).
- **Decisions requested.** (1) Accept FP on internal ticket ID or fund recogniser tuning; (2) sign off on parallelising output classifiers (adds ~$800 / month compute); (3) agree to treatment plan for LLM09.

That report is what a product manager can read. It is also what an auditor consumes as evidence.

## Per-slot residual-risk update into the register

Each measurement row updates one or more mod-103 register rows. The update pattern:

- **Residual score credit.** A row scored residual on the assumption that "output classifier catches disallowed self-harm content" now has a measured number for that credit. If the measured recall is high enough that the credit was justified, the residual number stands. If not, the residual number moves up.
- **Control-defeated calibration.** The control-defeated score for "output classifier down" is a hypothetical without measurement. With measurement, the score for "output classifier down but Presidio still up" or "output classifier down but rule layer still up" is now a specific number based on the recall the rule layer provides on that adversarial category on its own.
- **Change log.** Every register update names the measurement run that supports it. When the run version changes, the change log rolls forward.

The peer eval track (`ai-eval-engineer`, level 30) may run additional statistical calibration on these numbers (bootstrap CIs, judge-vs-human agreement). Reference their calibration in the register row rather than duplicating it.

## Anti-patterns

Four anti-patterns specific to effectiveness measurement:

- **Adversarial-only measurement.** Recall without precision. Ship the legitimate set alongside every adversarial category or the number is misleading.
- **Unpinned versions.** A number that cannot be reproduced next week is not evidence. Pin the set version and the control version.
- **Cost and latency measured in the classifier, not in the app.** Server-side timing on the classifier ignores network cost. Measure at the middleware.
- **Report the number, hide the tradeoff.** Telling the product manager "false-positive rate 5%" is meaningless without example FPs and a volume estimate. The FP / FN report shows the tradeoff explicitly.

## Cross-references

- **Mod-103 ch 01** — the residual and control-defeated columns are what the measurement rows feed. Without measurement, both columns are unsupported.
- **Mod-104** — hosts the adversarial and legitimate sets. This module consumes; it does not fork.
- **Mod-105** — supplies robustness thinking for how the numbers may degrade under distribution shift or attack transfer; measurement should be re-run after any shift in the attack surface.
- **Chapter 06** — the cost and coverage rows here become the build-vs-buy matrix input.
- **Mod-109** — the classifier verdicts feed monitor signal; the drift of the measured recall over time is what mod-109 monitors on the guardrail side.
- **Mod-111** — an incident that discovers an FN case (harm that a guardrail should have caught) triggers a fixture back-feed into the mod-104 adversarial set and a re-run.
- **Peer level-30 `ai-eval-engineer`** — receives the measurement contract for downstream statistical calibration; do not duplicate their bootstrap CI methodology here.

## Summary

- Every control has a measurement row: slot, category, TP / FP / TN / FN, precision, recall, cost, latency, pinned set version, pinned control version, run id.
- The adversarial set and the legitimate set both come from the mod-104 harmful-payload workflow; do not fork.
- Cost and latency are measured in the request path in the same currency and time units as the app's budgets, then rolled up to the surface's monthly view.
- The FP / FN report is one page per surface, names the top three FPs and FNs with example payloads, reports coverage and cost and latency and trends, and lists decisions requested.
- Every measurement run feeds a residual-risk update into the mod-103 register with a change-log entry and a run-manifest link.
- Anti-patterns: adversarial-only measurement, unpinned versions, wrong-place cost / latency measurement, tradeoff-hiding reports.
- Exercise 05 produces the report on your running system and back-feeds the register.
