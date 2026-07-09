# Grounding the Harm Model in Incident Corpora

An unevidenced harm claim is a *worry*. An evidenced harm claim is a *finding*. The difference matters when a regulator or an internal audit asks *"why is this scenario in your harm model — and why is that one not?"* This chapter tells you where to source the evidence and how to attach it to the harm-model rows produced in chapter 01.

## Motivation

Three groups will read your harm model — a governance analyst who files it, a peer engineer who will build a red-team fixture against it, and an auditor who will check that it exists. Each of them opens the model looking for the same thing: *does this row correspond to something that has actually happened to somebody, or to a plausible near-analog?* Rows that answer yes get taken seriously; rows that don't get flagged as unfounded and get you asked to defend the model as a whole. Grounding the harm model in public incident corpora is the cheapest way to keep that from happening.

Grounding also cuts the other way. A row that is *not* in your model but is well-represented in the corpora is a coverage gap. Regularly diffing your row set against the growing corpora is a maintenance discipline, not a one-shot activity.

## The three primary corpora

There are three public incident / risk corpora a level-25 harm-modeller has to know by name. Each has a different sampling bias; you use all three so no single bias governs your model.

### AI Incident Database (AIID)

Maintained by the Responsible AI Collaborative. AIID indexes real-world AI incidents reported in the public record — press coverage, court filings, regulator statements — and groups them into *incident* records, each linked to one or more *reports* (individual articles / documents). Fields on an incident include a narrative summary, the deployed system, the developer / deployer parties, the harmed parties, and taxonomic tags.

- URL: <https://incidentdatabase.ai/>
- API and CSV exports are available; the discovery app lets you search by keyword, deployer, or taxonomy tag.
- Bias to be aware of: *press-reported* incidents. AIID over-represents incidents involving publicly named companies and under-represents unreported enterprise incidents. Treat it as a floor for known-and-reported, not a census.

You cite an AIID incident by its stable `Incident N` identifier (and, when you're pointing at a specific source, one of its report IDs). Every harm-model row that plausibly matches an AIID incident should include the incident ID in `evidence_refs`.

### OECD.AI Incidents Monitor (AIM)

The OECD's AI Policy Observatory runs an AI Incidents Monitor that tracks AI-related incidents and hazards drawing on news and academic sources. Its taxonomy is aligned to the OECD's own AI classification and to OECD Recommendation on AI principles, which makes it useful for cross-referencing when you are filing to jurisdictions that reference the OECD framework.

- URL: <https://oecd.ai/en/incidents>
- Search and filter by country, principle affected, severity, and AI Principles categories.
- Bias to be aware of: overlaps significantly with AIID at the incident level but tags differently. Cite both when both apply — the tags feed different downstream consumers.

### MIT AI Risk Repository

Released by MIT FutureTech (Slattery et al., 2024), the MIT AI Risk Repository is an aggregation of AI risks extracted from a large corpus of existing risk frameworks and taxonomies. It ships two cross-cutting classifications you can use directly:

- A **causal taxonomy** — organising each risk row by who caused it (human vs. AI), whether the causation was intentional or unintentional, and the timing (pre- or post-deployment).
- A **domain taxonomy** — organising each risk row by top-level domain (for example: discrimination & toxicity, privacy & security, misinformation, malicious actors & misuse, human-computer interaction, socioeconomic & environmental, AI system safety & failures) and by sub-domain within each.

<!-- needs-research: cite the specific paper title, arXiv ID, and version number of the repository release referenced here; confirm exact count of source frameworks and top-level domain count against the current published version. -->

- URL: <https://airisk.mit.edu/>
- The repository is downloadable as spreadsheets; each row cites the source framework it was extracted from.
- Bias to be aware of: it is a *literature aggregation*, not an incident log — it tells you what has been *named as a risk*, not what has already occurred. Use it to check *coverage* rather than to prove *incidence*.

## How the three combine

You use each corpus for a different job:

| Corpus | Best used to answer |
|---|---|
| AIID | "Has a harm of this shape actually been reported, and to whom?" |
| OECD.AI AIM | "Which of the OECD AI Principles does this harm violate, and in which jurisdictions has it been logged?" |
| MIT AI Risk Repository | "Which categories in the aggregated risk literature does this harm map to — and which categories does my model not yet touch?" |

A defensible harm-model row cites at least one of AIID / OECD.AI *plus* MIT AI Risk Repository. If a row cites only MIT AI Risk Repository, it is a *coverage* row (something the literature warns about but no incident has manifested for a system like yours) — that's fine, but the row should say so, so the auditor can price the confidence appropriately.

## Adjacent corpora and reports worth pulling from

Beyond the three primary corpora, a working level-25 harm-modeller regularly pulls from:

- **Stanford CRFM AIR-Bench 2024** — categories derived from real government regulations and corporate policies; useful in chapter 03 as a taxonomy overlay and also as a bank of concrete policy-derived risk statements.
- **OWASP Top 10 for LLM Applications** — specific attack patterns and their harmful outcomes; already used in mod-101 ch 02.
- **MITRE ATLAS** — adversarial ML tactics and techniques with linked case studies; the case-study links are treatable as informal AIID adjuncts.
- **AVID (AI Vulnerability Database)** — vulnerabilities cross-referenced with SEP (Security, Ethics, Performance) categories. Complements AIID by covering vulnerabilities that never became publicly reported incidents.
- **AI Safety Institute (UK AISI, US AISI) publications** — evaluation reports on frontier systems; useful for corroborating capability-surface harm claims.
- **Academic literature and pre-prints** on specific harm classes (indirect prompt injection, memorisation, backdoored embeddings, jailbreak taxonomies). Cite the paper, not a summary.

<!-- needs-research: confirm current URL and maintainer status of AVID; if the project is inactive or has moved, adjust or remove. -->

## The evidence workflow — from corpus row to harm-model row

For each of your draft harm scenarios from chapter 01:

1. **Search AIID** for the deployment domain and the harm mechanism. Read the top few incident summaries and their linked reports. Note the ones that are analogous, even if the exact system differs.
2. **Search OECD.AI AIM** for the same. Prefer filters that align with your jurisdiction and with the OECD principle at issue (e.g., "human-centred values and fairness" for a bias scenario).
3. **Open MIT AI Risk Repository** and find the closest domain × sub-domain cell. Read the rows in that cell and look for language that describes your scenario. If none exists, that is worth flagging — it is either a coverage gap in the aggregated literature or a signal that your scenario is under-specified.
4. **Attach** the incident IDs (AIID `Incident N`, OECD AIM record link) and the MIT AIR row identifiers to the row's `evidence_refs` field.
5. **Note the analogy strength.** Not every match is direct. If the analogy is "similar system, similar harm mechanism, different target population," write that in the row. Auditors are fine with disciplined analogy; they object to unmarked analogy dressed up as evidence.
6. **Diff the reverse direction.** Skim the domain × sub-domain cell for MIT AIR rows that would apply to your system but are not represented in your model. Each of those is a candidate new row.

Steps 1–4 are the day-to-day evidencing workflow. Steps 5–6 are the *audit-survival* discipline. Both are trainable and repeatable.

## What to do when no incident matches

Some harm-model rows describe scenarios that have not yet manifested publicly — because your system is novel, because the harm is under-reported, or because it has occurred only inside enterprises that didn't disclose. In that case:

- Cite the closest **near-analog** in AIID / OECD.AI with an explicit *near-analog:* prefix in the row's `evidence_refs`.
- Cite the strongest **literature** row from MIT AI Risk Repository or from a published paper (e.g., an indirect-prompt-injection research paper) as the *pattern* citation.
- Mark the row's *confidence* as *literature-only* rather than *reported-incident* — mod-103 will use that field to price the row differently.
- Set a **watch** trigger on the row: when a matching AIID incident is added, the row is upgraded and the harm-model changelog notes the upgrade.

That last practice — literature-only rows carrying a watch trigger — is what keeps the harm model an *evidenced* document over time even for novel systems.

## A worked example — evidencing HR-CS-INJ-01 from chapter 01

Recall the customer-support RAG assistant's indirect-prompt-injection row from chapter 01. Populating `evidence_refs` and `taxonomy_tags` might look like this (evidence side only — taxonomy tags are chapter 03):

- **AIID:** search for `RAG`, `prompt injection`, `chatbot`, `customer support`. Cite any incident summaries where a chatbot returned unintended content because of upstream content it retrieved or ingested. Prefix each with `AIID Incident N — <one-line summary>`. If no exact match exists, mark as *near-analog*.
- **OECD.AI AIM:** find records tagged with `robustness` / `security` and involving user-facing conversational systems. Cite each by its record URL.
- **MIT AI Risk Repository:** find the "Privacy & Security" domain and its sub-domain covering AI system security failures / prompt injection. Cite the row identifiers.
- **Literature:** cite Greshake et al. on indirect prompt injection; cite OWASP LLM Top 10 categories LLM01 (Prompt Injection) and LLM06 (Sensitive Information Disclosure).

The row's `evidence_refs` after this becomes a compact list; the row is now defensible.

<!-- needs-research: replace the schematic "AIID Incident N" references with actual current AIID incident IDs when the module is used as a curriculum; the IDs change as the corpus grows and should be looked up at authoring time. -->

## Cadence

Incident corpora move. Set your review cadence explicitly:

- **Weekly:** skim AIID / OECD.AI new incidents; open any that touch your deployment domain.
- **Monthly:** check MIT AI Risk Repository release notes; check AIR-Bench, MITRE ATLAS, and AVID for updates.
- **Every release cycle:** diff new incidents against your harm model; every gap becomes either a new row or an explicit "we considered and ruled out" annotation.
- **Ad-hoc:** any major reported incident in your domain triggers an out-of-band review.

The cadence is not optional. An outdated harm model is worse than none, because it manufactures a false sense of coverage.

## Summary

- Harm-model rows without evidence are worries; grounding them in incident corpora turns them into findings.
- Use three primary corpora: AIID (reported incidents), OECD.AI AI Incidents Monitor (principle-tagged), and MIT AI Risk Repository (aggregated literature).
- Cite at least one incident source plus MIT AIR per row; literature-only rows must be marked as such and carry a watch trigger.
- Diff both directions — from your rows to the corpora and from the corpora to your rows — to catch coverage gaps.
- Refresh cadence: weekly on incident logs, monthly on taxonomy releases, every release cycle against the harm model.
- Exercise 02 walks the evidencing workflow end-to-end on your running system.
