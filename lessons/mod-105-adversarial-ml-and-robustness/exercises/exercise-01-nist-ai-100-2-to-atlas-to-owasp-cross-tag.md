# Exercise 01 — NIST AI 100-2 → MITRE ATLAS → OWASP ML Top 10 Cross-Tag Sheet

**Estimated effort:** 2 hours

## Objective

Build the **cross-tag sheet** that chapter 01 defines, for the ONE running AI system you will carry through exercises 02–05. Every attack scenario in scope for the system lands as one row, tagged in all three vocabularies (NIST AI 100-2 e2025 category + threat-model axes, MITRE ATLAS tactic + technique id, OWASP ML Top 10 entry + OWASP LLM Top 10 where dual-domain), traced back to a harm-model row (mod-102) and forward to a register row (mod-103). The sheet is the *upstream* artifact: nothing you engineer in exercises 02–04 gets run until its row exists here.

## Prerequisites

- Chapter 01 read; chapters 02–06 skimmed enough to know which attacks will land on the sheet.
- The running system chosen — the one carried through mod-101–103, or a fresh one with a real classifier / language-model surface. Note the choice and any switch on the deliverable.
- The mod-102 harm-model rows for this system (from mod-102 chapter 05 / mod-102 exercise 05) and the mod-103 register rows (from mod-103 exercise 05). Every row on your cross-tag sheet must trace to at least one of each.
- Working knowledge of the three source vocabularies:
  - NIST AI 100-2 e2025 read once end-to-end, focused on the six-category partition and the three-axis threat-model rubric.
  - MITRE ATLAS matrix walked once — you should be able to locate the Reconnaissance, ML Attack Staging, ML Supply Chain Compromise, ML Model Access, Exfiltration via ML Inference API, and Impact tactics without a search.
  - OWASP ML Top 10 read once; OWASP LLM Top 10 read if your system has a generative LLM surface.
- Access to a spreadsheet, table-friendly markdown, or a small YAML file — whatever medium your team's register uses. Do not invent a novel schema; the sheet is meant to be readable by a reviewer with no prior context.

## Problem statement

Enumerate the attack scenarios in scope for the running system across all six NIST AI 100-2 categories — evasion, poisoning, extraction, inversion, membership inference, availability — filtered by *what the system actually exposes*. For each scenario, produce one row that:

- Names the attack in NIST AI 100-2 terms (category + concrete technique) with the three-axis threat model (knowledge / capability / goal) filled in.
- Maps the technique to a MITRE ATLAS tactic and at least one technique id, citing the ATLAS release / snapshot date.
- Maps the scenario to an OWASP ML Top 10 entry (and OWASP LLM Top 10 entry where dual-domain), citing the version of each list.
- Traces back to at least one mod-102 harm-model row and forward to at least one mod-103 register row.
- Flags scope: `in-scope-this-module` (engineered in exercises 02–04), `handoff:mod-107` (privacy-metric depth), `handoff:mod-108` (guardrail deployment), `handoff:sec-L35` (runtime hardening at the inference edge), or `deferred:<reason>`.

The output is the sheet plus a short cover memo that argues *why* the six-category coverage is complete for this system — the memo is what an auditor reads first.

## Requirements

Produce all six items below. Missing any is an audit finding on the sheet itself.

1. **Cross-tag sheet** — `risk/mod-105/<system-id>/cross-tag.md` (or `.yaml` / `.csv` as your team's register prefers). Columns exactly as chapter 01's shape:
   - `scenario_id` — stable identifier reused across exercises 02–05 and in the register.
   - `nist_category` — one of `evasion / poisoning / extraction / inversion / membership_inference / availability`.
   - `attack_technique` — concrete technique name (e.g., `PGD-L∞`, `BadNets`, `Knockoff-Nets`, `LiRA`).
   - `attacker_knowledge` — `white_box / grey_box / black_box`.
   - `attacker_capability` — `passive / active_querier / training_data_contributor / supply_chain_participant`.
   - `attacker_goal` — `targeted / untargeted / availability / extraction`.
   - `atlas_tactic` — one or more ATLAS tactic names.
   - `atlas_technique_ids` — one or more technique ids (e.g., `AML.T00XX`).
   - `owasp_ml_top10_entry` — the ML Top 10 entry name and rank.
   - `owasp_llm_top10_entry` — the LLM Top 10 entry name and rank, or `n/a`.
   - `harm_model_ref` — mod-102 row id(s).
   - `register_ref` — mod-103 row id(s).
   - `scope_flag` — `in-scope-this-module` / `handoff:mod-107` / `handoff:mod-108` / `handoff:sec-L35` / `deferred:<ticket-or-reason>`.
   - `notes` — chain notes (recon → extract → surrogate-craft → evade), any known-weakness caveats on the mapping.
2. **Category-coverage cover memo** — `risk/mod-105/<system-id>/coverage-memo.md`. Two to three paragraphs arguing coverage across all six NIST categories:
   - For each category, either a list of scenario ids on the sheet, or a *justified* deferral / not-applicable statement (e.g., "availability is deferred because the endpoint sits behind an autoscaling gateway operated by the peer at sec-L35; the peer owns cost / DoS reporting").
   - No category may be silently omitted. A missing category is a coverage lie.
3. **Vocabulary-version pins** — at the top of the sheet, pin:
   - NIST AI 100-2 revision (e2025 or the version you read) plus URL / DOI.
   - MITRE ATLAS snapshot date and matrix URL.
   - OWASP ML Top 10 version and URL.
   - OWASP LLM Top 10 version and URL (if used).
   - Vocabularies drift; the sheet is undefendable in six months if the versions are not pinned.
4. **Trace back and forward** — every row must:
   - Point at ≥1 mod-102 harm-model row (`harm_model_ref`). If no harm-model row exists, either the harm model is incomplete (file a mod-102 update) or the scenario is out of scope (mark `deferred:` and cite the reason).
   - Point at ≥1 mod-103 register row (`register_ref`). If none exists, open one via the mod-103 process and cite the new row id.
   - No orphan rows. Chapter 01's invariant is that the sheet is a *routing* artifact, not a new source of truth.
5. **Chain-note callouts** — for any scenario that a real adversary would reach only via a chain of prior attacks (recon → extraction → surrogate-craft → evasion; supply-chain compromise → backdoor → trigger delivery), state the chain in `notes` and cite the upstream scenario ids. The sheet's rows are per *primary* category; the chain relations live in notes.
6. **Handoff callouts** — for every row with `scope_flag` other than `in-scope-this-module`, name the peer / module that owns the next step and the artifact that will carry the handoff (e.g., "mod-107 chapter 03 owns LiRA calibration; MIA baseline advantage here is the intake"; "sec-L35 handoff contract row for aggregate-query anomaly monitor; specification lands in exercise 04").

Cross-cutting rules:

- **Every attack you *implement* in exercises 02–04 must appear as a row here first.** If you cannot name the attack in NIST + ATLAS + OWASP terms, don't run it yet. Fix the taxonomy debt on this sheet before touching the tool.
- **Do not collapse NIST and OWASP into a single tag.** The two labels answer different review questions (taxonomy defensibility vs. AppSec routing) and different reviewers read them; keep both columns filled per row.
- **Dual-domain scenarios get a dual OWASP tag.** A safety-classifier evasion that is also a jailbreak on the underlying LLM gets both an ML Top 10 entry and an LLM Top 10 entry; the mod-104 red-team programme reads the LLM column and the mod-105 eval reads the ML column.

## Starter guidance

- **Start from the mod-102 harm model, not from the attack list.** Walk the harm-model rows and ask, "what adversarial-ML technique would realise this harm?" That gives you the in-scope scenarios grounded in real system risk. Starting from an attack catalogue tends to import scenarios the system does not actually expose.
- **Availability is easy to under-cover.** Sponge examples, adversarial-cost attacks, and energy-latency attacks are legitimate AI 100-2 rows. Even if the runtime enforcement is owned by the peer at sec-L35, this module names the scenario and files a handoff row — don't leave the category empty by default.
- **The three-axis threat model matters even when it looks obvious.** A "black-box, active-querier, extraction" scenario against a public-facing endpoint has a very different defence surface from a "grey-box, supply-chain-participant, poisoning" scenario. Filling in the three axes forces the granularity the defence work will need.
- **ATLAS technique ids drift.** Cite the technique id *and* the ATLAS snapshot date. When ATLAS renames or reorganises a cell between releases (it does), you want the pinned reference.
- **Do not chase completeness against every published technique.** The sheet is for *your* system. A row about federated-learning gradient inversion in a system that has no federated pipeline is noise, not coverage. Deferrals with a reason are the correct outcome for those.
- **The chain notes are where a real adversary lives.** Real incidents are chains, not single techniques. If your sheet has no `notes` column entries linking scenarios, the sheet is probably classifying techniques in isolation instead of thinking about how an attacker would compose them.
- **If the OWASP ML Top 10 has no clean match for a scenario, cite the closest entry and note the mismatch.** OWASP is a rank-ordered app-security list, not an exhaustive taxonomy; a rare-but-real scenario may sit under a nearest-neighbour entry. Do not invent an entry that isn't in the current version.

## Acceptance criteria

The submission is accepted when:

- The cross-tag sheet exists at the specified path, uses the exact chapter-01 column shape, and pins the versions of NIST AI 100-2, ATLAS, and OWASP ML Top 10 (and LLM Top 10, if used) at the top.
- Every row has all thirteen columns filled — no blank cells; deferrals are explicit with a reason.
- All six NIST AI 100-2 categories are addressed on the coverage memo: either with populated scenario rows or with a justified deferral / not-applicable statement per category.
- Every row's `harm_model_ref` points at a real mod-102 row; every row's `register_ref` points at a real mod-103 row. New register rows opened during the exercise are cited by id.
- Every row's `scope_flag` is set; every non-`in-scope-this-module` row names the receiving peer / module and the handoff artifact.
- Attack chains are called out in `notes` where a real adversary would compose scenarios; single-technique rows are the exception, not the rule.
- A reviewer opening the sheet and the memo can, in fifteen minutes, answer: which attacks will be engineered in exercises 02–04, which are handed off, which are deferred and why, and which register rows will move as a result.

## Stretch goals

- Draft the **register-row shape** the mod-103 owner will need for a new adversarial-ML row (columns, evidence pointers). This is the input to the "open a mod-103 row" step; formalising it as a small template saves round-trips in exercises 03–05.
- Contribute a **novel row not already in ATLAS** — a technique your harm model surfaces that the matrix does not yet have a cell for. Draft the tactic + technique proposal in `risk/mod-105/<system-id>/atlas-proposal.md`; note it as a candidate community contribution, do not submit unless you have the org's clearance.
- Write a one-paragraph **taxonomy-drift note** — what will change on this sheet when NIST AI 100-2 next revises, when OWASP ML Top 10 next re-ranks, and when ATLAS next reorganises. The paragraph is the maintenance ticket a future you will read.
- For one scenario where OWASP ML Top 10 has no clean match, propose which entry should absorb it (or which new entry the community should consider). Do not fabricate an entry; state the mismatch openly.

## Deliverable format

A single markdown document `mod-105-ex01-cross-tag.md` that indexes the artifacts, plus the artifacts themselves. The index document contains:

1. *Chosen running system* — one paragraph naming the system, the release under test, and (if different from your mod-101–103 system) why the switch.
2. *Version pins* — the four vocabulary versions and URLs (NIST AI 100-2, ATLAS, OWASP ML Top 10, OWASP LLM Top 10).
3. *Category coverage summary* — one row per NIST category with the count of scenarios and the pointer to the memo justification.
4. *Sheet-shape sanity* — a screenshot or a rendered slice of the first three rows demonstrating the column shape.
5. *Traceability index* — every `scenario_id` mapped to its `harm_model_ref` and `register_ref`. Missing traces are audit findings on the sheet.
6. *Handoffs opened* — every non-`in-scope-this-module` row and the receiving peer / module.
7. *New register rows opened* — the ids and their scenarios, if any.
8. *Chain notes* — the attack chains called out in `notes`.
9. *Open items* — anything flagged for exercises 02–04 or for upstream mod-102 / mod-103.

## Submission

Attach the index markdown and the cross-tag artifact to the module submission channel. If the running system is under NDA, redact system identifiers but preserve the NIST / ATLAS / OWASP columns, the threat-model axes, the scope flags, and the trace-refs — a reviewer cannot review a cross-tag whose taxonomy columns are blanked out.
