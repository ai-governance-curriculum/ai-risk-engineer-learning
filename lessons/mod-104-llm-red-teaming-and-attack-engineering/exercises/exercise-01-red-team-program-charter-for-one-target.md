# Exercise 01 — Red-Team Program Charter for One Target

**Estimated effort:** 2 hours

## Objective

Produce the **five artifacts** chapter 01 defines — target profile, coverage matrix, attack suite (stub), replay bundle (structure only), triage log (empty template) — plus the **cadence table**, the **release-gate thresholds**, and the **harmful-payload-out-of-repo manifest**, for the ONE running AI system you will carry through exercises 02–06. This charter is what the rest of the module populates; every downstream exercise reads from it.

## Prerequisites

- Chapter 01 read; chapters 02–06 skimmed enough to know which suite each cadence row will call.
- The running system chosen. The module README asks you to use the same system you carried through mod-101–103; if that is not viable, pick a fresh one now and document why. It must be a system you can actually invoke (model + prompt + tools + retrieval, as applicable) — a system you cannot run cannot be red-teamed.
- Access to the system's model IDs, prompt commit hash, tool schemas, and retrieval-source list at a pinned release. If any of these are opaque to you, note it as a scoping gap on the deliverable rather than inventing values.
- The mod-102 harm-model rows and the mod-103 register rows for this system. Every coverage-matrix row you claim to cover must trace to at least one of them.
- A location for the **out-of-repo payload store**. A private object bucket, an internal vault, or (for the exercise only) a local directory *outside* the source repo. Do not use the source repo itself.

## Problem statement

Author the charter for your one target in a new subtree `red-team/` of whatever repo you are treating as this system's risk-engineering repo. The charter is *not* the runnable suite — that arrives in exercises 02–06. It is the **contract the runnable suite fills**. When you are done, a reviewer opening `red-team/` should be able to answer, in under twenty minutes:

- *What system, at what version, on which surfaces, at which deployment tier?* (target profile)
- *What technique classes are claimed covered, deferred, escalated, or n/a on which surfaces?* (coverage matrix)
- *Which suites will run, at which commit hash, on which cadence?* (attack-suite stub + cadence table)
- *Where do harmful payloads live, and how does the repo reference them without containing them?* (payload manifest)
- *What is the pass rule for a release, and which register rows does it defend?* (threshold document)
- *When findings arrive, what shape will they take, and where will they be triaged?* (replay-bundle structure + triage-log template)

## Requirements

Produce all seven items below. Missing any is an audit finding on the charter itself.

1. **Target profile** — `red-team/targets/<target-id>.yaml`. Populate every field chapter 01 names: system identifier + version, model / prompt / tool inventory at pinned versions, in-scope surfaces, out-of-scope surfaces (named, not implied), personas modelled (from the mod-102 four-persona list), deployment tier, data-provenance rules for test inputs, and success criteria framed as attack-success-rate ceilings per technique class (leave the specific numbers as `<threshold-tbd>` if you have not negotiated them yet; do not invent).
2. **Coverage matrix** — `red-team/coverage/<target-id>.md`. Rows: the technique classes from chapter 01's technique-class table. Columns: the capability surfaces from your target profile. Every cell must be `covered:<suite-id-placeholder>`, `n/a`, `deferred:<ticket-or-reason>`, or `escalated:L40`. Blank cells are not allowed. Suite-id placeholders can point at the exercise that will produce them (e.g., `covered:ex02-garak-smoke`).
3. **Attack-suite stub** — `red-team/suites/README.md`. One paragraph per suite you intend to compose in the module (garak smoke, PyRIT orchestrator library, AgentDojo / InjecAgent environment run, GCG + many-shot payload set, Inspect-hosted internal suite). Name the tool, the exercise that produces it, and the coverage-matrix cells it will claim. Do not write suite code here — that is exercises 02–06.
4. **Replay-bundle structure** — `red-team/replays/README.md`. A schema (fields, not values) for a single replay bundle per chapter 01's list: payload reference (hash + store URI, never the raw string), provenance, target-profile hash, observed behaviour, expected behaviour, severity, triage disposition. Include one *empty* example file `red-team/replays/EXAMPLE.yaml` with the fields present and every value set to `<tbd>` or a hash placeholder.
5. **Triage-log template** — `red-team/triage/<target-id>/TEMPLATE.md`. Columns per chapter 01's triage-row list: candidate id, suite + run hash, replay-bundle hash, disposition, severity, register-row link, regression-fixture id, owner. The template is empty of rows; exercise 02 files the first real ones.
6. **Cadence table** — inside the target profile or as `red-team/cadence.md`. Rows per chapter 01's cadence table: per-commit smoke, per-release full, quarterly deep, event-triggered. For each, name the trigger, the suite subset (by placeholder id), and the register-row-review trigger in mod-103 it maps to.
7. **Payload-hosting manifest** — `red-team/payloads/MANIFEST.md` (plus `MANIFEST.yaml` if you prefer machine-readable). State the out-of-repo store's location *type* (bucket, vault, filesystem path pattern; do not commit real credentials or paths that leak), the access model (allow-list, access-logged), and the in-repo linking convention (hash + store URI, signed manifest). Include a one-paragraph "why not in-repo" statement — the four reasons chapter 01 gives, in your own words. Add a repo-root check: a pre-commit hook stub or a documented CI check that fails if any file matching a harmful-payload signature lands in the repo tree. The check itself does not need to be armed in this exercise, but its intent must be documented.

Cross-cutting rules:

- **Every coverage-matrix cell marked `covered:*` must trace to at least one mod-102 harm-model row or mod-103 register row.** Add the trace in a footnote table on the coverage matrix. Cells that cannot trace are either miscategorised (fix the coverage) or reveal a gap in the harm model (file it upstream).
- **Every `deferred:*` cell must name either a ticket id or an explicit reason** — "no tool-calling surface exists in v1" is a valid reason; "we'll get to it" is not.
- **Every `escalated:L40` cell must name the frontier-agent depth it hands off** — sub-agent spawning, deep multi-agent planning, capability-elicitation against a novel tool topology, etc. — matching chapter 01 and chapter 06's L40 boundary.
- **The threshold document must be versioned.** Even a stub with `<threshold-tbd>` values carries a version tag and a "last-negotiated" date; the release-gate contract does not float.
- **No harmful strings in the repo.** If you need an example payload to illustrate the structure, use `<harmful-payload-placeholder-sha256:...>` — never the string.

## Starter guidance

- Do the **target profile first**. Everything else keys off it — the coverage-matrix columns come from the in-scope surfaces list; the cadence's per-commit trigger is defined in terms of the model / prompt / tool inventory; the threshold contract references the deployment tier.
- On surfaces, use the capability-surface axis from mod-102 chapter 01 (text generation, retrieval, tool calling, memory / state, multimodal input, output rendering, human-oversight, admin surface). If a surface is absent from your system, mark it `n/a` on the coverage matrix rather than deleting the column — reviewers should see the axis in full.
- On personas, match the mod-102 four-persona list (adversarial user, insider, upstream data supplier, downstream integrator). A programme that only models adversarial-user is under-scoped; write it down if that is the case.
- On the coverage matrix, walk chapter 01's technique-class table row by row and *deliberately* place each cell. Blank cells are an audit finding; a matrix full of `covered:*` on first pass usually means you have not thought hard about what your system actually exposes — most first-pass matrices carry legitimate `n/a` and `deferred:*` entries.
- On the payload manifest, remember that suite-provided payloads (garak's probe corpus, InjecAgent's payload set) live *bundled with the tool*, not in your project repo. Your project repo carries only *hashes and store URIs* for the payloads that are yours (internal-suite additions, replays of confirmed findings). Chapter 01's table on where-payloads-live captures the split.
- On the threshold document, resist writing "no findings" as the pass rule. Chapter 01 is explicit that the rule reads as attack-success-rate ceilings per technique class; the ceilings are register-row-derived. If you do not know the ceilings yet, mark them `<threshold-tbd>` and note which register-row you will read them from.
- On the pre-commit check, chapter 01's discipline is that *cloning the repo should never expose a new engineer to harmful strings*. The check does not need to be sophisticated (a size + entropy + known-hash-list filter is enough as a stub); it needs to *exist and be documented*, so that arming it is a one-line change in exercise 02.

## Acceptance criteria

The submission is accepted when:

- All seven items above exist at the specified paths and are internally consistent.
- The target profile pins system id + version, model + prompt + tool inventory at pinned versions, in-scope + out-of-scope surfaces, personas, tier, and provenance rules.
- The coverage matrix has zero blank cells; every `covered:*` traces to a mod-102 or mod-103 row; every `deferred:*` names ticket or reason; every `escalated:L40` names the frontier-agent depth handed off.
- The attack-suite stub names one suite per exercise 02–06 with the coverage-matrix cells it will claim.
- The replay-bundle README defines the fields; the `EXAMPLE.yaml` has the fields present with placeholder values and no harmful strings.
- The triage-log template has every chapter-01 field and no rows.
- The cadence table has all four cadence rows (per-commit, per-release, quarterly, event-triggered), each mapped to the mod-103 register-review trigger it feeds.
- The payload manifest states store type + access model + in-repo linking convention, has the four-reason "why not in-repo" statement, and documents the repo-root check (armed or stubbed).
- The threshold document exists and is versioned, even if numbers are `<threshold-tbd>`.
- A reviewer given only the `red-team/` subtree can answer, per the six review questions in the problem statement, without asking a follow-up.

## Stretch goals

- Arm the pre-commit or CI check so that a deliberately-introduced test file containing a known harmful-string hash *fails* the check. Include the failing run in the deliverable.
- Draft one **plausible first triage-log row** end to end — using a *fabricated* candidate id and a *placeholder* replay-bundle hash — to prove the schema round-trips. The row must be marked `SIMULATED` and must not contain a real payload.
- Write a one-paragraph **charter-vs-programme note** distinguishing what this exercise produces (the charter, empty of findings) from what exercises 02–06 produce (findings filling the charter). This is the paragraph you show a stakeholder who confuses "we have a red-team programme" with "we have a red-team charter."
- Sketch the **handoff contracts** to `ai-eval-engineer` (L30) and `agentic-safety-engineer` (L40) as one-page each: what each peer consumes from your charter, what each returns, and which coverage-matrix cells sit on the boundary. Chapter 01 and chapter 06 name the boundaries; this stretch writes the interface.

## Deliverable format

A single markdown document `mod-104-ex01-charter.md` that indexes the `red-team/` subtree, plus the subtree itself. The index document contains:

1. *Chosen running system* — one paragraph naming the system, the release under test, and (if different from your mod-101–103 system) why the switch.
2. *Subtree map* — the file tree of `red-team/` with a one-line description per file.
3. *Target profile summary* — the profile fields at a glance; the full YAML lives in the subtree.
4. *Coverage-matrix summary* — the cell-distribution counts (`covered / n/a / deferred / escalated`) and the list of `deferred` and `escalated` cells with their reasons.
5. *Cadence summary* — the four cadence rows and their mod-103 review-trigger mappings.
6. *Threshold-document status* — versioned; numbers or `<threshold-tbd>` markers.
7. *Payload-manifest summary* — store type, access model, linking convention, repo-root-check status (stubbed / armed).
8. *Traceability index* — every `covered:*` cell to its mod-102 / mod-103 row.
9. *Open items* — anything flagged for exercises 02–06 or for upstream mod-102 / mod-103.

## Submission

Attach the index markdown and the `red-team/` subtree archive to the module submission channel. Do **not** attach the payload store contents; the store lives out-of-repo by design and its contents are not part of the deliverable. If the running system is under NDA, redact system identifiers, model IDs, and tool schemas, but preserve the coverage-matrix shape, the cadence table, the threshold-document versioning, and the payload-manifest structure — a reviewer cannot review a charter whose structural fields are blanked out.
