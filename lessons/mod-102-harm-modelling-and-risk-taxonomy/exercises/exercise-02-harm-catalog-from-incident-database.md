# Exercise 02 — Harm Catalog From Incident Database Evidence

**Estimated effort:** 3 hours

## Objective

Take the persona × capability-surface matrix from exercise 01 and turn every occupied cell into a *harm-catalog row* with real evidence citations from AIID, OECD.AI AI Incidents Monitor, and MIT AI Risk Repository. The output is a defensible harm catalog whose every claim points at something a reviewer can independently verify.

## Prerequisites

- Chapter 02 (this module) read.
- Exercise 01 completed — the matrix and stable IDs are the input to this exercise.
- Working access to:
  - AI Incident Database — <https://incidentdatabase.ai/>
  - OECD.AI AI Incidents Monitor — <https://oecd.ai/en/incidents>
  - MIT AI Risk Repository — <https://airisk.mit.edu/>
- A note-taking tool. This exercise is search-heavy.

## Problem statement

For each occupied cell in your exercise-01 matrix, produce a *harm-catalog row* that includes the immediate → secondary → cascading → systemic chain (chapter 01) and at least the following evidence references:

- ≥ 1 direct or *near-analog* AIID incident (cited by AIID incident ID).
- ≥ 1 OECD.AI AIM record where applicable (cited by record URL); mark *no matching AIM record* explicitly if none is found after a genuine search.
- ≥ 1 MIT AI Risk Repository row identifier (cited by the row identifier used in the current repository release).
- Additional citations from OWASP LLM Top 10, MITRE ATLAS, AVID, or academic literature where relevant.

Rows where no incident matches must be marked *literature-only* and carry a *watch* trigger for future upgrade (chapter 02).

## Requirements

- Output as a single markdown document `mod-102-ex02-harm-catalog.md` that supersedes the persona × capability-surface matrix and now becomes the canonical harm catalog for the running system.
- Cover *all* occupied cells from exercise 01. If a cell now looks like a duplicate under evidence review, collapse it and record the collapse.
- Each row has the fields: `id` (from exercise 01), `persona`, `capability_surface`, `immediate_harm`, `secondary_harm`, `cascading_effect`, `systemic_effect`, `evidence_refs`, `confidence`.
- `confidence` is one of `reported-incident` (at least one direct AIID / OECD.AI match), `near-analog` (matches exist but differ in system class or population), or `literature-only` (only MIT AIR / literature citations).
- Every AIID incident cited must be by its stable ID; every OECD.AI record must be by record URL; every MIT AIR row must be by its release row identifier and the release version.
- Include a *dated cadence marker* at the top ("evidence collected against AIID / OECD.AI / MIT AIR as of `<YYYY-MM-DD>`") so future re-checks can diff against a known corpus state.
- The reverse-direction sweep (chapter 02) is required: skim MIT AIR and AIID for harms *your model doesn't yet cover* and file each as a *coverage-gap candidate* in a separate section, not as new rows.

## Starter guidance

- Start by searching AIID for the deployment domain of your running system (healthcare triage, customer support, hiring, credit scoring, code assistant, etc.) — not by the personas. Deployment-domain-anchored searches surface the analog incidents fastest.
- For each analog incident you find, note which of your exercise-01 rows it evidences and which it *doesn't*. The rows it doesn't evidence are the ones you go looking for separately.
- Use OECD.AI AIM's principle filters to double-index — a harm that violates "human-centred values and fairness" often has records that AIID has not surfaced.
- For MIT AIR, open the domain × sub-domain cell you expect first; the sub-domain rows are dense and will give you multiple candidate citations at once.
- When you cannot find a direct incident match, be honest: mark `confidence: literature-only` and add a watch trigger. A watched literature-only row is defensible; a dressed-up literature-only row is not.
- Do not paste incident narratives into your document verbatim if they are behind press paywalls or contain personal data — cite the incident ID and a two-sentence summary in your own words.

## Acceptance criteria

The submission is accepted when:

- Every occupied row from exercise 01 has been either evidenced or collapsed with a note.
- Every row has a `confidence` field with a value the citations actually support.
- Every AIID / OECD.AI citation is by stable identifier or URL; every MIT AIR citation is by row identifier and release version.
- The dated cadence marker is present.
- The coverage-gap candidate list contains at least three items (a genuine reverse sweep will find candidates; zero suggests the sweep wasn't done).
- A reviewer can click through the citations and confirm the analogy strength for at least three sampled rows within 20 minutes.

## Stretch goals

- Add a *counter-evidence* column — for each row, note any published evidence that the harm has *not* materialised in similar systems, or note that no counter-evidence was found. Counter-evidence is a real signal for mod-103's likelihood pricing.
- Publish the harm catalog as YAML alongside the markdown; a machine-readable form is what the mod-103 register will read.
- Draft a monthly diff cadence with the analyst function (mod-102 ch 06) that specifies who watches which corpus and how updates flow into the harm catalog.

## Deliverable format

A single markdown document (and optional YAML) with the following sections:

1. *Running system + intake reference* (unchanged from exercise 01).
2. *Cadence marker* — the corpus dates against which evidence was collected.
3. *Harm catalog* — one row per persona × capability-surface cell, populated per the requirements above.
4. *Collapsed rows* — rows from exercise 01 that were merged or dropped, with the reason.
5. *Coverage-gap candidates* — rows the corpora surface that your model does not yet cover; each with a *proposed* new row-ID and a one-sentence description. Do not silently promote these to catalog rows; they wait for review.

## Submission

Attach the markdown (and YAML if produced) to the module submission channel. If any evidence is behind paywalls or under NDA, cite the identifier only and do not paste the source text.
