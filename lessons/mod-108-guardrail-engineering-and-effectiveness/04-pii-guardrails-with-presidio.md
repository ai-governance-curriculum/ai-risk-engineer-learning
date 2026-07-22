# PII Guardrails with Microsoft Presidio — Ingest, Retrieval, and Output

The classifier layer (chapter 03) handles semantic policy. PII handling is a different problem: the categories are enumerable, the detection is a mix of pattern-and-context recognition, and the redaction / anonymisation step is deterministic. **Microsoft Presidio** is the open-source library the industry defaults to for this layer. This chapter teaches Presidio's shape, where it sits in ingest, retrieval, and output paths, how to tune recognisers, how to account for false-positives and false-negatives, and how to mitigate the "PII already in the RAG store" case that classifier-only guardrails cannot fix.

## Motivation

PII guardrails are usually shipped in one of two broken shapes:

- **Output-only Presidio.** A Presidio pass runs on the model's final answer. It catches PII the model *generated* or *quoted*. It does not catch PII that made it into the retrieval index because there was no ingest-time filter. When an operator asks "why is this SSN in the index," the answer is "the classifier saw it in the answer and stripped it — but it is still in the index for the next query." That answer does not survive a privacy review.
- **Ingest-only Presidio.** A Presidio pass runs at ingest, redacting PII from documents before they enter the index. It does not catch PII the model *infers* from non-PII context, PII a user *pastes into a prompt*, or PII that survives ingest because a recogniser has drifted. Output is unguarded.

Presidio has to live in all three paths — ingest, retrieval, output — with independent recogniser configurations tuned to the failure modes at each path. This chapter walks the three integrations.

## What Presidio is

Presidio is an open-source PII detection and anonymisation SDK maintained by Microsoft. Two components matter for this chapter:

- **Presidio Analyzer** — detects PII in text. Ships with a set of built-in recognisers (email address, credit card number, SSN, IBAN, phone number, IP address, date of birth, medical licence numbers, and language- or region-specific identifiers). Each recogniser returns entity spans with type labels and confidence scores. Recognisers can be pattern-based (regex plus checksum), context-augmented (regex plus surrounding-word context), or model-based (spaCy / transformer NER, or a plugged-in third-party model).
- **Presidio Anonymizer** — transforms detected PII. Ships with operators for `replace` (fixed placeholder), `mask` (character-level masking), `redact` (remove), `hash`, `encrypt`, and `custom` (any function you write). Supports both anonymisation (irreversible) and pseudonymisation (reversible with a key).

<!-- needs-research: cite the current Presidio version and confirm which built-in recognisers ship by default vs. which require plugin install; the recogniser catalogue evolves per release. -->

Presidio is *not* a full data-loss-prevention platform. It does not manage keys, retention, or access control. Those live in your data plane and are the domain of `ai-privacy-engineer` and the security track — see mod-107 for the boundary.

## The three integration paths

### Ingest path

**Where.** In the pipeline that populates the RAG store, fine-tuning corpus, or evaluation set — anywhere content becomes indexable, embeddable, or trainable material.

**What Presidio does.** Analyzer runs on every document during ingest. Detected PII is either redacted (documents shipped to the index with PII removed), pseudonymised (documents shipped to the index with reversible tokens; the mapping stored in a controlled key store), or *routed* — documents whose PII cannot be safely handled are rejected from the index and flagged for review.

**Why it matters.** Ingest is the only path that can prevent PII from ever entering the retrieval surface. Every guardrail applied downstream is playing catch-up. And every downstream guardrail's FP / FN measurement is polluted if the underlying store contains PII that should not have been indexed.

**Common recogniser tuning at ingest.** More aggressive on precision (do not lose non-PII to over-redaction because you are creating the KB), backed by a review queue for rejected documents so that legitimate content is not silently dropped.

### Retrieval path

**Where.** After the retriever returns chunks, before those chunks are assembled into the model's context.

**What Presidio does.** Analyzer scans retrieved chunks; any PII detected is either redacted, replaced with a placeholder, or the chunk is dropped from the assembly. The scan is *per query* — different callers may have different entitlement to see PII, so the retrieval-side scan is caller-aware in enterprise deployments.

**Why it matters.** Even if ingest was perfectly clean today, PII can enter the index later (a new source, a mis-configured connector, a document that was updated to add PII, a synonym that was not in the recogniser at ingest time but is now). The retrieval-side scan is the second line.

**Common recogniser tuning at retrieval.** Latency-sensitive; the retrieval-side scan sits in the request path and adds to time-to-first-token. Use fast recognisers (regex-and-checksum ones) here; defer heavy transformer-based recognisers to ingest where offline batch is fine.

### Output path

**Where.** After the model produces the response, before the response leaves the application.

**What Presidio does.** Analyzer scans the response text; detected PII is redacted, replaced, or flagged. For structured outputs, Presidio runs per-field, and each field can carry a different policy (e.g., `answer` field: redact PII; `citations` field: allow PII if from a source the caller is entitled to; `tool_arguments`: schema-validate but leave PII untouched because the tool needs it).

**Why it matters.** The model can *generate* PII it inferred from context ("your account number is 12345"), *quote* PII from retrieved chunks that survived the retrieval-side scan, or *fabricate* PII-shaped content that a downstream consumer would treat as real. The output scan is the last line.

**Common recogniser tuning at output.** Balanced precision / recall; false positives are annoying to users (words that look like SSNs get masked), false negatives are the privacy incident.

## Recogniser tuning — the FP / FN accounting

Presidio's confidence scores are not magic thresholds; they are values you tune. Two disciplines that turn "we have Presidio" into a defensible control:

- **Per-recogniser threshold per path.** The credit-card recogniser's threshold is not the same at ingest (aggressive precision) as at output (balanced) as inside a caller-authorised branch (relaxed). Tune per path. Document the thresholds in the recogniser config.
- **Labelled fixture set per recogniser.** Every recogniser has a fixture set of known-PII strings it must catch and known-non-PII strings it must not flag. The fixture set is versioned in the repo. Regressions on either side of the fixture set block the release.

The FP / FN report chapter 05 produces has one row per recogniser per path. Recognisers whose FP rate exceeds the product manager's patience are candidates for tuning down (raise threshold, tighten context requirement) or replacement (custom recogniser, model swap). Recognisers whose FN rate leaves a category unaddressed are candidates for augmentation (additional pattern, added context word list, added model).

### Custom recognisers

Presidio ships an SDK for authoring custom recognisers. Two cases where custom recognisers are load-bearing:

- **Domain-specific identifiers.** Employee number formats, internal ticket IDs, customer account number formats, industry-specific medical or financial codes. Presidio's built-in set does not know these; author a `PatternRecognizer` with your patterns and check-digit rules and add it to the analyzer.
- **Locale-specific identifiers.** National ID formats, tax numbers, phone number formats, address shapes for the geographies your product serves. Presidio ships some locale packs; you may need to author others.

Custom recognisers get the same treatment: versioned, fixture-tested, threshold-tuned per path, measured on the FP / FN board.

## The "PII already in the RAG store" mitigation

The failure mode: the ingest pipeline was not running Presidio (or was running it with an incomplete recogniser set); the RAG store already contains PII; you cannot re-ingest overnight because the store is large / has legacy sources / is jointly owned with another team. The retrieval-side scan catches most of it but is a rate-limited backstop for a systemic problem.

Mitigation pattern:

1. **Audit the store.** Batch-run the full Presidio analyzer (including heavy transformer-based recognisers, since this is offline) against the store; produce a report of every chunk that contains detected PII and the recogniser that fired.
2. **Prioritise remediation.** Chunks with high-confidence PII in high-frequency retrieved documents get remediated first; low-confidence PII in rarely retrieved documents can be scheduled.
3. **Re-ingest the remediated documents.** With the tuned recogniser set in place, so future updates do not re-introduce.
4. **Harden the retrieval-side scan while the audit runs.** Raise threshold tolerance on the retrieval-side scan for the categories the audit is finding, until the store is remediated. Document the temporary configuration in the register.
5. **Standing recogniser-refresh cadence.** Every recogniser addition or tuning triggers a re-scan of the store (or the delta since the last scan) so drift does not accumulate.

The audit output feeds mod-107 (privacy risk) and mod-109 (monitoring); the register update is a mod-103 residual-risk revision.

## Worked composition — customer-support RAG assistant

Reusing the running scenario. Presidio integration per path:

| Path | Presidio configuration | Composition point |
|---|---|---|
| Ingest | Full analyzer (regex + context + spaCy NER); anonymiser redacts direct identifiers, pseudonymises account numbers with a keyed token to a controlled vault | Runs in the ingest DAG; documents are rejected to a review queue on unrecognised high-confidence entities |
| Retrieval | Fast analyzer (regex + context only, no NER for latency); redacts direct identifiers; drops chunks whose PII density exceeds a per-caller-entitlement threshold | Runs inside the LangChain retriever wrapper before context assembly |
| Output | Full analyzer on the response body; per-field policy in the output JSON schema; masks PII the caller is not entitled to see; audit log of every redaction | Runs inside the Guardrails AI output guard as a PII validator |

Custom recognisers: internal ticket ID (`TKT-NNNNNNNN` with mod-97 checksum), customer account number (16-digit with company-specific prefix), employee ID (`EMP-NNNNNN`). All three are versioned in the repo.

FP / FN measurement (chapter 05) is per recogniser per path.

## Anti-patterns

Four anti-patterns specific to the PII layer:

- **Ingest-only or output-only Presidio.** Missing paths leaves the other paths unguarded. Ship all three.
- **Same recogniser threshold everywhere.** Ingest, retrieval, output have different cost profiles; use different thresholds per path.
- **Recogniser drift without fixture regression.** A recogniser is tuned "for now" against ad-hoc examples; six months later the FP rate is way up because the tuning was never fixtured. Fixture every recogniser.
- **Pseudonymisation without key management.** Pseudonymised tokens are reversible with a key; if the key is stored next to the data, the pseudonymisation is theatre. Key management is not Presidio's job; the boundary lives in mod-107 and with the security track.

## Cross-references

- **Mod-107** — supplies the PII taxonomy this chapter enforces; the privacy-risk positions there inform which categories matter at which path.
- **Chapter 01** — Presidio populates the PII cell across three of the five slots (retrieval, output, and — as an ingest-time equivalent — the "before the index exists" pre-slot).
- **Chapter 02** — Guardrails AI wraps Presidio validators on the output slot; the custom middleware wraps it on the retrieval slot.
- **Chapter 03** — classifiers and Presidio are complementary; a classifier does not know your custom identifier formats, and Presidio does not know what "medical advice" is.
- **Chapter 05** — FP / FN measurement per recogniser per path is what turns Presidio from "installed" to "defensible control."
- **Mod-109** — the retrieval-side scan and output scan produce redaction events; the count of events per category per day is monitor signal.
- **Mod-111** — an incident that discovers PII leakage feeds a regression fixture into the recogniser set and (in the "PII already in the store" case) a re-audit trigger.

## Summary

- Presidio is the industry-default PII detection and anonymisation library; Analyzer detects, Anonymizer transforms.
- Integrate it into three paths — ingest, retrieval, output — with recogniser configurations tuned per path.
- Every recogniser has a threshold, a fixture set, a per-path FP / FN measurement, and a version.
- Custom recognisers cover domain-specific and locale-specific identifiers the built-in set does not know.
- The "PII already in the RAG store" case is a systemic mitigation: audit, prioritise, re-ingest, harden retrieval-side scan, standing recogniser-refresh cadence.
- Anti-patterns: single-path Presidio, uniform threshold, unfixtured tuning, pseudonymisation without key management.
- Exercise 04 puts you through the three-path integration on the running system.
