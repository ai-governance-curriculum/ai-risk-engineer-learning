# Exercise 02 — SR 23-4 Third-Party Model / API Due Diligence

**Estimated effort:** 2 hours

## Objective

Run one **third-party model or API vendor** through the SR 23-4 lifecycle end-to-end — planning, due diligence, contract, ongoing monitoring, termination — and produce the vendor file that plugs into the *Third-party components* heading of the MRM packet you assembled in exercise 01. The output is the file a US financial-institution examiner would open when they ask "how do you govern this vendor's model, not just its uptime."

## Prerequisites

- Chapter 02 read in full. The five lifecycle stages, the AI-specific extension of the due-diligence questionnaire, the contract-clause table, and the "monitor on your own signal, not the vendor's status page" rule are the reference shapes.
- Chapter 01 read. The vendor file lives inside the MRM packet, under the *Third-party components* heading — the exercise-01 packet reserved a slot for it.
- The primary running system carried since mod-101/mod-102 and its **most critical third-party AI dependency** — a foundation-model API, a hosted embedding model, an off-the-shelf classifier, a labelled-data vendor, an OCR provider, or similar. Pick the vendor whose failure or silent change would move the largest mod-103 register row.
- The mod-102 harm-model rows and mod-103 register rows the vendor participates in. The planning-stage tiering decision cites both.
- Access to whatever public materials the vendor already provides — Model Card / system card, safety card, terms of service, data-processing addendum, transparency report, sub-processor list. Public materials are enough for this exercise; do not attempt to open a new contract negotiation with the vendor for the purpose of a training drill.
- A pointer to the peer L25 or L30 who owns the runtime integration with the vendor (typically `ai-infra-mlops` or `ai-eval-engineer`). The ongoing-monitoring stage cites them.

## Problem statement

Author the vendor file for one third-party AI dependency of the primary running system. The file has five sections, one per SR 23-4 lifecycle stage, and it plugs into the exercise-01 MRM packet at `11-third-party-components/<vendor-id>/`.

### The vendor file layout

```
mrm-packet/<system-id>/<packet-version>/11-third-party-components/<vendor-id>/
  README.md                        # tier decision + coverage summary
  01-planning.md                   # harm-model rows + register rows the vendor participates in + criticality tier
  02-due-diligence-questionnaire.md   # the AI-extended questionnaire filled from public materials
  03-contract-clause-table.md      # attempted vs won vs walked-away
  04-ongoing-monitoring.md         # the mod-109 monitor spec for vendor-side signal
  05-termination-plan.md           # fallback design + runway + data-retrieval plan
  disclosure-gaps.md               # every question the vendor did not answer + compensating control
  register-cross-refs.md           # which mod-103 rows this vendor moves and how much
```

### The tier decision

The `README.md` opens with the tier decision. The three permitted tiers, adapted from SR 23-4's risk-based lens:

| Tier | Definition | Required due-diligence depth |
|---|---|---|
| Critical | Vendor's component participates in a register row whose residual sits at or near appetite; failure or silent change would put the system outside appetite | Full questionnaire, contract clause table, own-signal monitor, termination-runway drill |
| Elevated | Vendor's component participates in a material register row but residual has slack; failure would be noticed but not immediately outside appetite | Questionnaire, contract clause table, own-signal monitor |
| Baseline | Vendor's component is used only internally, not tied to any register row above the appetite band | Standard SaaS due diligence plus a short AI-extension checklist |

The tier decision cites the specific register rows that drive it. A vendor whose tier is not justified by register rows is under-tiered by construction.

### The AI-extended questionnaire

Chapter 02 §"Due diligence and third-party selection" defines the question areas: model identity and version, change notification, training-data provenance, safety-classifier / guardrail posture, evaluation and red-team disclosure, adversarial-robustness posture, fairness / bias posture, privacy, incident disclosure, right-to-audit, sub-processors, termination / model portability. Every area has at least one concrete question the vendor was (or was not) asked. Every unanswered question is a *disclosure gap* and lands in `disclosure-gaps.md`.

## Requirements

Produce all seven items:

1. **The vendor file folder** — every file in the layout above exists with real content. Files whose content depends on private contractual detail may be redacted, but the *shape* must remain and the redaction must be labelled.
2. **The tier decision** in `README.md` — cites the specific mod-103 register rows that drive the tier and names the peer L25 / L30 who agreed with the tier.
3. **The AI-extended questionnaire** in `02-due-diligence-questionnaire.md` — every question area from chapter 02 has at least one concrete question with either the vendor's answer (from public materials) or a `disclosure-gap` reference.
4. **The contract clause table** in `03-contract-clause-table.md` — every clause area from chapter 02's contract table (version pinning, change-notification SLA, safety-metric SLA, right-to-audit, data use, incident notification, sub-processor disclosure, termination assistance, liability and indemnity) has one row with status: `won` / `walked-away` / `deferred` / `unknown`. `unknown` is only acceptable when the actual contract is not accessible to you; each `unknown` row cites who would know.
5. **The own-signal monitor spec** in `04-ongoing-monitoring.md` — at least four monitor rows in the mod-109 signal-contract shape covering: behaviour-drift on a fixed prompt / input set, safety-metric measured on a rolling production sample, cost / usage anomaly detection, and pinned red-team fixture regression. If any of the four cannot be built with current tooling, mark it `blocked_on: <owner>` rather than omitting.
6. **The termination plan** in `05-termination-plan.md` — names the fallback model (a specific alternative vendor / self-hosted option), the runway (canary percentage, wrapper mechanism, config-swap approach), the data-retrieval plan, and the downstream-consumer notification plan.
7. **The register cross-reference** in `register-cross-refs.md` — one table row per mod-103 register row this vendor participates in, with the vendor-credited controls listed. When a control is contractually not enforceable or was walked-away in the clause table, the row's control-defeated column names the failure mode explicitly.

## Starter guidance

- **Tier from the register, not from the invoice.** The temptation is to tier by contract value; SR 23-4 tiers by *criticality of the relationship to the institution's risk profile*. A cheap embedding API that sits underneath every customer interaction is critical; an expensive OCR vendor used only in a back-office reconciliation may be baseline.
- **The questionnaire is the artefact, not the interview.** Even if you cannot get the vendor on a call, the questionnaire is worth completing from public materials — Model Card, system card, safety card, terms of service, data-processing addendum, transparency report. What the vendor *chose not to publish* is often more informative than what they did.
- **Silence is a disclosure gap.** If the vendor's Model Card does not mention prompt-injection resistance, that is a gap — file it. It is not a pass because the vendor did not say it was failing.
- **Contract clauses you walked away from become compensating-control rows.** A vendor who refuses to sign a safety-metric SLA is not automatically a rejected vendor — but the *compensating control* (your own safety monitor on their output) has to be real, budgeted, and reflected in the register.
- **Own-signal monitoring is what closes the vendor loop.** The vendor's own status page is not sufficient; chapter 02 is emphatic on this point. If your monitor spec has no rows that measure the vendor's *behaviour* on *your* signal, it is a trust-me plan and the tier decision should be reconsidered.
- **Termination readiness is the SR 23-4 heading that is most often skipped and most likely to bite you.** The exit path is not "we would evaluate alternatives at that time." It is a named alternative, a canary mechanism, and a data-retrieval process — with rehearsal cadence.
- **This is a training drill, not a legal negotiation.** Do not open a new commercial conversation with the vendor for the purpose of this exercise. Use public materials. Mark private contractual detail as `unknown / private` rather than inventing it.
- **The vendor file lives inside the MRM packet.** Do not produce it as a stand-alone artefact. Its path is `mrm-packet/<system-id>/<packet-version>/11-third-party-components/<vendor-id>/`. Update the exercise-01 packet's `11-third-party-components/README.md` to point at it.

## Acceptance criteria

The submission is accepted when:

- The vendor file folder exists at the required path inside the exercise-01 packet.
- The tier decision cites specific mod-103 register rows and is agreed to by a named peer.
- The AI-extended questionnaire covers every question area from chapter 02 with concrete questions and either answers or disclosure-gap references.
- The contract clause table has one row per clause area with a `won` / `walked-away` / `deferred` / `unknown` status; `unknown` cites who would know.
- The own-signal monitor spec has at least four rows covering drift, safety, cost anomaly, and red-team fixture regression, in the mod-109 signal-contract shape.
- The termination plan names a specific fallback, a specific runway mechanism, a data-retrieval plan, and a downstream-consumer notification plan.
- The register cross-reference maps every register row the vendor participates in to the vendor-credited controls, with the control-defeated column populated for walked-away clauses.
- The exercise-01 MRM packet's `11-third-party-components/README.md` is updated to point at this vendor file.

## Stretch goals

- **Second-vendor pass** — pick a second vendor of a *different* tier (elevated or baseline) and produce a compressed version of the file. Comparing two vendor files side-by-side reveals whether your tier definitions actually shape the artefact set.
- **Public-materials trace** — for each vendor answer sourced from a public document, cite the document URL and the retrieval date. Vendor materials change; the trace lets a future re-review know what was current at authoring time and whether it has since drifted.
- **Silent-change detection drill** — set up the behaviour-drift monitor from the own-signal monitor spec and record 7 days of daily runs against a fixed prompt set. Even if the vendor did not change, the exercise is proof that the monitor works and that the drift-comparison baseline is captured.
- **Sub-processor cascade** — the vendor's sub-processor list is itself a set of third parties. For any sub-processor that materially handles data or compute, note whether it has its own tier decision on your side; sub-processor tiering is under-practiced.
- **Waiver-and-escalation drill** — for one walked-away clause, draft the risk-council escalation memo per chapter 02: clause requested, vendor's refusal, compensating control offered, residual-risk elevation, decision requested (tolerate vs seek alternative vendor vs remove vendor from scope).

## Deliverable format

A single markdown document `mod-110-ex02-vendor-file.md` that indexes the vendor file. The index contains:

1. *Vendor and tier summary* — one paragraph naming the vendor, the vendor's component's role in the running system, the tier decision, and the register rows that drive it.
2. *Questionnaire coverage* — a table of question areas × answered / gap / not-applicable counts, with pointers to the full questionnaire.
3. *Contract clause status table* — the clause table quoted verbatim from `03-contract-clause-table.md`.
4. *Own-signal monitor summary* — the four (or more) monitor rows summarised in the mod-109 signal-contract shape.
5. *Termination plan summary* — one paragraph naming the fallback, the runway, and the data-retrieval plan.
6. *Register cross-reference summary* — one table row per register row the vendor moves, with the delta the vendor's presence creates.
7. *Disclosure-gap list* — the list of every unanswered question and its compensating control (if any).
8. *Open items* — anything blocked on the vendor, on a peer, or on a platform capability that does not yet exist.

## Submission

Attach the index markdown plus the vendor file folder. If the vendor is under NDA or the contract is not publicly disclosable, redact identifiers and specific clause language — but preserve the questionnaire structure, the clause-table statuses, the monitor spec, and the termination plan. A reviewer needs to see the *shape* of the file even when the private detail is not visible. Update the exercise-01 packet to reference this vendor file in its third-party components section.
