# Exercise 03 — MIT AI Risk Repository Taxonomy Overlay

**Estimated effort:** 2 hours

## Objective

Overlay the **MIT AI Risk Repository causal taxonomy** and **domain taxonomy** onto every row of the harm catalog from exercise 02. The output is a catalog whose every row carries a stable, defensible taxonomy tag that a reviewer can read against the current MIT AIR release.

## Prerequisites

- Chapter 03 (this module) read.
- Exercise 02 completed — the evidenced harm catalog is the input.
- Access to the MIT AI Risk Repository — <https://airisk.mit.edu/> — including the downloadable causal and domain taxonomy spreadsheets for the current release.
- Note the release version you're working against; taxonomy content changes across releases and your citations must be pinned.

## Problem statement

For every row of your harm catalog, tag it with:

- **Causal taxonomy** — the MIT AIR causal dimensions (Entity: Human vs. AI vs. Other; Intent: Intentional vs. Unintentional vs. Other; Timing: Pre-deployment vs. Post-deployment vs. Other). Every row gets exactly one code per dimension unless the scenario truly splits, in which case use two rows with a shared `id_group`.
- **Domain taxonomy** — the MIT AIR top-level domain and its sub-domain. Every row gets one primary domain / sub-domain pair; secondary pairs are allowed and encouraged where the row genuinely spans.
- **Row identifiers cited** — for each tag, note which MIT AIR row(s) you consulted from the current release to justify the classification. The tag without a cited row is your interpretation; with a cited row it is defensible.

<!-- needs-research: at time of authoring, learners must verify the exact domain and sub-domain labels against the current MIT AIR release; historical documentation cites 7 top-level domains and ~23 sub-domains, but the release-specific counts and labels should be checked live. -->

## Requirements

- Output as `mod-102-ex03-taxonomy-overlay.md` — either amending the exercise-02 catalog in-place with new columns, or as a separate overlay document that references the exercise-02 IDs.
- Cite the MIT AIR release version you consulted at the top of the document.
- Every row from exercise 02 gets a causal (Entity / Intent / Timing) triple and a primary domain / sub-domain pair. Secondary domain / sub-domain pairs are optional but noted where they apply.
- Every tag cites at least one MIT AIR row identifier from the current release as the justification. Multiple cited rows are welcome; zero-cited rows are unacceptable.
- Add a *reverse-sweep* section that lists MIT AIR sub-domains where your harm catalog currently has *no* rows, and asks explicitly: is this sub-domain out of scope for the running system, or is it a coverage gap? Answer each question in one sentence.
- Add a *distribution* section that summarises how many of your catalog rows fall in each domain and each causal-triple combination. The distribution is a first look at where the harm model is dense vs. thin — a signal downstream mod-103 uses.

## Starter guidance

- Begin with the causal taxonomy — it is only three dimensions and the classification usually falls out of the persona and capability-surface columns.
  - `Entity` mirrors the persona (adversarial user / insider = usually Human; upstream data supplier can be Human or AI-authored content; downstream integrator = Human or AI depending on the integrator; the model itself acting is AI).
  - `Intent` mirrors whether the persona is trying to cause the harm (adversarial user = usually Intentional; insider = varies; upstream data supplier = varies; downstream integrator = usually Unintentional).
  - `Timing` mirrors when the risk is realised (Pre-deployment for training-time / data-time origins; Post-deployment for runtime realisations).
- Then walk the domain axis. Read the domain descriptions in the current MIT AIR release and pick the *closest match*; if two domains fit equally, pick the one whose sub-domain rows include a row that matches your scenario, and list the other as secondary.
- If your catalog has more than one row that ends up with the same causal triple + same domain + same sub-domain, verify they aren't duplicates. If they aren't, note briefly why they are distinct despite the identical tag.
- If a scenario doesn't cleanly fit any MIT AIR domain, that is a data-point — write a one-paragraph reason and file it under exercise 04 as an *ontology-gap* note. Do not force-fit.

## Acceptance criteria

The submission is accepted when:

- Every row from exercise 02 has a causal triple, a primary domain / sub-domain pair, and at least one MIT AIR row identifier cited per tag.
- The MIT AIR release version is cited at the top of the document.
- The reverse-sweep section addresses *every* MIT AIR sub-domain that is unrepresented in your catalog, with an in-scope / out-of-scope disposition.
- The distribution section is present and shows the row counts per domain and per causal-triple.
- A reviewer can, in under 15 minutes, sample three rows and confirm the tag matches the cited MIT AIR row.

## Stretch goals

- Cross-walk each domain / sub-domain to the NIST GenAI Profile risk (mod-101 ch 02) that most closely aligns. This bridges exercise 04 in advance.
- Draft a *taxonomy watch* note that lists which MIT AIR sub-domains your catalog is *thin* in and would benefit from a targeted deeper harm-modelling pass; feed it to exercise 05 or to mod-103.
- Publish the tags as YAML alongside the markdown — one file whose entries carry both the exercise-02 evidence and the exercise-03 tags. That YAML is what the register (mod-103) will read.

## Deliverable format

A single markdown document with:

1. *MIT AIR release version and access date* at the top.
2. *Overlay* — per-row causal triple + primary domain / sub-domain + cited row identifiers.
3. *Reverse-sweep dispositions* — one paragraph per MIT AIR sub-domain not currently represented.
4. *Distribution* — small table showing row counts per domain and per causal-triple.
5. *Ontology-gap notes* — any scenarios that resist MIT AIR classification, with a proposed handling.

## Submission

Attach the markdown (and YAML if produced) to the module submission channel.
