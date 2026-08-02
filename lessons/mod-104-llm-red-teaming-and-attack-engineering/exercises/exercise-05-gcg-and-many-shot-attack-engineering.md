# Exercise 05 — GCG and Many-Shot Attack Engineering

**Estimated effort:** 3 hours

## Objective

Populate the two technique-class rows on your coverage matrix that chapter 05 singles out for special handling — **GCG-style adversarial suffixes** and **many-shot in-context** — with coverage that is *live*, not lexical. For GCG, assemble a hash-referenced replay set against every target on the coverage matrix, and — *only if* an open-weight target is in scope — document the fresh-optimisation cadence. For many-shot, build the demonstration-corpus manifest and run a shots-per-context sweep against every target whose context window admits it. All harmful payloads stay out of the repo per chapter 05's payload discipline; the repo receives manifests, hashes, sweep configurations, and result tables only.

## Prerequisites

- Chapter 05 read; chapters 01 and 03 re-skimmed (coverage matrix from ch. 01; PyRIT converters and payload-store wiring from ch. 03).
- Exercise 01 completed — the coverage matrix from ex01 is the artifact this exercise updates; the two rows *GCG-style adversarial suffixes* and *many-shot in-context* must already be present (as `deferred:ex05` or equivalent) so this exercise can flip them to `covered:<suite-id>`.
- Exercise 03 completed — the PyRIT attack library, its payload-store wiring, and its scorers are the substrate this exercise adds converters and orchestrators to.
- Access to the out-of-repo payload store from ex01 / ex03 (with access logging enabled) and write permission to add new manifest entries.
- The list of targets on the coverage matrix, split into **closed-weight** targets (API-only; no gradients available) and **open-weight** targets (locally hosted or otherwise gradient-accessible). If no open-weight target is in scope, the fresh-optimisation portion below is documented as `n/a` with the rationale — it is not skipped silently.

## Problem statement

Deliver, for the running system, the artifacts that turn the GCG and many-shot rows on the coverage matrix from `deferred` to `covered:<suite-id>` under chapter 05's rules:

1. **GCG replay set.** A hash-referenced manifest of published GCG suffixes (from the Zou et al. paper and public follow-on work cited in `resources.md`), loaded from the out-of-repo payload store by a PyRIT converter, applied to every target on the coverage matrix. Per-suffix ASR and aggregate replay ASR are reported per target.
2. **GCG fresh-optimisation cadence (conditional).** *If* any open-weight target is in scope, the cadence for re-optimising GCG suffixes against the current checkpoint of that target — with the resulting suffixes added to the payload store and included in the replay set as transfer tests against closed-weight targets. If no open-weight target is in scope, this section is `n/a` with a one-line rationale citing the target inventory.
3. **Many-shot demonstration-corpus manifest.** A category-tagged manifest of `(policy-negative ask, model-would-comply response)` demonstration pairs stored in the payload store. Categories match the internal policy categories (mod-102 harm-catalog rows). Only the manifest — IDs, hashes, category tags, provenance — enters the repo; the pairs themselves do not.
4. **Shots-per-context sweep.** A sweep configuration and result table showing ASR per shot count for each in-scope target, plus at least one cross-category test (demonstrations in category A, final-turn ask in category B). The shot counts are chosen against each target's context window; do not invent numbers you cannot execute.
5. **Payload discipline audit.** A short section demonstrating that none of the above artifacts (manifests, sweep configs, result tables, converters) contain a raw suffix, a raw demonstration, or any other harmful string — only hashes, IDs, provenance metadata, and aggregate numbers.

## Requirements

- Output as `mod-104-ex05-gcg-and-many-shot.md`, plus the manifests and converter / orchestrator configuration under a `red-team/` directory following the ex03 layout.
- **GCG replay** — manifest lists every pinned suffix by ID, hash, provenance (paper or public follow-on source), and the target set it is applied to. Per-suffix and aggregate ASR are reported per target. The ASR threshold per target is the register-row target for the corresponding harm category (mod-103 anchor).
- **GCG fresh optimisation** — *if in scope*, document the cadence (chapter 05 anchors this as quarterly), the target checkpoint identifier, the compute footprint estimated (not measured — this exercise does not require running the optimisation), and where the resulting suffixes will land in the payload store. Include a transfer-test plan: closed-weight targets are re-run against the freshly optimised suffixes as replay entries. If out of scope, write `n/a` with the target-inventory rationale.
- **Many-shot corpus manifest** — every demonstration pair has an ID, a hash, a category tag matching a mod-102 harm-catalog row, and a provenance note (curated in-house, drawn from a published research corpus by hash, etc.). The manifest lives in the repo; the pairs do not.
- **Shots-per-context sweep** — the sweep configuration names the shot-count ladder per target (chosen against the target's context window, not invented), the harm categories exercised, the cross-category test at minimum, and the scorer that decides refusal vs. compliance on the final turn (reused from ex03's scorer library). Result table shows ASR per shot count per target.
- **Refusal-robustness threshold** — for each target, name the shot count at which refusal degrades meaningfully against your appetite band, and comment on whether that shot count is reachable in the target's real usage (long documents, chained tool responses, multi-turn history).
- **Payload discipline audit** — a section titled *Payload discipline audit* that lists every file authored under this exercise and asserts (with `grep`-style checks or equivalent) that no raw suffix or demonstration string is present. Include the access-log configuration for reads from the payload store.
- **Coverage matrix update** — the two rows *GCG-style adversarial suffixes* and *many-shot in-context* flip from their prior state to `covered:<suite-id>` on the matrix, with a link back to the manifest and result tables produced here.
- Every metric reported is measured against your target — do not paste numbers from the Zou et al. paper, the Anil et al. work, or any other source as if they were your results.

## Starter guidance

- Do the manifest layout before you populate anything. The manifest is what your reviewer reads; the payload store is what the manifest points at. If the manifest shape is wrong, every subsequent step gets refactored.
- The GCG replay set is the cheap coverage — it does not require gradients on your target. Get it running end-to-end (one suffix, one target, one ASR number) before scaling to the full pinned set.
- The fresh-optimisation cadence is a *documented cadence and plan*, not a run of the optimisation. Running the optimisation is out of scope for a 3-hour exercise; the plan (target checkpoint, cadence, compute footprint estimate, payload-store landing location, transfer-test wiring) is in scope.
- Choose the shot-count ladder against each target's actual context window. If a target's window admits many demonstrations, sweep across a wide range; if it admits few, sweep across what it admits and note the limit. The number is the target's, not yours.
- Cross-category test — pick two mod-102 categories that are policy-adjacent but distinct (e.g., one operational-harm category and one content-harm category). The result tells you whether refusal degradation is shot-count-driven or category-specific.
- Scorer reuse — do not re-invent the refusal/compliance scorer here. Point at the ex03 scorer and reuse it. If the ex03 scorer does not fit the many-shot final-turn shape, note the gap and defer to ex06 (Inspect-based authoring) rather than authoring a new scorer under time pressure.
- Payload discipline is the single hardest requirement. Before you commit anything, `grep` your diff for any string that looks like a suffix or a demonstration; if you see one, the payload discipline has already failed. Fix it before the commit lands, not after.
- If you find yourself wanting to paste a suffix or a demonstration into the deliverable "for illustration," write `<!-- payload-out-of-repo: hash=<sha256>, store=<store-id> -->` in its place. That is what chapter 05 requires.

## Acceptance criteria

The submission is accepted when:

- Both target technique-class rows on the coverage matrix are `covered:<suite-id>` with resolvable links to the manifest and result tables.
- The GCG replay manifest lists every pinned suffix by ID, hash, and provenance, and per-target ASR is reported against the register-anchored threshold.
- The GCG fresh-optimisation section is either a real cadence + plan (if any open-weight target is in scope) or `n/a` with the target-inventory rationale — not silent.
- The many-shot manifest is category-tagged against mod-102 rows, hashes resolve into the payload store, and the pairs are not in the repo.
- The shots-per-context sweep result table shows ASR per shot count per target, includes at least one cross-category test, and names the refusal-robustness shot-count threshold per target with a comment on real-usage reachability.
- The payload discipline audit section demonstrates that no raw suffix or demonstration string appears in any file authored under this exercise, and that payload-store reads are access-logged.
- A reviewer with only the deliverable and the coverage matrix can, in under 20 minutes, answer: which suffixes are pinned, which target they were run against, what the fresh-optimisation plan is (or why it is out of scope), where the many-shot demonstrations live, what shot count breaks refusal against appetite, and whether any harmful payload has leaked into the repo.

## Stretch goals

- Wire the GCG replay set into the release-gate suite from ex01 so it runs on every gate, not only on demand. Report the added wall-clock cost.
- Add a *retired* section to the GCG manifest for suffixes that once transferred but no longer do after a provider safety-tuning update, per chapter 05's rotation discipline. Preserve them (archived, flagged) rather than deleting, so historical measurements remain reproducible.
- Extend the cross-category many-shot test to a full category × category matrix (demonstrations in each category, final-turn ask in each other category) — even a small matrix reveals whether refusal degradation is category-specific or generalises.
- Sketch the detector-logic hand-off to mod-109: what signal on a production request (long context, high demonstration-shaped structure) would flag as a candidate many-shot attempt? Note this as an interface for mod-109 to pick up, not as a detector you build here.
- Draft the incident-response fixture shape (mod-111 interface): if a confirmed finding from this suite produced a regression fixture, what does it pin — the suffix hash, the demonstration-corpus hash, the shot count, the scorer version?

## Deliverable format

A single markdown document with the following sections, plus the manifest and configuration artifacts under `red-team/`:

1. *Target inventory* — the closed-weight vs. open-weight split; whether fresh-optimisation is in scope.
2. *GCG replay* — manifest reference, per-target and aggregate ASR table, threshold per target from the register anchor.
3. *GCG fresh-optimisation* — cadence, target checkpoint, compute-footprint estimate, payload-store landing, transfer-test wiring; or `n/a` with rationale.
4. *Many-shot corpus manifest* — reference to the manifest, category coverage against mod-102 rows, provenance summary.
5. *Shots-per-context sweep* — sweep configuration, result table, cross-category test, refusal-robustness threshold per target with reachability comment.
6. *Payload discipline audit* — the grep-style checks, the access-log configuration, the assertion that no raw suffix or demonstration is in the repo.
7. *Coverage matrix update* — the diff on the two rows, from prior state to `covered:<suite-id>`.
8. *Open questions* — anything flagged for exercise 06 (Inspect-based authoring) or for the mod-108 / mod-109 / mod-111 interfaces.

## Submission

Attach the markdown and the `red-team/` artifact tree to the module submission channel. If the running system is under NDA, redact target identifiers but preserve manifest shape, ASR tables, sweep configuration, and the payload discipline audit — a reviewer cannot review this exercise from a deliverable whose measurements are blanked out. Under no circumstances attach raw suffixes, raw demonstrations, or any other harmful payload to the submission; if the submission channel does not itself enforce the payload discipline, this exercise's whole point has been lost.
