# Exercise 05 — Regulator-Facing Incident Summary Authoring

**Estimated effort:** 3 hours

## Objective

Reshape the post-mortem you authored in exercise 03 into two mandatory regulator-facing extracts — the EU AI Act Article 73 serious-incident package and the SR 11-7-shaped MRM incident memo — plus one concurrency-driven extract (HIPAA breach notification or GDPR Article 33, chosen based on the scenario's data-subject residency). Then produce the cross-package consistency-check file that confirms timeline dates, causal-chain wording, and impact counts agree across the extracts, and the submission-log entries that would land on the incident record when each extract is submitted under its clock. The exercise trains the "one incident, three regulator-facing packages" discipline that chapter 05 defines: a single post-mortem as source of truth, declarative extracts as views, and CI-style consistency checks so the regulator does not read the disagreement between packages as the finding.

## Prerequisites

- Chapter 05 read in full — the three obligation shapes, the timing envelopes, the extract-generation shape, and the consistency-check discipline are the reference shapes.
- Chapter 03 read for the ten-section post-mortem shape — the extracts are lifts from the post-mortem's sections, not re-authored documents.
- Exercise 03 completed — the post-mortem file (`post-mortem.md`) and its `extracts/regulator-extract.md` are the inputs; every field in every package is a lift from a section of the post-mortem, per chapter 05's field-mapping tables.
- Access to the primary regulatory texts cited in `resources.md`: EU AI Act Regulation (EU) 2024/1689 Articles 3, 72, 73; SR 11-7 (Federal Reserve) and OCC 2011-12; 21 CFR Part 803 (FDA MDR) if you extend to MDR; HIPAA Breach Notification Rule (45 CFR Part 164 Subpart D); GDPR Article 33 (Regulation (EU) 2016/679). Use the primary texts, not paraphrases.
- Legal counsel and compliance function pointers (or peer placeholders) — the risk engineer *authors* the extracts; legal and compliance sign off and submit. Chapter 05 §"Practical authoring notes" pins this.

## Problem statement

The mock incident INC-2026-08-14-cs-rag-inj-01 involves a customer-service RAG assistant deployed to EU and US customers. The exfiltrated data is a customer's email address; the customer is an EU resident (personal data of a data subject in the EU). The organisation is a US financial institution offering health-insurance-adjacent customer-service journeys, which brings SR 11-7 (through the assistant's role in customer-decisions on billing) and HIPAA (through the narrow subset of PHI-touching flows) into scope, in addition to the EU AI Act obligation on the deployer of the high-risk system. The GDPR Article 33 72-hour clock runs concurrently on the personal-data breach.

You are the risk engineer. The engineering post-mortem is signed (drill). Legal and compliance need three extracts on their desk so they can submit under the correct clocks. Chapter 05 §"The 'one incident, three regulator-facing packages' pattern" defines the folder shape and the extract-generation discipline; your job is to produce the folder for this incident and the consistency-check that keeps the packages from drifting.

Note the scope boundary: this exercise authors *extracts*. It does not actually submit. The submission-log entries are records of what *would* be submitted, to which authority, at which timestamp under the clock — the record shape from chapter 05 §"The 'one incident, three regulator-facing packages' pattern," not a real regulator interaction.

## Requirements

Produce all five deliverables:

1. **The EU AI Act Article 73 serious-incident package** — a markdown file `extracts/eu-ai-act-article-73-v1.md` with every field from the table in chapter 05 §"EU AI Act — Article 73" filled by lift from the post-mortem. Explicitly:
   - Name the Article 3 sub-category the incident falls under (death / serious health damage; serious-and-irreversible disruption of critical infrastructure; infringement of Union law obligations intended to protect fundamental rights; serious damage to property or environment). For this incident, argue the category with the specific evidence — the incident may or may not meet a sub-category floor; if it does not, the extract records that finding with the criteria that were checked.
   - Record the timing envelope with the clock start explicitly named. Chapter 05 §"Timing envelopes summarised" gives the envelopes (without undue delay / 15 days for the general case; 2 days for widespread infringement or critical-infrastructure disruption; 10 days for death). The clock starts at *awareness* of the serious incident, not at internal confirmation — cite the timeline row on the post-mortem that establishes awareness.
   - Fill every remaining field: provider / deployer identity, AI system identification, date and place, facts, immediate mitigating actions, corrective measures with due dates, preliminary root cause (the two-sentence causal chain from exercise 02), post-market monitoring signal that surfaced the incident (the mod-109 monitor from exercise 01), follow-up reports schedule.
   - Where any field cannot be filled, fill it explicitly as "not applicable" or "not yet known — expected by <date>" with the reason. Blank is a defect.
2. **The SR 11-7-shaped MRM incident memo** — a markdown file `extracts/sr-11-7-mrm-memo-v1.md` with every field from the table in chapter 05 §"SR 11-7" filled by lift from the post-mortem. Explicitly:
   - State the model tier / materiality assessment (a customer-service assistant that touches account information is not the same materiality as a credit-decision model; make the argument for tier from the scoring scheme mod-110 pins). If the organisation's MRM guidance sets a materiality threshold that this incident does *not* meet, the memo records that with the criteria — but chapter 05 pins that SR 11-7 is guidance, not a hard-clock rule, so the memo goes on the MRM committee cadence regardless.
   - Fill every remaining field: model identifier, business use, incident / discovery / mitigation dates, impact on model output and downstream decisions, root cause, ongoing-monitoring evidence, controls affected (which controls were defeated; which are re-credited on the CAP), validation impact (whether re-validation is triggered), reporting to MRM governance and committee with the sign-off routing.
   - Where the incident brings a third-party model provider into scope (the assistant's underlying model is a vendor model), append the SR 23-4 attribute block: supplier identifier, contract-anchored control credits, supplier-side incident references (if any).
3. **The concurrency-driven extract (HIPAA breach OR GDPR Article 33)** — a markdown file at `extracts/hipaa-breach-notification-v1.md` OR `extracts/gdpr-article-33-v1.md`. The scenario has a PII exposure of an EU-resident customer's email address; the scenario mentions health-insurance-adjacent flows but does not confirm PHI was in the exfiltrated payload. You choose the extract based on the data-subject residency and payload type, and *you record the reasoning explicitly*. If GDPR is chosen, the extract records the 72-hour clock start and the supervisory-authority routing per Article 33. If HIPAA is chosen, the extract records the 60-day individual / HHS clock start and the ≥ 500-affected media-notification threshold check. If both apply, choose the one whose clock fires first and record the reasoning; the other stub is drafted at the placeholder level with a "concurrency: this obligation also applies; separate extract in progress" note.
4. **The cross-package consistency-check file** — a markdown file `diff-checks/eu-vs-sr-11-7-vs-<hipaa-or-gdpr>-consistency.md` per chapter 05 §"The 'one incident, three regulator-facing packages' pattern." The file walks three axes:
   - *Timeline dates* — for every date field appearing in any two extracts (incident date, discovery date, mitigation date, notification date, follow-up dates), quote the field from each extract side by side and confirm they match. Every mismatch is called out with a justification (e.g., "SR 11-7 memo cites the MRM-committee-notification date; EU package cites the market-surveillance-authority-notification date — these are two different events by design").
   - *Causal-chain wording* — quote the causal-chain paragraph from each extract; confirm they cite the same delta as cause and the same delta as symptom. Divergence in causal framing between extracts is a defect chapter 05 §"Motivation" — the re-authored post-mortem — is designed to prevent.
   - *Impact counts* — for named-party count, population-scale estimate, controls-defeated count, controls-added-on-CAP count, and any monetary or duration figure that appears in more than one extract, confirm the numbers agree. Numbers that must differ (an SR 11-7 memo may express *materiality* as a bucket while the Article 73 package expresses *categorisation* under Article 3 — different framings, both legitimate) are documented with the deviation reason.
5. **The submission-log entries** — a folder `submission-log/` with one markdown file per submission that *would* be filed. For this scenario, produce at minimum:
   - `submission-log/2026-08-14-eu-ma-provisional.md` — a record for the provisional Article 73 submission: what was submitted (extract v1), to which authority (member-state market-surveillance authority; identify by convention if you have no real one), at which timestamp (the clock-start timestamp from the EU extract plus the envelope), and by whom (the accountable submitting role — not the risk engineer).
   - `submission-log/<date>-sr-11-7-mrm-committee.md` — a record for the MRM committee filing per the memo's routing.
   - `submission-log/<date>-<hipaa-or-gdpr>.md` — a record for the concurrency extract per its clock.
   Each log entry records the extract version submitted, the authority, the timestamp, the submitting role, and a placeholder for the receiving-authority acknowledgement.

## Starter guidance

- **The extract is a lift, not a rewrite.** Chapter 05 §"The 'one incident, three regulator-facing packages' pattern" pins this. Every field in every package points at a specific section of `post-mortem.md`; if you find yourself authoring new content in an extract, either you are updating the post-mortem (which is fine, then regenerate the extract) or you are drifting (which is not fine, and the consistency check will catch it).
- **The clock starts at awareness, not at confirmation.** Chapter 05 §"Motivation" — the clock miscount — is explicit. For this scenario, "awareness" is the moment on-call opened the incident at the monitor fire (2026-08-14 09:12Z), not the moment the post-mortem was signed (drill 2026-08-25). Cite the timeline row that establishes awareness. If you slip the clock start, the regulator interaction opens with a lateness finding.
- **Use the language of the obligation.** Chapter 05 §"Practical authoring notes" pins this. Article 73's "serious incident" categories, SR 11-7's "material" framing, and (if you extend to MDR) 21 CFR Part 803's "serious injury" definition each carry specific meaning in the source text. Do not paraphrase the categorisation; quote the definitional language and argue the fit with evidence.
- **"Not reportable" is a documented decision, not an omission.** If, on the argument, this incident does not meet an Article 3 sub-category floor for a "serious incident" (a customer-email exfiltration may not, depending on argument and jurisdiction), the extract records that decision explicitly with the criteria checked. Chapter 03 §"8. Regulator classification and reporting log" pins the same rule for the post-mortem's regulator section; the extract inherits it.
- **The risk engineer authors; legal + compliance submit.** Chapter 05 §"Practical authoring notes" pins the role boundary. Do not put the risk engineer on the submitting-role line of any log entry — that is the accountable-submitting role your organisation names (typically compliance or legal counsel with a delegated authority).
- **The concurrency stub is not busywork.** For this scenario, both GDPR Article 33 (EU-resident data subject, 72-hour clock) and HIPAA (US health-insurance-adjacent flow, 60-day clock) can arguably run. Chapter 05 pins that concurrency handling is what motivates the extract pattern; the exercise trains the reasoning. Pick one to author fully, stub the other with a note.
- **The consistency check has three axes and every axis catches a real drift.** Timeline drift is the most common — teams re-author dates in different formats and they diverge. Causal-chain drift is the most damaging — a regulator that reads two different root causes across two packages sees an organisation that does not know why the incident happened. Impact-count drift is the most embarrassing — an Article 73 package that reports "one affected party" and a HIPAA notification that reports "affected individuals include..." must reconcile.
- **Chapter 05's needs-research markers apply here.** The EU AI Act Article 73 implementing acts, the FDA MDR forms, and the HHS breach portal fields all evolve. Where a specific field format is required by the obligation and you cannot verify the current form at authoring time, write `<!-- needs-research: verify current field format at authoring time -->` and continue. Do not invent form field names.

## Acceptance criteria

The submission is accepted when:

- The EU Article 73 extract has every field from the chapter 05 §"EU AI Act — Article 73" table filled by lift from the post-mortem; the Article 3 sub-category argument is present with cited evidence; the clock start is named with a specific timeline row.
- The SR 11-7 memo has every field from the chapter 05 §"SR 11-7" table filled; the model-tier / materiality argument is present; the MRM governance sign-off routing is named.
- The concurrency-driven extract (HIPAA or GDPR) is authored fully with the correct clock start and routing; the *other* concurrent obligation has a placeholder stub with the reasoning for concurrency.
- The consistency-check file walks all three axes (timeline dates, causal-chain wording, impact counts) and calls out every deviation with a justification.
- The submission-log folder has at least three entries — one per authored extract — each recording the extract version, authority, timestamp, submitting role (not the risk engineer), and receiving-authority acknowledgement placeholder.
- Every field marked "not applicable" or "not yet known" cites the criteria checked or the expected-by date. No silent blanks.
- The extracts do not introduce content that is not in the post-mortem; every fact in every extract is a lift, and the consistency check confirms it.
- Language is the obligation's own — Article 3 categories are named by the Article's terms; SR 11-7 "material" is used as the guidance frames it; the HIPAA / GDPR clock language matches the rule.
- A legal or compliance reader can pick up any single extract and know: what obligation it addresses, when the clock started, when it expires, who submits, and where the source-of-truth facts live.

## Stretch goals

- **Add the FDA MDR extract.** If the running system has any SaMD-adjacent surface, author `extracts/fda-mdr-3500a-v1.md` per the table in chapter 05 §"FDA MDR." Note the 30-calendar-day and 5-working-day envelopes and cite the mock event class. Even for this exercise's non-SaMD scenario, drafting an "MDR not applicable — criteria checked" stub records the reasoning and closes the concurrency question.
- **Write a consistency-check script.** Chapter 05 §"The 'one incident, three regulator-facing packages' pattern" implies a CI-style check. Write a small script (`diff-checks/consistency_check.py` or equivalent) that reads each extract, extracts the date fields, the causal-chain paragraph, and the count fields, and fails when any drift is found. Running this in CI turns "extracts drifted" from a manual find into a build failure.
- **Round-trip with legal or compliance.** Hand the extracts to a peer or a real compliance function (redact identifiers if necessary). Collect the feedback on which language does not track the obligation and which fields the accountable-submitting role would rewrite anyway. That feedback is the actual authoring bar; the exercise's acceptance criteria are the floor.
- **Author the follow-up-report schedule.** Every obligation expects follow-ups — Article 73 progress reports, an SR 11-7 update to the committee once the CAP closes, a HIPAA / GDPR annual-log entry. Add a `follow-ups/` folder listing the scheduled follow-up reports with due dates and the sections of the post-mortem they will lift from.

## Deliverable format

A single index markdown document `mod-111-ex05-regulator-packages.md` that contains, in order:

1. *Summary* — one paragraph naming the incident, the three obligations in scope, the concurrency reasoning, and the clock starts.
2. *Folder layout* — the `regulator-packages/INC-2026-08-14-cs-rag-inj-01/` tree per chapter 05 §"The 'one incident, three regulator-facing packages' pattern," with `post-mortem.md`, `evidence-pack/`, `extracts/`, `submission-log/`, `diff-checks/` all named and populated.
3. *Article 73 extract excerpt* — the header, the Article 3 sub-category argument, the timing envelope, and the follow-up schedule quoted inline.
4. *SR 11-7 memo excerpt* — the materiality argument, the MRM governance routing, and the SR 23-4 supplier attributes (if applicable) quoted inline.
5. *Concurrency extract excerpt* — the choice, the reasoning, and the clock start quoted inline; the stub for the non-chosen concurrent obligation quoted inline.
6. *Consistency-check excerpt* — the three-axis table quoted inline, with every deviation flagged.
7. *Submission-log entries* — one line per entry in the folder.
8. *Cross-references* — pointers to the post-mortem source of truth (exercise 03), the CAP evidence-of-completion (exercise 04's fixture ID), and the RCA evidence pack (exercise 02).
9. *Open items* — anything blocked on real legal / compliance sign-off, a real supervisory-authority identifier, a real HHS portal submission field, or the needs-research markers from chapter 05 that could not be resolved at authoring time.

## Submission

Attach the index markdown plus the full `regulator-packages/INC-2026-08-14-cs-rag-inj-01/` folder. If any part of the extract references real regulator names, supervisory authority identifiers, or accountable-submitting roles from your organisation, redact them per the organisation's information-classification policy but preserve the extract structure and the clock records. The extracts are not submissions; they are the drafts that go to legal and compliance for sign-off and submission. The consistency-check file and the submission-log folder are the audit-facing artefacts that show the organisation ran the "one incident, three regulator-facing packages" discipline for this incident — the audit reads them before it reads the extracts themselves.
