# Taxonomy Overlays: NIST GenAI Profile, ISO/IEC 23894, AIRO, and AIR-Bench 2024

Chapter 01 defined the harm-model artifact. Chapter 02 said every row must be *evidenced*. This chapter says every row must also be *tagged* — cross-walked to the standardised taxonomies auditors and consumers of your model expect. Tagging is what turns your harm model from an in-house document into something a level-15 analyst, an ISO auditor, an EU AI Act filer, and a peer-track engineer can all *read*.

## Motivation

If four different roles need to read your harm model, expect four different vocabularies. The analyst files it under an ISO/IEC 23894 risk source; the release-assurance engineer reads it against the NIST GenAI Profile checklist; the platform team wants an AIRO-shaped record for the internal risk API; the frontier-facing security lead wants AIR-Bench 2024 policy categories so a benchmark run can be attributed to a policy line. Rather than owning four separate harm files (drift-guaranteed), you tag each row with **all four** categorisations and let each consumer view the model through their preferred lens.

The discipline is called *taxonomy overlay*: one row, many tags, one source of truth.

## The four overlays this module requires

Every shippable harm-model row carries tags from all four of the following taxonomies. If any tag column is empty, the row is a draft.

| Overlay | What it categorises | Where downstream consumes it |
|---|---|---|
| NIST AI RMF Generative AI Profile (AI 600-1) risks | Twelve GenAI-specific risks | The applicability sheet and MAP function outputs from mod-101 |
| ISO/IEC 23894 risk sources | Sources of AI risk (data, model, use, deployment) | ISO/IEC 42001 AIMS records; ISO/IEC 42005 impact assessments |
| AIRO ontology categories | Machine-readable AI risk / harm ontology | Internal risk-graph APIs; cross-tool interchange |
| AIR-Bench 2024 policy-derived risk categories | Risk statements extracted from government + corporate policies | Benchmark runs attributable to policy lines; regulator-facing evidence |

Each is introduced below with enough detail to tag correctly; the exercise walks the full cross-walk.

## Overlay 1 — NIST GenAI Profile (AI 600-1) risks

You have this list already from mod-101 ch 02. The twelve risks — CBRN, Confabulation, Dangerous / Violent / Hateful Content, Data Privacy, Environmental Impacts, Harmful Bias / Homogenisation, Human-AI Configuration, Information Integrity, Information Security, Intellectual Property, Obscene / Degrading / Abusive Content, and Value Chain / Component Integration — form the *starting checklist* against which your harm model is evaluated. Publication reference: NIST AI 600-1 (July 2024). See mod-101 chapter 02 for the tabular breakdown.

**How to tag a row:** pick the single risk that best matches the immediate harm; add a second if the scenario materially involves it (e.g., a bias harm that also implicates human-AI configuration). Reference by the profile's risk number and short name. Rows tagged with none of the twelve are either (a) an out-of-scope risk (write it in), (b) a candidate future risk category to raise with `senior-ai-governance-architect` (level 50), or (c) a mis-scoped row.

**Applicability sheet reminder:** mod-101 ch 02 requires a twelve-row applicability sheet listing *for each of the twelve*, whether the risk is in-scope for your system. Every "in" row of that sheet must map to at least one harm-model row here. Every harm-model row must map to at least one "in" row of that sheet, or explicitly flag the sheet as needing an update.

## Overlay 2 — ISO/IEC 23894 risk sources

**ISO/IEC 23894:2023 — Information technology — Artificial intelligence — Guidance on risk management** is the ISO risk-management guidance for AI systems. It builds on ISO 31000 (general risk management) and specialises it for AI. Its Annex A enumerates *AI risk sources* — origins from which risk arises across the lifecycle. A working (non-exhaustive) grouping used at this level:

| Group | Example risk sources within the group |
|---|---|
| Data | Data quality, dataset representativeness, labelling error, dataset drift, data governance failures |
| Model | Training procedure, model complexity, generalisation failure, robustness, transparency of learned behaviour |
| Use | Intended purpose vs. actual use, task suitability, over-reliance, deployment context misalignment |
| Human oversight | Insufficient oversight design, poorly designed user interface for oversight, operator competence |
| Lifecycle | Change management, retirement, decommissioning, third-party components in the lifecycle |
| Broader environment | Physical environment, legal / regulatory environment, socio-technical environment |

<!-- needs-research: replace the working grouping above with the exact clause / annex identifiers and phrasing from ISO/IEC 23894:2023 Annex A once accessed through an ISO national body or licensed lead-implementer materials. -->

**How to tag a row:** pick the risk source that best names the *origin* of the harm scenario. Multiple sources are common (e.g., a bias scenario originates from Data *and* Use). Cite the annex sub-heading identifier.

**Why this overlay matters:** the ISO/IEC 42001 auditable management-system standard treats risk sources as first-class inputs. When your program is 42001-certifying, the auditor reads harm scenarios by their 23894 risk source — that is the lens they know.

## Overlay 3 — AIRO ontology

The **AI Risk Ontology (AIRO)** is a machine-readable ontology developed by the ADAPT Centre (Golpayegani et al.) that models AI system risk concepts — AI systems, capabilities, purpose, risk, risk sources, consequences, harms, impacted stakeholders, mitigations — using W3C-compatible ontology conventions. Its identifier space is `https://w3id.org/airo/` and it is designed to be aligned with ISO/IEC standards (23894, 42001) and with the EU AI Act's obligations. It also has a Vocabulary of AI Risks (VAIR) sibling project that classifies specific risks and harms.

<!-- needs-research: confirm the current AIRO version and any renamed / added classes; verify the current status of the VAIR sub-vocabulary and its exact stable URL. -->

At the level-25 harm-modelling scope, treat AIRO as *the machine-readable target* for your row tags. A useful minimum tag set per row:

- `airo:hasRiskSource` — one or more concepts from AIRO's risk-source class (data, model, use, deployment, etc.), analogous to 23894 sources.
- `airo:hasConsequence` — the consequence class that describes the outcome (e.g., discrimination, privacy breach, security breach, physical harm).
- `airo:hasImpactedStakeholder` — the class of impacted party (end user, data subject, third party, operator, society at large).
- `airo:hasMitigation` — either the class or the row identifier from your mitigation catalogue (populated later in mod-108).

**How to tag a row:** for each row, populate the four AIRO fields above. If your risk-engineering repo can hold structured YAML per row (recommended), materialise the AIRO tags as an object rather than a string blob; you will thank yourself when the internal risk-graph platform (a level-50 concern) asks for machine-readable exports.

**Why this overlay matters:** it is what turns your harm model from a Markdown table into a data structure a broader governance platform can query, aggregate across systems, and reason about at portfolio scale.

## Overlay 4 — AIR-Bench 2024 policy-derived risk categories

**AIR-Bench 2024** (Stanford CRFM, Zeng et al.) is an AI risk benchmark whose risk taxonomy is *derived from real government regulations and corporate policies* — it inverts the usual direction of the arrow: instead of authoring a taxonomy and then trying to map policies to it, it extracts risk categories from policies as the primary source and organises them into a hierarchy. The benchmark then tests whether models refuse or produce outputs that violate each policy-derived category.

<!-- needs-research: confirm exact counts (number of policies analysed, number of categories in the current released hierarchy, hierarchy depth) from the AIR-Bench 2024 paper and repository; only cite specific numbers once verified. -->

- Paper landing / dataset: check the current CRFM publication and the associated benchmark repository for exact identifiers.
- Category tags use a hierarchical code that identifies the top-level category, sub-category, and specific risk statement.

**How to tag a row:** where a harm scenario corresponds to a policy-derived category, cite the AIR-Bench category code. If a scenario has no matching AIR-Bench category, that is legitimately worth flagging — either the scenario is genuinely novel to your context or the category is buried and needs to be looked for more carefully.

**Why this overlay matters:** AIR-Bench 2024's category codes are directly attributable back to *policy language*. When a regulator asks "which policy does this evaluation defend," an AIR-Bench code answers the question by construction. It also lets you *run a benchmark* against the row — mods 104 and 108 use this hook.

## Cross-walking the four overlays

Not every row has a tag from every taxonomy — but the discipline is: *try*. If a row cannot be tagged in one of the four overlays, that is a datum, not a shrug. Some patterns you will see:

| Pattern | What to do |
|---|---|
| All four overlays tag cleanly | Row is a *well-covered* harm; it is the easy case. |
| NIST GenAI Profile is empty | Either the harm is not GenAI-specific (fine — this track covers non-GenAI harms too) or the scenario needs re-scoping. |
| ISO/IEC 23894 is empty | Very unlikely; if so, the row's `cause_bucket` (chapter 05) is probably mis-set. |
| AIRO is empty | The scenario touches an ontology class not yet represented — feed back to the ontology's issue tracker or add a bridging note. |
| AIR-Bench 2024 is empty | Either the harm has no corresponding policy language, or the scenario is enterprise-internal (e.g., internal-tool misuse) and no external policy captures it. |

Recording the reason for an empty tag is as important as recording the tag itself. It is the evidence that you tried.

## An overlay cross-walk table for the running example

Continuing HR-CS-INJ-01 (customer-support RAG assistant, indirect prompt injection via retrieved KB article) from chapters 01-02:

| Overlay | Tag |
|---|---|
| NIST GenAI Profile (AI 600-1) | Risk 9 — Information Security (primary); Risk 7 — Human-AI Configuration (secondary — insufficient oversight design over retrieved-content trust) |
| ISO/IEC 23894 risk sources | Data (upstream corpus quality / provenance); Use (retrieval trust boundary) |
| AIRO | `hasRiskSource`: data + model deployment context; `hasConsequence`: security breach + integrity of information provided; `hasImpactedStakeholder`: end user + operator; `hasMitigation`: provenance filter + tool allow-list |
| AIR-Bench 2024 | Category for security / prompt-injection policy-derived risks (identify the exact code in the benchmark) |

<!-- needs-research: replace the qualitative AIR-Bench line above with the exact hierarchical code once verified against the current AIR-Bench 2024 release. -->

Once every row of your harm model looks like this, downstream consumers can pick their lens.

## What the overlays do *not* replace

The overlays are labels; they are not:

- **A control catalogue.** The tags tell you *what class of harm* — not *what mitigation to deploy*. The control catalogue is chapter 05 / mod-108 territory.
- **Severity or likelihood.** Those live in the risk register (mod-103).
- **Regulatory obligation identification.** ISO/IEC 42001 clauses, EU AI Act articles, SR 11-7 requirements are mapped separately (mod-101 ch 03 and ch 04). Overlays are *for the harm model*; obligation maps are *for the framework map*.

Keeping these distinctions crisp is how the harm-modelling repo stays a single-purpose artifact.

## Summary

- Every harm-model row carries tags from four overlays: NIST GenAI Profile risk #, ISO/IEC 23894 risk source(s), AIRO ontology classes, AIR-Bench 2024 policy-derived category.
- Overlays let four different consumers — analyst, ISO auditor, platform team, benchmark run — read the same harm model through their preferred lens.
- Empty tags are recorded with a reason, not silently omitted; the reason is itself evidence.
- Overlays are labels, not controls, not severities, and not obligation maps — those live in other artifacts.
- Exercise 03 walks the MIT AI Risk Repository domain × causal overlay for the running system; exercise 04 completes the AIRO and NIST GenAI Profile cross-tag.
