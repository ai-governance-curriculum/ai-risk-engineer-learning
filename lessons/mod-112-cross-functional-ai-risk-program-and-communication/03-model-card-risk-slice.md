# The Risk-Engineering Slice of Model, System, and Dataset Cards

Model cards, system cards, and dataset cards (Mitchell et al. 2019 for model cards; Gebru et al. 2021 for datasheets; Google's Data Cards Playbook; Hugging Face's model-card specification for the operational format most projects use in practice) are the external-facing surface of the risk-engineering work you have done across mods 102-111. This chapter is about *what belongs in the risk-engineering slice of a card* and *how the same evidence that filled your register defensibly fills the card without a second authoring pass*.

## Motivation

Cards fail in three patterns that a level-25 risk engineer sees repeatedly:

- **Marketing card.** The card exists, but reads as product copy. Sections that should carry evaluation numbers ("Risks, Limitations and Biases," "Intended Use," "Out-of-Scope Use") carry paragraphs of language that could equally describe half the models on the Hub. No numbers, no citations, no evidence anchors.
- **Duplicate-authoring card.** The card exists, is well-populated, and was written from scratch by the risk engineer *in addition to* the register. The card and the register drift; six months later they disagree about the fairness delta or the memorisation rate, and the auditor asks which is authoritative.
- **Card without residual claim.** The card describes evaluations, red-team findings, and known limitations, but never asserts *the residual position* — what risk remains after mitigations, on which population, in which context, and how monitoring is meant to catch violation of that claim.

The productive frame: the *card is a projection of the register*. Every risk-slice section on the card is a view over evidence you already produced upstream. The chapter's job is to name those views, tie them to the modules that authored the underlying evidence, and lay out the residual-risk claim as a first-class card section.

## The three card families

| Card family | Primary reference | Unit of description | Where the risk-engineering slice lives |
|---|---|---|---|
| Model card | Mitchell et al., "Model Cards for Model Reporting" (2019) | A trained model | "Uses," "Bias, Risks and Limitations," "Evaluation," "Environmental Impact," "Technical Specifications" sections |
| System card | e.g., OpenAI GPT-4 system card; Anthropic model / system cards | A deployed system (model + scaffolding + policy) | "Safety evaluations," "Red-teaming," "Preparedness assessment," "Mitigations," "Monitoring" sections |
| Dataset card / datasheet / data card | Gebru et al., "Datasheets for Datasets" (2021); Google Data Cards Playbook | A dataset (training, evaluation, or fine-tuning) | "Motivation," "Composition," "Collection process," "Preprocessing," "Uses," "Distribution," "Maintenance" sections; risk lives in "Uses" and "Distribution" |

The three families are complementary. A single deployed AI system typically has: one or more model cards (per component model), one system card (for the deployed composition), and one or more dataset cards (per dataset the system trained or was evaluated on). The risk-engineering slice is *the same discipline applied at three levels of granularity*.

<!-- needs-research: cite the exact current URLs for the OpenAI GPT-4 system card and one of the recent Anthropic system cards; system-card publications are versioned and named per release. -->

## The Hugging Face model-card spec — the operational format

The Hugging Face model-card specification is the format most public model releases use in practice. It provides a YAML metadata block at the top of the card, then a set of standard markdown sections beneath. The specification is the *default schema* an external consumer will look for.

Sections in the Hugging Face model-card template, mapped to their risk-engineering content:

| Section | Content the risk engineer supplies |
|---|---|
| Metadata (YAML) | `license`, `tags`, `datasets`, `metrics`, `model-index` (with the safety-eval numbers where you want them structured) |
| Model Details | Author, contact, licence, versioning; date of the card |
| Uses — Direct Use | Intended, in-scope uses; ties to the deployment tier (this module ch 04) |
| Uses — Downstream Use | Fine-tuning / integration guidance; risk considerations that pass through to downstream integrators |
| Uses — Out-of-Scope Use | Explicit misuse and off-label uses; ties to the harm model (mod-102) and the deployment-tier contract |
| Bias, Risks and Limitations | The **integrator section** for mods 104-108 findings |
| Recommendations | Concrete guidance to consumers (deployment tier suitability, mitigations to keep in place) |
| Training Data | Provenance summary; dataset-card pointers; mod-107 privacy-risk summary |
| Training Procedure | Data preprocessing; supervision; hyperparameters (as relevant) |
| Evaluation | Safety-evaluation numbers (mod-104), adversarial-robustness delta (mod-105), fairness slice measurements (mod-106), privacy-risk findings (mod-107) — with pointers to the underlying report |
| Environmental Impact | Compute / carbon (often kept lightweight at the level-25 boundary; grows with mod-109 / MRM overlap) |
| Technical Specifications | Model architecture, size, dependencies |
| Citation | How to cite |
| Model Card Authors | Named authors; risk engineer typically co-listed |
| Model Card Contact | Escalation contact for external users |

The Hugging Face format is a defaultable schema. Where an internal template deviates, the deviation should be *named* rather than silent — an internal system-card template that omits, say, "Bias, Risks and Limitations" has a compliance gap unless the omission is justified in writing.

<!-- needs-research: pin the current Hugging Face model-card specification URL (documentation and template repo) at authoring time; the schema is stable but occasionally extended. -->

## The risk-engineering slice — anatomy

For a card at any of the three levels, the risk-engineering slice has seven components. Each is a *view over an artefact already authored*, so authoring the card is largely a matter of extracting and cross-linking.

### 1. Intended and out-of-scope use

| Source | Card location |
|---|---|
| Harm-model persona / capability-surface enumeration (mod-102 ch 01) | "Uses — Direct Use" and "Uses — Out-of-Scope Use" |
| Deployment tier (this module ch 04) | Tier explicitly named alongside the intended use |
| Legal / obligation constraints (mod-101 ch 04, ch 05) | Named obligations mentioned as constraints, not as legal opinions |

The out-of-scope section names *specific* off-label uses. "Do not use for hiring decisions" beats "not intended for high-stakes decisions."

### 2. Safety evaluations

| Source | Card location |
|---|---|
| Red-team programme run reports (mod-104) | "Evaluation" or a dedicated "Safety evaluations" subsection; report the run scope, attack corpus size, pass / fail thresholds, and citation to the run report |
| Guardrail effectiveness numbers (mod-108) | "Evaluation" — per-guardrail F1 (or the metric your programme uses), coverage grid, known defeat modes |

Numbers, not adjectives. "Guardrail G-01 attained 0.94 F1 on the retention corpus (n=1,200; report Q2)" beats "guardrails performed well."

### 3. Adversarial-robustness delta

| Source | Card location |
|---|---|
| Adversarial-ML evaluation harness (mod-105) | "Evaluation" — accuracy under clean vs. under a named perturbation set; report the deltas and the tolerated worst case |

Adversarial robustness is a *delta claim*: the model's clean accuracy vs. its worst-case accuracy under the perturbations you evaluated. If the delta is large, name the mitigation that reduces exposure at deployment.

### 4. Fairness slice measurements

| Source | Card location |
|---|---|
| Subgroup slice definitions (mod-106) | "Evaluation" — per-slice accuracy / FPR / calibration numbers, with the subgroup definitions cited |
| Explainability tool output (mod-106) — where relevant | "Evaluation" — narrative on where explanations were and were not stable |
| Caveats sheet (mod-106) | "Bias, Risks and Limitations" — what the fairness measurements *do not* cover |

Fairness reporting has a discipline: report the slice definition alongside the number, and cite the caveat sheet honestly. A card that reports fairness deltas without slice definitions is not useful evidence.

### 5. Privacy findings

| Source | Card location |
|---|---|
| Memorisation / MIA / linkage findings (mod-107) | "Bias, Risks and Limitations" and "Training Data" — the summary and the pointer to the report |
| DPIA input trail (mod-107) | "Training Data" — provenance, lawful-basis notes, data-subject rights posture |

The privacy summary on the card should not restate the DPIA; it should say what the risk-engineering evaluations *found* and how the deployed system reduces that exposure.

### 6. Residual-risk claim

| Source | Card location |
|---|---|
| Residual-risk narrative for the covered systems (mod-103 ch 04) | A **first-class** section — either "Bias, Risks and Limitations" or a dedicated "Residual Risk" subsection |

The residual-risk claim is the *load-bearing* card component. A working shape:

> After the mitigations described above, we assess residual risk as **[level, per programme rubric]** on the [population, cohort, use case] evaluated. This claim holds subject to [named monitoring signals from mod-109] remaining within [named thresholds]. If any of the following triggers fire, the claim is invalidated and the system reverts to [tier / mitigated state]: [named triggers, per this module ch 04].

The claim is testable, tied to monitoring, and reversible. A residual-risk claim without an invalidation condition is not defensible.

### 7. Monitoring plan

| Source | Card location |
|---|---|
| Monitor spec, thresholds, escalation (mod-109) | "Recommendations" or a dedicated "Monitoring" subsection |

The card names the monitors that keep the residual-risk claim honest, the alerting thresholds, and the escalation path for downstream integrators. External integrators need this to know what they are inheriting.

## A worked short-form model-card risk slice

Below is a compact worked example of the seven-component slice for the customer-support RAG assistant. It is illustrative — a real card would carry more prose in each section — but shows the pattern.

```markdown
## Uses

**Direct use.** English-language first-line customer support for [product X], up to and including drafting responses that are reviewed by a human agent before send. Deployment tier: T2 (limited external, human-in-the-loop).

**Downstream use.** Not licensed for redeployment. Any integration outside the described product should re-run the mod-102 harm model against the intended context.

**Out-of-scope.** Not for use in: (a) hiring, promotion, or termination decisions; (b) credit / benefits / lending decisions; (c) medical, legal, or tax advice; (d) legally binding communications without human review; (e) minors as end users.

## Bias, Risks and Limitations

The following residual limitations have been evaluated and are being monitored:

- **Prompt-injection defeat.** Baseline guardrail G-INJ-01 attained 0.93 F1 on the retention corpus (n = 2,000; report `2026-q2-guardrail-effectiveness.md`). Known defeat modes: [named modes]. Monitor `mon-cs-injection-rate` (mod-109) tracks live defeat rate.
- **Fairness slice deltas.** Accuracy delta between [named language / dialect slices] is X percentage points (mod-106 report `2026-q2-fairness-slice.md`). Caveats: [named caveats]. Accepted above appetite for a 90-day window under decision DEC-2026-Q2-006.
- **Privacy — memorisation.** Retention corpus n = 5,000 canaries; extraction rate 0.02 percent (mod-107 report `2026-q2-privacy.md`). Data-subject rights procedures per DPIA §4.

## Residual Risk

After the above mitigations, we assess residual risk as **Medium-low** on the English-language EU + US customer cohort evaluated. This claim holds subject to `mon-cs-injection-rate` remaining below 0.5 percent (30-day rolling), fairness-slice delta not widening beyond [threshold], and no severity ≥ 2 incident. Rollback triggers and thresholds are enumerated in the deployment-tier contract (this module ch 04, exercise 04).

## Recommendations

Downstream integrators must (i) retain `mon-cs-injection-rate` and `mon-cs-fairness-delta` in their pipeline; (ii) route severity ≥ 2 incidents to the risk-council escalation contact within 24 hours; (iii) re-run the harm model for any new capability surface introduced by integration.
```

That card slice was written by *extracting from existing artefacts*: no fresh authoring beyond the connectives. This is the card-as-evidence-source pattern in practice.

## Internal templates vs the Hugging Face schema

Many organisations run an *internal* model / system card template — richer than the Hugging Face schema in some respects (audit-trail fields, regulator citations, internal deployment IDs) and reduced in others (no public-facing environmental impact numbers, no citation for external re-use). At the level-25 boundary, the productive posture is:

- Use the Hugging Face schema as the *external* face for anything the organisation publishes.
- Use the internal template as the *authoritative internal* face; treat the public card as a projection of the internal card that redacts internal-only fields.
- Never let the two disagree on evidence-linked fields. If the external card reports a fairness delta of X and the internal card reports Y, one of them is wrong and the audit trail must show which.

<!-- needs-research: many organisations have not yet standardised an internal system-card template; the pattern above is the working recommendation, not a citation. -->

## The card-as-evidence-source pattern

The card is an evidence source in two directions:

- **Downstream** — for consumers (integrators, auditors, regulators, customers), the card is the summary they read first.
- **Upstream** — for the register (mod-103) and the council packet (this module ch 02), the card is *evidence in itself*: proof that a residual-risk claim has been published, that intended and out-of-scope uses have been named, that a monitoring plan is externally committed to.

The card is therefore both a *view* over the register and an *artefact indexed by* the register. Register rows carry a `card_ref` field pointing at the card version and hash that publicly represents them.

## Interfaces to earlier modules

- **mod-101** — the framework / obligation mapping determines which citation and disclosure fields the card must include (e.g., EU AI Act Article 13 transparency obligations for high-risk systems inform "Intended Use" and "Uses — Out-of-Scope Use" content).
- **mod-102** — the harm model authors the persona and out-of-scope enumeration.
- **mod-103** — the register hosts the residual claim; the card *publishes* a redacted view of that claim.
- **mod-104** — red-team programme run reports feed the "Evaluation — Safety" section.
- **mod-105** — adversarial-ML numbers feed the "Evaluation — Robustness" section.
- **mod-106** — fairness slice numbers, caveats, and explainability findings feed both "Evaluation" and "Bias, Risks and Limitations."
- **mod-107** — privacy findings and DPIA linkage feed "Training Data" and "Bias, Risks and Limitations."
- **mod-108** — guardrail effectiveness numbers feed "Evaluation" and "Recommendations."
- **mod-109** — monitor spec and thresholds feed "Recommendations" and the "Residual Risk" invalidation conditions.
- **mod-110** — supply-chain / MRM evidence attaches at "Training Data" and "Technical Specifications"; SBOM / model-BOM references live here or in a linked artefact.
- **mod-111** — the incident log is not a public card section, but the escalation contact and the recurrence guarantees are.

Chapter 04 continues into the deployment-tier contract that is referenced across the seven card components.

## Summary

- Model / system / dataset cards are the external-facing surface of mods 102-111; the card *is a projection of the register*, not a fresh authoring pass.
- The Hugging Face model-card specification is the default operational schema; internal templates may extend but should not silently deviate.
- The risk-engineering slice has seven components — intended / out-of-scope use, safety evaluations, adversarial-robustness delta, fairness slice measurements, privacy findings, residual-risk claim, monitoring plan — and each maps to an earlier module.
- The residual-risk claim is load-bearing: it must be *testable*, tied to *named monitors*, and *reversible* by pre-registered triggers.
- Cards act as evidence in both directions: downstream for consumers and upstream as an audit-linkable artefact indexed by the register.
- Exercise 03 authors a full risk-slice card for the running system in the Hugging Face schema.
