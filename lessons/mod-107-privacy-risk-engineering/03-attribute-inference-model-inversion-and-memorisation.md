# Attribute Inference, Model Inversion, and LLM Memorisation Evaluations

Membership inference (chapter 02) is one axis of privacy leakage. Three others — **attribute inference**, **model inversion**, and **LLM memorisation / verbatim extraction** — are engineered differently, defended differently, and reported differently. This chapter walks each one: what the attacker asks, what the canonical eval looks like, what the residual-risk signal is, and where each row lands on the mod-103 register.

## Motivation

A privacy engineer who runs only MIA and stops has two blind spots.

- **Tabular and vision models with sensitive columns** can leak attributes and reconstructions even when membership is well-protected. A model with mediocre MIA can still be an inference oracle for the sensitive-column value.
- **LLMs** leak differently. MIA on an LLM is possible but methodologically noisy at scale; the operationally significant leak is *verbatim memorisation* — the model emits training strings in response to natural or adversarially crafted prompts. This is a controller obligation in its own right and it does not correlate cleanly with MIA scores.

Each attack in this chapter closes one of those blind spots.

## 1. Attribute inference

### What the attacker asks

Given some attributes of an individual (age, ZIP, sex, some medical history) and access to the trained model, infer a sensitive missing attribute (a specific diagnosis, a genotype, an income band, a political preference).

### Canonical publications

- Fredrikson, Lantz, Jha, Lin, Page, Ristenpart (USENIX Security 2014) — "Privacy in Pharmacogenetics: An End-to-End Case Study of Personalized Warfarin Dosing" — attribute-inference on a deployed dosing model that leaks genotype from clinical features + dosing outputs.
- Yeom, Giacomelli, Fredrikson, Jha (CSF 2018) — "Privacy Risk in Machine Learning: Analyzing the Connection to Overfitting" — attribute-inference formalisation and the connection between attribute-inference advantage and generalisation gap.

### Threat model

Attribute inference is a *query-time* attack; the attacker has:

- some attributes of the target individual (call them the *quasi-identifiers*),
- query access to the model,
- knowledge of the population marginal for the sensitive attribute (the base rate).

The attack "wins" if the attacker's posterior probability on the sensitive attribute — after querying the model — is *meaningfully higher* than the base-rate prior.

### Pipeline

1. **Assemble a labelled evaluation set.** Records with the true sensitive attribute known to you as evaluator but treated as unknown to the attacker.
2. **Simulate the attacker's information.** Give the attacker the quasi-identifiers and query access.
3. **Attacker's decision rule.** The attacker either fits an explicit inversion model (e.g., a classifier that maps quasi-identifiers + model outputs → sensitive attribute) or uses maximum-likelihood over model outputs.
4. **Measure.** Attribute-inference *advantage* = attacker accuracy − base-rate accuracy. Report per-slice.

### Residual-risk signal

The two numbers to file:

- **Attribute-inference advantage** over the base-rate prior, per sensitive attribute. An advantage of 0 says the model adds nothing to what the attacker could already guess from demographics; a positive advantage says the model is leaking.
- **Calibration** of the attacker's confidence: how often is the attacker *confidently right*? A model can leak in the tail (a small number of very-confident correct inferences) while the average advantage looks modest.

### How this row lands on the register

Attribute inference is a **model-caused** or **data-caused** row depending on the root cause: model-caused if the leakage comes from overfitting, data-caused if the sensitive attribute was *in the training data with no legitimate purpose*. Chapter 06 uses the cause bucket to route the mitigation — DP-SGD closes the model-caused leak; column-dropping or lawful-basis review closes the data-caused leak.

## 2. Model inversion

### What the attacker asks

Given the trained model, produce a plausible reconstruction of a training example — for instance, a face image that a human labeller would identify as the correct labelled class in a per-person face recogniser.

### Canonical publication

- Fredrikson, Jha, Ristenpart (CCS 2015) — "Model Inversion Attacks that Exploit Confidence Information and Basic Countermeasures" — the reference reconstruction attack on face-recognition classifiers.

Later work (Zhang et al., "Secret Revealer" CVPR 2020 and successors) sharpens the reconstruction using generative priors; treat those as attack-strength ceiling references, not baseline.

<!-- needs-research: confirm the current strongest published model-inversion baseline for face recognisers and its metric convention at authoring time. -->

### Threat model

- Query access to the model (probabilities per class, or full logits).
- Knowledge of the label space (per-person classifiers make the label space *the identities*).
- No training-data access.

Model inversion is most damaging on **small-population classifiers**: per-person face recognisers, per-user personalisation models, per-patient models. The class *is* the person; a plausible reconstruction of the class is a plausible reconstruction of the person.

### Pipeline

1. **Fix a target class** (an identity, for a face recogniser).
2. **Optimise an input** — starting from a random image or a prior — to maximise the target class's confidence under the model.
3. **Report** the reconstruction with a quality metric: structural similarity to a reference training image (if you have one — for the eval), human-recognisability rating (crowdsourced or expert), or a task-specific classifier's confidence on the reconstruction against the ground-truth identity.

### Residual-risk signal

Two numbers, together:

- **Reconstruction quality** on a chosen metric (SSIM, LPIPS, or human-recognisability).
- **Identifiability** — whether the reconstruction is a face a human can *name*. This is the qualitative claim; the quality metric is the quantitative one. Both are reported.

A reconstruction that scores well on SSIM but is unrecognisable is a low-severity finding; a reconstruction a human can name is high-severity, regardless of SSIM.

### Register row

Model inversion is a **model-caused** row (small class sets and confident classifiers are the underlying enabler). Defences: don't expose confidence vectors on production APIs (return top-1 label only), DP-SGD training, larger class sets, or don't build per-person classifiers at all where the use case does not require them.

## 3. LLM memorisation and verbatim extraction

### What the attacker asks

*Can I make the LLM emit verbatim training-data strings — names, phone numbers, addresses, code secrets, private notes — in response to natural or adversarially crafted prompts?*

### Canonical publications

- Carlini, Tramèr, Wallace, Jagielski, Herbert-Voss, Lee, Roberts, Brown, Song, Erlingsson, Oprea, Raffel (USENIX Security 2021) — "Extracting Training Data from Large Language Models" — the GPT-2 extraction study; introduced the *k-eidetic memorisation* definition and demonstrated extraction of hundreds of unique verbatim strings.
- Carlini, Ippolito, Jagielski, Lee, Tramèr, Zhang (ICLR 2023) — "Quantifying Memorization Across Neural Language Models" — the scaling law: memorisation rises with model size, training-data duplication, and prompt-prefix length.
- Nasr, Carlini, Hayase, Jagielski, Cooper, Ippolito, Choquette-Choo, Wallace, Tramèr, Lee (2023) — "Scalable Extraction of Training Data from (Production) Language Models" — extraction from production aligned chatbots via divergence prompting (the "repeat this word forever" attack).

### The two axes: memorisation vs. extraction

- **Memorisation** is a property of the model: given the model and the training set (which you as evaluator have access to for a fine-tuned model or a canary-planted model), how many training strings can be elicited verbatim under a prompt budget?
- **Extraction** is a property of the attacker: given only the model, how many training strings can the attacker recover without training-set access, verifying by cross-checking against a public corpus or against structural regularities (SSN patterns, e-mail formats)?

Memorisation is the tighter measurement; extraction is the more realistic threat.

### Canary-based memorisation eval (recommended for level 25)

The most defensible memorisation eval is a **canary-planted** one:

1. Before fine-tuning (or before any training run you control), plant a labelled set of *canary* strings — synthetic, unique, structurally realistic (fake SSNs, fake e-mails, fake phone numbers, fake credentials) — at controlled duplication counts in the training data. Record the canary strings and their duplication counts.
2. Fine-tune the model.
3. Query the model with prompts designed to elicit the canaries: prefixes drawn from the surrounding training context; direct completion prompts; the divergence-prompting patterns from Nasr et al.
4. **Measure extraction rate** = fraction of canaries successfully elicited verbatim within a prompt budget, per duplication-count bucket.
5. Report the extraction-rate table by duplication count. The Carlini 2023 scaling result is: memorisation rises with duplication; canaries at duplication 1 are rarely elicited; canaries at duplication 10-100 are frequently elicited on capable models.

Two engineering choices to be explicit about:

- **Prompt budget.** State the number of prompts per canary attempted (say, 100, or 1000). Extraction rate is meaningless without it.
- **Elicitation strategies attempted.** State the full list (direct completion, prefix-from-context, divergence prompting, etc.). Missing an obvious attack strategy is an eval defect, not evidence of a safe model.

### Alternative: production-shape extraction eval without canaries

If you cannot plant canaries (e.g., evaluating a base model you did not train), the eval is *lower-bound* extraction: attempt to elicit strings that are known-in-training (matched against a public corpus, or against a known-in-your-corpus reference file), report the count. This is a valid eval — it is what Carlini 2021 and Nasr 2023 do at production scale — but the number is a lower bound on model memorisation, not a per-canary rate.

### Residual-risk signal

The reporting table for a canary-based eval:

| Canary duplication count | Prompts per canary | Extraction rate | Notes |
|---|---|---|---|
| 1 | 100 | 0.00 | Below noise floor at this budget |
| 3 | 100 | 0.01 | 1/100 canaries elicited |
| 10 | 100 | 0.07 | Rising |
| 30 | 100 | 0.24 | Rising sharply |
| 100 | 100 | 0.65 | Highly duplicated strings routinely elicited |

The shape of the curve is more informative than any single number. A curve that rises sharply at low duplication signals a model with high memorisation propensity; a curve that stays low across the whole range signals meaningful protection.

### What defends against memorisation

Layered:

- **De-duplication of training data** (Lee et al., "Deduplicating Training Data Makes Language Models Better", ACL 2022) — the largest single lever, since duplication is the strongest predictor of memorisation.
- **DP-SGD training** (chapter 04) — provides a formal bound; effective but with utility cost.
- **PII redaction on training corpora** (chapter 05) — Presidio-shaped ingest-side pipelines remove candidate secrets before they enter training.
- **Output-side filters at inference** (mod-108) — catch known-secret patterns before they leave the model.
- **Prompt filters against divergence attacks** — the "repeat this word forever" pattern of Nasr et al. is now well-known and provider APIs typically block it; internally-hosted models need their own filter.

## Rolling up the three attacks into the register

Continuing the mod-103 register pattern, an internal fine-tuned LLM's privacy row might look like:

| Field | Inherent (no DP, no dedup, no redaction) | Residual (dedup 1x; Presidio redaction; DP-SGD ε=8; divergence-prompt filter) | Control-defeated: divergence filter off | Control-defeated: PII redaction off |
|---|---|---|---|---|
| Extraction rate at dup=10, 100 prompts / canary | 0.28 | 0.02 | 0.05 | 0.02 (redaction removed the canaries before training; already gone) |
| Extraction rate at dup=100, 100 prompts / canary | 0.81 | 0.09 | 0.19 | 0.09 |
| PII-string elicitation rate (real PII in fine-tune data) | Not meaningful — no defences | 0.005 | 0.011 | 0.14 (this is the load-bearing control) |
| Credited controls | none | dedup + redaction + DP-SGD + divergence filter | (divergence filter absent) | (redaction absent) |

Two observations the report should draw explicitly:

- The **control-defeated variant that hurts most** identifies the *load-bearing* control. In this row it is the redaction, not the DP training — a useful signal that upstream data hygiene carries more of the risk mitigation than the DP knob.
- The residual number is not zero. That is deliberate; "no extraction possible" is not a claim these evals can support. What the report can defend is "extraction rate under threat model T, prompt budget B, and attack list A is X."

## Cross-references

- **mod-101** — GenAI Profile risk "data privacy" and RMF MEASURE 2.7 consume these numbers.
- **mod-102** — every attack row in this chapter has a corresponding harm-model scenario.
- **mod-103** — the residual and control-defeated columns above go straight onto the register.
- **mod-105** — model-inversion and MIA share adversarial-ML machinery; extraction shares LLM red-teaming machinery from mod-104.
- **mod-108** — output-side PII filters and divergence-prompt filters are guardrail catalogue entries.
- **mod-109** — canary elicitation monitored in production (shadow-run canaries against the live model at cadence) is a monitoring signal.

## Summary

- Three attacks in addition to MIA: attribute inference, model inversion, LLM verbatim memorisation / extraction. Each is engineered separately and reported separately.
- Attribute inference: measure *advantage over base rate* per sensitive attribute; the row's cause bucket routes the fix (DP-SGD for model-caused, column drop for data-caused).
- Model inversion: measure *reconstruction quality* plus *identifiability*; small-population classifiers (per-person, per-user) are the danger shape.
- LLM memorisation: canary-planted evals produce the tightest number; report the extraction-rate curve by duplication count and prompt budget. Cite Carlini 2021, Carlini 2023, Nasr 2023 as the reference frame.
- The register row uses the numbers directly; the control-defeated column identifies the load-bearing control — often ingest-side data hygiene rather than the DP training.
- No eval supports the claim "no extraction possible." The claim is always "under threat model T, budget B, attacks A: X."
