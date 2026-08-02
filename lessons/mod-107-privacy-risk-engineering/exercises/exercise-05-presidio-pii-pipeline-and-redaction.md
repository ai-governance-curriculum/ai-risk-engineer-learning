# Exercise 05 — Presidio PII Pipeline and Redaction Across Ingest, Retrieval, and Output

**Estimated effort:** 2 hours

## Objective

Assemble a **Microsoft Presidio** PII detection / redaction pipeline in the three chapter-05 positions — **ingest**, **retrieval**, **output** — for the running system (assume a RAG or fine-tune-plus-serving shape; if the running system is a pure classifier, use its input-side text fields as the ingest surface). Author at least one **custom recogniser** for an identifier Presidio does not cover out of the box; report **precision / recall / F1 per entity type** against a labelled evaluation slice drawn from production-shape data; and produce the credited-control evidence that the mod-103 register row for PII / memorisation consumes as the layered defence alongside the DP-SGD variant from exercise 04.

## Prerequisites

- Chapters 01 and 05 read; exercise 03 completed (the memorisation / extraction eval identifies which PII types the model has actually leaked, which is what an ingest-side redaction pipeline is designed to prevent).
- **The running system's data corpus.** Either a slice of the actual training / RAG corpus or a production-shape sample of similar text. The eval slice must be a **labelled** subset — spans annotated with entity type — either by you or by a small labelling pass. Aim for at least 200-500 labelled spans across at least 4 entity types.
- **Microsoft Presidio** installed at a pinned version: `presidio-analyzer` and `presidio-anonymizer` at minimum; the image redactor is optional.
- A spaCy language model appropriate to your corpus (`en_core_web_lg` for English, or the language-specific analogue). Presidio's default `PERSON` recogniser is a spaCy NER wrapper.
- **A retrieval / RAG plumbing surface** to integrate against. Either the running system's actual RAG stack or a small local stand-in (a FAISS or Chroma vector store loaded with 100-1000 document chunks that mimic the production shape). The exercise's point is not to build a RAG system but to insert Presidio at the three positions.
- **A serving surface** for the output-side integration — the running system's actual serving path or a thin FastAPI / Flask wrapper around the model.
- The mod-103 register rows updated by exercises 03 and 04. You will populate the **credited-control** column with Presidio evidence and the **control-defeated: recogniser disabled** column.

## Problem statement

Deploy Presidio in the three chapter-05 positions and measure its precision / recall per entity type per position. Author at least one custom recogniser. Ship the credited-control evidence in the register-row shape.

### The three positions

- **Ingest** — a batch or streaming pipeline that reads records before they enter training or the RAG store. For each record: Presidio Analyzer at a chosen confidence threshold; Anonymize with a placeholder-preserving strategy (`<PERSON_1>`, `<EMAIL_1>` etc.); emit the anonymised record downstream; log detection counts by entity type and source.
- **Retrieval** — Presidio applied after retrieval, before the retrieved chunk enters the prompt. Per-chunk Analyzer; access-control check against the current user's authorisation grant (for the exercise, either use a simple RBAC stub — the current user has read on entity types {X, Y} — or wire against the running system's actual access layer); anonymise or drop chunks the user is not authorised to see.
- **Output** — Presidio applied to the model's response before it reaches the user. Analyzer over the response; whitelist entities present in the user's own input or in the authorised retrieved context; redact the rest; log high-confidence detections passed through vs. redacted for monitoring.

### The custom recogniser

Author at least one custom recogniser for an entity type Presidio does not cover out of the box. Options:

- An internal-order-number / internal-user-ID pattern specific to the running system (a regex plus context words plus, ideally, a checksum).
- A jurisdiction-specific identifier Presidio's built-in list does not include (a specific country's national ID, a specific industry's identifier scheme).
- A domain-specific identifier (a medical device serial, a specific insurance policy number shape, a vehicle-registration-format).

Ship the custom recogniser with a labelled test set of at least 20 positive + 20 negative examples pulled from real data (not from your imagination), and calibrate the `score` against that test set — do not hard-code `1.0`.

### The precision / recall accounting

Per pipeline position, per entity type, report:

- **Precision** — of the spans Presidio flagged as this entity type, what fraction were actually this entity type in the ground-truth labelling?
- **Recall** — of the ground-truth spans of this entity type, what fraction did Presidio flag?
- **F1** — harmonic mean.
- **Notes** — where Presidio's defaults were strong (typically e-mails, credit cards, US SSNs) vs. weak (typically `PERSON` in unstructured chat, `PHONE_NUMBER` on international formats). Chapter 05's reporting table is the reference shape.

Slice the eval by language, domain, and formality where the corpus supports it (English customer-support chat is a different eval slice than English formal document, is a different eval slice than any non-English text).

## Requirements

Produce all eight items:

1. **Labelling manifest** — `risk/mod-107/<system-id>/presidio/labelling.md`. Names the eval slice's provenance, the entity types labelled, the labelling protocol (single annotator / dual annotator / adjudication rule), the inter-annotator agreement if two annotators overlapped, and the exact span count per entity type. If a single annotator labelled everything, state that; do not claim inter-annotator agreement you cannot document.
2. **Ingest pipeline** — `risk/mod-107/<system-id>/presidio/ingest.py`. Reads records; runs Analyzer at the chosen threshold; anonymises with placeholder-preserving strategy; writes anonymised records and a detection-log JSON.
3. **Retrieval pipeline** — `risk/mod-107/<system-id>/presidio/retrieval.py`. Wraps the local RAG store's retrieval call; runs Analyzer on returned chunks; applies the RBAC stub / real ACL; emits sanitised chunks + a chunk-level redaction log.
4. **Output pipeline** — `risk/mod-107/<system-id>/presidio/output.py`. Post-processes the model's response; runs Analyzer; applies the input / authorised-context whitelist; redacts non-whitelisted PII; emits a response-redaction log.
5. **Custom recogniser** — `risk/mod-107/<system-id>/presidio/custom_recognizers/<entity>.py` — a Pattern + PatternRecognizer registered against the analyzer, plus `custom_recognizers/<entity>_tests.json` with the labelled positive and negative examples and a `pytest` (or plain assertion script) that verifies precision / recall on the test set.
6. **Precision / recall report** — `risk/mod-107/<system-id>/presidio/pipeline-report.md`. Chapter-05 fixed shape: per-position × per-entity table with precision / recall / F1 / span counts / notes; per-slice breakdown (language / domain / formality) where the corpus supports it; a "coverage boundary" section naming what Presidio does *not* fix (chapter 05 §"Where Presidio does not fix things" — semantic PII, sensitive inference, out-of-support-language PII).
7. **Register updates** — the mod-103 rows for the PII / memorisation scenarios updated: credited-control column populated with the Presidio pipeline evidence (per-position precision / recall / F1); control-defeated column populated with the "recogniser off" or "output filter off" variant (either derived from the "no pipeline" number or explicitly generated by running the pipeline with the recogniser disabled). Chapter 05 §"The RAG-store PII problem" and chapter 03's control-defeated row are the reference shapes.
8. **Reproducibility manifest** — `risk/mod-107/<system-id>/presidio/manifest.json`. Pins Presidio versions (analyzer, anonymiser, image redactor if used), spaCy model + version, custom-recogniser test-set hash, eval-slice fingerprint, confidence-threshold choice, and script commit hash.

Cross-cutting rules:

- **Presidio is not ground truth.** The precision / recall / F1 numbers measure *how well Presidio matches the ground-truth labelling* on your eval slice — they are not statements about the world's PII. The eval slice's provenance and labelling protocol are what make the numbers defensible.
- **Both false positives and false negatives.** False positives = utility cost (over-redaction breaks the product); false negatives = leak (the extraction risk from exercise 03 comes right back). Chapter 05 §"False positives and false negatives — the accounting" is the reference.
- **Score calibration on the custom recogniser is required, not optional.** A custom recogniser with `score=1.0` cited without a test set is a red flag. Chapter 05 §"Custom recognisers — the pattern" is the reference.
- **Slice by language and formality where the corpus supports it.** A single English-formal precision number hides that the pipeline breaks on Japanese chat or on OCR'd PDFs.
- **Name what Presidio does not fix.** Semantic PII, sensitive-inference PII, and structured PII outside supported languages / jurisdictions must appear in the coverage-boundary section of the report. Chapter 05 §"Where Presidio does not fix things" is the reference.
- **Whitelist the user's own PII in the output pipeline.** A user asking about their own account is entitled to see their own e-mail; redacting it breaks the product. Chapter 05 §"Output" is the reference.

## Starter guidance

- **Prove the ingest pipeline on 20 records first.** Run Presidio Analyzer over 20 labelled records; hand-check that the flagged spans match your labelling for the entity types Presidio covers by default. If you see obvious mismatches (Presidio flagging `LOCATION` where you labelled `PERSON`, or missing something you consider unambiguous PII), that is either a threshold problem (drop the threshold), a spaCy-model problem (upgrade to the larger model), or a custom-recogniser gap (which is what the exercise addresses). Do not scale to 500 records until 20 look sane.
- **Confidence threshold is a knob, not a fixed value.** Presidio's default is often 0.5; lowering to 0.35 catches more true positives on `PERSON` in noisy text at the cost of more false positives. Sweep threshold over a small grid on the labelled eval slice and pick the threshold that maximises F1 on the entity types that matter most to your running system. Report the swept curve as an appendix.
- **The custom recogniser test set matters more than the regex.** A regex that looks tight can catch surprising false positives in real data; a labelled test set of even 20-40 examples is what surfaces those. Pull the test-set examples from the same corpus your production data comes from.
- **Context words on the custom recogniser lift precision.** A regex for an internal order number that also requires the words "order," "invoice," or "shipment" within a small window is much less noisy than the regex alone. Chapter 05 §"Custom recognisers — the pattern" is the reference.
- **RBAC stub is fine for the exercise.** For the retrieval-side integration, a hand-rolled RBAC — "user X has read on {`EMAIL_ADDRESS`, `PHONE_NUMBER`} but not `US_SSN`" — is enough to exercise the pattern. In the real system this is a policy call, not Presidio's job; Presidio flags, the access layer decides. The exercise is about wiring the two together.
- **Output-side whitelist logic is subtle.** Extract PII entities from the user's own input + the authorised retrieved context; add them to the whitelist for that turn; redact everything else in the response. This means the whitelist is *per-turn*, not global; do not accidentally leak PII across turns.
- **Log everything for the mod-109 monitor.** The three log files (ingest detection log, retrieval redaction log, output redaction log) are what the mod-109 monitoring stack reads to compute drift — do not treat them as debug output; treat them as first-class artefacts.
- **Coverage-boundary is a real section, not a footnote.** The reader who trusts Presidio's flagged spans as complete PII will miss the semantic-PII and sensitive-inference leaks Presidio does not catch. Naming what Presidio does not fix is chapter 05's discipline.

## Acceptance criteria

The submission is accepted when:

- The labelling manifest is complete and honestly documents the annotation protocol.
- All three pipeline positions are implemented and produce their respective detection / redaction logs on a run over the eval slice.
- The custom recogniser is authored, its labelled test set exists, and its precision / recall on the test set is reported in the pipeline report.
- The precision / recall report has a per-position × per-entity table with span counts and notes; slice breakdowns exist where the corpus supports them.
- The coverage-boundary section names semantic PII, sensitive inference, and any out-of-scope language / jurisdiction gaps for the running system.
- The mod-103 register row's credited-control column cites the Presidio pipeline with the per-position numbers; the control-defeated column cites the "recogniser / output filter off" number.
- The reproducibility manifest pins Presidio versions, the spaCy model + version, the eval-slice fingerprint, the custom-recogniser test-set hash, the confidence threshold, and the script commit.
- One paragraph interprets the results: which position was the load-bearing control (typically ingest for training data, retrieval for RAG, output for arbitrary user prompts), and how does this pair with the DP-SGD variant from exercise 04?

## Stretch goals

- **PII in a RAG store, end-to-end shape.** Assemble the chapter-05 §"The RAG-store PII problem" full pattern: tag at ingest (retain span metadata but not the raw PII on the default retrieval surface), redact-by-default at retrieval, un-redact only on authorised entity types, filter again on output. Report the residual claim in the shape chapter 05 pins.
- **Multi-language eval.** Add a second language (Japanese, French, Spanish — whichever your corpus supports) and run Presidio with the appropriate language pack. Report the precision / recall / F1 delta from the English slice. This is chapter 05 §"Structured PII outside supported languages" made concrete.
- **Obfuscation-adversarial output test.** Craft a small set of prompts that induce the model to emit PII in obfuscated form — leet-speak, base64, spelled-out numbers. Report which forms slipped past the output-side filter. This is chapter 05's cross-reference to mod-105 (obfuscated-PII output requires layered defence: a filter that decodes candidate strings before checking). A stretch that shows the reader where the Presidio-only defence bounds.
- **Image / OCR redaction.** If the running system ingests PDFs or images, add Presidio Image Redactor to the ingest pipeline; report precision / recall on a small labelled OCR slice. This is where structured PII in image form typically leaks.
- **Cross-cutting mod-108 handoff.** Author a one-page "Presidio in the guardrail catalogue" entry that mod-108 can consume: the entity types covered, the per-position false-negative envelope, the confidence-threshold recommendation, and the custom-recogniser authoring rubric. Chapter 05's cross-reference to mod-108 is the tie-in.
- **Cross-cutting mod-109 handoff.** Author a monitor-config stub that reads the three log files (ingest, retrieval, output) and emits per-day per-entity-type detection counts as metrics with a drift alarm. The metric plumbing lands in mod-109; this exercise contributes the intake shape.

## Deliverable format

A single markdown document `mod-107-ex05-presidio.md` that indexes the artefacts. The index document contains:

1. *System and corpus summary* — one paragraph naming the running system, the corpus slice, and whether the RAG store and serving surface are the real ones or the stand-in.
2. *Labelling protocol* — the block from `labelling.md`, verbatim.
3. *Pipeline diagram* — a one-figure or one-paragraph description of the three positions and where each fires in the request lifecycle.
4. *Per-position precision / recall table* — one section per position, one table per position, with the per-entity-type rows.
5. *Custom recogniser* — the entity name, the regex, the context words, the test-set precision / recall, and the calibrated `score` justification.
6. *Coverage boundary* — the section naming semantic PII, sensitive inference, out-of-scope languages, and any structured identifier the running system uses that Presidio does not cover (and whether you wrote a custom recogniser for it).
7. *Register handoff* — the mod-103 rows with credited-control and control-defeated columns updated.
8. *Interpretive paragraph* — which position was load-bearing? How does this compose with the DP-SGD variant from exercise 04?
9. *Reproducibility manifest* — pointer to `manifest.json`.
10. *Open items* — anything for the module-final privacy-risk report (chapter 06 §"The report shape"), for mod-108 (guardrail-catalogue entry), and for mod-109 (drift monitor).

## Submission

Attach the index markdown, the pipeline report, the custom-recogniser source + test set, the labelling manifest, and the three log files (or their fingerprints) to the module submission channel. If the corpus is under NDA or contains real PII, redact the corpus content but preserve the labelling protocol, the precision / recall tables, and the reproducibility manifest — a reviewer cannot review a redaction pipeline whose numbers are blanked out.
