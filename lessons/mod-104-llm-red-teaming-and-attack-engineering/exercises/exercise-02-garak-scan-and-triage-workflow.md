# Exercise 02 — garak Scan and Triage Workflow

**Estimated effort:** 3 hours

## Objective

Stand up the garak smoke suite against the running target from exercise 01, wire it into CI on the cadence you set there, run it, and walk chapter 02's five-step triage on the real candidates it produces. The output is a passing (or defensibly-failing) smoke run, a triage log with dispositioned rows, register updates against the mod-103 rows the findings touch, and — where a true positive is fix-shipped inside the exercise window — a regression fixture that references its payload by hash rather than embedding it.

## Prerequisites

- Chapter 02 read; chapter 01 available as reference for the five artifacts and the harmful-payload-out-of-repo discipline.
- Exercise 01 completed. You will consume the target profile (generator wiring), the coverage matrix (which probe families you claim), and the cadence / threshold decisions (what the CI gate checks and how often).
- The running system you have carried through mod-101–103 is reachable from the CI runner (or from your workstation acting as a stand-in runner) via whatever generator shape the target profile named — REST endpoint, provider SDK, or local HuggingFace model.
- A garak install pinned to a specific release version; note the version in the suite config. <!-- needs-research: recommend a specific minimum garak release once the module authoring team has picked a floor. -->
- The out-of-repo payload store from exercise 01 is writable — replay bundles and regression fixtures land there, not in the repo.
- Access to the mod-103 register (from mod-103 exercise 05) so triaged true positives can update real rows.

## Problem statement

Compose the smoke suite as chapter 02 describes — generator, probe set, detector set, seed — for your target. The suite has to satisfy three constraints simultaneously:

1. It runs in single-digit minutes on the CI runner, so it can gate every commit that changes the model, the prompt, the retrieval index, or the tool schema.
2. It exercises *at least one probe per probe family* that your exercise-01 coverage matrix claims as `covered` under garak. A family with zero probes running is a coverage lie.
3. It emits a JSON / JSONL report your gate script can read against a versioned thresholds file, and it exits non-zero when a threshold is breached.

Run the suite against the target, take the candidates it produces, and walk them through the chapter 02 triage — dedup within run, false-positive review, dedup across runs, severity + register attachment, regression fixture. Every candidate that leaves triage lands somewhere: a disposition line in the triage log, a row update in the register, or a fixture reference in the out-of-repo store.

## Requirements

- Output as `mod-104-ex02-garak-smoke-and-triage.md` plus the following artifacts committed to the red-team repo:
  - `red-team/suites/garak-smoke/config.yaml` — the smoke-suite config: generator (as named in the target profile), probe list (one per claimed family), detector list, seed, pinned garak version.
  - `red-team/suites/garak-smoke/thresholds.yaml` — per-probe hit-rate ceilings the gate enforces. Ceilings are argued from the target's deployment tier and appetite band, not chosen to make the first run pass.
  - `red-team/tools/garak_gate.py` (or the equivalent in your language) — reads the garak report, compares against thresholds, exits non-zero on breach. Small; testable; versioned with the suite.
  - CI configuration wiring the two above into the pipeline your exercise-01 cadence named — per-commit for model / prompt / retriever / tool changes, at minimum.
  - `red-team/triage/<target-id>/<run-id>.md` — the triage log for this run, one row per unique candidate after step 1 dedup, with the disposition columns from chapter 02.
- The smoke suite is pinned to a specific garak release; the release version is stored in the config and echoed in the report header. Bumping garak is a change-controlled event, not an incidental upgrade.
- The probe list matches the coverage matrix. If the matrix claims a family under garak, the smoke suite exercises it; if the smoke suite skips a family, the matrix cell moves to `deferred:<reason>` or `escalated:L40`. The two must not disagree.
- The detector list is annotated by kind — regex / classifier / LLM-judge — so triage step 2 knows which false-positive discipline applies per row.
- Run the suite at least once against the live target. The run identifier (commit SHA or timestamp) and the raw report are referenced from the triage log; the raw report itself is stored per your repo's convention (typically not in-repo if it contains harmful strings — reference by hash to the out-of-repo store).
- Every candidate emitted by the run is dispositioned in the triage log with:
  - `probe`, `detector`, `detector-kind`, `response-fingerprint`, `disposition` (TP / FP / duplicate / accepted / mitigated), and — for TPs — `severity`, `harm-scenario-ref` (mod-102 row), `register-row-ref` (mod-103 row), `replay-bundle-hash`, `mitigation-owner`.
  - Duplicate rows cite the prior finding they collapse into.
  - False positives cite the detector-kind-specific reviewer disposition from chapter 02's table (regex negative-example, classifier owner note, LLM-judge calibration set).
- Payload discipline: no harmful prompt strings, no harmful model responses, and no full raw reports containing either land in the repo. Every such artifact lives in the out-of-repo payload store and is referenced by hash. The triage log rows and the register updates refer to payloads by hash only.
- Register attachment: every novel TP updates at least one mod-103 register row — likelihood-anchor evidence at minimum, control-effectiveness evidence where the finding demonstrates a guardrail defeat. Where no register row matches, open one via the mod-103 chapter 06 process and cite the new row's ID.
- Regression fixture: for any TP that is fix-shipped inside the exercise window, add a fixture row that references the replay bundle by hash and would fail the release-gate suite if the fix regressed. The fixture is stored per chapter 02's mod-111 handoff (payload by hash, not embedded).
- The CI gate is demonstrated in *both* modes: a passing run on the current build, and a deliberately-breached run (raise a threshold, or introduce a probe that will hit, or point at a stub target that fails) proving the gate exits non-zero. Screenshots or console captures are attached.

## Starter guidance

- Wire the generator first. If the target profile says REST, prove a one-probe end-to-end run against the live endpoint before you expand the probe list. Auth and network issues will burn more time than probe selection will.
- Choose probes cheap-first inside each family. Chapter 02 explicitly reserves depth for the release-gate suite and for PyRIT / Inspect; the smoke suite is broad and shallow by design.
- Thresholds: do not pick them to make the first run pass. Argue each ceiling from the deployment tier and the appetite band from mod-103. If the first run breaches a ceiling, that is the register update the exercise is asking for.
- Seed the run. A `--seed` value in the config makes the run reproducible; the same `(generator, probe-set, detector-set, seed)` tuple has to produce the same finding set for the triage log to mean anything.
- Fingerprint response text before you dedup. Whitespace, model-generated IDs, and timestamps make near-identical responses look distinct; normalise them out before you hash.
- Do false-positive review with the detector kind in front of you. A regex hit and an LLM-judge hit demand different evidence to overturn; conflating them is how false-positive review devolves into vibes.
- Do not open a new register row for every TP. Chapter 02's step 3 dedup runs *across runs*; step 4 opens a new row only when the response-fingerprint is novel *and* no existing row already frames the scenario.
- The out-of-repo payload store is not optional. If your only writable store today is a local directory, that is fine for the exercise — document it and note the production-hosting gap on the deliverable.

## Acceptance criteria

The submission is accepted when:

- The smoke-suite config, thresholds, gate script, and CI wiring are committed and reference a specific pinned garak release version.
- The probe list covers every garak-claimed family on the exercise-01 coverage matrix, and any gaps are reflected as `deferred:<ticket>` or `escalated:L40` on the matrix (both sides consistent).
- One live run against the target is captured with a run identifier; the raw report is referenced (by hash to the out-of-repo store if it contains harmful strings).
- Every candidate from that run has a disposition in the triage log using the chapter 02 columns, including detector-kind-driven FP dispositions.
- Every novel TP is attached to a mod-102 harm-scenario row and a mod-103 register row, with the register update visible; new register rows opened during the exercise are cited by ID.
- Fix-shipped TPs (if any inside the window) have a regression fixture referencing the payload by hash, ready for the mod-111 fixture store.
- No harmful prompt or response strings are committed to the repo — grep of the diff for the payload store's prefix, and a spot-check of the triage log, confirm hash references only.
- The gate is demonstrated in both passing and breaching modes.
- A reviewer with no prior context can, from the deliverable, answer: which probe families ran, which candidates fired, which of those became register updates, which became regression fixtures, and which were dispositioned FP with what evidence.

## Stretch goals

- Wire the smoke-suite hit-rate trend into a small dashboard (or a scheduled report) that mod-109 can consume. Chapter 02 names this as the monitoring seam; this stretch closes it.
- Add a *nightly full-suite* job that runs the broader garak probe library with the tuned (classifier / LLM-judge) detectors and compare its finding set against the smoke suite's on the same run. The delta is the argument for what the smoke suite is *not* catching.
- Draft the garak-version-bump runbook: what re-baselining looks like when the pinned garak version moves, which thresholds are permitted to shift, and who signs off. This is the change-controlled discipline chapter 02 names.
- For one TP where the detector was an LLM-judge, generate a small judge-calibration set (a handful of positive and negative examples with human labels) and hand it off with a named reference to the `ai-eval-engineer` L30 judge-calibration methodology.
- Contribute one novel probe upstream — an attack pattern your triage surfaced that garak's library does not yet exercise. Contribution is optional; drafting the probe in `red-team/suites/garak-smoke/local-probes/` is enough to satisfy the stretch.

## Deliverable format

A single markdown document with the following sections, plus the artifacts named in Requirements:

1. *Target and generator* — one paragraph naming the target (from exercise 01), the generator shape used, and any auth / network scaffolding notable enough for the reviewer to know.
2. *Suite composition* — the probe list per family, the detector list annotated by kind, the seed, and the pinned garak version. Cross-reference the coverage matrix.
3. *Thresholds and gate* — the per-probe ceilings and the argument for each, plus the gate script's pass rule.
4. *Run summary* — run identifier, timing, candidate count before and after step 1 dedup, and any operational notes (probe timeouts, generator flakiness).
5. *Triage log* — the dispositioned rows as described in Requirements. Payloads by hash only.
6. *Register updates* — the list of mod-102 harm-scenario rows and mod-103 register rows touched, with links / IDs. New rows opened during the exercise are called out.
7. *Regression fixtures* — the fixtures created (if any) with their payload hashes and the release-gate suite entry they were added to.
8. *Gate demonstration* — the passing run and the breaching run, with the console / screenshot evidence.
9. *Payload-discipline audit* — the check that confirms no harmful strings landed in-repo (command run, output).
10. *Stretch goals attempted* — the notes.

## Submission

Attach the markdown, the red-team repo diff (or a link to the PR), the CI run URLs for the passing and breaching demonstrations, and the triage log to the module submission channel. If the target is under NDA, redact system identifiers but preserve the probe list, detector kinds, thresholds, disposition columns, register-row IDs, and fixture hashes — a reviewer cannot review a garak triage whose dispositions and register attachments are blanked out.
