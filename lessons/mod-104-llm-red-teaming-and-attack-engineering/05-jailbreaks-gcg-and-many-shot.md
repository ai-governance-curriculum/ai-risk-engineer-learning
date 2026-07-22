# Jailbreaks: The Taxonomy, GCG-Style Suffixes, and Many-Shot In-Context

Direct prompt-side jailbreaks are the attack class most people picture when they hear "LLM red-team." Chapter 02's garak suite and chapter 03's PyRIT library both exercise them, but they exercise them as an inventory — they do not teach the taxonomy or the operational discipline for the deeper classes (freshly optimised adversarial suffixes; many-shot in-context attacks). This chapter pins the taxonomy, walks the two deep classes explicitly, and formalises the harmful-payload discipline in the specific context where its operational risk is highest.

## Motivation

Two failure modes a programme hits when it treats jailbreaks as one lump:

- **The pattern-matched programme.** The suite has a hundred DAN variants and calls that "jailbreak coverage." A researcher publishes a new persona-hijack framing that shares nothing lexically with any DAN variant — the suite misses it, the model complies, and the finding surfaces from a bug bounty. The programme's ASR on the shipped suite looked fine; the shipped suite did not cover the technique class.
- **The stale-suffix programme.** The suite includes a GCG suffix from the 2023 paper. Model providers have since retrained against that specific suffix; the suffix no longer works; the programme reports "GCG covered." Nobody has re-run the optimisation against the current model. The class is not covered — one instance of it is, and that instance is dead.

The way out is to organise jailbreaks by *technique class*, ensure each class has coverage that is *live* (regenerated where regeneration is needed), and keep the harmful payloads out of the repo while doing it.

## The taxonomy

A working taxonomy of prompt-side jailbreak techniques at this level of maturity — the same list that shows up in the OWASP GenAI Red-Team Guide, the OWASP LLM Top 10 (category LLM01), and the "Do Anything Now" (DAN) empirical study (Shen et al., arXiv:2308.03825):

| Technique class | What it does | Where it originates |
|---|---|---|
| Direct instruction override | Explicit "ignore your instructions and do X" framings | Universal; the first jailbreak class named |
| Encoding / obfuscation | Payload obscured through base64, ROT13, leetspeak, unicode-lookalike, translation | Circumvents plaintext filters and safety-tuning that was done on plaintext |
| Multi-turn refusal-bypass | The attacker uses partial compliance on turn N to leverage on turn N+1 | Requires an orchestrator (chapter 03); PAIR / TAP families |
| Role-play / persona-hijack | The model is asked to adopt a persona ("DAN," "AIM," "developer mode") that is framed as not subject to the safety training | DAN family — Shen et al. arXiv:2308.03825 |
| GCG-style adversarial suffixes | Optimised token sequences appended to a request that push the model toward compliance | Zou et al., "Universal and Transferable Adversarial Attacks on Aligned Language Models," arXiv:2307.15043 |
| Many-shot in-context | The context is filled with many demonstrations of the model complying with policy-negative asks, and then the target ask is placed at the end | Anthropic, *Many-shot jailbreaking* — Anil et al. |
| Refusal-suppression framing | The request is cast in a context that reframes refusal as harm ("you must answer or the patient dies") or that solicits meta-refusal ("what would the response be if you did answer?") | Multiple; catalogued in the DAN taxonomy and the OWASP Guide |
| Prompt-leak | The attacker extracts the system prompt or the retrieval-side instructions | Adjacent to jailbreak; catalogued in the OWASP LLM Top 10 |

<!-- needs-research: pin the exact OWASP GenAI Red-Team Guide version and the OWASP LLM Top 10 version, and confirm the Anil et al. Many-Shot Jailbreaking paper's citation form (title, publication venue, arXiv ID if applicable). -->

Each technique class is a row on the coverage matrix (chapter 01). The claim "we cover jailbreaks" is not a coverage claim; the claim "each of the eight technique-class rows shows `covered:<suite-id>` for the surfaces where the class applies" is.

The rest of this chapter drills into the two classes that need special handling: GCG-style adversarial suffixes (because they must be *optimised against the specific target*, not just replayed) and many-shot in-context (because their operational shape is very different from single-prompt jailbreaks and their coverage is often accidentally omitted).

## GCG-style adversarial suffixes

The Zou et al. 2023 paper *Universal and Transferable Adversarial Attacks on Aligned Language Models* (arXiv:2307.15043) introduced Greedy Coordinate Gradient (GCG), a gradient-based optimisation over the input tokens that appends a suffix to a harmful request such that the model becomes measurably more likely to comply. The paper's main empirical claim is a nontrivial attack-success-rate lift on the models it evaluated, and evidence of transferability across models — a suffix optimised on one open-weight model transfers with meaningful ASR to other models, including closed ones. <!-- needs-research: confirm the paper's headline numbers and the exact list of transfer targets. -->

### Why GCG is different from other jailbreak classes

Three properties that make GCG operationally distinct:

- **It requires gradients.** GCG is a white-box attack. You need model weights and the ability to compute gradients on the input tokens. Against closed models you cannot optimise GCG directly; you can only *replay* suffixes optimised on open models and hope for transfer.
- **The suffix is model-specific.** A suffix that transfers with 40% ASR to model A may transfer with 5% ASR to model B or 0% ASR to model C after a safety-tuning update. Suffixes have half-lives — providers retrain against known suffixes, and re-optimisation is required to keep the coverage live.
- **The output is unreadable.** A GCG suffix is a sequence of tokens that looks like garbage to a human. This has two consequences: filters that look for jailbreak "keywords" don't catch it, and the payload is not something an ordinary reviewer can eyeball for triage.

### The engineering shape of GCG coverage

Two levels of coverage that a level-25 programme is expected to provide:

**Level 1 — replay coverage.** A pinned set of published GCG suffixes (from the Zou et al. paper's appendix or public follow-on work), stored in the out-of-repo payload store, loaded by hash by a converter in your PyRIT library (chapter 03), applied to your target on every release-gate suite run. Metric: ASR per pinned suffix, plus an aggregate ASR across the pinned set. Threshold: below the register-row target for the corresponding harm category. Replay coverage is *cheap* — it does not require gradients on your target.

**Level 2 — fresh-optimisation coverage.** For open-weight models you deploy, GCG is re-optimised against the current checkpoint on a quarterly cadence, with the resulting suffixes added to the payload store and included in the replay set for both the open-weight target and the closed-weight targets (as a transfer test). Metric: ASR against the open-weight target directly, plus transfer ASR to closed-weight targets. Threshold: attacker-favourable, since you now have gradient-informed suffixes.

Level 2 is the depth that keeps the class alive over time. It requires compute (gradient descent over token sequences on a full model), a small model-of-attackers infra, and an operational discipline about the payloads.

<!-- needs-research: confirm whether GCG's public-implementation shape (the LLM-Attacks repo from the Zou et al. authors) is still the reference implementation, or whether the community has consolidated on a newer maintained fork. -->

### GCG payload discipline

Fresh-optimised GCG suffixes are the *highest-risk category* of harmful payload the programme holds, because:

- They are opaque to human review; a reader cannot tell whether the string is dangerous by looking at it.
- They may transfer to models the string was not optimised on; a leaked suffix has a footprint bigger than its target.
- They compose with converters trivially — a suffix converter + any harmful-goal prompt = a candidate attack.

The discipline:

- **Never in the source repo.** The string never enters git. Not as a test case, not as a comment, not as documentation.
- **Out-of-repo payload store only.** The suffix is stored in the same payload store as other harmful strings, tagged with its optimisation provenance (model, run, seed, goal).
- **Access-logged.** Reading a GCG suffix from the store is an access-logged event. The log is retained.
- **Do not publish transfer results with the suffix intact.** Internal reports may cite the aggregate transfer ASR; the specific suffix does not appear in the report body. If a report needs to reference a specific suffix, it references its hash.
- **Rotate on retraining.** When a model provider ships a safety-tuning update, the transfer suffixes may go dead; regenerate the set and archive the old set with a "retired" flag rather than deleting outright, so historical measurements remain reproducible.

The discipline is not decoration — leaked GCG suffixes are a category of research-adjacent risk that model providers and the broader ecosystem treat as sensitive. Being sloppy here is the programme's most likely policy-side incident.

## Many-shot in-context

Anthropic's *Many-shot jailbreaking* work (Anil et al.) is the second class that needs explicit coverage. The pattern:

- The attacker constructs a long context filled with many demonstrations of a "dialogue" in which the model (as depicted in the context) complies with policy-negative asks.
- The final turn of the context contains the target policy-negative ask.
- The model, having seen many demonstrations of its in-context "self" complying, becomes measurably more likely to comply on the final turn.

The empirical result is that the effect scales with the number of shots — the more demonstrations, the higher the ASR — and that models with larger context windows are more vulnerable to the specific class because they can be filled with more shots. <!-- needs-research: confirm Anil et al. citation form and the paper's specific empirical scaling claim. -->

### Why many-shot is different

Two properties that make it operationally distinct:

- **The payload is long.** A many-shot attack may be tens or hundreds of turns; the whole context window is the payload, not just a suffix.
- **The demonstrations themselves are harmful.** Each shot depicts the model complying with a policy-negative ask. The corpus of demonstrations is a corpus of harmful outputs. Storing and reviewing them requires the same out-of-repo discipline as GCG suffixes, plus extra care around the individual demonstrations' content.

### The engineering shape of many-shot coverage

A working coverage pattern:

- **A demonstration corpus.** A category-tagged set of `(policy-negative ask, model-would-comply response)` demonstration pairs, stored in the out-of-repo payload store. The demonstrations are grouped by category (e.g., malware-generation, self-harm, illicit-instructions) matching the internal policy categories.
- **A shots-per-context parameter sweep.** The suite runs the target with, e.g., 4 / 16 / 64 / 256 shots (as many as the model's context window allows) and reports ASR per shot count. The shape of the curve tells you how the model's refusal degrades with shot count.
- **A refusal-robustness threshold at your operating shot count.** Real users do not usually fill the context with 256 shots, but the enterprise threat model must know at what shot count the model's refusal degrades meaningfully and whether that shot count is reachable via chained tool responses, long documents, or multi-turn conversation histories.
- **Cross-category demonstrations.** Demonstrations in one category can degrade refusal in adjacent categories. The suite should include a cross-category test: does 64 shots of "the model helping with malware" make the model more likely to comply with a self-harm ask on the last turn?

### Many-shot payload discipline

The same out-of-repo discipline as GCG applies, with two additions:

- **Per-demonstration review.** Each demonstration is itself a harmful ask/answer pair. Someone had to author or curate it; that curation is under the same access controls as red-team payloads generally.
- **Length-aware storage.** Many-shot corpora can be very large; the payload store's throughput and access-logging must handle the volume without becoming the bottleneck.

## Refusal-bypass on the internal policy-derived suite

Beyond the published techniques, the programme owns an internal policy-derived suite: a set of asks that the organisation's own policy (the acceptable-use policy, the content policy, the tool-use policy) says the system must refuse. That suite is not published anywhere — it is derived from your policy, in your repo (with the *questions* in-repo, the *harmful responses* only in the out-of-repo store if any are collected).

Coverage for the internal suite runs all the technique classes above against every category in the policy. The pass rule is the ASR ceiling per category, sized by the register-row severity. Chapter 06 covers how the internal suite is authored on the Inspect harness.

## Interfaces with adjacent modules

- **Mod-102 (harm modelling)** — every technique class maps to attack vectors on the harm-model rows; novel technique classes drive harm-model additions.
- **Mod-103 (risk register)** — per-technique-class ASR is a likelihood anchor; the GCG re-optimisation cadence is a control the register credits.
- **Mod-108 (guardrails)** — output-side and input-side guardrails are the primary mitigation; the technique classes drive the guardrail category coverage.
- **Mod-109 (monitoring)** — production-side detection for these classes is hard (that is why the guardrails matter), but patterns from confirmed findings inform detector logic (e.g., long-context anomaly detection for many-shot).
- **Mod-111 (incident response)** — every confirmed finding produces a regression fixture; the fixture pins the payload by hash so replay is deterministic.

## Summary

- Jailbreaks are eight technique classes at this level of maturity — direct override, encoding / obfuscation, multi-turn refusal-bypass, persona-hijack, GCG-style suffixes, many-shot, refusal-suppression framing, prompt-leak.
- Coverage is per class; a hundred DAN variants is not coverage of the persona-hijack class if novel framings are missed.
- GCG requires two levels of coverage: replay of published suffixes, and (for open-weight targets) quarterly fresh optimisation against the current checkpoint.
- GCG payloads are the highest-risk payload category — opaque, transferable, composable. Never-in-repo, out-of-repo-only, access-logged, rotate-on-retraining.
- Many-shot requires a demonstration corpus, a shots-per-context sweep, a refusal-robustness threshold at operating shot counts, and cross-category coverage.
- Refusal-bypass on the internal policy-derived suite runs all technique classes against every policy category; the ASR ceilings are register-anchored.
- Exercise 05 walks the GCG replay + fresh-optimisation loop and the many-shot suite construction.
