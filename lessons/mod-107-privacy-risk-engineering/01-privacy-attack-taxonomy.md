# The Privacy Attack Taxonomy This Module Engineers Against

Privacy risk for ML and LLM systems is not a single failure mode. It is a small family of *empirically distinct* attacks — membership inference, attribute inference, model inversion, memorisation / extraction — each with its own evaluation, its own defence, and its own legal footprint. This chapter fixes the taxonomy the rest of the module works against, and pins each attack to the standards vocabulary (NIST AI 100-2 e2025) and the regulatory vocabulary (GDPR, NIST Privacy Framework) so that when your report says "residual MIA TPR at 1% FPR is X," a governance reader can trace the number to a legal concept.

## Motivation

An engineer who says "the model is privacy-safe" is asserting four separate things at once, whether or not they know it:

1. That an adversary who queries the trained model **cannot reliably decide whether a specific record was in the training set** — the membership-inference claim.
2. That an adversary who knows some attributes of an individual **cannot infer the sensitive missing attributes** from model outputs — the attribute-inference claim.
3. That an adversary **cannot reconstruct a plausible representation of a training example** from model outputs — the model-inversion claim.
4. That the model **does not emit verbatim training data** in response to natural or adversarially crafted prompts — the memorisation / extraction claim.

Each claim needs its own evaluation, its own metric, its own defence, and its own control-defeated row in the mod-103 register. Collapsing them into one sentence is how privacy incidents get missed at review.

The rest of this module engineers the four evaluations and the two main defences (DP-SGD and PII pipelines) that answer them.

## The four attacks

### 1. Membership inference (MIA)

**Question the attacker asks.** *Was record X in the model's training set?*

**Why it matters.** Membership itself is often the sensitive fact: membership in a health-outcome cohort, in a fraud-suspect dataset, in a labelled clinical-trial arm, in a labelled political-preference dataset. Under GDPR, membership in any dataset whose subject population is a special category (Article 9 — health, ethnic origin, religion, biometrics for ID, sexual orientation, trade-union membership) is itself special-category personal data, and disclosure is a controller obligation.

**Canonical publications.**
- Shokri, Stronati, Song, Shmatikov (S&P 2017) — "Membership Inference Attacks Against Machine Learning Models" — the shadow-model attack that seeded the field.
- Carlini, Chien, Nasr, Song, Terzis, Tramèr (S&P 2022) — "Membership Inference Attacks From First Principles" — the Likelihood Ratio Attack (LiRA) and the argument that AUC averages hide the useful signal; report TPR at very low FPR instead.

**Working eval harness.** ML Privacy Meter (NUS Data Security and Systems Group). Chapter 02 walks the build.

**Residual-risk signal.** LiRA-style TPR at low FPR (typically 0.1% and 1%), reported per-population slice. Chapter 02 discusses why AUC alone is not enough.

### 2. Attribute inference

**Question the attacker asks.** *Given some attributes of an individual, can I infer sensitive missing attributes from the model's behaviour on that individual's partial record?*

**Why it matters.** Attribute inference converts a *seemingly innocuous* deployed model into an inference oracle for the sensitive fields the training data contained. A model trained with a sensitive column can leak that column even when the model's task has nothing to do with it — the leakage is a side-effect of overfitting on the joint distribution.

**Canonical publications.**
- Fredrikson, Lantz, Jha, Lin, Page, Ristenpart (USENIX Security 2014) — "Privacy in Pharmacogenetics: An End-to-End Case Study of Personalized Warfarin Dosing" — the first widely-cited attribute-inference attack on a real deployed model.
- Fredrikson, Jha, Ristenpart (CCS 2015) — "Model Inversion Attacks that Exploit Confidence Information and Basic Countermeasures" — combines attribute inference with reconstruction on vision.

**Residual-risk signal.** Advantage over an attacker with only the marginal / base-rate prior; report separately per sensitive attribute.

### 3. Model inversion

**Question the attacker asks.** *Given the trained model, can I reconstruct a plausible representation of a training example — for instance, a recognisable face for a labelled class?*

**Why it matters.** Model inversion is what makes small-population classifiers (per-person face recognisers, per-user personalised models) inherently privacy-fragile: the "class" is a person, and a plausible reconstruction of the class is a plausible reconstruction of the person's likeness.

**Canonical publication.** Fredrikson, Jha, Ristenpart (CCS 2015), as above.

**Residual-risk signal.** Reconstruction quality (structural-similarity, human-recognisability, or a task-specific metric); combined with the *identifiability* claim — a reconstruction that a human can name is a different kind of finding from one that only preserves coarse features.

### 4. Memorisation and extraction (LLM-specific)

**Question the attacker asks.** *Can I make the model emit verbatim training-data strings — a phone number, a code secret, a full personal note?*

**Why it matters.** Modern LLMs are trained on datasets large enough that individual records are, in principle, memorised at low rate; and memorisation is *sample-efficient* to elicit if the prefix is known. Under GDPR this converts training-data provenance into an ongoing controller obligation: the model itself may be personal data with respect to a subject whose record was memorised.

**Canonical publications.**
- Carlini, Tramèr, Wallace, Jagielski, Herbert-Voss, Lee, Roberts, Brown, Song, Erlingsson, Oprea, Raffel (USENIX Security 2021) — "Extracting Training Data from Large Language Models" — the seminal GPT-2 extraction study.
- Carlini, Ippolito, Jagielski, Lee, Tramèr, Zhang (ICLR 2023) — "Quantifying Memorization Across Neural Language Models" — scaling law for verbatim memorisation.
- Nasr, Carlini, Hayase, Jagielski, Cooper, Ippolito, Choquette-Choo, Wallace, Tramèr, Lee (2023) — "Scalable Extraction of Training Data from (Production) Language Models" — extraction from production-scale aligned models via divergence prompting.

**Residual-risk signal.** Extraction rate on a labelled canary set at a stated prompt budget; verbatim k-eidetic memorisation counts (Carlini 2021 definition) if you have training-set access.

## Mapping to NIST AI 100-2 e2025 (Adversarial ML taxonomy)

NIST AI 100-2 e2025 ("Adversarial ML: A Taxonomy and Terminology of Attacks and Mitigations") groups attacks by learning stage (training vs. inference) and by attacker goal (availability / integrity / privacy). This module lives in the **privacy** column of that taxonomy. The mapping this module uses:

| This module's attack | NIST AI 100-2 e2025 category |
|---|---|
| Membership inference (MIA / LiRA) | Privacy → membership inference |
| Attribute inference | Privacy → attribute inference / property inference |
| Model inversion | Privacy → model inversion / reconstruction |
| Memorisation / verbatim extraction | Privacy → data extraction / training-data extraction (GenAI-specific rows) |

The publication is versioned; treat the taxonomy rows above as the direction and pin the exact §-numbers at authoring time.

<!-- needs-research: pin the exact section identifiers and any new privacy sub-categories in the current published revision of NIST AI 100-2 e2025 at authoring time. -->

## Mapping to GDPR concepts

GDPR (Regulation (EU) 2016/679) is the reference regulation for this module's report shape. The relevant concepts each attack invokes:

| Attack | GDPR concept invoked | Article |
|---|---|---|
| Membership inference | *Personal data* if the training-set identity is identifiable; *special-category personal data* if the set encodes any Article 9 category | Article 4(1), Article 9 |
| Attribute inference | *Personal data* (the inferred attribute is personal data about the subject); *profiling* if the inference feeds a decision | Article 4(1), Article 4(4) |
| Model inversion | *Personal data* (a reconstructed identifier is personal data); *biometric data* if the reconstruction is a face | Article 4(1), Article 4(14), Article 9(1) |
| Memorisation / extraction | *Personal data* if extracted strings identify a subject; potentially *processing without a lawful basis* if the training data was scraped | Article 4(1), Article 6, Article 9 |
| Any attack driving an automated decision | *Automated individual decision-making, including profiling*; subject has rights under Article 22 | Article 22 |
| Any attack with material risk to subjects | Controller obligation to run a *Data Protection Impact Assessment* | Article 35 |

Chapter 06 turns these mappings into the report structure. Legal *interpretation* is out of scope to counsel; this module gives you the technical evidence counsel needs, not the legal opinion on top of it.

## Mapping to the NIST Privacy Framework

The NIST Privacy Framework (v1.0, January 2020) organises privacy-risk work into five functions, each split into privacy-specific categories and sub-categories. Each attack in this module maps naturally into one or more functions:

| NIST PF Function | What it asks | Where this module contributes |
|---|---|---|
| **IDENTIFY-P** | Understand privacy risk — inventory data, processing, and privacy risks | The four attack evaluations are the *privacy-risk assessments* that IDENTIFY-P asks for |
| **GOVERN-P** | Policies, processes, and workforce for privacy risk | You feed engineering-side evidence that the policies are testable and tested |
| **CONTROL-P** | Enable data-processing management by individuals and organisations | Presidio redaction, DP-SGD (ε, δ) budgets, and the retrieval-side filters are CONTROL-P mechanisms |
| **COMMUNICATE-P** | Enable dialogue about privacy risks with stakeholders | The privacy-risk report (chapter 06) is the communication artifact |
| **PROTECT-P** | Data-processing safeguards | DP-SGD, redaction, output filtering, access controls |

The NIST Privacy Framework references NIST SP 800-53 privacy controls; treat 800-53 rev 5 Appendix J and the PF Core as the joint reference.

## Where each attack surfaces in the harm model (mod-102) and register (mod-103)

Every attack in this module is a *scenario row* in the mod-102 harm model and a *quantified row* in the mod-103 register.

- The **persona** field on the row is usually `adversarial user` (for query-time attacks) or `downstream integrator` (for extraction of the model's outputs into third-party context). Occasionally `insider` (a partner with elevated query rate).
- The **capability surface** is the model's *query interface* — probability outputs, top-k labels, log-probabilities on a token stream, embedding vectors, sampled generations. Chapter 02 uses this to explain why LiRA needs logits or losses and why an API that returns only labels bounds the attack.
- The **cause bucket** is usually `model-caused` (memorisation, overconfidence on training points) but can be `data-caused` (sensitive attribute in training data with no purpose) or `integration-caused` (PII in a RAG store).
- The **residual-risk score** in mod-103 is directly the MIA / LiRA TPR@low-FPR (or the attribute-inference advantage, or the extraction rate) after the credited defences are applied. Chapter 06 shows the row shape.
- The **control-defeated** column is what happens when a defence is bypassed: DP-SGD noise multiplier tuned down for utility, Presidio recogniser off, output filter muted, retrieval-side allow-list disabled. Each is a monitoring priority.

## Attack surface vs. deployment shape

Not every attack applies to every system. This module teaches you which apply to which shape.

| Deployment shape | MIA | Attribute inference | Model inversion | Extraction / memorisation |
|---|---|---|---|---|
| Public prediction API (classification) | High relevance | High relevance where sensitive columns in training | High relevance for small class sets (e.g. per-person recognisers) | n/a |
| Public prediction API (regression) | Relevance depends on output continuity; MIA weaker in principle | High relevance | Lower relevance | n/a |
| Internally-hosted LLM (not fine-tuned on customer data) | Low relevance (train data is public-ish) | Low | Low | Medium — depends on training corpus |
| Internally-hosted LLM (fine-tuned on customer data) | High relevance | High relevance if PII in fine-tune corpus | Low | High — extraction is the primary risk |
| RAG system on internal documents (no fine-tune) | Not directly on the base model | Not directly | Not directly | Not from the model — from the *retrieval store*, which is the PII-pipeline problem (chapter 05) |
| Embedding API | Membership on the embedded corpus | Attribute inference through embedding similarity | Inversion possible on some embedding models | n/a |

The rows above are engineering rules of thumb, not legal advice. Chapter 06 is where they turn into a report row.

## Where this module *does not* go

To keep the boundary clean:

- **Cryptographic privacy techniques** — homomorphic encryption, secure multi-party computation, secure aggregation — are named where they intersect DP training but are not engineered here. They belong to a separate specialisation.
- **Federated learning attack surface** — client-update leakage, gradient inversion — is real but out of scope; DP-SGD's role in federated settings is noted in chapter 04 without being the module's focus.
- **Legal interpretation of GDPR / EU AI Act privacy provisions** is out of scope to counsel. This module produces the evidence; counsel produces the opinion. Chapter 06 draws the line.
- **Deeper methodological calibration of eval error bars, statistical-significance framing, calibration under distribution shift** lives with `model-evaluation-engineer` at level 30. This module gives you the evaluation shape sufficient to file a residual-risk claim; the peer role gives it audit-grade rigour.

## Cross-references

- **mod-101** — the framework tie-in. The NIST GenAI Profile lists "data privacy" and "information integrity" among the twelve risks; the EU AI Act cites GDPR at Article 10 (data governance) and elsewhere. This module engineers evaluations against those obligations.
- **mod-102** — every scenario evaluated here starts as a row in the harm model.
- **mod-103** — every attack's numeric output becomes a residual-risk score (and a control-defeated variant) on a register row.
- **mod-105** — MIA and extraction share adversarial-ML machinery (query-based attacks, gradient-based reconstruction); this module reuses that machinery for the privacy-goal column of NIST AI 100-2 e2025.
- **mod-108** — Presidio (chapter 05) is one of the guardrail components mod-108 catalogues; the DP-SGD-trained model itself is a control credited in the register.
- **mod-109** — the privacy signals produced here (extraction-canary hit rate, MIA drift, Presidio false-negative rate on shadow traffic) feed the continuous monitoring stack.

## Summary

- Privacy risk decomposes into four empirically distinct attacks: MIA, attribute inference, model inversion, memorisation / extraction. Each has its own eval and its own defence.
- The taxonomy pins to NIST AI 100-2 e2025 (privacy column) for the technical vocabulary and to GDPR / NIST Privacy Framework for the reporting vocabulary.
- Every attack maps to a scenario row in mod-102 and a quantified row in mod-103; the residual-risk column carries the MIA / LiRA TPR, the extraction rate, or the inference advantage.
- Attack applicability depends on deployment shape — public API vs. RAG vs. fine-tuned LLM. The applicability table above is the working rule of thumb.
- Legal interpretation is out of scope to counsel; deeper methodological calibration escalates to `model-evaluation-engineer` at level 30.
