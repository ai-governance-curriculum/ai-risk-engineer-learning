# Exercise 03 — Classifier Guardrails with Llama Guard and ShieldGemma

**Estimated effort:** 3 hours

## Objective

Stack the **classifier layer** on top of the rule / flow layer built in exercise 02. Wire **Meta Llama Guard** into the input and output slots and **Google ShieldGemma** into the retrieval and output slots for the running system, then compose the classifiers with the rules so the composition is defence in depth (chapter 03 §"Composing classifier + rule + retrieval"), *not* two variants of the same classifier at two thresholds. Tune each classifier's threshold on a small labelled slice (a real threshold discipline, not the vendor default). Exercise 05 will run the full FP / FN / cost / latency measurement on the composed stack; this exercise ships the classifier bodies and calibrates the thresholds.

## Prerequisites

- Exercises 01 and 02 completed. The stubs in the NeMo GR actions (`llama_guard_action`, `shieldgemma_action`) get their real bodies here.
- Chapter 03 read; chapter 05 skimmed at the summary-table level so the threshold discipline this exercise applies makes sense against the measurement discipline that lands there.
- Access to at least two of the four classifiers:
  - **Meta Llama Guard** — the currently released version (Llama Guard 3 or 4; pin the exact version at authoring time — see resources.md's `needs-research` note). Self-hosted on GPU if data residency requires; hosted through a model-serving provider otherwise. Read the model card before shipping (chapter 03 §"Meta Llama Guard").
  - **Google ShieldGemma** — a size that fits your latency and cost budget (2B, 9B, or 27B). Self-hosted on GPU. Read the technical report before shipping.
- **Optional additional classifiers**: OpenAI Moderation (only if data residency permits vendor egress on user content; chapter 03 §"OpenAI Moderation" is explicit on the disqualification). Anthropic Constitutional Classifiers as an *approach* if your policy is bespoke and no off-the-shelf taxonomy fits (chapter 03 §"Anthropic Constitutional Classifiers" is explicit that this is an approach, not a drop-in library — do not conjure a product that does not exist for you at this level).
- A **labelled threshold-tuning slice** — a small labelled set (200–500 items minimum) that overlaps with the mod-104 hosted adversarial-and-legitimate set for the categories relevant to your running system. This is *not* the final measurement set (that stays in exercise 05); it is a calibration slice for finding the threshold before the full run.

## Problem statement

Replace the classifier stubs from exercise 02 with real classifier calls; tune per-classifier per-slot thresholds against the labelled slice; document the composition so a reviewer can see that two slots addressing the same harm class use *different classifiers with different training data*, not two variants of the same model.

### Which classifier goes where

Follow chapter 03 §"Worked composition — customer-support RAG assistant" as the reference. For the running system:

- **Input slot.** Llama Guard action on the user turn, wrapped by the NeMo GR input rail from exercise 02. Cheap self-hosted call before the main model runs. Blocks obvious unsafe / injection-shaped inputs.
- **Retrieval slot.** ShieldGemma action on each retrieved chunk, called from the NeMo GR retrieval rail from exercise 02. Score per policy — use ShieldGemma's per-policy probability API to score for "injection-likely," "unsafe," and "sensitive-info" at separate thresholds. Chunks that fail any threshold either drop from the assembly or are transformed.
- **Output slot.** *Two independent classifiers, both must pass.* Llama Guard on the response text (safety taxonomy) plus ShieldGemma on the response text (per-policy score). Chapter 03 §"Composing classifier + rule + retrieval" is explicit that two variants of the same classifier at two thresholds do not count as defence in depth; two different classifiers with different training data do.
- **Refusal-shaping slot.** No classifier; templated content. Confirms the exercise-01 design choice.

If your running system has a different shape (no retrieval, no tools, single-endpoint completion, etc.), adjust which classifier lands where but preserve the discipline: any slot the harm model says is defence-in-depth must have *two independent classifiers or one classifier plus one strong rule*.

### Threshold discipline

Chapter 03 §"Anti-patterns" is explicit that shipping with the vendor default threshold is a bug — the default is calibrated for the vendor's risk appetite, not yours. Tune every threshold on the labelled slice:

- **Llama Guard.** Threshold-style tuning is done by taxonomy scope (which categories are blocking vs. warning). For each slot, pick the categories that are blocking on the calibration slice such that FP on legitimate items is inside the product manager's patience. Document the taxonomy scope chosen per slot.
- **ShieldGemma.** Per-policy probability lets you plot ROC per policy. For each policy at each slot, pick the threshold that gives the FP / FN trade you want on the calibration slice. Document the threshold value per policy per slot.
- **OpenAI Moderation (if used).** Use raw per-category probabilities; do not rely on the API's boolean flags. Chapter 03 §"OpenAI Moderation" is explicit.

The calibration on this slice is not the final measurement (exercise 05 runs the full measurement). It is the tuning step so the full measurement is not chasing an obviously bad threshold.

### The composition discipline

For each slot that stacks a classifier on top of a rule (input, retrieval, output), name the composition explicitly:

- Which rules fire first (chapter 02 layer).
- Which classifier(s) fire on what passes the rules.
- What happens when classifiers disagree at the output slot ("both must pass" = both `safe` verdicts required; either `unsafe` reject).
- What the trace record looks like when both classifiers fire on the same item.

Do not stack "Llama Guard threshold-A on input plus Llama Guard threshold-B on output" and call it defence in depth. That is the anti-pattern from chapter 03 §"Composing classifier + rule + retrieval". The composition must involve *different underlying mechanisms*.

## Requirements

Produce all eight items:

1. **Classifier actions** — `risk/mod-108/<system-id>/classifiers/llama_guard.py` and `risk/mod-108/<system-id>/classifiers/shieldgemma.py`. Each exposes a Python callable used by the NeMo GR action layer. Pins the model checkpoint, the runtime (transformers / vLLM / Triton / TGI / hosted endpoint identifier), and the taxonomy / policy set the classifier is scored against.
2. **Wired-in rails** — the exercise 02 NeMo GR actions swapped from stubs to the real classifier calls. Diff-worthy commit; call out that the trace records now carry classifier-specific fields (category codes from Llama Guard, per-policy probabilities from ShieldGemma).
3. **Threshold-tuning slice** — `risk/mod-108/<system-id>/classifiers/tuning/slice.md` describing the labelled slice (source, labelling protocol, item counts per category). The slice files themselves (or a fingerprint if they cannot be committed) live alongside.
4. **Threshold calibration report** — `risk/mod-108/<system-id>/classifiers/tuning/thresholds.md`. For each classifier at each slot: the chosen threshold (or Llama Guard taxonomy scope), the FP / FN observed on the calibration slice at that threshold, and the sweep summary (thresholds tried, FP / FN at each). Include a ROC-style plot per ShieldGemma policy if the calibration slice is large enough to produce one honestly.
5. **Composition specification** — `risk/mod-108/<system-id>/classifiers/composition.md`. For each slot: the ordered stack (rules first, classifiers second), the disagreement handling at the output slot, the trace-record shape when both classifiers fire.
6. **Latency budget check** — `risk/mod-108/<system-id>/classifiers/latency.md`. p50 and p95 added latency per classifier per slot on the calibration slice. Chapter 03 §"Meta Llama Guard" and chapter 05 §"Latency accounting" define the discipline: measure in the middleware, not on the classifier server side. Any slot's p95 above the exercise-01 latency budget is flagged for a treatment plan.
7. **Cost budget check** — `risk/mod-108/<system-id>/classifiers/cost.md`. Mean cost per call per classifier per slot at production pricing (self-hosted GPU-hour amortised, or hosted per-call). Roll up to per-day at your production request volume. Chapter 05 §"Cost accounting" is the reference.
8. **Failure-mode probe report** — `risk/mod-108/<system-id>/classifiers/failure-modes.md`. For each of chapter 03 §"Anti-patterns", one section:
   - **Vendor threshold.** Named the threshold you tuned; confirmed no vendor default is shipped.
   - **Category-taxonomy mismatch.** Named at least one harm-model category your chosen classifier's taxonomy does not cover; named the supplementary check (rule, ShieldGemma custom policy, deferred to exercise 04's Presidio, or filed as open work).
   - **Same-model stack.** Confirmed the output slot uses two mechanisms trained on different data. If cost forces a single classifier at output, named the second mechanism that fills the defence-in-depth requirement (a strong rule, an ingest-side Presidio pass, an out-of-band review sampling).
   - **Missing FP report.** Named that exercise 05 produces the full FP report and that this exercise's calibration report is a preview.

Cross-cutting rules:

- **Pin model checkpoints and taxonomies.** Every classifier call carries the model checkpoint identifier and the taxonomy / policy version in the trace. Chapter 05 §"The measurement contract" requires it for reproducibility; wire it here rather than back-fitting later.
- **Fail closed on classifier error** (chapter 02 §"Anti-patterns" and chapter 03's implicit corollary). A classifier that returns HTTP 500, times out, or returns an unparseable response must produce a "reject" verdict at input and retrieval slots. At the output slot, the fail-closed shape is a refusal, not a raw model response.
- **Do not swap OpenAI Moderation's booleans for your risk appetite.** If you use it, use raw probabilities and calibrate; if you use booleans, the residual number cannot credit the calibration you did not do.
- **Do not fabricate a Constitutional Classifier body.** Chapter 03 §"Anthropic Constitutional Classifiers" is explicit that this is an approach, not a drop-in you install; if bespoke policy is on your roadmap, note the approach as the treatment plan and cite the paper. Do not conjure a `constitutional_classifier.py` that pretends to be a productised library.
- **Different classifiers, different mechanisms.** The composition test in chapter 03 §"Composing classifier + rule + retrieval" is the one an auditor will run against your stack. Answer it now, not at review time.

## Starter guidance

- **Read the Llama Guard model card and the ShieldGemma technical report before writing code.** Both name the taxonomy scope, the training data source categories, and the intended usage. Skipping this step and later discovering that "medical advice" is not in the Llama Guard taxonomy is a costly detour.
- **Sizing choice matters.** ShieldGemma at 27B is a much stronger classifier than 2B but has a much larger latency and cost tail. Start with the 2B or 9B; measure; step up only if the FP / FN numbers on the calibration slice force it.
- **Self-host vs. hosted matters for cost.** A self-hosted classifier's cost is GPU-hour amortised across throughput; if throughput is low, per-call cost is bad. A hosted classifier's cost is per-call directly. Chapter 06's build-vs-buy TCO analysis will bite here; do the amortisation honestly, not "GPU already there so free."
- **Threshold sweeping is grid-search on a small grid.** Try 5–7 thresholds per policy (e.g., 0.3, 0.4, 0.5, 0.6, 0.7); plot FP and FN on the calibration slice; pick the point where FP and FN both sit inside the exercise-01 appetite. If no point sits inside both bounds, either the classifier is not strong enough for this slot (treatment: swap to a stronger classifier or add a rule at that threshold) or the appetite is wrong (treatment: escalate).
- **Parallelise the output slot's two classifiers.** If Llama Guard and ShieldGemma both fire on the model response, they should run in parallel, not sequentially. Chapter 05 §"Latency accounting" describes the parallel-vs-serial reporting shape.
- **The trace record from two classifiers at one slot is a shape decision.** Log a single "output verdict" record with sub-fields for each classifier's verdict, threshold, and version. Mod-109 will want to alert on classifier drift per classifier, not aggregated.
- **The calibration slice is not the final measurement set.** Do not overfit thresholds to the calibration slice. Exercise 05 runs the full measurement on the mod-104-hosted set; the calibration set is scaffolding.
- **Fail-closed is subtle at the output slot.** If the output classifier errors and you reject, the caller sees a refusal for what may have been a perfectly safe response. That is the price of fail-closed; document it and instrument the error rate so drift is visible.

## Acceptance criteria

The submission is accepted when:

- Real Llama Guard and ShieldGemma actions exist and are wired into the NeMo GR rails from exercise 02; the stubs are gone.
- Every classifier call pins the model checkpoint, the taxonomy / policy version, and the threshold or taxonomy scope in the trace record.
- The threshold calibration report shows a sweep, a chosen threshold, and the FP / FN observed on the calibration slice for each classifier at each slot.
- The composition specification names the ordered stack per slot, the disagreement handling at the output slot, and the trace-record shape.
- The latency and cost budget checks are populated with numbers from the calibration slice, and any slot over budget is flagged.
- The failure-mode probe report covers all four chapter-03 anti-patterns and either resolves each or files a follow-up.
- The output slot uses two different classifiers (or one classifier plus one strong rule); the two-variants-of-the-same-classifier anti-pattern is not shipped.
- Fail-closed behaviour is proved with a fault-injection test (force a classifier to raise; observe rejection).

## Stretch goals

- **A custom ShieldGemma policy for a harm-model category the built-in policies do not cover.** Chapter 03 §"Google ShieldGemma" describes the custom-policy-prompt pattern. Author a policy prompt for one of your running system's bespoke categories (e.g., "medical advice not from a licensed source," "financial advice outside our jurisdiction," "quotes about a competitor product"); score against your calibration slice; report FP / FN vs. the built-in policies.
- **Latency parallelisation on the output slot.** Ship the parallel-vs-serial variant of the Llama Guard + ShieldGemma output pair (chapter 05 §"Latency accounting"). Report the p95 delta and the compute-cost delta of parallel-with-speculative-cancellation vs. plain-sequential.
- **A per-language calibration.** If your running system serves more than one language, run the threshold calibration per language. Llama Guard and ShieldGemma both degrade meaningfully on lower-resource languages; the threshold that works for English may not work for German. Report the per-language deltas.
- **A no-classifier baseline for comparison.** Turn every classifier off; run the calibration slice through the rules-only stack from exercise 02; report the FP / FN delta so the classifier layer's contribution is explicit. Chapter 05 §"Per-slot residual-risk update" describes how this baseline populates the control-defeated column.
- **A supplementary check for a taxonomy-mismatch case.** For one harm-model category where Llama Guard's / ShieldGemma's taxonomy is a mismatch, ship a rule-based supplementary check that catches the category, wired alongside the classifier at the same slot; report the improvement in recall on the calibration slice.
- **A cross-track handshake with mod-105.** Chapter 03 §"Cross-references" flags that mod-105's adversarial-ML literature shapes how classifiers degrade under distribution shift and attack transfer. Skim mod-105's chapters on classifier robustness and write a one-paragraph note on which of your classifiers is most at risk (typically the smallest and most in-distribution model).

## Deliverable format

A single markdown document `mod-108-ex03-classifiers.md` that indexes the artefacts. The index document contains:

1. *System summary* — one paragraph naming the running system, the slot-to-classifier assignment, and the deployment shape (self-hosted / hosted) per classifier.
2. *Model versions and taxonomies pinned* — Llama Guard version + taxonomy version; ShieldGemma checkpoint + policy set; runtime versions (transformers, vLLM, TGI, endpoint identifier).
3. *Threshold-tuning slice* — provenance, labelling protocol, per-category item counts.
4. *Threshold calibration report* — per classifier per slot, chosen threshold, sweep summary, ROC plot where the slice supports it.
5. *Composition specification* — slot-by-slot ordered stack; disagreement handling; trace-record shape.
6. *Latency budget check* — p50 / p95 per classifier per slot; slots over exercise-01 budget flagged.
7. *Cost budget check* — mean cost per call, per-day roll-up at production volume.
8. *Failure-mode probe report* — the four chapter-03 anti-pattern sections.
9. *Fail-closed proof* — the fault-injection test result.
10. *Open items* — any category the taxonomy does not cover; any slot over latency or cost budget; any classifier that needs replacement.

## Submission

Attach the index markdown plus the eight artefacts. This exercise is a precondition for exercise 05 (measurement) — the classifier bodies and threshold choices land here; the measurement on the full mod-104 set lands there.
