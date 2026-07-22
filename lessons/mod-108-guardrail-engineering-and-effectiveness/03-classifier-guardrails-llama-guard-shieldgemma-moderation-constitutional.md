# Classifier Guardrails — Meta Llama Guard, Google ShieldGemma, OpenAI Moderation, Anthropic Constitutional Classifiers

Rule and flow (chapter 02) catches the enumerable. The classifier layer catches what the rules cannot see — paraphrase, obfuscation, novel jailbreak shapes, semantic policy violations. Four classifiers cover almost every enterprise deployment: **Meta Llama Guard** (open-weights safety classifier, versions 3 / 4), **Google ShieldGemma** (open-weights safety classifier, Gemma family), **OpenAI Moderation** (hosted API), and **Anthropic Constitutional Classifiers** (research publication and applied approach for jailbreak-resistant filtering). This chapter teaches how each is trained, where each fits, and how to compose them with rules and retrieval-based defence in depth.

## Motivation

Two failure modes an experienced reviewer catches at first glance:

- **One classifier, one threshold.** The application ships one moderation call at one threshold on both input and output. When the classifier drifts, both slots drift together. When a paraphrase bypass appears, both slots miss. The residual number in the register is wrong the day after the classifier is bypassed and nobody knows because the *number* still comes out clean.
- **Classifier as belt-and-braces without owning the false-positive cost.** The application stacks four classifiers, each cutting off ~5% of "risky-looking" traffic. Together they reject 15–18% of *good* traffic. The product manager finds out through user complaints. There is no FP report; nobody can point at which classifier is doing what.

The right shape is: know each classifier's training scope, know its threshold's cost in FP and FN, place it at a slot where a *different* mechanism also fires, and measure the composition on the mod-104 adversarial set before promoting to production.

## Meta Llama Guard

**What it is.** Llama Guard is an open-weights, LLM-based *input / output content classifier* from Meta. It reads a conversation turn (user prompt, model response, or both) and returns a label from a policy taxonomy — safe / unsafe, and for unsafe outputs a set of category codes from a published taxonomy (e.g., violent crimes, non-violent crimes, sex crimes, hate, self-harm, elections, sexual content, weapons, code-interpreter abuse, privacy, indiscriminate weapons, defamation, specialised advice, intellectual property, and additional categories per version).

**Versions.**

- **Llama Guard 1 / 2** — earlier Llama-based safety classifiers with a smaller taxonomy.
- **Llama Guard 3** — 8B model, expanded taxonomy including tool-use / code-interpreter abuse categories.
- **Llama Guard 4** — most recent version at authoring time, expanded taxonomy and multimodal support in some variants. <!-- needs-research: pin the exact Llama Guard 4 release date, size options, and taxonomy shipped in the model card, since the taxonomy evolves per major version. -->

**How it's trained.** Fine-tuned on human-labelled safety data against the published taxonomy; released under the Llama community license. The model card names the taxonomy version, the training data source categories, and the intended usage. Read the model card as authoritative before shipping.

**Where it fits.**

- Input slot — classify user turns before the main model is called. Catches obvious unsafe intents cheaply.
- Output slot — classify model responses before returning to the caller. Catches unsafe outputs the main model produced despite the input filter passing.
- Retrieval slot — classify retrieved chunks before context assembly, on applications where retrieved content may contain unsafe material.

**Deployment shape.** Self-hosted on GPU (small vs. main-model inference cost), or hosted through a model-serving provider that offers it as an endpoint. Choose based on your latency budget and data-residency constraints.

**Threshold discipline.** Llama Guard returns a hard verdict (safe / unsafe) plus the category codes; it does not natively expose a probability threshold you tune. Threshold-style tuning is done by changing the taxonomy scope (which categories you treat as blocking vs. warning), or by fine-tuning a per-category acceptance rule on your own labelled set. Both paths are documented in the model card.

## Google ShieldGemma

**What it is.** ShieldGemma is an open-weights content-moderation classifier family from Google, built on Gemma 2. It is scored per policy — you pass a policy prompt describing what "unsafe" means and receive a probability that the content violates that policy. Ships in multiple sizes (2B, 9B, 27B) to trade off cost and coverage.

**Policies.** ShieldGemma's default policies cover categories like sexually explicit content, dangerous content, harassment, and hate speech. You can also supply your own policy prompts to score against custom categories.

**How it's trained.** Fine-tuned on a large synthetic-and-real safety corpus with policy-conditioned prompts, released with a technical report describing the methodology. <!-- needs-research: cite the exact ShieldGemma paper / model-card version current at authoring time; Google has released ShieldGemma and ShieldGemma 2 with additional multimodal capability. -->

**Where it fits.**

- Output slot — probability-scored classification lets you tune a threshold with FP / FN accounting.
- Retrieval slot — score per policy on retrieved chunks; drop chunks whose probability exceeds threshold for a policy that matters at retrieval time.
- Custom-policy slot — supply your own policy prompt for a category not in the default taxonomy.

**Threshold discipline.** Because ShieldGemma emits per-policy probability, you can plot the ROC curve on your labelled adversarial set and pick the threshold with the FP / FN trade you want. That measurement — one threshold per policy per slot — is the deliverable of chapter 05 for ShieldGemma-based controls.

## OpenAI Moderation

**What it is.** OpenAI Moderation is a hosted API from OpenAI that classifies text (and, in newer versions, images) against a fixed policy taxonomy — hate, hate/threatening, harassment, harassment/threatening, self-harm and its subtypes, sexual and its subtypes, violence and its subtypes, illicit content. Returns per-category probabilities and boolean flags.

**Versions.**

- **`text-moderation-*` series** — earlier text-only endpoints.
- **`omni-moderation-*` series** — current multimodal moderation endpoint at authoring time.

<!-- needs-research: pin the exact current model identifier for OpenAI Moderation and its documented category list; the API and its taxonomy evolve. -->

**How it's trained.** Trained on OpenAI's internal safety-labelled corpus; the specific training methodology is not fully public but the category definitions and the API contract are documented.

**Where it fits.**

- Input slot — cheap hosted call before your main model runs.
- Output slot — cheap hosted call before returning to caller.
- Bounded-scope moderation on user-generated content that flows through your application (chat inputs, uploaded documents summarised by the model).

**Deployment shape.** Hosted only. Data leaves your environment. For applications with data-residency constraints or a strict no-vendor-inference policy on user content, this is disqualifying — go Llama Guard or ShieldGemma self-hosted instead.

**Threshold discipline.** Per-category probabilities allow you to tune thresholds. The API returns both booleans (at OpenAI's default thresholds) and raw probabilities; use the raw probabilities and calibrate your own thresholds on your adversarial set. Do not ship blindly with the default booleans; they are calibrated for OpenAI's own risk appetite, not yours.

## Anthropic Constitutional Classifiers

**What it is.** Constitutional Classifiers are a research approach (published by Anthropic, "Constitutional Classifiers: Defending against Universal Jailbreaks across Thousands of Hours of Red Teaming") for training input and output classifiers using synthetic data generated against a written *constitution* — a set of natural-language rules describing what is and is not allowed. The approach is designed to be robust to jailbreaks that transfer across models.

**How it's trained.** The constitution is authored; synthetic training data is generated to cover diverse allowed and disallowed content according to the constitution; classifiers are trained on the synthetic data and evaluated against held-out red-team payloads.

**Where it fits.** As an *approach* rather than a drop-in library. You use it when your policy is bespoke (industry-specific, product-specific) and the off-the-shelf taxonomies of Llama Guard / ShieldGemma / OpenAI Moderation do not match. You author the constitution, generate the training data, and produce a classifier that scores against *your* policy specifically. Anthropic also uses classifiers of this style to filter its own products; see the Anthropic Trust Center for the current productisation state.

<!-- needs-research: confirm current availability of Constitutional-Classifiers-derived filters as a hosted service through Anthropic (vs. only as a research approach at authoring time); update citation accordingly. -->

**Threshold discipline.** Whatever classifier architecture you land on, the threshold is tuned on your labelled adversarial set. The value the approach adds is *robustness to novel jailbreak shapes*, evidenced by the red-team methodology in the paper. Reproduce the evaluation methodology, not just borrow the number.

## When to pick which

A summary table for the working set:

| Classifier | Deployment | Threshold surface | Custom policies | Best fit slot | Data-residency friendly |
|---|---|---|---|---|---|
| Llama Guard 3 / 4 | Self-hosted GPU | Category-scope tuning | Via fine-tuning | Input, output, retrieval | Yes |
| ShieldGemma (2/9/27B) | Self-hosted GPU | Per-policy probability | Yes (policy prompts) | Output, retrieval | Yes |
| OpenAI Moderation | Hosted OpenAI API | Per-category probability | No (fixed taxonomy) | Input, output on non-sensitive traffic | No |
| Constitutional Classifiers | Approach; you build the classifier | Whatever your classifier exposes | Yes (constitution) | Input, output for bespoke policies | Depends on your build |

The composition point is that these are *different mechanisms trained on different data with different taxonomies*. Two of them at two slots addressing the same harm class is defence in depth; the same one at two slots is not.

## Composing classifier + rule + retrieval

Three composition patterns:

- **Rule cheap-catch, classifier semantic-catch, retrieval-side provenance.** The rule catches known injection strings (a few microseconds); Llama Guard catches paraphrased injection (tens to hundreds of milliseconds on GPU); the retrieval-side provenance rule ensures untrusted chunks never reach the assembler regardless of injection content.
- **Structural + semantic on output.** Guardrails AI JSON-schema check (structural) plus ShieldGemma per-policy score on the free-text fields (semantic). Structural fails cheaply and often; semantic runs on what passes structural.
- **Two independent classifiers on the same slot, both must pass.** Both Llama Guard and ShieldGemma classify the output; the caller sees the response only if both return safe. Cost doubles; residual improves because a single-classifier miss no longer defeats the slot.

The composition to *avoid* is stacking two variants of the same classifier at different thresholds and calling it defence in depth. Different threshold on the same model is still the same model — a bypass of the underlying model bypasses both variants.

## Worked composition — customer-support RAG assistant

Reusing the running scenario. Classifier-layer choices per slot:

| Slot | Classifier(s) | Rationale |
|---|---|---|
| Input filter | Llama Guard 3 (self-hosted) | Data-residency requirement; unified taxonomy across input and output; NeMo GR action wraps it |
| Retrieval filter | ShieldGemma 9B (self-hosted) with custom policies for injection-shape content | Per-policy scoring lets us tune "injection-likely" threshold aggressively without collapsing the "generally-unsafe" threshold |
| Output validator | Llama Guard 3 (self-hosted) + ShieldGemma 9B (self-hosted), both must pass | Two independent mechanisms on the primary caller-facing slot |
| Refusal shaping | No classifier | Templated content per category |

OpenAI Moderation is not selected here (data-residency constraint on customer content). Constitutional-Classifier-style approach is deferred to a later phase when the bespoke-policy set stabilises.

Cost accounting for the classifier layer for a representative daily volume is what chapter 05 measures.

## Anti-patterns

Four anti-patterns specific to the classifier layer:

- **Blindly trusting the vendor threshold.** Default thresholds are calibrated for the vendor's risk appetite, not yours. Tune on your labelled set or you inherit a residual you did not design.
- **Category-taxonomy mismatch.** You care about a category the classifier does not have (e.g., "medical advice not from a licensed source" is not in Llama Guard's taxonomy). Ship a supplementary check — either a rule, a Constitutional-Classifier-style bespoke classifier, or a ShieldGemma custom policy — rather than pretending the closest existing category covers it.
- **Same-model stack.** Two classifiers, both descended from the same base model, both fooled by the same jailbreak. Independence requires different underlying data and preferably different architectures.
- **Missing FP report.** The classifier layer's false-positive rate is the product manager's problem the moment it exceeds their patience. Measure and report it (chapter 05) before they discover it.

## Cross-references

- **Chapter 01** — this classifier layer populates several slots; the coverage table shows which.
- **Chapter 02** — the rule / flow layer calls these classifiers via NeMo GR actions or Guardrails AI validators.
- **Chapter 04** — Presidio (the PII layer) is a rule-and-analyzer library, not a classifier in the Llama-Guard sense; the two compose.
- **Chapter 05** — the FP / FN / cost / latency measurement is what turns these classifier choices into defensible residual numbers.
- **Mod-104** — supplies the adversarial payload set the classifiers are measured on. Do not build a private classifier eval; consume the hosted one.
- **Mod-105** — supplies the adversarial-ML literature that shapes how each classifier degrades (transfer attacks, prompt-based evasion, distribution shift).
- **Mod-109** — consumes the classifier verdicts as monitor signal; drift on classifier output distribution is a monitoring priority.
- **Mod-111** — consumes the classifier logs at incident time; the classifier version and prompt version are part of the incident evidence.

## Summary

- Four classifiers cover most enterprise deployments: Llama Guard 3 / 4 (open, taxonomy-driven), ShieldGemma (open, per-policy probability), OpenAI Moderation (hosted, per-category probability), Constitutional Classifiers (approach for bespoke policy).
- Each has a documented training methodology, taxonomy, and threshold surface. Read the model card / paper as authoritative before shipping.
- Compose classifier + rule + retrieval-based defence in depth; do not stack two variants of the same classifier and call it defence in depth.
- Threshold discipline: tune on your labelled adversarial set; do not ship the vendor default without measurement.
- Anti-patterns: vendor threshold, taxonomy mismatch, same-model stack, missing FP report.
- Exercise 03 puts you through a Llama Guard + ShieldGemma composition on the running system.
