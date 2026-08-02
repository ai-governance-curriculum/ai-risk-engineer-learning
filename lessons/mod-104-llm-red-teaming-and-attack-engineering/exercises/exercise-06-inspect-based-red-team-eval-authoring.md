# Exercise 06 — Inspect-Based Red-Team Eval Authoring and Level-Ladder Handoff

**Estimated effort:** 2 hours

## Objective

Author the policy-derived internal red-team suite for the running system as Inspect (UK AISI) `Task` files, integrate HarmBench and AIR-Bench 2024 as reference eval-sets against the same target, and sign the two level-ladder handoff contracts named in chapter 06 — the app-side safety measurement contract to `ai-eval-engineer` (L30) and the frontier-agent depth boundary to `agentic-safety-engineer` (L40). This is the terminal exercise for the module; it closes the reference-eval rows on the coverage matrix and the two handoff seams.

## Prerequisites

- Chapter 06 read; chapters 01-05 read and their exercises completed on the same target system.
- Exercise 01 outputs available — target profile, coverage matrix, threshold and cadence, replay-bundle layout, triage log.
- Exercise 02 garak smoke suite wired in and its results-format known (you will compose with it).
- Exercise 03 PyRIT attack library and scorer-calibration evidence available.
- Exercise 04 AgentDojo / InjecAgent suite and any tool-topology-specific environments available.
- Exercise 05 GCG replay set and many-shot corpus (out-of-repo; referenced by hash).
- An installed and version-pinned Inspect release. Note the release version alongside every measurement.
- A local or remote HarmBench install (behaviour set + attack corpus + classifier) — outputs land in the out-of-repo payload store; only run metadata and per-category scores commit.
- A local or remote AIR-Bench 2024 install (prompt corpus + judge configuration) — same payload discipline.
- Access to the target model endpoint under the same configuration the release-gate suite will use — same system message hash, same tool topology, same guardrail stack.
- Access to the internal policies that constrain the target's outputs and actions (acceptable-use, content, tool-use, jurisdiction / product carve-outs).

## Problem statement

Produce three artifacts against the running system:

1. **The Inspect-authored internal policy-derived suite** — a directory of Inspect `Task` files, one per policy category, walking chapter 06's six steps: enumerate categories from policy → define per-category pass rule → author or curate prompts → author scorers (with calibration evidence from chapter 03's discipline) → wire onto Inspect → put on the release gate.
2. **Reference eval-set integration** — a HarmBench run and an AIR-Bench 2024 run against the target's release-gate configuration, with results-only committed and payloads referenced by hash. Each reference run produces a comparability claim against published numbers for the target model where available, or an explicit note where it is not.
3. **The two signed handoff contracts** — the L30 app-side safety measurement contract to `ai-eval-engineer` and the L40 frontier-agent depth boundary to `agentic-safety-engineer`, each in the shape chapter 06 names.

## Requirements

- Output as `mod-104-ex06-inspect-and-handoff.md` plus the Inspect suite directory (in-repo) and the reference-run result artifacts (in-repo results, out-of-repo payloads).
- The internal suite:
  - At least **six** policy categories authored, each as a separate Inspect `Task`. Every category cites the policy clause it derives from.
  - For each category: the pass rule, the prompt corpus (surface-form, encoded, multi-turn refusal-bypass, persona-hijack difficulty classes represented across the suite), the scorer selection with calibration evidence pinned, and the per-category ASR ceiling that will gate the release.
  - Every task version-pinned in-repo; every dataset that carries harmful strings loaded by hash from the out-of-repo payload store; no harmful strings committed to the suite repo.
  - Cross-walk against AIR-Bench 2024's category taxonomy — every AIR-Bench category not covered by the internal suite is either dispositioned as `n/a` with a reason or logged as `deferred:<ticket>`. This closes the category-coverage gap chapter 06 names.
  - Suite runs on the same invocation in dev, CI, and release-gate. Include the invocation.
- The reference runs:
  - HarmBench run on the target's release-gate configuration, per-category ASR reported, published numbers for the target model cited for comparability where available. Note the HarmBench release version and classifier version alongside the numbers.
  - AIR-Bench 2024 run on the same configuration, per-category scores reported. Note the AIR-Bench version and judge-model configuration.
  - Both runs are re-runnable — the run configuration is committed; the payloads are not.
- The handoff contracts:
  - **L30 (ai-eval-engineer) contract** — names the split chapter 06 defines: you author suite composition, scorer selection at the suite level, harmful-payload discipline, and produce per-category ASR; L30 owns app-side release-gate metrics, statistical framing (CIs, judge calibration studies, agreement metrics), and scorer *methodology-of-methodology*. Contract lists the artifacts you hand over, the artifacts L30 hands back (calibration flags, advisory-only scorer notices), and the escalation loop when a scorer's calibration is questionable.
  - **L40 (agentic-safety-engineer) contract** — names the coverage chapter 06 defines: you cover static tool topology, enumerable retrieval sources, single-agent or one-hop delegated systems, indirect injection through named surfaces, published GCG-suffix replay (plus fresh optimisation against open-weight targets you deploy), many-shot at your context lengths, and PyRIT / Inspect / garak-shaped tooling; L40 picks up dynamic tool composition, multi-agent orchestration with credential passing, browser-driving agents against unbounded live web, novel-attack research, capability-elicitation on frontier models with new scaffolding, frontier context-length pushes, and RSP-informed dangerous-capability evaluations. Contract lists which coverage-matrix cells are `escalated:L40`, the escalation ticket references, and the state each cell is in.
- Update the coverage matrix (exercise 01) — every internal-suite category becomes a `covered:<inspect-task-id>` cell; every reference-eval category maps to `covered:<harmbench>` or `covered:<air-bench-2024>`; every L40-scope row becomes `escalated:L40` with the ticket link.

## Starter guidance

- Do steps 1-2 (enumerate categories, define pass rules) *before* touching Inspect. The chapters are clear that the suite is a translation of policy into tests; if you cannot state the pass rule in one sentence per category, you do not have a testable claim yet.
- Author one `Task` end-to-end — dataset loader, solver, scorer, calibration pin, invocation — before scaling to the rest. The first task will surface every dataset-loading, payload-storage, and scorer-plumbing decision you will otherwise repeat wrong six times.
- Keep harmful payloads out of the repo without exception. Store them by hash in the payload store from chapter 01; the Inspect `Dataset` loads them by reference. If a payload leaks into the repo, roll back the commit — do not attempt to redact in-place.
- On scorer selection, prefer substring / classifier scorers where the pass rule is crisp; reserve LLM-judge scorers for rubric-based policy categories where the language is inherently judgement-shaped. Every LLM-judge task carries its calibration evidence (agreement with a human-labelled sample, per chapter 03) pinned alongside the task file.
- For HarmBench, run a subset of the attack corpus first (direct + one adaptive) to confirm the classifier's behaviour on your model's output style before committing to a full run. If the classifier is out of distribution for your model, mark the score advisory-only and note it in the reference-run write-up.
- For AIR-Bench, do the category cross-walk against your internal suite before you report scores. The cross-walk is the primary value of the run; the scores are secondary.
- Do not invent the L30 or L40 contract terms. Chapter 06 gives the split explicitly — copy the shape into the contract and fill in the target-specific references.
- If a policy category cannot be tested because the pass rule is not writable (a policy that says "be helpful" is not testable as-is), file that as a policy-authoring finding upstream, not as a suite gap.

## Acceptance criteria

The submission is accepted when:

- The Inspect suite has at least six versioned `Task` files, one per policy category, each with dataset loader, solver, scorer, and per-category ASR ceiling; every task runs and reports.
- No harmful strings are committed in the suite repo; every payload reference resolves to the out-of-repo store by hash.
- Every task's scorer has calibration evidence pinned; every LLM-judge task carries agreement-with-human evidence.
- The AIR-Bench 2024 category cross-walk is complete — every AIR-Bench category has a disposition (`covered`, `n/a` with reason, or `deferred:<ticket>`).
- HarmBench and AIR-Bench 2024 reference runs execute against the target's release-gate configuration; per-category numbers are reported with release / classifier / judge versions; a comparability claim (or an explicit "no published comparator available") is written for each.
- The internal suite runs with the same invocation in dev, CI, and release-gate; the invocation is committed.
- The L30 contract is signed and covers artifact-handover, artifact-return, and the scorer-calibration-questionable escalation loop.
- The L40 contract is signed and names the specific coverage-matrix cells escalated, with ticket references.
- The coverage matrix (exercise 01) is updated so every cell resolves to `covered:<suite-id>`, `n/a`, `deferred:<ticket>`, or `escalated:L40`. No cell is blank.
- A reviewer given the deliverable can, in under 30 minutes, answer: what does the internal suite cover, how does it compose with garak / PyRIT / AgentDojo, how do the reference runs compare against published numbers, and where does the L25 programme's scope stop.

## Stretch goals

- Wire the mod-111 regression-fixture store onto the same Inspect substrate — take one confirmed finding from exercise 02, 03, 04, or 05's triage log and author it as an Inspect `Task` that is expected to *pass* (the fix holds). Chapter 06 names this seam explicitly.
- Draft the release-gate composition — the single command that runs the garak smoke, the PyRIT attack library, the AgentDojo suite, and the Inspect internal suite into a single results format the release-gate metric consumer reads. This is the artifact L30 will wrap with statistical framing.
- Author a *policy-drift* check — a scheduled job that diffs the current internal-policy document against the last-authored suite version and opens an issue when a policy clause has moved but no suite task has moved with it. This is what keeps the suite a live policy translation, not a snapshot.
- Sketch the mod-108 (guardrails) hand-back — for one category where the internal suite ASR exceeds the ceiling, name the guardrail addition or tuning that would close the gap, and pre-write the suite-addition that would confirm the guardrail's effect.

## Deliverable format

A single markdown document with the following sections, plus the suite directory and the reference-run artifacts:

1. *Target and configuration* — one paragraph naming the target, the release-gate model configuration, the Inspect release version, and the HarmBench / AIR-Bench versions used.
2. *Internal suite* — the six-plus categories, per-category pass rule, prompt-corpus difficulty-class coverage, scorer selection, calibration evidence pointer, ASR ceiling. One table row per category.
3. *AIR-Bench 2024 category cross-walk* — every AIR-Bench category with a disposition and (for `covered`) the internal task ID.
4. *HarmBench reference run* — per-category ASR, published-comparator numbers where available, classifier-calibration note.
5. *AIR-Bench 2024 reference run* — per-category scores, judge-model configuration, category coverage note.
6. *Invocation* — the one command the suite runs under in dev / CI / release-gate.
7. *L30 handoff contract* — the artifacts you hand to `ai-eval-engineer`, the artifacts they hand back, the scorer-calibration escalation loop, signatories and date.
8. *L40 handoff contract* — the coverage-matrix cells escalated to `agentic-safety-engineer`, the ticket references, the state per cell, signatories and date.
9. *Coverage matrix update* — the diff against exercise 01's matrix; confirmation that no cell is blank.
10. *Stretch goals attempted* — the notes.

## Submission

Attach the markdown, the Inspect suite directory (source only; no payloads), and the reference-run result artifacts to the module submission channel. If the target is under organisational NDA, redact identifiers but preserve the per-category ASR numbers, ceilings, cross-walk dispositions, invocation, and handoff-contract terms — a reviewer cannot review a reference-eval integration or a handoff contract whose scoped fields are blanked out.
