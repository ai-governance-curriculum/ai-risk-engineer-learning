# Exercise 03 — PyRIT Attack Library and Orchestrator

**Estimated effort:** 3 hours

## Objective

Build a Microsoft PyRIT attack library for the running system that carries the multi-turn and converter-chained depth the garak smoke suite in exercise 02 cannot express. Compose the six PyRIT roles — targets, prompt converters, attack strategies, orchestrators, memory, scorers — into a named, versioned, reproducible library, and calibrate every scorer credited in the release-gate pass rule against a labelled fixture set before the library is considered shippable.

## Prerequisites

- Chapter 03 read.
- Chapter 01 skimmed — the harmful-payload-out-of-repo discipline applies here without exception.
- Exercise 01 completed — the programme charter (target profile, coverage matrix, threshold, cadence) is the input this exercise consumes.
- Exercise 02 completed — the garak smoke suite is already running and its triage workflow is the one PyRIT results will feed.
- Access to the running system chosen in exercise 01 (same target profile, same environment, same credential path).
- An out-of-repo payload store (from exercise 01) that already holds any DAN-style, GCG-style, or category-specific harmful strings by hash. If it does not exist yet, stand it up before starting.
- A separate attacker model endpoint you can call for the multi-turn adaptive strategy — an internal instruction-following model whose only job is to try adversarial framings. It must not be the target.
- A small labelled fixture set of `(prompt, response, label)` triples for each scorer you plan to credit — see the scorer calibration requirement below.

## Problem statement

Produce a PyRIT attack library for the running system, laid out per chapter 03's file layout (`red-team/suites/pyrit/…`), that composes at least the following named attacks:

- One **single-turn** attack using a PyRIT target + scorer pair without converters — proves the target and scorer wiring end-to-end.
- One **converter-chained** attack using at least two converters in a chain (e.g., an encoding converter and a persona wrapper). The chain must be *data*, not baked into the attack logic — swapping one converter for another produces a new attack with no other code changes.
- One **multi-turn adaptive** attack using an attacker-model target and an objective string, with a turn limit and a full-transcript scorer. The objective must be tied to a category from your coverage matrix.

Each attack composes a target, a converter chain (possibly empty), an attack strategy, an orchestrator, a scorer, and writes to memory. Attacks are separate modules; converters, scorers, and targets are separate modules; a new attack is a composition of existing pieces.

Wire two orchestrators: a **smoke** orchestrator (cheap subset, deterministic seeds, runs alongside garak-smoke) and a **release-gate** orchestrator (full library, adaptive strategies to their turn limits, calibrated scorers, full transcripts captured). Both write to the PyRIT memory store; the memory store lives out-of-repo.

## Requirements

- Output as `mod-104-ex03-pyrit-library.md` plus the library itself under `red-team/suites/pyrit/` in a repository you can share with a reviewer.
- The directory tree matches chapter 03's layout: separate modules under `library/`, `converters/`, `scorers/`, `orchestrators/`, `targets/`, and a `config/` for the smoke and release-gate configurations.
- The target is a **build function** imported by every attack, not a class re-instantiated per attack. A change to the target definition propagates uniformly.
- At least **three named attacks** as above (single-turn, converter-chained, multi-turn adaptive). Each attack is a module that exports a build function returning an orchestrator instance.
- At least **two converters** authored under `converters/`, used in at least one attack chain. Converters that load harmful strings do so from the out-of-repo payload store, by hash, at runtime.
- At least **two scorers** authored under `scorers/`, of *different shapes* from chapter 03's scorer table (substring / structured match, classifier, LLM-as-judge — pick any two). Each scorer credited in the release-gate pass rule carries calibration evidence — see below.
- The **smoke** orchestrator config names a small, deterministic-seeded subset of the library and runs against the same CI trigger as the garak smoke suite from exercise 02. Wall-clock and API-cost budgets are documented on the config.
- The **release-gate** orchestrator config runs the full library with adaptive strategies to their turn limits and writes full transcripts to memory. It runs per release candidate, not per commit.
- Every attack row is present on the coverage matrix from exercise 01, either as `covered:pyrit:<attack-id>` or as an update to an existing row that was previously `deferred`.
- Findings from a release-gate run feed the same triage workflow exercise 02 walked for garak. Attach one worked triage of a real PyRIT candidate (any disposition — confirmed, false-positive, duplicate-of-garak, deferred).
- **Scorer calibration discipline (non-negotiable):** for every scorer credited in the release-gate pass rule, produce a calibration record before the library is called shippable. For classifier scorers, calibration is precision / recall on a held-out labelled sample of your system's actual outputs. For LLM-judge scorers, calibration is agreement with a human-labelled sample using an inter-rater metric (e.g., Cohen's kappa). Substring / structured-match scorers require a documented false-negative check (a small sample of paraphrased successful attacks that the substring rule misses). The calibration record is pinned to the scorer's release and referenced by hash from the release-gate config. A scorer without a calibration record can seed triage but cannot gate the release alone.
- **Harmful-payload discipline:** no harmful prompt, no attacker-model raw output, no DAN framing, no GCG suffix, and no harmful string generated during a run lives inside the source repo. Converters reference payloads by hash; the PyRIT memory store lives out-of-repo under the same access controls as the payload store. The repo carries pointers and hashes only.
- The library's release version is pinned to a specific PyRIT release tag; the tag is recorded in the config alongside every measurement.

## Starter guidance

- Author the target build function first. Every attack imports it. Get one single-turn attack running against it end-to-end before you write any converters, scorers, or multi-turn strategies — this is the fastest way to surface target-wiring problems.
- Write the substring / structured-match scorer next — it is the cheapest to calibrate and its false-negative check is quick to author. Use it to prove the memory store, scorer plumbing, and triage log integration before you invest in an LLM-judge scorer.
- Only after the single-turn end-to-end path is green, add the converter chain. Chain composition surfaces its own bugs (converter ordering, prompt-template escaping) that are easier to diagnose in isolation.
- The multi-turn adaptive attack is the piece scanners cannot express — write it last, and expect the attacker-model objective string to need several iterations. If the attacker model refuses its own task, either the objective is over-broad or the attacker model is over-safety-tuned for red-team use.
- Calibrate scorers against fixtures drawn from *your* system's outputs, not from published benchmarks. A scorer's precision on HarmBench outputs is not evidence of its precision on your target's outputs.
- Do not invent scorer thresholds. If your calibration says the LLM-judge agrees with humans at kappa 0.42, that is what you write down — a low number is a real finding, not a scoring bug to hide.
- If a converter needs a harmful payload (GCG suffix, DAN framing), commit the *hash and pointer*, not the payload. If you find yourself pasting a payload into a test, stop — write the payload-store loader first.
- The smoke config exists to run alongside garak-smoke, not to duplicate it. Its job is coverage of *your* converter chains and multi-turn shapes — not another pass at published single-turn probes.

## Acceptance criteria

The submission is accepted when:

- The `red-team/suites/pyrit/` tree exists in the shape chapter 03 describes, with separate modules for converters, scorers, targets, orchestrators, and attacks.
- At least three named attacks are runnable: one single-turn, one converter-chained (chain length ≥ 2), one multi-turn adaptive with a turn limit and full-transcript scorer.
- The target is a build function imported uniformly; changing the target definition in one place propagates.
- At least two converters and at least two scorers of different shapes are authored and used.
- The smoke orchestrator runs on the same CI trigger as the garak smoke suite; wall-clock and cost budgets are documented and respected.
- The release-gate orchestrator runs the full library with adaptive strategies to their turn limits and writes full transcripts to the memory store.
- Every scorer credited in the release-gate pass rule has a calibration record pinned to its release, referenced by hash from the config. Uncalibrated scorers are marked as triage-only and cannot fail the build alone.
- Every harmful payload, attacker-model raw output, and generated harmful string lives out-of-repo; the repo carries hashes and pointers only.
- The coverage matrix from exercise 01 is updated so every PyRIT attack shows as `covered:pyrit:<attack-id>` on the matching row.
- One worked triage of a real PyRIT candidate is attached to the deliverable, with a disposition and a register-row link (or an explicit "no register-row impact" justification).
- The PyRIT release tag is pinned and recorded alongside every measurement.
- A reviewer can, from the repo alone, reproduce a smoke run and a release-gate run against the same target, and can point at the calibration record for every scorer that gated a release-gate finding.

## Stretch goals

- Add a **second converter chain** to the converter-chained attack that swaps one converter for another with no other code change. Show the two attacks side by side to make the composition point concrete for a reviewer.
- Add a **tree-of-attacks-shaped** multi-turn strategy alongside the linear adaptive one — same target, same scorer, different attacker-model driving pattern. Report the delta in success rate and wall-clock cost.
- Wire the PyRIT memory store into the same triage log that exercise 02 built for garak, so a triage reviewer sees garak candidates and PyRIT candidates in one queue with a `source:` field.
- Add a **regression-fixture emitter**: on any confirmed release-gate finding, auto-generate an Inspect / PyRIT single-turn fixture that reproduces the attack deterministically. Chapter 06 and exercise 06 will consume these; seeding the emitter now closes the loop early.
- Prototype an **attacker-model swap**: run one multi-turn adaptive attack with two different attacker models and report the delta. This is the cheapest way to show that the attacker-model choice is itself a measurement variable.

## Deliverable format

A single markdown document with the following sections, plus the library repository itself:

1. *Target profile carried forward* — one paragraph naming the target and the environment, and confirming the exercise-01 target profile is unchanged.
2. *Library layout* — the directory tree, with a one-line purpose per file.
3. *Named attacks* — one short section per attack: strategy shape, target, converter chain, scorer, orchestrator, memory-store pointer, coverage-matrix row it satisfies.
4. *Scorer calibration records* — one entry per credited scorer: shape, fixture-set size and provenance, calibration metric, result, pinned release hash. Uncalibrated scorers are listed here explicitly as triage-only.
5. *Orchestrators and CI wiring* — smoke config (trigger, budget, cost) and release-gate config (trigger, cadence, transcript retention).
6. *One worked PyRIT triage* — the candidate, the five-step walk, disposition, register-row link.
7. *Payload and memory-store discipline* — pointer to the out-of-repo payload store; pointer to the memory-store location; the access-control note.
8. *Coverage-matrix delta* — the rows that moved from `deferred` or `garak-only` to `covered:pyrit:<attack-id>`.
9. *Stretch goals attempted* — the notes.

## Submission

Attach the markdown and a tarball / git bundle of the `red-team/suites/pyrit/` tree to the module submission channel. If your running system is under NDA, redact identifiers but preserve the directory layout, the scorer calibration records, the orchestrator configs, and the coverage-matrix delta — a reviewer cannot review a PyRIT library whose composition, calibration, and coverage are blanked out.
