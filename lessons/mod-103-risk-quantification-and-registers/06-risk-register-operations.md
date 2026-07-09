# Register Operations: Fields, Cadence, Ownership, Sign-Off; GRC-for-AI Platforms and the Spreadsheet Fallback

Chapters 01-05 defined what a scored risk row *is*. This chapter says what it takes to run a register *as an operation* — who owns rows, on what cadence they refresh, how sign-off is routed, how the register lives in a GRC-for-AI platform (Credo AI, Holistic AI, ModelOp, Monitaur) or in a spreadsheet fallback when a platform isn't available. The register is not a document; it is a live artifact with a cadence and an audit trail.

## Motivation

Two failure modes for a register that has the right schema:

- **Stale rows.** The register was populated once at project kickoff and hasn't been touched since. Residual scores reflect the world at authoring time; auditors reading a six-month-old register see a system that has changed and a register that hasn't. Finding: "risk-management process is not operating on a defined cadence."
- **Ownership fog.** A row lists an owner in prose. When the row's residual moves outside appetite, nobody is on the hook to act. Finding: "controls exist but responsibility for their operation is not assigned."

The operational discipline in this chapter — fields, cadence, ownership, sign-off, evidence trail — is what turns a schema from chapters 01-05 into a live risk-management practice.

## Fields the row must carry to be operationally live

Chapter 03 gave the schema for the *scoring* content. Below are the *operational* fields the row also needs. These are what turn a static score into a piece of running work.

```yaml
# Operational fields (add to the chapter 03 schema)

# Ownership
row_author_role: ai-risk-engineer-L25
row_author_person: <named individual>
system_owner_role: system-owner
system_owner_person: <named individual>
control_owner_roles:                        # per control credited on residual
  CTL-CS-RAG-PROV-01: platform-eng-lead
  CTL-CS-RAG-ALLOWLIST-01: platform-eng-lead
  CTL-CS-RAG-MON-01: sre-oncall
mitigation_owner_role: ai-risk-engineer-L25
mitigation_owner_person: <named individual>

# Cadence
review_cadence: quarterly                   # or event-triggered (see below)
last_reviewed: 2026-06-15
next_review_due: 2026-09-15
review_type_last: scheduled                 # or event-triggered

# Sign-off
sign_off_routing:
  - role: ai-risk-engineer-L25
  - role: system-owner
  - role: security-risk-partner (27005 side)
  - role: risk-council (enterprise) [conditional if above appetite]
sign_off_status: current
sign_off_history:
  - version: v3
    date: 2026-06-15
    signatories: [ai-risk-engineer-L25, system-owner, security-risk-partner]

# Evidence trail
evidence_manifest:
  - control_id: CTL-CS-RAG-PROV-01
    evidence_type: guardrail-effectiveness-report
    location: evidence/ctl-cs-rag-prov-01/2026-q2/report.json
    hash: sha256:...
  - control_id: CTL-CS-RAG-ALLOWLIST-01
    evidence_type: release-config-hash
    location: releases/2026-05-22/allow-list.yaml
    hash: sha256:...
change_log_ref: register-changelog-2026-q2.md
```

These fields are boring on their own. The failure of any one is an audit finding, so the fields are shown out in the open.

## Cadence: scheduled and event-triggered

A register review has two triggers.

### Scheduled cadence

Every row has a *default* review cadence. Rows do not all review at the same rate — riskier rows review more often. A working tiering:

| Row profile | Default cadence |
|---|---|
| Residual well below appetite; no active monitors firing | Annual |
| Residual near appetite; monitors active | Quarterly |
| Residual at or above appetite (treatment in progress) | Monthly during treatment; return to quarterly on treatment closure |
| Rows attached to a system in an active change window (new release, new use case, new integration) | Ad-hoc plus scheduled |

The register file carries `next_review_due` and reporting drops flagged rows that are past their due date onto an exception list.

### Event-triggered cadence

Some events force a review regardless of the scheduled cadence:

- **System change** — a new capability surface, a new tool, a new user population, a new jurisdiction added → all rows for that system are re-reviewed for exposure and taxonomy tags.
- **Model change** — a model swap, a fine-tune, a prompt change → rows depending on the model recompute residual likelihood; suite runs (mods 104-108) refresh.
- **Control change** — a control's config changes, a monitor is muted, a guardrail model is swapped → the credited-controls list on affected rows updates and control-defeated variants are re-scored.
- **Incident or near-miss** — an incident post-mortem names an unmodelled or under-scored row → the row is corrected and a change-log entry filed.
- **Framework or standard update** — NIST AI RMF revision, ISO/IEC 25059 or 23894 amendment, EU AI Act delegated act, sector overlay revision → coverage grid is re-checked and rubric versions bump.
- **Base-rate shift in the incident corpus** — a class of scenario that was Rare is now Possible in the incident corpus → likelihood anchors update.
- **Red-team or eval finding** — a mod-104 or mod-105 run produces a novel finding → new rows are added and existing rows have their likelihood re-anchored.

Each event source has a wiring plan in the register operations doc. If an event source has no wiring, its updates arrive silently and the register drifts.

## Ownership contract

Each named field points at a named role and, ideally, a named person on the row at review time. Roles are ISO/IEC 42001-shaped (or your equivalent management-system role set) and reflect the ladder (mod-101 ch 06, mod-102 ch 06).

Three ownership contracts sit on every row:

- **Row author** — the risk engineer who created and maintains the row. Owns row correctness and evidence quality.
- **System owner** — the deployment-side accountable person. Owns the system's operating decisions and accepts residual risk.
- **Control owner(s)** — the operating teams for each credited control. Own control availability and evidence generation.

None of the three should be the same person on a system operating at any real scale. If two overlap (a small deployment where the risk engineer is also the system owner), *record that fact*; do not silently collapse.

## Sign-off routing

Sign-off is the moment a version of the row is *accepted* as the current position on the risk. Every substantive change to a row triggers a sign-off round; every scheduled review either produces a sign-off (if unchanged) or triggers a change round.

A defensible routing pattern:

| Trigger | Signatories |
|---|---|
| Row create | row author + system owner + security-risk partner (27005 side) |
| Residual state change (control credited or dropped) | row author + system owner + affected control owner |
| Residual moves above appetite | above + risk council for tolerance decision |
| Control-defeated exceeds appetite AND monitor cannot catch defeat inside tolerable window | above + risk council + affected control owner escalation |
| Framework version change | row author + system owner (rubric-version acknowledgement) |
| Incident-driven change | row author + system owner + incident-response owner |

Sign-off history is *append-only*. Old sign-offs are not overwritten; they are versioned. The audit reads the history to confirm the row's current sign-off matches the current row content.

## The GRC-for-AI platforms

Four platforms commonly referenced at level-25 maturity are Credo AI, Holistic AI, ModelOp, and Monitaur. Each represents a slightly different point in the space between AI governance, MLOps, and enterprise ERM.

At the level of *what a risk engineer does with them* rather than a vendor comparison, the pattern is the same:

| Function of the platform | What you use it for |
|---|---|
| System / model inventory | The intake record for each system (mod-102 ch 06); the anchor for register rows |
| Policy / obligation catalogue | Framework citations (NIST RMF, ISO 42001, EU AI Act, sector overlays) attached to the system |
| Risk / control catalogue | The register rows and their control credits; the taxonomy tags |
| Evidence collection | Evidence-manifest pointers (control reports, monitor snapshots, release configs) |
| Workflow / sign-off | Reviewer routing, cadence enforcement, sign-off audit trail |
| Reporting / dashboards | The aggregation views from chapter 05 |
| Integrations | MLOps / CI hooks that stream test results and monitor data in |

Whatever the platform, the risk engineer is responsible for the *content* landing in it. The platform enforces workflow; you enforce data quality.

### What to look for when selecting or configuring a platform

At level-25 scope, the platform-selection conversation is above your role (owned by `senior-ai-governance-architect` at L50 or `head-of-ai-governance` at L60). Your input to that conversation should focus on the operating capabilities that make the register defensible:

- Does the platform's row schema carry inherent / residual / control-defeated distinctly, or does it collapse to a single score? If the latter, you will need to represent the three states in row prose or an escape-hatch field.
- Does the platform version the rubric and the row together, or does a rubric update silently re-score existing rows?
- Can the platform export the register to a structured format (JSON / YAML / CSV) for the aggregation reports in chapter 05, or does it lock you into its built-in reports?
- Does it integrate with the enterprise ERM / 27005 register, or does it silo the AI register (chapter 03's anti-pattern)?
- Does it support the sector overlays your program spans (SR 11-7 attributes, EU AI Act article citations, FDA GMLP fields, Colorado AI Act fields)?
- Are its integrations with your MLOps and monitoring stack (mod-109) live, or are they promised?

The exercise (exercise 05) walks the population step on a chosen platform (or the spreadsheet fallback) and the reporting export.

<!-- needs-research: any specific platform capability claims (Credo AI, Holistic AI, ModelOp, Monitaur) should be verified against the current published product documentation at authoring time; feature sets change per release. -->

## The spreadsheet-shaped fallback

Many organisations at level-25 maturity do not yet have a GRC-for-AI platform in place, or the AI register is being piloted before platform selection. In that case the register is a set of files (YAML / CSV / a spreadsheet) in a versioned repo. This is a legitimate operating position at this maturity level — audits will accept it if the *operating discipline* is intact.

A working file layout for a spreadsheet-shaped register:

```
risk-register/
  README.md                            # register operating doc
  rubric/
    scoring-rubric-v1.3.md             # chapter 02 rubric
    scoring-matrix-v1.3.md             # ordinal-matrix definition
    appetite-bands-v2.md               # enterprise appetite
  taxonomy/
    iso-25059-characteristics.md       # applicable quality characteristics
    nist-rmf-measure-subcats.md        # cited sub-categories
    iso-23894-risk-sources.md          # cited risk sources
  systems/
    cs-rag-assistant-01/
      intake-ref.md                    # pointer to intake record inv-2025-014
      quality-profile.md               # 25059 characteristics in scope
      rows/
        HR-CS-INJ-01.yaml
        HR-CS-DRIFT-02.yaml
        ...
  aggregation/
    reports/
      2026-q2-heat-map.md
      2026-q2-above-appetite.md
      2026-q2-control-defeated.md
      2026-q2-shared-component.md
      2026-q2-coverage-grid.md
      2026-q2-jurisdiction-exception.md
    scripts/
      build-reports.py                 # stateless reduction over rows
  change-log/
    register-changelog-2026-q2.md
  sign-off/
    2026-q2-signoff-records.md
```

Points to note about this layout:

- Rows are YAML files, one per row. That is what makes the aggregation reduction (chapter 05) run automatically.
- Rubric, taxonomy, and appetite are versioned files. When they change, git records both the diff and the reviewer.
- Sign-off records live *in the repo*, not in an email thread.
- The change log is the register-scale one; each row's `change_log_ref` points at it.
- The `build-reports.py` script is a stateless reduction. Running it should be a one-line command in CI.

The spreadsheet fallback is not a downgrade of the GRC platform; it is the *first-order* form of the register. If the platform ever ingests the register, it ingests from this file set.

## Interfaces with adjacent operations

- **Mod-101 ch 01 evidence pipelines** — every credited control's evidence pointer is a link into the mod-101 five-column translation. If a control's evidence link is stale, the credit lapses.
- **Mod-108 (guardrails)** — control creation, versioning, and effectiveness measurement land as evidence. The register is where those numbers become credits on residual scores.
- **Mod-109 (continuous monitoring)** — monitors are the wire from control-defeated to signal. Monitor status is a row field.
- **Mod-110 (MRM and supply chain)** — the shared-component report from chapter 05 feeds the supply-chain review. Rows tied to third-party components carry supplier-context fields.
- **Mod-111 (incident response)** — post-mortem findings that name unmodelled rows create incident-triggered register updates; residual scores may move in response to a validated near-miss.
- **Mod-112 (cross-functional AI risk program and communication)** — the risk-council packet is the mod-112 output; the aggregation reports from chapter 05 are what fills the packet.

## Register hygiene checks

Add these checks to CI so they run whenever a row file changes:

- Every row's `next_review_due` is not in the past unless the row is flagged.
- Every credited control has a live evidence pointer and the pointer resolves.
- Every row's `iso_25059_characteristics` and `nist_ai_rmf_measure` are populated with valid identifiers.
- Every row's `sign_off_status` matches the last row-content change (if content changed since last sign-off, status is `stale`).
- Every row's `taxonomy_tags` fields are non-empty (per mod-102 ch 03's discipline).
- Every above-appetite row has a `treatment` field with a due date.
- No row references a component with `depends_on_components` that isn't in the shared-component catalogue.

CI failing on any of these means the register is not shippable. The discipline is not optional; the audit reads the register as if it were.

## Anti-patterns

- **The living-document without a diff.** Rows are edited in place; you cannot tell what changed. Fix: put rows in git or in a platform with row-versioning; either way, diff-able.
- **The sign-off email chain.** Sign-off exists in a mailbox somewhere. Fix: sign-off records are register-repo artifacts, or GRC-platform audit-log rows.
- **The permission-drift.** Everyone can edit any row; no ownership is enforced. Fix: `row_author_role` gates edits; system-owner sign-off gates residual state changes.
- **The frozen taxonomy.** Rubric v1.0 was defined at project kickoff; two years and three framework releases later, no version bump. Fix: version the rubric with the framework it cites; bump on framework update.

## Summary

- The register carries operational fields — ownership, cadence, sign-off, evidence trail — alongside the scoring content from chapters 01-05.
- Cadence is both scheduled (per-row default) and event-triggered (system change, control change, incident, framework revision, base-rate shift, red-team finding).
- Ownership sits on three roles per row — row author, system owner, control owner — never silently collapsed.
- Sign-off is versioned, append-only, and routed by the change type.
- The register lands in a GRC-for-AI platform (Credo AI, Holistic AI, ModelOp, Monitaur) or in a spreadsheet-shaped fallback that is a proper first-class file-and-CI operation.
- Register hygiene checks in CI keep the operational fields clean; failing checks make the register non-shippable.
- Exercise 05 walks the population and export on either a platform or the fallback.
