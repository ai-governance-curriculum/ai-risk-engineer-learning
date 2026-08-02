# Exercise 04 — ML-BOM, SPDX 3.0 AI Profile, and SLSA Provenance for One Deployed Model

**Estimated effort:** 3 hours

## Objective

Assemble the full **provenance package** — the five layers from chapter 04 — for one deployed model of the primary running system. The output is a folder an auditor can open cold and, from the files alone, answer: *which inputs went into this model*, *how was it built*, and *how can any of that be verified without trusting a wiki page*. This package is what chapters 05 (signing) and 06 (scanning) subsequently lock down.

## Prerequisites

- Chapter 04 read in full. The five layers (data → model → ML-BOM → SPDX AI profile → SLSA), the field tables for each, the worked-example folder layout, and the "regenerated on every release, not assembled after the fact" discipline are the reference shapes.
- The primary running system carried since mod-101 through mod-103, and one specific deployed model within it — a fine-tune, a safety classifier, an embedding model, or a base model you self-host. Pick the model that (a) has the most third-party inputs (foundation-model base, external datasets, off-the-shelf safety classifier) and (b) has real training or fine-tuning steps you can describe honestly.
- Access to the datasets that entered training, fine-tuning, evaluation, and retrieval — enough to author the Data Cards. If some corpora are third-party and their Data Card / Datasheet is not published, the exercise output includes a *disclosure-gap* note (per chapter 04's "un-audited data" discipline).
- Access to the model card (or model-card-shaped notes) for the base checkpoint and any fine-tune deltas. Public Hugging Face model cards are enough for the base checkpoint.
- Access to the build system (CI runner, training-pipeline runner) enough to describe the builder identity, source references, and invocation parameters for a SLSA provenance record. If the build system does not currently emit provenance, the exercise output describes the *manual* provenance record and flags the CI wiring as a gap for exercise 05 or for the L35 platform team.
- The CycloneDX 1.6 (or current) ML-BOM specification and the SPDX 3.0 AI Profile specification open in a tab. The exercise expects you to cite the specification version you used.

## Problem statement

Author the provenance package for one deployed model. The package is the folder from chapter 04's worked-example diagram, with every file populated with real (not placeholder) content. Where content is genuinely unknown, the file exists with a `disclosure-gap` block that names the missing information, the owner, and the target date.

### The package layout

Reproducing chapter 04's example:

```
provenance/<model-name>/<release-tag>/
  README.md                          # overview, how-to-verify, version pins
  model-card.md                      # this model's Model Card
  parent-model-card.md               # or a link to the upstream base's Model Card + hash
  datasets/
    <corpus-1>.datacard.yaml
    <corpus-1>.license.txt
    <corpus-2>.datacard.yaml
    ...
  mlbom.cyclonedx.json               # CycloneDX 1.6 ML-BOM
  sbom.spdx.json                     # SPDX 3.0 with AI profile
  slsa-provenance.json               # SLSA provenance for the training / build run
  attestations/
    training-run.intoto.jsonl        # in-toto attestation of the training run
  evaluation/
    eval-report.md
    metrics.json
```

Layers 6 (signing) and 7 (scanning) land in exercises 05 and 06; leave the placeholders for them (`attestations/signature.sig`, `attestations/scan-report.json`) so the folder shape is stable.

### The verify-from-the-folder rubric

Every provenance package must, from the files alone, let an auditor answer three questions without external context:

1. *Which corpora entered this model?* — answered by walking the Model Card's dataset references to the `datasets/` folder and reading the Data Cards.
2. *Which parent model is this derived from?* — answered by the Model Card's parent link plus `parent-model-card.md` (or an upstream Model Card link with hash).
3. *How was this artefact built?* — answered by the SLSA provenance record and the in-toto attestation: builder identity, source references, invocation parameters, produced artefacts.

If any of those three questions cannot be answered from the folder alone, the package is incomplete.

## Requirements

Produce all seven items:

1. **The package folder** — every file in the layout above exists with real content or a `disclosure-gap` block. No silent blanks.
2. **The Model Card** (`model-card.md`) — covers every field from chapter 04's Model Card table: model identity (repo path, revision hash, weights file hashes), base checkpoint identity, fine-tuning corpus references (persistent IDs of the Data Cards), training procedure, evaluation results (aggregate and disaggregated), intended use (from mod-101), known limitations, safety and ethical considerations (cross-referenced to mod-102 rows and mod-103 register rows), version history.
3. **A Data Card per corpus** (`datasets/<corpus>.datacard.yaml`) — covers every field from chapter 04's Data Card table for at least three corpora (training, evaluation, and one retrieval or fine-tuning corpus). If the corpus is third-party and no Data Card is published upstream, author the *outer-shape* Data Card with `provenance: third-party-not-disclosed` and file a compensating disclosure-gap block.
4. **The ML-BOM** (`mlbom.cyclonedx.json`) — a syntactically valid CycloneDX 1.6 (or current version — cite the version) document enumerating: the deployed model artefact, the base model, every fine-tune adapter / LoRA delta, every safety classifier, every embedding model, every training and evaluation dataset, every third-party model API called at inference time. Every ML component carries name, version, hash where computable, format, licence, and reference to the corresponding Data Card / Model Card.
5. **The SPDX 3.0 record** (`sbom.spdx.json`) — a syntactically valid SPDX 3.0 document using the AI Profile and the Dataset Profile, expressing at minimum: one `AIPackage` per model, one `Dataset` per corpus, `trainedOn` relationships from AI packages to datasets, `derivedFrom` relationships across the fine-tune / adapter / quantised chain, and licence expressions per package. Cite the SPDX 3.0 minor version you used.
6. **The SLSA provenance record** (`slsa-provenance.json`) — a syntactically valid SLSA v1.0 provenance record naming the builder, the source references (training-code commit, dataset URIs, base model URI), the invocation parameters (hyperparameters, seeds, config hashes), and the produced artefacts (weights file(s), evaluation reports). Cite the SLSA specification version and the current Build level the platform provides. If the build platform does not emit provenance automatically, hand-author the record and flag CI emission as a gap for the L35 platform team.
7. **The `README.md`** — opens with the how-to-verify block: three numbered steps that let a reader answer the three verify-from-the-folder rubric questions using only the files in the folder. Includes the version pins for CycloneDX, SPDX 3.0, and SLSA specification versions, and a change log for the package (v0.1 minimum).

## Starter guidance

- **Pick the model with the most external inputs, not the fewest.** A model that is only a base checkpoint with no fine-tune has a shallow provenance graph; a model with a fine-tune on a mixed internal-and-third-party corpus with a downstream quantisation exercises every layer. The training value is proportional to the fan-in.
- **Start from the deployed weights, work backwards.** The exercise is easiest when you start with the artefact in production and walk backwards through fine-tune script → training dataset → base checkpoint → upstream Model Card. Starting from "I want to build the ML-BOM" makes the omissions harder to spot.
- **Every corpus without a Data Card is a disclosure gap.** Chapter 04 is clear: un-audited data should not enter training or evaluation for a model that will be validated. If you find such corpora, that is *the finding* — file the gap, propose the retroactive Data Card owner, and continue. Do not fabricate provenance.
- **Model formats matter for the ML-BOM.** Record the exact on-disk format for every model artefact (safetensors, GGUF, ONNX, TorchScript, pickle-based `.bin`). Chapter 06's scanner suite will read this field to decide which scanners to run; a wrong format field breaks the downstream gate.
- **The SLSA record is a machine-readable answer to "how was this made."** It is not a narrative. If you find yourself writing prose in the SLSA JSON, translate it to structured fields per the spec — or, if the field does not exist, note the extension and cite the SLSA spec version.
- **CycloneDX and SPDX are complements, not alternatives.** Chapter 04 says a mature programme produces both; do both. Regulators lean SPDX; security tooling leans CycloneDX. If your organisation has only chosen one, produce that one plus a stub of the other with a `TODO: extend to full` note.
- **Version-pin everything.** CycloneDX 1.6 is not the same as CycloneDX 1.5; SPDX 3.0 is being revised; SLSA levels have moved between v0.1 and v1.0. In `README.md`, name the versions you used. When those versions change, the package's `<release-tag>` is bumped and the pins are updated.
- **The package regenerates on every release.** This exercise is a snapshot, but the package's purpose is to be produced by a build step, not by hand. Where you had to author fields manually, note them as gaps for CI wiring.
- **Where regulator-facing wording matters, prefer SPDX.** SPDX 3.0 with the AI Profile is what a European or a US regulator asking for an AI-BOM is most likely to accept. CycloneDX ML-BOM is what your internal security tooling will consume.

## Acceptance criteria

The submission is accepted when:

- The package folder exists with every file populated as real content or a `disclosure-gap` block.
- The Model Card covers every field from chapter 04's Model Card table, with cross-references to mod-102 harm rows and mod-103 register rows.
- Data Cards exist for at least three corpora (training, evaluation, and one retrieval or fine-tune corpus). Third-party corpora without upstream Data Cards have outer-shape Data Cards with a labelled disclosure gap.
- The CycloneDX ML-BOM is syntactically valid, enumerates every ML component from the required list, and version-pins the specification.
- The SPDX 3.0 record is syntactically valid, uses the AI Profile and Dataset Profile, expresses `trainedOn` and `derivedFrom` relationships, and version-pins the specification.
- The SLSA provenance record is syntactically valid, names builder / source references / invocation parameters / produced artefacts, and version-pins the specification and level.
- `README.md` opens with a how-to-verify block that answers the three verify-from-the-folder questions using only the folder's contents.
- The exercise-01 MRM packet's `03-data.md` and `11-third-party-components/` folders are updated to reference this provenance package.

## Stretch goals

- **CI wiring proposal** — write a short design (`README.ci.md` in the package folder) for how the provenance package would be produced by the training pipeline automatically. Which build step emits which file, which secrets are needed, which failure modes exist. This is the hand-off to the L35 platform team.
- **Second model, cross-reference verified** — pick a second model in the same system (e.g., the safety classifier that runs alongside the primary model) and produce its provenance package. Then verify that where the two packages should share a dataset citation or a shared base model, they cite the same identifier. Divergence there is a real risk — two models that were trained on "the same data" but cite different provenance is a common source of audit findings.
- **Diff against a public model card** — pick a public Hugging Face model whose Model Card you cited as parent. Produce a small diff showing what your fine-tune's Model Card *adds* (fine-tune-specific metrics, downstream harms) and what it *does not need to duplicate* (base-model architecture, base-model training corpus). This is the discipline for how children of a Model Card reference their parent.
- **Round-trip through a downstream consumer** — feed the CycloneDX ML-BOM into a downstream security tool (an OSS scanner that consumes CycloneDX; a policy engine like OPA) and confirm the tool successfully consumes it. Failure to consume is a signal the ML-BOM is not tool-ready, not a signal the tool is wrong.
- **Disclosure-gap remediation plan** — for every disclosure gap in the package, write a one-line remediation entry (owner, action, target date). The set of remediation entries is the runway to a fully audited provenance package.

## Deliverable format

A single markdown document `mod-110-ex04-provenance-package.md` that indexes the package folder. The index contains:

1. *Model and package summary* — one paragraph naming the model, the release tag, and the specification versions pinned (CycloneDX, SPDX, SLSA).
2. *File inventory* — the folder listing with per-file completeness status (content / disclosure-gap / placeholder-for-exercise-05-06).
3. *Model Card summary* — the Model Card's fields at a glance, with cross-references to mod-102 / mod-103.
4. *Data Card summary* — one line per corpus: name, origin (internal / third-party / vendor), Data Card completeness status.
5. *ML-BOM summary* — one line per ML component: name, format, licence, reference to Data Card / Model Card / vendor questionnaire.
6. *SPDX record summary* — packages listed, relationships expressed, licence expressions used.
7. *SLSA record summary* — builder, source references, level attested to.
8. *Verify-from-the-folder walk-through* — the three-question walk-through quoted verbatim from `README.md`.
9. *Disclosure gaps* — table of missing content, owner, target date.
10. *Open items* — anything blocked on the L35 platform team, on a peer, or on a specification version pin that has not been ratified.

## Submission

Attach the index markdown plus the provenance package folder. If any dataset or model artefact is under NDA, redact identifiers but preserve the schema, the cross-references, and the version pins — a reviewer must be able to check the shape even when the content is not visible. Update the exercise-01 MRM packet's data and third-party-components sections to reference this package as the file that fills those slots.
