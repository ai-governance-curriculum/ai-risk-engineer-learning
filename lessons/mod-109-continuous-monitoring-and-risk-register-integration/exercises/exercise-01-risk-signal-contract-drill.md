# Exercise 01 — Risk-Signal Contract Drill

**Estimated effort:** 2 hours

## Objective

Draft the **signal contract** for the running system carried since mod-103. Produce one row per `(monitor, register-field)` pair, with every one of the fifteen required columns filled or explicitly deferred. The output is the artifact chapters 02, 03, 04, and 05 populate — this exercise is the empty-contract drill that binds each later exercise to a concrete row set.

## Prerequisites

- Chapter 01 read in full. The fifteen-column schema, the "monitor vs. dashboard" test, the cadence / threshold discipline, and the `deferral_boundary` pattern are the reference shapes.
- The mod-103 register for the running system (from mod-103 exercises 01 and 05). Every signal contract row cites a `register_row_id` that must exist in that register.
- The mod-102 harm-model persona / capability-surface list. It is the source for which harms need runtime signals, and therefore for which register rows need signal contract rows.
- The mod-108 guardrail inventory (from mod-108 exercises 01 and 05). Guardrail firing rates are one of the upstream signal sources for the safety-monitor family (chapter 03).
- A pointer to the peer L25 (`ai-infra-mlops`) contact who owns the monitoring pipeline. The `signal_source_owner` and `deferral_boundary` columns cite them by role.
- A pointer to the peer L30 (`ai-eval-engineer`) contact for signals sourced from the app-eval online loop.

## Problem statement

Author the signal contract for the running system as a versioned artifact. Every register row whose residual depends on a runtime signal gets at least one contract row. Every contract row has every one of the fifteen columns from chapter 01 filled, or the row is a `draft` and is called out as such.

### The row shape

Chapter 01 §"What a signal contract is" defines the columns. For reference:

`contract_id`, `register_row_id`, `register_field`, `monitor_id`, `signal_source_owner`, `signal_semantics`, `cadence`, `threshold`, `severity`, `register_effect`, `alert_routing`, `owner_on_call`, `escalation_path`, `evidence_pointer`, `deferral_boundary`.

Rows are versioned as a set (the contract has a semantic version); every row also carries a `sign_off` block (row author + system owner + control owner) mirroring mod-103 chapter 06.

### The row set

Draft at least one contract row for each of the seven signal families named in the module:

1. Model / data drift.
2. Quality regression.
3. Safety-monitor (guardrail firing rate, jailbreak, prompt-injection).
4. PII / sensitive-data egress detection.
5. Near-miss cumulative counts.
6. Cost anomalies.
7. Latency anomalies.

If a family does not apply to the running system, write a one-sentence justification in the contract's front-matter (not a per-row `TBD`). Auditors ask for the negative-space justification too.

For each row, the `deferral_boundary` column cites either `ai-infra-mlops` (L25) on the pipeline plumbing or `ai-eval-engineer` (L30) on the app-eval online loop, or both, per chapter 06's boundary tables.

### The contract front-matter

Above the row set, the contract has a front-matter block naming:

- The system this contract covers, keyed to the mod-102 intake identifier.
- The contract's semantic version and change-log pointer.
- The sign-off routing (row author, system owner, control owner, and — for high-risk-classified systems under mod-101 chapter 04 — the risk council chair).
- The plan version this contract is bound to (the Article 61 / 72 plan, if applicable; chapter 04).
- The list of families in scope and the justifications for any family scoped out.

## Requirements

Produce all six items:

1. **Signal contract file** — `risk/mod-109/<system-id>/signal-contract.md` (or `signal-contract.yaml` if your register is machine-readable). Contains the front-matter block plus the row set. Every row has all fifteen columns populated. Draft rows are marked `status: draft` with the missing columns named explicitly; do not leave silent blanks.
2. **Family coverage table** — `risk/mod-109/<system-id>/signal-contract-coverage.md`. Two-column table: family name × contract rows drafted. If a family is scoped out, the cell holds the justification and cites the row of the mod-103 register that would otherwise carry the signal.
3. **Deferral map** — `risk/mod-109/<system-id>/signal-contract-deferrals.md`. Per contract row, the `deferral_boundary` clause quoted, plus the named contact and the last-touched date. This is what the auditor reads when they ask "who owns the pipeline for this signal."
4. **Change log** — `risk/mod-109/<system-id>/signal-contract-changelog.md`. Every row insertion, deletion, and threshold change is a log entry with a reviewer. Even the initial draft is v0.1 with an "initial draft" log entry.
5. **Anti-pattern probe** — `risk/mod-109/<system-id>/signal-contract-anti-patterns.md`. Four short sections, one per chapter-01 anti-pattern:
   - **Ad-hoc dashboard credited as a monitor.** Confirmed every row satisfies all five conditions (signal / threshold / cadence / routing / evidence). Any row that fails is called out.
   - **Silent Slack-only alert.** Confirmed every row has both a first responder *and* a register-side updater. No signal reaches only the on-call.
   - **Unpinned threshold.** Confirmed every threshold has a defended origin (statistical band, SLO burn, absolute rule) recorded on the row or in the evidence pointer.
   - **Missing deferral clause.** Confirmed every row cites the L25 / L30 boundary where applicable.
6. **Cross-reference note** — `risk/mod-109/<system-id>/signal-contract-cross-refs.md`. Two subsections:
   - *Register rows without a signal contract row.* List every mod-103 register row whose residual depends on a runtime observation and does *not* have a contract row. Each entry is either a fix in exercise 02 / 03 or a justification (residual credited to a static control, not a runtime signal).
   - *Contract rows without a register row.* List every contract row that does not join to a mod-103 register row. Each entry is either a fix in the register (add the row) or a justification (signal is monitored for engineering reasons, not risk-register reasons — file the row as `status: informational`).

## Starter guidance

- **Start from the register, not from the monitors.** The contract's purpose is to bind register rows to signals. Walking the mod-103 register row-by-row and asking "which runtime observation would move this row's residual" produces a shorter and more defensible row set than starting from "which monitors do we have."
- **A `TBD` column is a draft row.** Do not fake completion by filling in placeholders. Chapter 01 §"What a signal contract is" says a row missing any column is a draft; keeping the draft-vs-shipped distinction visible is what makes the contract signable.
- **The `signal_semantics` sentence is the one auditors read.** "Prompt-injection detector firing rate up" is not a signal semantics; "5-minute rolling firing rate of the input-side prompt-injection classifier over evaluated requests, computed in the middleware" is. Write the sentence with unit, window, and denominator.
- **Cadence choices are defended per row.** Chapter 01 §"Cadence and threshold as engineering choices" names the four cadence families (streaming with window, sub-hourly batch, daily batch, weekly rolling). Pick the shortest cadence whose sensor cost you can defend; do not default to daily because it is convenient.
- **Thresholds must have a citation.** Statistical thresholds cite the drift-metric convention (PSI band, KS alpha, MMD kernel + test-set size); SLO thresholds cite the SLO document; absolute-rule thresholds cite the harm-model row. `> 0.2` is not a threshold; `> 0.2 PSI on the top-20 features, band from Siddiqi + validated against this system's reference window` is.
- **The `register_effect` column is deterministic, not narrative.** "Residual state moves to `control-defeated`, treatment flag opens with a 5-day due date" is deterministic. "Investigate and update the register" is not.
- **Per-slice rows are separate rows.** Chapter 02 §"Per-slice monitoring" and mod-106 discipline both push against folding per-slice signals into the aggregate row. If the system has protected-attribute slices, draft a per-slice row alongside the aggregate row for at least drift and quality regression, and cite mod-106 on the fairness liaison in `alert_routing`.
- **`draft` is the correct status when a signal source is not yet owned.** If the pipeline for a signal is not built yet, keep the row as `draft` with a `blocked_on` note pointing at the MLOps issue. Do not delete the row — the draft is what tells you what to build.
- **The contract is repository-versioned.** Chapter 01 §"Versioning and change control" pins this. Do not treat the contract as a Notion page or a slide. It is a file in the risk-engineering repo with diffs and reviewers, alongside the register.

## Acceptance criteria

The submission is accepted when:

- Every register row whose residual depends on a runtime observation has at least one contract row (or an explicit note explaining why not, in the cross-reference file).
- Every one of the seven signal families is either represented by at least one row or scoped out with a written justification.
- Every contract row has all fifteen columns populated; any that cannot yet be populated is marked `status: draft` with the missing columns named.
- Every row's `deferral_boundary` cites the L25 or L30 peer where applicable.
- The anti-pattern probe resolves each chapter-01 anti-pattern with a named row-level check.
- The change log carries at least the initial draft as v0.1, with a reviewer.
- The contract is committed to the risk-engineering repo alongside the register and referenced from the mod-103 register's operational fields.

## Stretch goals

- **Contract-linter script.** Write a small script (`risk/mod-109/<system-id>/lint-contract.py` or equivalent) that reads the signal contract file and reports any row missing a required column, any row whose `register_row_id` does not resolve, any row without a `deferral_boundary` where the signal source is external, and any row whose threshold field is missing a defended citation. The linter runs in CI and blocks merges of contract changes that fail.
- **Register-side back-link.** In the mod-103 register file, add a `signal_contracts` field per row listing the contract identifiers that feed it. This makes the two-way lookup trivial and closes the "orphan signal, orphan register row" anti-pattern by construction.
- **Historical residual overlay.** Take the last 90 days of tripped-signal records (if the pipeline exists) and overlay them on the register's residual-state history. If a residual moved without a tripped signal, or a signal tripped without a residual move, that is a bug in either the pipeline or the contract; file it as a fix.
- **Sign-off dry run.** Circulate the drafted contract to the row author, system owner, and control owner named in the front-matter and collect a written sign-off (or a written objection). This is the discipline chapter 01 pins; running the dry run before exercise 02 is what makes exercise 02 land.

## Deliverable format

A single markdown document `mod-109-ex01-signal-contract-drill.md` that indexes the artifacts. The index contains:

1. *System and register summary* — one paragraph naming the running system, the mod-102 intake record, and the mod-103 register version.
2. *Contract front-matter* — quoted verbatim from the contract file.
3. *Row set summary* — a family × row-count table, with pointers to the full contract file.
4. *Deferral map excerpt* — the deferral clauses for the S1-severity rows, quoted verbatim.
5. *Anti-pattern probe report* — the four chapter-01 sections resolved.
6. *Cross-reference note* — orphan register rows and orphan contract rows.
7. *Change log* — the v0.1 entry and any subsequent entries.
8. *Sign-off state* — pending vs. signed per named reviewer.
9. *Open items* — any row still `draft`, with the `blocked_on` reference.

## Submission

Attach the index markdown plus the six artifacts. The signal contract file is the load-bearing artifact for the module; exercises 02, 03, 04, and 05 all read from it. Treat it as the artifact you would hand a market-surveillance inspector — because that is exactly what it is.
