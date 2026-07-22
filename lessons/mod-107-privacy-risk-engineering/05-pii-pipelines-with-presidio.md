# PII Detection and Redaction Pipelines with Microsoft Presidio

DP-SGD (chapter 04) bounds the leakage of records that entered training under a mechanism. Presidio bounds the leakage of *strings* — names, phone numbers, e-mails, national identifiers, credit-card numbers, per-jurisdiction identifiers — that flow through the system at ingest, inside a retrieval store, or on the way to a user. This chapter engineers Presidio into three places: **ingest** (before data lands in training or a RAG store), **retrieval** (inside the RAG loop), and **output** (before the model's response leaves the system). It then engineers the false-positive / false-negative accounting the report has to file.

## Motivation

Two common failure modes for a level-25 PII pipeline:

- **Presidio as a single output-side filter.** The pipeline redacts PII only in the model's outputs. Meanwhile PII enters the training corpus and the RAG store unfiltered, and the "leakage" that the extraction eval catches (chapter 03) is exactly the PII the ingest-side pipeline should have caught. The output filter is the last line of defence, not the whole defence.
- **Presidio treated as ground truth.** The pipeline redacts everything Presidio flags, ships, and calls it done. In production the false-negative rate turns out to be substantial on custom identifiers (internal user IDs, structured order numbers, jurisdiction-specific numbers Presidio does not recognise by default) and the false-positive rate turns out to be substantial in domains where names collide with common words. Neither number was measured.

This chapter engineers around both failure modes.

## What Presidio provides

**Microsoft Presidio** (`microsoft.github.io/presidio`, open source, MIT-licensed) is a Python library and container stack for PII detection and de-identification. Two components matter at level 25:

- **Presidio Analyzer** — detects PII entities in text using a combination of regex-based recognisers, spaCy / NER-model-based recognisers, checksum validators (Luhn for credit cards, national-ID checksums), and per-language rulesets. Output: a list of `RecognizerResult` objects with entity type, span, and confidence score.
- **Presidio Anonymizer** — takes analyser results and applies an anonymisation strategy: redact, replace with a placeholder, hash, encrypt (reversible with key), or apply a custom operator.

A separate **Presidio Image Redactor** handles OCR-based PII detection in images / PDFs; the pattern this chapter teaches applies to it too.

Entity types Presidio recognises out of the box include (non-exhaustive): `PERSON`, `EMAIL_ADDRESS`, `PHONE_NUMBER`, `CREDIT_CARD`, `IBAN_CODE`, `IP_ADDRESS`, `URL`, `US_SSN`, `US_ITIN`, `US_DRIVER_LICENSE`, `US_BANK_NUMBER`, `LOCATION`, `DATE_TIME`, `NRP` (nationality / religious / political group), `MEDICAL_LICENSE`, `UK_NHS`, `AU_TFN`, `SG_NRIC_FIN`, and language-specific national identifiers for a range of jurisdictions.

<!-- needs-research: pin the complete current list of built-in Presidio recognisers per jurisdiction at authoring time; the set grows release to release. -->

## The three pipeline positions

### 1. Ingest — before data enters training or a RAG store

**What it protects.** Training-data memorisation risk (chapter 03); PII surfacing later through retrieval. Ingest-side redaction is the single largest leverage point for LLM memorisation risk because a string that never entered training cannot be memorised.

**Shape.** A batch job (for training-data preparation) or a streaming pipeline (for continuously indexed RAG corpora):

1. Read record.
2. Presidio Analyzer over all text fields at a chosen confidence threshold.
3. Anonymize: redact or replace with a stable placeholder (e.g. `<PERSON_1>`, `<EMAIL_1>`).
4. Emit the anonymised record downstream.
5. Log detection counts and confidence distributions per entity type per source for monitoring.

**Engineering decisions:**

- **Placeholder vs. hash vs. drop.** Placeholders preserve structure (useful when the model must learn to handle "a person" without learning the specific person); hashes preserve joinability across records (useful when the same subject must be traceable across records for legitimate analytics); dropping the record is the strictest option.
- **Confidence threshold.** Presidio recognisers return a confidence in [0, 1]. Lower threshold catches more true positives at the cost of more false positives. The threshold is chosen against the labelled evaluation (below), not by feel.
- **Custom recognisers.** Every organisation has PII types Presidio does not know: internal user IDs of a specific shape, per-product order numbers, per-jurisdiction identifiers Presidio does not cover. Author a custom recogniser (below).

### 2. Retrieval — inside the RAG loop

**What it protects.** The "PII in the RAG store" problem. Even if the model is not fine-tuned on customer data, a RAG system that retrieves customer records and injects them into the prompt gives the model access to those records at inference time. Failure modes: (a) the retrieved chunk contains PII belonging to a different customer than the current user (cross-customer leakage); (b) the retrieved chunk contains PII the current user is not authorised to see; (c) the model quotes the chunk verbatim including the PII.

**Shape.** Presidio applied *after retrieval, before the chunk enters the prompt*:

1. Retrieval returns top-K chunks.
2. Per-chunk Presidio Analyzer detects PII entities.
3. Access-control check: is the current user authorised to see each entity? (Usually a *role-based* or *record-owner* check that is out of Presidio's scope — Presidio flags; the access layer decides.)
4. Anonymise / redact chunks where the user is not authorised; drop chunks where the entire chunk is another user's PII with no legitimate access.
5. Emit sanitised chunks into the prompt.

**Anti-pattern.** Trusting the vector store's access-control alone. Vector stores partition on a per-document ACL model that does not know about PII spans *within* a document. Presidio adds the span-level check.

### 3. Output — before the model's response leaves the system

**What it protects.** Verbatim quoting of retrieved PII; extraction attempts against the base model; failure modes where the model synthesises PII-shaped strings (hallucinated e-mails, fabricated phone numbers).

**Shape.** Presidio on the model's response, before it reaches the user:

1. Model generates response.
2. Presidio Analyzer detects PII entities in the response.
3. If PII is present and the entity was not present in the *user's own* input or in the authorised retrieved context, redact.
4. Log the detection; feed the count into monitoring (mod-109) as an extraction-attempt indicator.

**Engineering decisions:**

- **Whitelist the user's own PII.** A user asking about their own account is entitled to see their own e-mail in the response; redacting it breaks the product.
- **Log high-confidence detections that were passed through vs. redacted.** The log is evidence for the incident-response and monitoring stacks.

## Custom recognisers — the pattern

Every serious deployment authors custom recognisers for identifiers Presidio does not cover. The Presidio API supports both **pattern-based** recognisers (regex + context words + optional checksum) and **entity-recognizer** (NER-model-based) additions.

Skeleton for a pattern-based recogniser:

```
from presidio_analyzer import Pattern, PatternRecognizer

order_number_pattern = Pattern(
    name="internal_order_number",
    regex=r"\bORD-[A-Z0-9]{4}-\d{6}\b",
    score=0.85,
)

order_recognizer = PatternRecognizer(
    supported_entity="INTERNAL_ORDER_NUMBER",
    patterns=[order_number_pattern],
    context=["order", "invoice", "shipment"],
    supported_language="en",
)

analyzer.registry.add_recognizer(order_recognizer)
```

Engineering discipline:

- **Score calibration.** The `score` on the pattern is the recogniser's asserted confidence. Set it against the labelled evaluation; do not set it to 1.0 unless the pattern is checksummed.
- **Context words** raise the confidence when the pattern appears near the listed words (analogous to Luhn's role for credit-card numbers). Choose context words from real corpora, not from imagination.
- **Language.** Register per language; do not assume `en` covers French or Japanese documents.
- **Test set.** Every custom recogniser ships with a labelled test set of at least positive and negative examples pulled from real data; the recogniser's precision / recall on that set is a release artifact.

## False positives and false negatives — the accounting

The two error modes have different costs:

- **False negative** (missed PII) is a *leak*. This is the failure that the extraction eval (chapter 03) will catch, and the failure that produces a subject-access-request incident.
- **False positive** (redacted non-PII) is a *utility cost*. Over-redaction degrades RAG relevance, breaks legitimate product surfaces, and drives users toward workarounds that leak more.

Both have to be measured, on a labelled evaluation set, per entity type.

**Reporting shape**, per pipeline position:

| Entity type | Recogniser source | Precision | Recall | F1 | Notes |
|---|---|---|---|---|---|
| `EMAIL_ADDRESS` | Presidio default | 0.98 | 0.96 | 0.97 | Strong out of the box |
| `PHONE_NUMBER` | Presidio default | 0.92 | 0.87 | 0.89 | Misses international-format numbers without country code |
| `PERSON` | Presidio default (spaCy) | 0.78 | 0.82 | 0.80 | Confounded with common English words in unstructured chat |
| `INTERNAL_ORDER_NUMBER` | Custom pattern | 0.96 | 0.99 | 0.97 | Regex-anchored with checksum |
| `US_SSN` | Presidio default | 0.99 | 0.94 | 0.96 | High precision (checksum + context); recall gap on SSNs written without dashes |

Two disciplines that keep the numbers honest:

- **Evaluation set is drawn from real production-shape data**, not from Presidio's own test corpus. Presidio recognisers are typically well-tuned to public benchmarks; production data always looks different.
- **Slice the eval by language, domain, and formality.** English-formal customer-support text is different from Japanese chat is different from OCR'd PDFs.

## The RAG-store PII problem — the working shape

A concrete pattern for a customer-support RAG system with PII in the historical ticket corpus:

1. **Ingest-time filter.** Every ticket, upon indexing, passes through Presidio; entity spans are tagged (not necessarily redacted — the *span metadata* is retained). The vector store records both the raw text (encrypted at rest, retrievable only under audit) and a redacted-and-tagged copy that is the default retrieval surface.
2. **Retrieval-time filter.** On retrieval, the redacted-and-tagged copy is returned. If the requesting user has an authorisation grant for a specific entity type (e.g., their own account manager entity), the tag is un-redacted for that user.
3. **Prompt assembly.** The retrieved chunk enters the prompt with placeholders in the PII spans. The model reasons over the abstract structure, not the identity.
4. **Output-time filter.** The response is passed through Presidio one more time to catch synthesised PII (hallucinated names, e-mails).
5. **Monitoring.** False-positive / false-negative estimates are re-computed on a shadow-labelled sample monthly; drift is a mod-109 signal.

The residual claim this shape supports: *no unauthorised PII appears in the user's assembled prompt* (modulo the false-negative rate); *no PII appears in the model's response except PII from the user's own input or from authorised retrieved context* (modulo the false-negative rate).

## Where Presidio does not fix things

Three claims Presidio does not support:

- **Semantic PII.** "The customer whose birthday is next Tuesday and who lives in the town of X." Presidio detects `DATE_TIME` and `LOCATION`, but the *combination* that identifies a specific person is not caught by span-level detection. This is the k-anonymity / re-identification problem and needs a separate treatment.
- **Sensitive-inference PII.** A response that says "based on your medical history, X" reveals a sensitive category through the answer itself, not through a detectable identifier. Presidio catches nothing here; the mitigation is a topic classifier and a policy check.
- **Structured PII outside supported languages / jurisdictions.** Presidio has strong coverage for a set of languages and a set of national-ID types; identifiers outside that set need custom recognisers or replacement tooling.

Chapter 06's report explicitly names these boundaries.

## Cross-references

- **mod-101** — the RMF MEASURE 2.7 sub-category, and the GenAI Profile "data privacy" and "information integrity" risks, consume Presidio evidence.
- **mod-102** — the RAG-PII scenario (chapter 01's `HR-CS-INJ-01`-shaped rows extended to include PII leakage) is the harm scenario Presidio addresses.
- **mod-103** — Presidio precision / recall / F1 by entity type is what credits (or fails to credit) the redaction control in the residual column.
- **mod-105** — output-side redaction interacts with adversarial-ML — a prompt that induces the model to output PII in an obfuscated form (leet-speak, base64) requires layered defence (an output filter that decodes candidate strings before checking).
- **mod-108** — Presidio is a *catalogued guardrail component*; the mod-108 guardrail catalogue names it and characterises its false-positive / false-negative envelope.
- **mod-109** — false-negative-rate drift monitoring on a shadow-labelled sample is a mod-109 monitor.

## Summary

- Presidio (analyser + anonymiser) is the reference PII pipeline component. Deploy in three positions: ingest, retrieval, output — each with its own protective purpose.
- Ingest-side redaction is the single largest leverage point for memorisation risk; do not let PII into training or into a RAG store if it does not need to be there.
- Custom recognisers cover the identifiers Presidio does not know out of the box; ship each with a labelled test set and calibrated confidence.
- False positives (utility cost) and false negatives (leak) both matter; both are measured per entity type on production-shape data, reported as precision / recall / F1.
- The RAG-PII shape: tag at ingest, redact-by-default at retrieval, un-redact only on authorised entity types, filter again on output, monitor drift.
- Presidio does not fix semantic PII, sensitive inference, or PII outside supported languages; name those boundaries in the report.
