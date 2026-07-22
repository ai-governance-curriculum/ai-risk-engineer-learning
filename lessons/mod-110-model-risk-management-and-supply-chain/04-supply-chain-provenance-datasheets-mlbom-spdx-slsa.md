# AI Supply-Chain Provenance — Datasheets, Model Cards, ML-BOM, SPDX 3.0 AI Profile, SLSA

An auditor asks a single question about a deployed model that decides most other audit questions: *given this artefact running in production, can you show me every input that went into it and every transformation it went through*. This is the *supply-chain provenance* question. It is the same question the software-supply-chain community has been standardising around for a decade under the umbrella of the SBOM (Software Bill of Materials), and it is now being extended to AI-specific inputs — training data, evaluation data, model weights, fine-tune deltas, safety classifiers, upstream base models — under CycloneDX's **ML-BOM** notion, SPDX's **3.0 AI profile**, and SLSA's **build integrity** levels. This chapter teaches the artefact set an auditor expects.

## Motivation

Three failure modes that show up when a real provenance question hits an unprepared team:

- **"We can find it, give us a week."** The team has the information — in Slack, in a training-script comment, on a former engineer's laptop, in an S3 bucket with no naming convention — and can reconstruct it under duress. That is not provenance. That is *reconstruction*.
- **"It's a Hugging Face model."** The model card names the base checkpoint. The team's fine-tune script points at "the customer-support corpus," which is a moving pointer to an S3 prefix, which has been silently updated three times. Provenance stops at the base checkpoint.
- **"The dataset speaks for itself."** The training set is a folder of CSVs with no schema, no licence file, no collection description, no known-limitations note. When a downstream team tries to reuse it, they cannot answer whether it is fit for their purpose, and when a fairness audit surfaces a demographic gap, nobody can trace whether the gap comes from source or labelling.

The engineering fix is the same for all three: turn provenance into *files with schemas* that live alongside the artefacts they describe, and turn assembly of those files into *build steps that fail if inputs are missing*, not a checklist that fills itself.

## The provenance stack in one diagram (as prose)

For a single deployed model, the provenance package has five layers, from raw input up to deployed artefact:

1. **Data provenance.** Datasheet for Datasets or Data Card per corpus that entered training, fine-tuning, evaluation, retrieval, or prompt context.
2. **Model provenance.** Hugging Face-style Model Card per model artefact (base, fine-tune, safety classifier), plus the Model Card's linkage to the datasets in layer 1.
3. **Component inventory.** An ML-BOM (CycloneDX) — a machine-readable inventory of every ML-specific component the deployed system depends on, extending the software SBOM shape.
4. **Cross-domain SBOM record.** An SPDX 3.0 document using the AI profile, describing the AI-specific relationships alongside the ordinary software SBOM in a single record.
5. **Build-integrity attestation.** A SLSA provenance attestation describing how the deployed artefact was built, by which builder, from which sources, at what level of assurance.

The five layers compose: layer 3 (ML-BOM) references layer 1 (Datasheets) and layer 2 (Model Cards); layer 4 (SPDX 3.0) can consume the ML-BOM as sub-document or express the same relationships natively; layer 5 (SLSA) attests that layers 1-4 were produced under a controlled build.

Chapters 05 and 06 add layer 6 (signing the model artefacts and the provenance documents themselves) and layer 7 (scanning the model artefacts for malicious content before ingestion).

## Layer 1 — Data provenance: Datasheets and Data Cards

The **Datasheet for Datasets** proposal (Gebru et al., initially published 2018; the current version was published in *Communications of the ACM*, 2021) is a structured questionnaire that documents a dataset along seven headings: motivation, composition, collection process, preprocessing/cleaning/labelling, uses, distribution, and maintenance. Each heading has a defined set of questions.

The **Data Cards** framework, published by Google Research in 2022 ("Data Cards: Purposeful and Transparent Dataset Documentation for Responsible AI"), is a slightly different structure aimed at the same goal, oriented toward practitioner and stakeholder consumption rather than academic completeness. Data Cards are typically shorter, use structured fields, and are designed to compose into a public-facing dataset catalogue.

For your purposes, the choice between Datasheet and Data Card is a matter of house style — you commit to one and produce it for every corpus that enters your ML system. A minimum-viable Data Card / Datasheet has fields for:

| Field group | What it captures |
|---|---|
| Identity | Name, version, DOI or persistent identifier if any, publication date |
| Motivation and provenance | Why the dataset was created; by whom; funded by whom; source of the underlying data |
| Composition | What is in each record; how many records; what representation |
| Collection | How data was collected, from whom, over what period, under what consent |
| Preprocessing / labelling | What transformations were applied; who labelled; on what schema; with what adjudication |
| Population coverage | Which populations are represented; which are not; known distributional biases |
| Uses | Intended uses; uses that are known to be unsafe or misleading; downstream tasks tested |
| Distribution and licence | How the dataset is distributed; under what licence; access controls if any |
| Maintenance | Who owns the dataset now; how updates happen; sunset policy |
| Known limitations | What the maintainers explicitly know the dataset does not represent well; what is known to be low quality |

Every corpus in scope of a mod-101 sub-category (MEASURE 2.11 fairness, MEASURE 2.3 deployment-shaped evaluation, and so on) needs one of these files. Corpora without a Data Card / Datasheet are un-audited data — they should not enter training or evaluation for any model that will be validated under MRM or FDA.

## Layer 2 — Model provenance: Hugging Face-style Model Cards

The Model Card format was proposed by Mitchell et al. in 2019 ("Model Cards for Model Reporting") and has become the de-facto standard through Hugging Face's model-hub adoption. A Model Card describes a specific *model version* — base model plus fine-tune deltas plus any post-training modifications — and covers model details, intended uses, factors, metrics, evaluation data, training data, quantitative analyses, ethical considerations, and caveats & recommendations.

For an internally deployed model the Model Card is not a marketing document; it is the *engineering source of truth* for the model artefact. Fields the Model Card must carry for provenance purposes:

| Field | What it captures |
|---|---|
| Model identity | Repository path, revision hash, weights file hashes |
| Base checkpoint identity | Upstream model name, upstream version, upstream licence, upstream Model Card link |
| Fine-tuning corpus reference(s) | Persistent identifier(s) of the Data Card / Datasheet for every dataset used |
| Training procedure | Hyperparameters, hardware, epochs, random seeds, framework versions |
| Evaluation results | Aggregate and disaggregated performance on named evaluations, with links to the reports |
| Intended use | The mod-101 intended-use statement |
| Known limitations and out-of-scope uses | Failures modes; uses explicitly discouraged |
| Safety and ethical considerations | Cross-reference to harm-model rows (mod-102) and register rows (mod-103) |
| Version history | Every previous version, what changed, when |

Fine-tune deltas produced in-house are Model Cards in their own right, with the base checkpoint identified as their parent. The full provenance chain — foundation model → fine-tune → adapter → distillation → quantisation — is a tree of Model Cards each referencing its parent.

Mod-112 chapter on the AI programme covers Model Cards as a *consumer-facing communication artefact*; this chapter's take is on Model Cards as *provenance evidence*. Both roles are compatible in a single file.

## Layer 3 — ML-BOM with CycloneDX

**CycloneDX** is an OWASP project that defines a lightweight SBOM standard originally aimed at software components. The 1.5 release (2023) added ML-specific component types under the notion of a **ML-BOM**: `component.type` values including `machine-learning-model` and `data`, plus fields for training data references, model formats, hyperparameters, energy consumption, and inference environment. The 1.6 release refined the ML component definitions further.

An ML-BOM extends the SBOM idea in one direction: it adds *ML components* (models, datasets) to the list of things you inventory alongside libraries and containers. A minimum-viable ML-BOM for a deployed AI service enumerates:

| ML component | What is captured |
|---|---|
| The deployed model artefact | Name, version, weights hash, format (safetensors, ONNX, GGUF, TorchScript), signing status (chapter 05) |
| The base model | Same, plus the upstream registry and licence |
| Every fine-tune adapter / LoRA / adapter delta | Same |
| Every safety-classifier model | Same |
| Every embedding model in the RAG stack | Same |
| Every training and evaluation dataset | Name, version, identifier of the Data Card / Datasheet, licence |
| Every third-party model API (bounded description) | Vendor, model, version pinned to, terms |

Alongside the classical software SBOM (libraries, container base images), the ML-BOM gives an auditor the complete component inventory for one deployed service. Modern registries (Anchore Syft, GitHub Advanced Security, cloud-provider registries) already emit CycloneDX; extending that emission to cover ML components is largely a matter of teaching the pipeline to include model-registry lookups.

## Layer 4 — SPDX 3.0 with the AI profile

**SPDX** (System Package Data Exchange) is a Linux Foundation project that standardises SBOM data exchange; SPDX 2.x has been the dominant SBOM interchange format for compliance and licence-management purposes for a decade. **SPDX 3.0**, published in April 2024, restructured SPDX as a set of *profiles* on a common core, with dedicated profiles for security, build, licensing, lite, and — the reason this chapter cares — an **AI profile** and a **Dataset profile** aimed at describing ML systems and their data inputs alongside the classical software components.

Where ML-BOM (CycloneDX) is a lightweight ML-focused inventory, SPDX 3.0 with the AI profile is a broader interchange format that expresses ML relationships in the same document as the software components, licences, and build steps. In practice a mature programme produces *both* — CycloneDX for internal tooling and downstream security-scanning integrations, SPDX 3.0 for external interchange with regulators, standards-body submissions, and enterprise customers who insist on SPDX.

The AI profile fields to know:

| SPDX 3.0 AI profile element | What it captures |
|---|---|
| `AIPackage` | An AI model as a first-class package, with fields for domain, hyperparameters, training data references |
| `Dataset` (from Dataset profile) | A dataset as a first-class package, with fields for confidentiality, sensor collection method, intended use |
| `Relationship: trainedOn` | The AI package was trained on the referenced dataset(s) |
| `Relationship: hasEvaluation` / evaluation attestations | Links from the AI package to evaluation results |
| `Relationship: derivedFrom` | Fine-tune / distilled-from / quantised-from linkage between AI packages |
| Licence fields | Model licence, dataset licence, tracked per SPDX's licence-expression syntax |

If your organisation is choosing one format only, choose based on the consumer. Regulators and enterprise procurement lean SPDX. Security tooling ecosystems lean CycloneDX. There is no incompatibility — both can be produced from a single provenance data model in your registry.

## Layer 5 — SLSA build-integrity levels

**SLSA** (Supply-chain Levels for Software Artifacts) is a framework maintained under the OpenSSF that assigns levels to the assurance you have about *how* an artefact was built. SLSA levels are cumulative:

| Level | Requirement (short) |
|---|---|
| SLSA 1 | Build process is scripted / automated and produces a *provenance* record describing what was built and by what |
| SLSA 2 | Provenance is signed by a hosted build service (not just self-attested); the build platform is authenticated |
| SLSA 3 | Source and build platforms meet specific security requirements; the provenance is *non-falsifiable* — the build platform enforces isolation such that a compromised build cannot rewrite its own provenance |
| SLSA 4 | (In v1.0 the top level is Build L3; some earlier drafts named a Level 4 with additional hermetic-build and two-party-review requirements — check the current SLSA specification version) |

<!-- needs-research: pin the current SLSA specification version and confirm the highest defined Build level at authoring time; the framework's level definitions have evolved between v0.1 and v1.0. -->

For AI, the SLSA provenance describes:

- which builder built the artefact (e.g., a GitHub Actions runner, a Vertex AI training job, an internal Kubernetes training pipeline);
- from which sources (the training code repo commit, the dataset URIs, the base model URI);
- with what invocation parameters (hyperparameters, seeds, config hashes);
- to produce which artefacts (weights file(s), evaluation reports).

SLSA compresses the "how was this made" question into a single signed record that a downstream consumer (an auditor, an incident responder, a compliance system) can verify without re-running the build. A model with SLSA provenance can be traced from deployed artefact back to inputs deterministically; a model without cannot.

Uplifting SLSA level is where mod-110 hands off to `security-learning` / `ai-infra-security` (level 35). Their programme owns the build platform, its signing keys, and its isolation properties. Your programme owns *whether the provenance record for each production model exists*, *whether the record's contents are correct*, and *whether the model registry gates on it* (chapter 05).

## Worked example — assembling the package for one model

To make the composition concrete, walk through what a "provenance package" folder looks like for one deployed model. The scenario continues the customer-support RAG assistant from mod-102 (specifically, an in-house fine-tune of a foundation model on internal support transcripts).

```
provenance/<model-name>/<release-tag>/
  README.md                          # overview + how to verify
  model-card.md                      # this fine-tune's Model Card
  parent-model-card.md               # or a link to the upstream base's Model Card + hash
  datasets/
    support-transcripts.datacard.yaml
    support-transcripts.license.txt
    eval-heldout.datacard.yaml
    eval-heldout.license.txt
    kb-corpus.datacard.yaml
  mlbom.cyclonedx.json               # CycloneDX 1.6 ML-BOM
  sbom.spdx.json                     # SPDX 3.0 with AI profile
  slsa-provenance.json               # SLSA provenance for the training run
  attestations/
    training-run.intoto.jsonl        # in-toto attestation of the training run
    signature.sig                    # cosign signature on the weights (chapter 05)
    scan-report.json                 # ModelScan + picklescan output (chapter 06)
  evaluation/
    eval-report.md
    eval-report.pdf
    metrics.json
```

An auditor pointed at this folder can answer *every input that went into the deployed artefact* by reading the Data Cards, the parent Model Card, and the SLSA provenance. The evidence is not "somewhere" — it is *here*, with defined names and formats, and it is regenerated on every release rather than assembled after the fact.

## Cross-references

- **mod-101 chapter 03** placed ISO/IEC 42001, 23894, 42005, and 25059 as the standards this evidence feeds; provenance evidence is one of the primary inputs to ISO/IEC 42001 records.
- **mod-102** harm model rows are cited in the Model Card safety-and-ethics section.
- **mod-103** register rows citing "unknown-provenance dataset" or "unpinned base model" as the risk source are closed by producing the provenance package.
- **mod-108** guardrail catalogue rows that credit "the base model's own safety training" get much stronger evidence when the base model's Model Card and evaluation history are captured in the provenance folder.
- **mod-109** monitors on drift or population stability rely on the training-data distribution being *knowable* — Data Cards make that possible.
- **mod-111** incident triage that traces a bad behaviour to an input source depends on complete provenance; without it, the "where did this come from" question dead-ends.
- **mod-112** Model Card as the consumer-facing communication artefact is filled by the same file this chapter treats as provenance evidence.
- Chapter 01 (SR 11-7) requires the datasheet / model-card / ML-BOM under the "Data" and "Third-party components" MRM headings.
- Chapter 02 (SR 23-4) requires the vendor's data / model provenance disclosures.
- Chapter 03 (FDA PCCP) requires provenance discipline under the modification-protocol's data-management block.
- Chapter 05 signs the artefacts inside this folder; chapter 06 scans them.

## Summary

- The provenance package has five layers: data (Datasheets / Data Cards), model (Model Cards), ML-BOM (CycloneDX), cross-domain SBOM (SPDX 3.0 AI profile), and build-integrity attestation (SLSA).
- The layers compose: ML-BOM references Data Cards and Model Cards; SPDX 3.0 with AI profile expresses the same relationships for external interchange; SLSA attests to how the artefact was built.
- Every corpus entering training, fine-tuning, evaluation, or retrieval needs a Data Card / Datasheet; every model artefact (base, fine-tune, safety classifier) needs a Model Card; every deployed AI service needs an ML-BOM and a SLSA provenance record.
- Uplifting SLSA level is a level-35 security platform responsibility; ensuring the record exists, is correct, and is gated on is this role's responsibility.
- Exercise 04 authors the ML-BOM + SPDX 3.0 + SLSA provenance for one deployed model of your choice.
