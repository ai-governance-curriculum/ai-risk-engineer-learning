# Exercise 05 — Risk Register Population in a GRC-for-AI Platform (or the Spreadsheet Fallback)

**Estimated effort:** 2 hours

## Objective

Stand up your register as a **live operating artifact** — populated with the rows from exercises 01-04, wired to a cadence, ownership assignments recorded, sign-off routed, evidence pointers resolvable, hygiene checks running, and portfolio reports exportable — in either a GRC-for-AI platform (Credo AI, Holistic AI, ModelOp, Monitaur, or your organisation's chosen platform) or the spreadsheet-shaped fallback from chapter 06. The output is the register in the shape it will be operated in from this exercise forward.

## Prerequisites

- Chapter 06 read; chapter 07 skimmed (evidence-pointer provenance is chapter 07's shape).
- Exercises 01-04 completed — three-state rows, versioned scoring scheme, 27005-aligned schema, and portfolio reports carry through.
- Access to *one* of:
  - A GRC-for-AI platform trial or your organisation's live tenant (Credo AI, Holistic AI, ModelOp, Monitaur are the common examples cited in chapter 06). Choose whichever your organisation uses or is evaluating.
  - The spreadsheet-shaped fallback: a git repo you can commit YAML / CSV / markdown to, and a CI system that can run a small reduction script.
- If neither is available, the spreadsheet fallback is always available — see chapter 06's file layout.

## Problem statement

Stand up the register in your chosen platform (or fallback) so that:

1. Every row from exercises 01-04 is populated with the fields chapter 03 requires plus the operational fields chapter 06 requires (ownership, cadence, sign-off, evidence-manifest).
2. Ownership is recorded on every row: `row_author_role` + person, `system_owner_role` + person, `control_owner_roles` + persons per credited control, `mitigation_owner_role` + person. On the exercise's provisional systems, name plausible role owners even if the person is a placeholder — but *say* the person is a placeholder.
3. Cadence is set per row per the tiering in chapter 06 (annual / quarterly / monthly during treatment / ad-hoc plus scheduled), and `next_review_due` is populated.
4. Event-triggered cadence hooks are documented: at minimum, name the wiring for a *model change*, a *control change*, and an *incident* triggering re-review — even if the wiring is manual (a runbook entry) rather than automated.
5. Sign-off routing is populated per chapter 06's table (row create / residual state change / above appetite / control-defeated exceeds appetite / framework version change / incident-driven change).
6. Every credited control has an evidence pointer that *resolves* — a URL, a file path, or a GRC-platform artifact reference. Broken pointers count as no evidence and the credit lapses.
7. Register hygiene checks (chapter 06's list) run in CI or as an equivalent platform validation, and the exercise deliverable includes the check configuration and a passing run.
8. Portfolio reports from exercise 04 are exported *from* the register — either via the platform's report builder or via the reduction script — so aggregation is a stateless read of the live register, not a hand-authored snapshot.

## Requirements

- Output as `mod-103-ex05-register-operation.md` plus artifacts:
  - On a GRC platform: screenshots (redacted as needed), platform-object IDs, export files (JSON / CSV) showing the row schema and one full row.
  - On the spreadsheet fallback: the `risk-register/` directory tree from chapter 06 populated to real content, the CI configuration, and one run of the hygiene checks passing (and one run failing on a deliberately-introduced defect, to prove the check works).
- Every row carries every chapter 03 field and every chapter 06 operational field. Where a field cannot be populated (e.g., control-owner person on a provisional system), a placeholder is used *and* an inline note names what it is a placeholder for.
- Cadence per row is chosen per the chapter 06 tiering table, and `next_review_due` is a real date.
- At least three event-triggered cadence hooks are documented — model change, control change, incident — with the wiring (runbook step, CI trigger, platform-workflow rule) named.
- Sign-off routing shows at least one *executed* sign-off in the sign-off history (even a self-sign for the exercise), with a version, date, and signatories list.
- Every credited control's evidence pointer resolves. Include the resolution demonstration — screenshot, `curl` output, `ls -la` of the file, etc.
- Register hygiene checks run and are demonstrated in both success and failure modes.
- Portfolio reports are regenerated from the register with a single command / a single platform action, and the output matches the exercise-04 reports (or shows a diff explained by rows added since).
- If you used a GRC platform, name the specific *choices* the platform forced on you (row schema mapped to platform fields; sign-off routed through platform workflow; evidence stored inside the platform vs. linked externally; version-of-rubric handling) and note where the chapter 06 anti-patterns were forced by the platform.

## Starter guidance

- Choose the platform / fallback deliberately. If your organisation has a platform commitment, use it — the exercise doubles as a productive population run. If not, the spreadsheet fallback is *not* a downgrade; it is the first-order form of the register and forces the discipline the platform will otherwise hide.
- Populate one row *fully* before you scale to the rest. The first row will surface every schema field you forgot; iterating on 20 rows in parallel spreads the pain out.
- Ownership placeholders on provisional systems: use role names for people ("`<system-owner-placeholder>`" is fine) but never leave a role field blank. Blank roles are the anti-pattern; placeholder people are honest.
- Evidence pointers: prefer pointers that *cannot rot silently* — a hash-pinned release config, a monitor dashboard URL, a test-run identifier. Confluence pages rot; test outputs and release configs do not (as long as your CI retains them).
- Hygiene checks: write the failing case first. It is the only way to know the check actually fails when the register is bad. A hygiene check that only ever passes is a check that has never been exercised.
- Sign-off history: even if the exercise is a solo run, execute the sign-off ritual — record the date, the signatory (yourself, with role), and the sign-off scope. The auditor's version of this will be someone else, but the shape of the ritual is the same.
- If your platform silos the register from the security-side 27005 register (chapter 03's anti-pattern), *note it* on the deliverable. The exercise is passable in that state but the finding is a real one and worth surfacing to whoever selects the platform.

## Acceptance criteria

The submission is accepted when:

- Every exercise 01-04 row is populated in the register with full chapter 03 + chapter 06 fields.
- Ownership is complete on every row (roles are named; persons are named or placeholder-marked).
- Cadence is set per row per the chapter 06 tiering; `next_review_due` is real.
- Event-triggered hooks (≥ 3) are documented with named wiring.
- Sign-off routing is defined and at least one sign-off has been executed with a version, date, and signatories.
- Every credited control has an evidence pointer and the pointer resolves.
- Register hygiene checks run in CI or platform validation, and both a passing and a failing run are demonstrated.
- Portfolio reports regenerate from the live register with a single command / platform action.
- If a GRC platform was used, platform-forced choices and any surfaced anti-patterns are documented.
- A reviewer given only the register (and its README / platform-tour) can, in under 20 minutes, understand who owns what, what refresh cadence applies, and where the aggregation reports come from.

## Stretch goals

- Wire an *event-triggered* re-review into a real trigger — e.g., a CI hook on the model-serving repo that opens an issue against the register when a model version bumps. Demonstrate the trigger firing on a synthetic model bump.
- Prototype the *staleness* alert: a scheduled task that emails / issues / posts on the register when any `next_review_due` slips past. Show a slip.
- Draft the *platform-migration* note — if you had to move from your chosen platform / fallback to a different one next quarter, what would need to be exported, transformed, and re-imported? This is the argument that the register lives in the *data*, not in the tool.
- Sketch the *evidence-manifest* verification job — a scheduled task that walks every credited control's evidence pointer and reports the resolvable / broken split. This turns evidence-freshness into a first-order metric.
- If your platform supports it, wire mod-109 monitor data into the register as a live control-status feed (green / amber / red per credited control). Chapter 06 named this as the mod-109 seam; this stretch closes the seam.

## Deliverable format

A single markdown document with the following sections, plus the register content itself:

1. *Chosen operating shape* — platform + version, or spreadsheet-fallback with repo layout; the reason for the choice.
2. *Row-population summary* — count of rows populated; any rows that could not be populated and why.
3. *Ownership completeness* — a small table listing every row × ownership-role, marking placeholders explicitly.
4. *Cadence and event-triggered hooks* — the tiering used and the wiring documented for each event trigger.
5. *Sign-off* — the routing table and the sign-off history entry executed for this exercise.
6. *Evidence-pointer demonstration* — one row walked end-to-end, showing each credited control's pointer resolving.
7. *Hygiene checks in CI* — the configuration and one passing + one failing run.
8. *Portfolio-report regeneration* — the one-line command / platform action, and a screenshot / dump of one regenerated report.
9. *Platform-forced choices and anti-patterns surfaced* — honest notes if using a GRC platform.
10. *Stretch goals attempted* — the notes.

## Submission

Attach the markdown, the register-content archive (repo tarball / platform export), and any screenshots to the module submission channel. If the register is under organisational NDA, redact identifiers but preserve the fields, cadence dates, ownership roles, sign-off structure, and hygiene-check output — a reviewer cannot review an operational register whose operating fields are blank.
