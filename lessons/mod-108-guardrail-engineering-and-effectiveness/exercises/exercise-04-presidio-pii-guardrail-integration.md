# Exercise 04 — Microsoft Presidio Integration Across Ingest, Retrieval, and Output

**Estimated effort:** 2 hours

## Objective

Ship the **PII guardrail** layer for the running system's guardrail architecture by integrating **Microsoft Presidio** into the **three chapter-04 paths** — ingest, retrieval, output — with recogniser configurations tuned per path. Author at least one **custom recogniser** for a domain-specific identifier your running system uses. Prove the "PII already in the RAG store" mitigation pattern on a small audit of the store you have (or a synthetic stand-in). Exercise 05 will run the full FP / FN measurement per recogniser per path; this exercise ships the integrations and calibrates initial thresholds.

## Prerequisites

- Exercises 01–03 completed. The output-slot integration below plugs into the Guardrails AI guard from exercise 02; the retrieval-slot integration plugs into the NeMo GR retrieval rail or the LangChain retriever wrapper from exercise 02. The ingest-slot integration is new plumbing.
- Chapter 04 read; chapter 05 skimmed at the "FP / FN report" section so the tuning discipline lands in a shape the measurement can consume.
- The mod-107 PII taxonomy and the privacy-risk positions from mod-107 exercise 05 (Presidio pipeline). If mod-107 exercise 05 already stood up a Presidio pipeline for privacy-risk purposes, this exercise **integrates** those pipelines into the guardrail-slot architecture — do not fork; extend.
- Python environment with:
  - **`presidio-analyzer`** and **`presidio-anonymizer`** at pinned versions.
  - **`presidio-image-redactor`** if your running system ingests images or PDFs (optional).
  - The appropriate **spaCy language model** for your corpus (`en_core_web_lg` for English or the language-specific analogue).
- The **running system's data corpus** — a slice of the actual RAG store (or fine-tuning corpus, if RAG is not the shape), plus a **labelled evaluation slice** (at least 100–300 labelled spans across at least 3 entity types) drawn from production-shape data. Presidio's precision / recall on your corpus cannot be measured without ground truth; label the slice or reuse the mod-107 exercise-05 labelling if it exists.
- A **RBAC stub or the running system's actual access-control layer** for the retrieval-side per-caller filtering. Chapter 04 §"Retrieval path" is explicit that the caller-aware scan matters at retrieval.

## Problem statement

Deploy Presidio in the three chapter-04 paths and integrate each into the slot the guardrail architecture from exercise 01 designated. Ship a custom recogniser for at least one domain-specific identifier. Prove the audit pattern on the existing store.

### The three integration paths

Chapter 04 §"The three integration paths" is the reference. For each path:

- **Ingest path.** In the pipeline that populates the RAG store, fine-tuning corpus, or evaluation set. Runs the full analyzer (regex + context + spaCy NER) offline where latency is not a concern. Anonymises with a placeholder-preserving strategy (`<EMAIL_1>`, `<CREDIT_CARD_1>`) or pseudonymises with a keyed token to a controlled vault. Documents rejected records to a review queue.
- **Retrieval path.** Wraps the retriever's returned chunks before assembly. Uses a **fast** analyzer configuration (regex + context only, no heavy NER) because it sits in the request path and adds to time-to-first-token. Applies per-caller entitlement — the caller is entitled to see PII types `{X, Y}`, redacts `{Z, W}` — using the RBAC layer.
- **Output path.** Scans the model response. Uses per-field policy on structured JSON responses (chapter 02 §"A worked composition" and chapter 04 §"Output path"). Whitelists PII entities the caller supplied in their own turn or that appear in authorised retrieved context; redacts the rest. Logs every redaction event for the monitor.

The output-path integration plugs into the Guardrails AI guard from exercise 02 as a validator; do not build a separate output layer.

### The custom recogniser

Chapter 04 §"Custom recognisers" is explicit that the built-in Presidio set does not cover domain-specific or locale-specific identifiers. Author at least one:

- Choose an identifier your running system uses that Presidio does not cover: internal ticket ID (`TKT-NNNNNNNN`), customer account number with a company-specific prefix, employee ID, medical device serial, an industry-specific policy or contract number, a jurisdiction-specific national ID / tax number.
- Ship as a `PatternRecognizer` (regex + optional check-digit + context word list) registered against the analyzer at each path where it fires (usually ingest and output; sometimes retrieval).
- Author a labelled test set of **at least 20 positive and 20 negative** examples pulled from real corpus data (not invented — chapter 04 §"Custom recognisers" is explicit; invented examples miss the surprising false positives that real data contains).
- Calibrate the recogniser's `score` against the labelled test set; do not hard-code `score=1.0`.

### The "PII already in the store" audit

Chapter 04 §"The 'PII already in the RAG store' mitigation" describes the systemic mitigation. Prove the pattern on either the real store (if you have it) or a small synthetic stand-in that mimics the shape:

- Batch-run the full Presidio analyzer (including heavy transformer-based recognisers, since this is offline) against the store.
- Produce a report naming every chunk that contains detected PII, the recogniser that fired, and the retrieval-frequency band of the containing document (if the running system tracks retrieval frequency).
- Draft the prioritised remediation queue (chapter 04 §"Mitigation pattern" step 2).
- Draft the temporary hardened retrieval-side threshold that runs while the audit is remediated (step 4).

Do **not** actually remediate the whole store in this exercise — that is a production project. Producing the audit report, the prioritised queue, and the interim-hardening threshold is enough to prove you can execute the pattern.

## Requirements

Produce all eight items:

1. **Ingest pipeline** — `risk/mod-108/<system-id>/presidio/ingest.py`. Reads records; runs analyzer with the ingest-path threshold and the full recogniser set (built-in + custom); anonymises with placeholder-preserving strategy; writes anonymised records and a `detection-log.jsonl` naming detection counts by entity type and source document. If a record contains a detected entity above the "reject" threshold, routes it to a review queue file instead of the index.
2. **Retrieval pipeline** — `risk/mod-108/<system-id>/presidio/retrieval.py`. Wraps the running system's actual retriever (LangChain retriever wrapper, LangGraph node, or the NeMo GR retrieval action from exercise 02). Runs the fast analyzer; applies the RBAC / entitlement stub; emits sanitised chunks and a `chunk-redaction-log.jsonl`. Latency instrumentation (p50 / p95 added to the retriever call) is required — chapter 05 §"Latency accounting" is the discipline.
3. **Output pipeline** — `risk/mod-108/<system-id>/presidio/output.py`. Runs as a Guardrails AI validator on the response (structured or free-text). Whitelists PII the caller supplied in their turn and PII present in authorised retrieved context; redacts everything else; emits a `response-redaction-log.jsonl`. Per-field policy is applied where the response is structured.
4. **Custom recogniser** — `risk/mod-108/<system-id>/presidio/custom_recognizers/<entity>.py` — a `PatternRecognizer` registered against the analyzer, plus `custom_recognizers/<entity>_tests.json` with the 20+20 labelled positive / negative examples, plus a small pytest / assertion script that confirms precision / recall on the test set is above the threshold you claim.
5. **Threshold-per-path config** — `risk/mod-108/<system-id>/presidio/thresholds.md`. Per recogniser per path: the confidence threshold in force, the reason (chapter 04 §"Common recogniser tuning" gives per-path defaults), the fixture examples that motivated the threshold. This is what exercise 05 will run the full FP / FN measurement against.
6. **Store audit** — `risk/mod-108/<system-id>/presidio/store-audit.md`. The full-scan report on the store slice (real or synthetic stand-in); the prioritised remediation queue; the temporary retrieval-side hardened threshold. Cross-reference to a mod-107 privacy-risk register row for the "PII in the store" scenario.
7. **PII layer trace-format update** — extend the exercise-02 trace record schema to carry Presidio-specific fields (`entity_type`, `confidence`, `recogniser`, `action_taken`, `caller_entitled_types`). Update `risk/mod-108/<system-id>/rails/traces/README.md`; mod-109 consumes this format.
8. **Anti-pattern probe report** — `risk/mod-108/<system-id>/presidio/anti-patterns.md`. Four sections, one per chapter-04 anti-pattern:
   - **Single-path Presidio.** Confirmed all three paths are integrated. If a path is missing (e.g., no ingest control because the RAG store is legacy), the mitigation from §"The 'PII already in the RAG store' mitigation" is filed.
   - **Uniform threshold.** Confirmed each path uses a different threshold configuration.
   - **Unfixtured tuning.** Confirmed every recogniser has a fixture set that regressions are gated against.
   - **Pseudonymisation without key management.** If pseudonymisation is used, confirmed the key management boundary is documented and lives outside Presidio (in mod-107's privacy engineering, in the security track's KMS).

Cross-cutting rules:

- **Presidio is not ground truth** (chapter 04 §"Recogniser tuning — the FP / FN accounting"). Every precision / recall number is against your labelled eval slice, not against the world's PII. Document the labelling provenance.
- **Ship all three paths, not one and a promise** (chapter 04 §"Motivation"). Ingest-only or output-only is a broken shape; if a path is genuinely infeasible in this exercise (e.g., the ingest pipeline is not yours to modify), the audit-and-hardened-retrieval mitigation covers it — but say that explicitly.
- **Latency at the retrieval path is a first-class concern.** The retrieval-side scan sits in the request path. Use the fast recogniser configuration; measure and report the p50 / p95 added latency; if it exceeds budget, escalate rather than silently accept.
- **Per-turn whitelist logic is not global.** The output-path whitelist is derived per-turn from the caller's input and the authorised retrieved context; do not persist it across turns. Chapter 04 §"Output path" and mod-107 exercise-05 §"Output-side whitelist logic is subtle" both name this.
- **Custom recogniser calibration is required.** A recogniser with `score=1.0` and no test set is a red flag. Score calibrated on the test set.

## Starter guidance

- **Reuse the mod-107 exercise-05 Presidio pipeline as the starting point if it exists.** Chapter 04 §"Cross-references" points at mod-107 for the taxonomy; the pipeline shape lives there. This exercise's contribution is the integration into the guardrail-slot architecture.
- **20 records first, then scale.** Chapter 04's implicit discipline (and mod-107 exercise-05 §"Starter guidance") — run Presidio on 20 labelled records, hand-check the flagged spans, tune, then scale. Do not run against 300 records and then debug why the flags are off.
- **The confidence threshold is a knob, not a fixed value.** Presidio's default is often 0.5; the right value depends on the recogniser and the corpus. Sweep 5–7 values per recogniser per path; pick the one that maximises F1 on the calibration slice for the entity types that matter.
- **Context words lift precision on custom recognisers.** A regex for an internal ticket ID that also requires "ticket", "invoice", or "case" within a small window is much less noisy. Chapter 04 §"Custom recognisers" is the reference.
- **The audit stand-in can be small.** If your running system's store is under NDA or not readable at exercise time, build a small synthetic stand-in (50–200 chunks) that mimics the shape; the audit pattern is what the exercise proves, not the size of the audit.
- **RBAC stub is fine.** For the retrieval-side per-caller filtering, a hand-rolled `entitled_types = {"EMAIL_ADDRESS", "PHONE_NUMBER"}` per test user is enough to exercise the pattern. Real ACLs are the concern of the security track.
- **The three log files feed mod-109.** Author the log schemas with mod-109's monitor in mind; per-day per-entity-type detection counts are the first metric the monitor will read.

## Acceptance criteria

The submission is accepted when:

- All three Presidio pipelines (ingest, retrieval, output) exist and are wired into the running system's actual serving path.
- The output pipeline is wired into the exercise-02 Guardrails AI guard as a validator.
- The retrieval pipeline is wired into the actual retriever, with latency instrumentation reporting p50 / p95.
- The custom recogniser exists, has a labelled test set (20+20 minimum from real data), and its `score` is calibrated on the test set.
- The threshold-per-path config documents every recogniser's threshold at every path with a defended reason.
- The store audit report exists (real store or synthetic stand-in) and produces the prioritised remediation queue and the interim-hardening threshold.
- The trace record schema is extended with Presidio fields and `traces/README.md` is updated.
- The anti-pattern probe report covers all four chapter-04 anti-patterns and either resolves each or files a follow-up.
- Every Presidio version, spaCy model version, and custom-recogniser test-set fingerprint is pinned in a manifest.

## Stretch goals

- **A second custom recogniser for a locale-specific identifier.** If your running system serves more than one jurisdiction, author a recogniser for a national-ID / tax-number format Presidio's built-in set does not cover in that locale. Score against a locale-specific labelled slice.
- **PDF / image ingest with Presidio Image Redactor.** If your running system ingests PDFs or images (OCR'd), add the image redactor to the ingest pipeline and report precision / recall on a small labelled OCR slice. Structured PII in image form is where the ingest layer often leaks.
- **The full "RAG store PII" end-to-end pattern.** Beyond the audit-and-hardened-retrieval interim, ship the full chapter-04 §"Mitigation pattern" flow on a subset of the store: tag at ingest (retain span metadata but not the raw PII on the default retrieval surface), redact-by-default at retrieval, un-redact only on authorised entity types, filter again on output. Report the residual claim in the register-row shape (chapter 04 §"Cross-references" to mod-103).
- **A recogniser that catches semantic PII the built-in set does not.** Chapter 04 §"Motivation" and mod-107 exercise-05 name semantic PII (the customer's employer, an inferred medical condition) as beyond Presidio's default. Ship a supplementary classifier (a small fine-tuned NER, or a Constitutional-Classifier-style scorer) that catches one semantic-PII case, and integrate it as a Presidio custom recogniser or as a separate validator in the output guard. Report precision / recall.
- **A monitor stub that reads the three log files.** Read `detection-log.jsonl`, `chunk-redaction-log.jsonl`, and `response-redaction-log.jsonl`; emit per-day per-entity-type counts and a drift alarm on any category whose count moves >2 standard deviations. This is a mod-109 preview.

## Deliverable format

A single markdown document `mod-108-ex04-presidio.md` that indexes the artefacts. The index document contains:

1. *System summary* — one paragraph naming the running system, the three Presidio integration points, and whether the RAG store is the real one or a stand-in.
2. *Presidio and dependency versions pinned* — Presidio analyzer, anonymiser, image redactor (if used), spaCy model + version.
3. *Ingest pipeline* — the shape; the detection log location; the review queue behaviour.
4. *Retrieval pipeline* — the shape; the RBAC / entitlement layer used; the latency instrumentation numbers.
5. *Output pipeline* — the shape; the per-turn whitelist logic; the wiring into the Guardrails AI guard.
6. *Custom recogniser* — the entity name, the regex, the context words, the test-set precision / recall, the calibrated `score`.
7. *Threshold-per-path config* — the block from `thresholds.md`.
8. *Store audit* — the block from `store-audit.md`; the prioritised queue; the interim-hardening threshold.
9. *Trace format update* — the diff to `traces/README.md`.
10. *Anti-pattern probe report* — the four sections.
11. *Open items* — any path not fully integrated; any recogniser that needs replacement; any pseudonymisation key-management gap escalated to mod-107 or the security track.

## Submission

Attach the index markdown plus the eight artefacts. The three log files (or their fingerprints if they contain real PII) attach as evidence. If the store audit was run on real PII data, redact the sample rows but preserve the counts and the recogniser attribution — a reviewer cannot review a redaction pipeline whose numbers are blanked out.
