# Exercise 03 — Post-Mortem Authoring for a Mock Jailbreak Incident

**Estimated effort:** 3 hours

## Objective

Author the full ten-section post-mortem for the mock jailbreak incident INC-2026-08-14-cs-rag-inj-01 whose RCA you produced in exercise 02. The post-mortem is a *single* file that carries the load for three audiences — engineering, governance council, regulator — with audience-specific extracts generated as views over the same file, not re-authored. Then produce the mod-103 register update patch, a stand-alone corrective-action-plan (CAP) table with named owners and evidence-of-completion artefacts, and a section-by-section self-check against the anti-patterns in chapter 03. The exercise trains the discipline of writing *one* controlled document instead of three drifting ones — the discipline the audit reads as the difference between an obligated organisation that has its story straight and an organisation whose engineering post-mortem, council briefing, and regulator submission disagree on the facts.

## Prerequisites

- Chapter 03 read in full — the ten-section shape, the three audience-specific extracts, the CAP schema, and the anti-pattern table are the reference shapes.
- Exercise 02 completed — the RCA file (`mod-111-ex02-rca.md`) supplies section 4 (causal chain), section 3 (timeline: detection, routing, mitigation), section 5 (evidence pack), and the hand-off records the post-mortem cites.
- Exercise 01 completed — the triage row for candidate C1 supplies the header fields (class, SEV, routing) and the initial regulator classification signal.
- Access to the mod-102 harm model (to reconcile against row `HR-CS-INJ-01` or a locally-named equivalent) and the mod-103 register (three-state — inherent / residual / control-defeated — chapter 03 of mod-103); the register-row update patch is a mandatory deliverable, not an afterthought.
- Access to the general SEV post-mortem template your organisation already uses; the AI-specific extensions in chapter 03 are additive to that template, not a replacement.

## Problem statement

The RCA for INC-2026-08-14-cs-rag-inj-01 has converged. The causal chain names the KB corpus refresh at 05:47Z as the delta that ingested a public help-forum thread containing indirect-prompt-injection strings, which the retriever selected, which induced the assistant to execute an injected `send_email` tool call, exfiltrating a customer email address before the guardrail rejected the pattern (ASR 0.19 vs floor 0.05 on suite 2026-08-14). Containment cut over at 09:41Z (the `send_email` tool was removed from the allow-list); the incident window closed at 10:14Z when no further affected traces were observed. The named customer whose email address was exfiltrated has been notified per the organisation's data-subject notification runbook; a refund of unrelated inconvenience has been processed.

The post-mortem now has to survive three readers who each ask a different question of it: the engineering team asking "what do we fix in CI"; the governance council (mod-112) asking "does the risk register still tell the truth and does our appetite still hold"; and the regulator asking "did we detect, report on time, act on correct information, and prevent recurrence." Chapter 03 §"The three audiences and what each needs" pins the framing; your job is to produce the one file whose sections meet all three loads and the three extracts generated from it.

## Requirements

Produce all five deliverables:

1. **The full ten-section post-mortem file** — `post-mortem.md`, in the exact order and with the exact section headings of chapter 03 §"The post-mortem shape":
   1. Header — incident ID, SEV, class, systems affected, date range, report status, signatories, cross-links (harm-model row, register row, red-team fixture placeholder, regulator-package placeholder).
   2. Impact statement — harm phrased in mod-102 language (immediate / secondary / cascading / systemic), named-party count vs population estimate, remediation status, regulatory classification.
   3. Timeline — one line per event in UTC, no analysis interleaved, per chapter 03 §"3. Timeline."
   4. Causal chain — the paragraph from exercise 02 requirement 6, expanded into a section that names the deltas, orders them by explanatory power, cites trace IDs and snapshot hashes, and separates *contributing factors* from *causes*.
   5. Evidence pack — links (or hash-pinned locations) for trace IDs, model manifest before / after, prompt / retriever / decoding config manifest before / after, guardrail-effectiveness report versions, drift dashboard snapshots, supplier / dataset change record, SOC hand-off record.
   6. Risk-model reconciliation — cite the harm-model row (add one if none matched — chapter 03 pins that the addition is a required post-mortem action for unmodelled scenarios), cite the register row, and state the residual-risk movement.
   7. Corrective action plan — the table from requirement 3 below, quoted inline.
   8. Regulator classification and reporting log — which classifications apply (Article 73 sub-category, SR 11-7 materiality, MDR, HIPAA, GDPR Article 33 concurrency); what was reported, to whom, when; what is still due. "Not reportable" is a documented decision with cited criteria, per chapter 03.
   9. Learning propagation — feedback to mod-102 (harm-model row edits), mod-103 (register update, credited-controls change), the red-team / app-eval suites (fixture ID, populated in exercise 04), mod-109 (new monitor / threshold), and mod-112 (council packet reference).
   10. Signatures — incident owner, product IR lead, security IR lead, governance IR lead, system owner, risk-engineer (this role); dated.
2. **The three audience-specific extract files** — as views over `post-mortem.md`, not re-authored:
   - `extracts/engineering-extract.md` — sections 1, 3, 4, 5, 7 (chapter 03 §"Engineering extract").
   - `extracts/governance-council-extract.md` — sections 1, 2, 6, 7 (CAP summary), 8, 9 (chapter 03 §"Governance-council extract").
   - `extracts/regulator-extract.md` — sections 1, 2, 3, 6, 7, 8 (chapter 03 §"Regulator extract"). This is the raw extract; exercise 05 reshapes it into the Article 73 / SR 11-7 / HIPAA / GDPR packages.
   Each extract file opens with a one-line "generated from post-mortem.md rev <hash>" pointer so the extract's provenance is auditable.
3. **The corrective action plan (CAP) table** — a stand-alone `cap.md` with 4-6 items, each row containing: `Action ID`, `Description`, `Owner (role / person)`, `Due date`, `Evidence-of-completion artifact`, `Register-row credit`. Include the four items sketched in chapter 03 §"7. Corrective action plan" and add one or two of your own driven by contributing factors from your causal chain. Every CAP item has an individual person (placeholder name is fine) — a role without a named individual is a CAP the audit reads as unowned. CAP-4 is the fixture back-feed placeholder for exercise 04 to land.
4. **The register-row update patch** — a short markdown file `register-update.md` that shows the mod-103 row (use `HR-CS-INJ-01` or your locally-named equivalent) *before* and *after* the incident with a diff of the changes: likelihood / severity anchoring evidence, credited-controls list (which controls were defeated and are dropped or marked "defeated pending re-evaluation," which new controls come from the CAP), recomputed residual, cadence change if residual moves above appetite, and a signatory block (row author + system owner + incident-response owner, per chapter 03 §"The incident-into-register update").
5. **The anti-pattern self-check** — a short `self-check.md` file with one row per anti-pattern from chapter 03 §"The anti-pattern of a post-mortem only engineering can read" (six rows minimum: no impact statement, timeline is Git commit messages, CAP is vague, regulator section is missing, no register-row link, no sign-offs). For each row, quote the specific section of your post-mortem that demonstrates you avoided the anti-pattern.

## Starter guidance

- **Author one file, then generate the extracts.** The three-audience extracts are views over the single post-mortem; do not author them independently or the audit will read the disagreement (chapter 03 §"Motivation" — the re-authored post-mortem). If you find yourself editing the regulator extract to say something the engineering extract does not, edit the post-mortem, then regenerate.
- **The impact statement uses mod-102 language, not log language.** Chapter 03 §"2. Impact statement" pins this. Say "immediate harm: named-party data disclosure (customer email exfiltrated); secondary harm: erosion of customer trust in the assistant; systemic harm potential: class of injection attack in this scenario has known variants on AIID" — not "the assistant emitted a `send_email` tool call and it went through."
- **Timeline is a fact chain in UTC.** No analysis. If a row of the timeline says "we realised that…" it is analysis; move it to section 4. Chapter 03 §"3. Timeline" pins the fact-chain rule.
- **Contributing factors are not causes.** In this incident, the corpus-refresh path bypassed the review that would have caught the injected content — that is a contributing factor, not a cause. Separate them; over-crediting a contributing factor with the outcome produces a CAP that fixes the wrong thing. Chapter 03 §"4. Causal chain" pins the separation.
- **CAP owners are individuals, not teams.** Chapter 03 §"7. Corrective action plan" is explicit. If you write "the platform team" on a CAP row, the CAP is unowned. Placeholder names are acceptable in the drill; anonymous roles are not.
- **"Not reportable" is a documented decision.** If any of the classifications in section 8 come out as not applicable, write the criteria you checked and the answer. Chapter 03 §"8. Regulator classification and reporting log" pins this — omission reads as evasion.
- **Register update happens on this exercise, not the next one.** The register-row patch is a mandatory deliverable here. Deferring it to exercise 04 (which back-feeds the fixture) breaks the chain: exercise 04 credits the fixture *on the row this exercise updates*.
- **Sign-off dates are load-bearing.** A "signed" post-mortem with no dated sign-offs is not signed. Chapter 03 §"10. Signatures" pins this. Placeholder dates are fine in the drill (`2026-08-25 (drill)`) but they must be present.

## Acceptance criteria

The submission is accepted when:

- The post-mortem file has all ten sections, in order, with headings that match chapter 03 §"The post-mortem shape" verbatim.
- Section 4 quotes the two-sentence causal chain from exercise 02 requirement 6 and expands into a full section that cites at least three trace IDs / snapshot hashes.
- Section 5 lists at least the six evidence categories in chapter 03 §"5. Evidence pack" (traces, model manifest, config manifest, guardrail-effectiveness report, drift snapshot, supplier record, SOC hand-off — six or seven items depending on SOC engagement).
- Section 6 cites a harm-model row (existing or newly-added) and a register row, and states the residual-risk movement in numeric or banded terms consistent with mod-103 chapter 03's scoring scheme.
- The CAP has 4-6 items; every row has an ID, an individual owner (role + name placeholder), a due date, a named evidence-of-completion artefact, and a register-row credit column.
- The three extract files each open with a "generated from post-mortem.md rev <hash>" pointer and contain only sections listed for that audience in chapter 03; nothing is added that is not in the source.
- The register-row update patch shows before / after and includes the signatory block.
- The self-check file addresses every anti-pattern from chapter 03 §"The anti-pattern of a post-mortem only engineering can read" and quotes the specific section of the post-mortem that avoids it.
- A reader can open the post-mortem, then the governance-council extract, then the regulator extract, and confirm that all three agree on the timeline, the causal chain, the impact count, and the CAP set. No fact drifts between them.

## Stretch goals

- **Add a generation script.** Write a short script (`extracts/generate.py` or equivalent) that reads `post-mortem.md`, splits by H2 heading, and emits the three extract files by section-number selection. Running the script in CI turns "extract drift" from a manual worry into a CI failure — the discipline chapter 03 §"Audience-specific extracts" implies.
- **Round-trip the extracts.** Hand `engineering-extract.md` to a peer engineer, `governance-council-extract.md` to a peer familiar with the risk council packet (see mod-112), and `regulator-extract.md` to a peer familiar with the regulator formats (or use your own second pass a day later). Collect what each reader says is missing; the disagreement is the effective challenge chapter 03 §"The three audiences and what each needs" implies.
- **Author the incident-into-register update in the real register tool.** If your organisation uses a GRC platform (Credo AI, Holistic AI, ModelOp, Monitaur) or a home-rolled register, land the update patch there and attach the tool's audit-log record to the register-update file. This turns the drill's markdown patch into a live register edit.
- **Draft the mod-112 council-packet entry.** The governance-council extract feeds a rolled-up entry in the next risk-council packet (mod-112). Draft the one-page entry: incident summary, register-row movement, appetite comparison, CAP status, cross-system exposure. Filing this alongside the extract closes the loop chapter 03 §"9. Learning propagation" implies.

## Deliverable format

A single index markdown document `mod-111-ex03-post-mortem.md` that contains, in order:

1. *Summary* — one paragraph naming the incident, the running system, and the state of the post-mortem (draft / under-review / signed-off (drill)).
2. *File layout* — the folder / file list for `post-mortem.md`, `cap.md`, `register-update.md`, `self-check.md`, and the three extract files.
3. *Post-mortem excerpts* — the header, the impact statement, the causal chain paragraph, and section 6 (risk-model reconciliation) quoted inline so a reader of the index can grasp the shape without opening the full file.
4. *CAP table* — quoted inline.
5. *Register-row before / after* — quoted inline.
6. *Anti-pattern self-check* — quoted inline.
7. *Extract pointers* — one line per extract with its intended audience and its "generated from" pointer.
8. *Cross-references* — pointers forward to exercise 04 (fixture back-feed lands CAP-4 and the fixture-ID credit on the register row) and exercise 05 (regulator packages reshape `regulator-extract.md` into the Article 73 and SR 11-7 formats).
9. *Open items* — anything blocked on a real signing routine, a real GRC-tool update, or a peer review.

## Submission

Attach the index markdown plus the full folder (`post-mortem.md`, `cap.md`, `register-update.md`, `self-check.md`, and `extracts/`). If the running system is under NDA, redact the named-party count and the customer identifier but preserve the shape — the audit reads the shape, not the identifiers. The post-mortem file is the source of truth for exercises 04 and 05, so treat it as versioned and controlled from this exercise forward.
