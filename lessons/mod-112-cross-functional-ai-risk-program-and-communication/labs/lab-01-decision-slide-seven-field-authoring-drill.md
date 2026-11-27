# Lab 01 — Council Decision-Slide Seven-Field Authoring Drill (One Decision per Slide + Seven Fields in Order + Rollback as a Required Cell)

**Duration:** 45 minutes · **Format:** guided practice, markdown authoring on a fixed fabricated LLM application

## Objective

Internalise the chapter-02 **decision-slide seven-field shape** — (i) decision requested; (ii) scope; (iii) evidence pointers; (iv) options and recommendation; (v) residual-risk claim; (vi) escalation dependency; (vii) rollback / reversibility — by authoring two complete decision slides on a fixed fabricated running system (`HelpDesk`), each shaped to force a different hard-to-defend judgement call. The lab drills the atomic slide shape chapter 02 pins so that when **exercise 02** assembles a full council packet (three decision slides, one-pager, decision-log entries, minutes) on your real primary running system, the engineering budget goes into picking the decisions that belong at council scope, not into relitigating what each field means or whether "we will revisit" counts as a rollback.

The chapter-02 anti-patterns the shape forecloses: the **read-out session slide** (a context slide pretending to ask for a decision; "the council should be aware that..."); the **ambush decision** (decision requested with no evidence pack); the **mixed-content slide** (a decision slide that also carries three signal paragraphs; nods-and-moves-on territory); and the **missing-rollback decision** (approved commitment with no reversibility clause; argued in real time when a monitor fires six months later).

## Prerequisites

- Chapter 02 read — specifically "The pre-read deck — 'one decision per slide'" (the three-slide-type structure — context, decision, signal — and the load-bearing *one decision per slide* rule); "The decision-slide template" (the seven fields and the discipline that "if any of the seven fields is empty, the slide is not ready for the council"); "A worked decision slide (short form)" (the shape at the reading level); and "Common failure modes and their fixes" (the seven anti-patterns and their mechanical fixes).
- Chapter 01 skimmed at "The decisions the programme owns" — the decision matrix, specifically the rows that route through the risk council (above-appetite acceptance, T3+ tier promotion, mute of tripwire monitors, external card publication).
- Chapter 04 skimmed at "2. Rollback criteria" — rollback is *mechanical* (a config flip), not a human decision at incident time; the inverse-of-threshold-with-tolerance pattern.
- A terminal where you can create a working directory and author a markdown file.
- No external access or compute required; the system brief, the register row stubs, the evidence-pointer IDs, and the two decision scenarios are fixed below.

## The fixed example system

Use this system throughout the lab — do *not* substitute your own. Carrying a fixed scenario keeps the lab bounded to the slide-authoring skill. The system matches the `HelpDesk` carried through the mod-111 labs; the chapter-01 and chapter-02 running examples (customer-support RAG assistant) are built on the same shape.

> **`HelpDesk`** — a customer-support RAG assistant for a mid-size B2B SaaS company. Serves English-language customers of a billing-and-provisioning product. Assembles context from a knowledge-base store (product docs + internal runbooks + a mirrored public help-forum corpus) and from the authenticated caller's own recent ticket history. Backed by an in-house fine-tune (`helpdesk-finetune` v1.4.0) of the vendor-hosted base `FoundationCo-LLM-3.5` v3.5.2 behind a FastAPI service wrapping a LangGraph agent. Two production tool calls: `escalate_ticket(destination, summary)` and `lookup_account(account_id)`.
>
> **Classification under the EU AI Act:** high-risk (Annex III use case affecting a provider's customers; account data in scope). Deployed to EU and US customers. The sponsoring organisation is a US bank subsidiary — SR 11-7 is the parent MRM shape; HIPAA applies on the narrow subset of health-insurance-adjacent customer-service journeys; GDPR Article 33 runs concurrently for EU-resident data subjects.
>
> **Current deployment tier:** T2 (limited external, human-in-the-loop). The quarterly council on 2026-09-24 will consider promotion to T3 (general external availability, human-in-the-loop).
>
> **Protected-attribute slice in scope:** tenant tier (`standard` vs. `enterprise`), used as a per-slice proxy for coverage / quality parity monitoring on `HR-CS-FAIR-08`.

### Fixed evidence-pointer IDs (do not add more)

Every slide's `Evidence pointers` field cites IDs from this set. The IDs follow the mod-103 ch 06 version-hash-pinned shape and the chapter-05 artefact-ID convention.

| Artefact ID | Scope line | Current version |
|---|---|---|
| `HR-CS-INJ-01` | Register row — prompt-injection control credit (defence in depth across input filter, retrieval filter, tool-invocation policy) | v2026.09 |
| `HR-CS-PII-04` | Register row — PII egress control credit (Presidio at ingest/retrieval/output and per-caller entitlement); absolute-rule row | v2026.09 |
| `HR-CS-FAIR-08` | Register row — per-slice quality parity on `tenant_tier` slice (standard vs enterprise); fairness-tagged | v2026.09 |
| `HR-CS-DRIFT-02` | Register row — covariate drift on top-20 input features and on prompt-embedding distribution | v2026.09 |
| `HR-CS-VEND-11` | Register row — vendor silent-swap risk on `FoundationCo-LLM-3.5` base model | v2026.09 |
| `ART-HD-RED-Q3-01` | Red-team run report (mod-104) — Q3 external partner engagement; 3 findings opened, 3 closed | v2.1, sha256:4e3a... |
| `ART-HD-GRD-Q3-01` | Guardrail effectiveness report (mod-108) — Q3 retention corpus (n=2,000); G-INJ-01 F1 0.93, G-PII-01 F1 0.96, G-TOXIC-01 F1 0.91 | v3.0, sha256:71bc... |
| `ART-HD-FAIR-Q3-01` | Fairness-slice report (mod-106) — Q3 `tenant_tier` per-slice measurement; standard-tier parity delta 3.1 pts on quality composite | v2.0, sha256:8acd... |
| `ART-HD-PRIV-Q3-01` | Privacy-risk report (mod-107) — Q3 canary set (n=5,000); extraction rate 0.03 percent | v2.0, sha256:a21e... |
| `ART-HD-MON-2026` | Monitor spec + Q3 digest (mod-109) — critical monitors green; `SC-HD-INJ-01` injection-defeat 30-day rolling 0.18 percent | v1.3, sha256:3cfd... |
| `ART-HD-INC-2026` | Incident log + post-mortems (mod-111) — no open severity ≥ 2; regression fixtures green | rolling |
| `ART-HD-CARD-2026-09` | Model / system / dataset cards (this module ch 03) — v3.0 published 2026-09-15 | v3.0, sha256:5f08... |
| `SC-HD-INJ-01` | Monitor ID (mod-109) — injection-defeat rate, 30-day rolling; threshold green < 0.3 percent, tripwire 0.3-0.5 percent, rollback > 0.5 percent for 3 consecutive days | — |
| `SC-HD-FAIR-01` | Monitor ID (mod-109) — `tenant_tier` per-slice parity, 7-day rolling; +/-5-point band | — |
| `SC-HD-PII-01` | Monitor ID (mod-109) — PII egress detector firing rate, per-call | — |

### The two decisions (do not substitute)

Use exactly these two. Each is deliberately shaped to force one judgement — decision D-01 forces the "monitored ramp with mechanically enforceable rollback" shape (B is the correct recommendation, not A), and decision D-02 forces the "above-appetite acceptance expires on a date, not on 'we will revisit'" shape plus the forced-decision-slide rule chapter 02 §"Common failure modes" names.

> **Decision D-01 — T2 → T3 promotion (`HelpDesk`).** The 2026-09-24 council is being asked to approve promotion of `HelpDesk` from T2 (limited external, human-in-the-loop) to T3 (general external availability, human-in-the-loop) for the English-language EU + US customer cohort, excluding the (separate) agentic ticket-triage variant. Current state: all register rows at or below appetite (mod-103 Q3 aggregation); Q3 red-team run closed all high-severity findings (`ART-HD-RED-Q3-01`); guardrail effectiveness above Q3 thresholds (`ART-HD-GRD-Q3-01`); monitor coverage green (`ART-HD-MON-2026`); no open severity ≥ 2 incidents (`ART-HD-INC-2026`); published card at v3.0 (`ART-HD-CARD-2026-09`). The three options the risk engineer wants on the slide: **A** — approve as scoped (straight promotion); **B** — approve subject to a 60-day monitored ramp with revert at 5 percent traffic; **C** — defer to 2026-Q4 pending external audit conclusion. The engineer's recommendation is **B**: a monitored ramp with pre-registered rollback triggers is the shape chapter 04 §"A worked short-form tier promotion" models and the shape the Q3 evidence supports without overclaiming at straight T3. The council-ownership check: chapter 01's decision matrix has "Approve promotion into T3 / T4 (external, high-stakes)" as risk-council-owned with release-assurance sign-off (`ai-evaluation-engineer` L35).
>
> **Decision D-02 — above-appetite acceptance on `tenant_tier` parity, 90-day window.** On 2026-09-10 the fairness-slice monitor `SC-HD-FAIR-01` flagged a 3.1-point widening of the `standard`-vs-`enterprise` per-slice parity delta on the composite quality metric; the register row `HR-CS-FAIR-08` was re-scored and is now **above** the programme's appetite band for the 90-day period while the mod-106 team runs a prompt-template refresh and re-balance intake. Monitoring stays green on the other critical monitors (`SC-HD-INJ-01`, `SC-HD-PII-01`). The council is being asked to **approve above-appetite acceptance for 90 days** (2026-09-24 → 2026-12-23) subject to: (i) weekly fairness-slice report during the window, (ii) auto-revocation of the exception on any further widening beyond 5 points, (iii) legal partner's reservation on EU AI Act Article 10 data-governance exposure recorded in the decision-log entry. Chapter 01's decision matrix routes this as "Accept residual *above* appetite for a fixed window" — system owner + risk council co-signed; chapter 02 §"Common failure modes" forces this onto the agenda via the **silent-above-appetite** rule ("make above-appetite acceptance a *forced* decision slide when appetite is breached; no drift by silence").

## Steps

### Step 1 — Create your working directory and the slide file

```bash
mkdir -p mod112-lab01
cd mod112-lab01
touch pre-read-decision-slides.md
```

Open `pre-read-decision-slides.md` and paste this scaffold. You will fill the two slides in order.

```markdown
# HelpDesk — pre-read decision slides (lab 01)

Programme: HelpDesk (customer-support RAG assistant; see lab brief)
Council date: 2026-09-24 (2026-Q3)
Pre-read version: v1, date: 2026-09-17, hash: sha256:<placeholder>
Chapter references: mod-112 ch 02 (decision-slide seven fields; one-decision-per-slide; common failure modes); ch 01 (decisions the programme owns; sign-off routing); ch 04 (rollback as mechanical config flip)
Running-system register version bound to: mod-103 register v2026.09
Running-system harm model: mod-102/harm-model/intake-INT-HD-001.md

## Decision slide D-01 — T2 → T3 promotion (HelpDesk)

(seven fields in order: Decision requested | Scope | Evidence pointers | Options and recommendation | Residual-risk claim | Escalation dependency | Rollback / reversibility)

## Decision slide D-02 — above-appetite acceptance on `tenant_tier` parity (90-day window)

(seven fields in order: Decision requested | Scope | Evidence pointers | Options and recommendation | Residual-risk claim | Escalation dependency | Rollback / reversibility)

## Readiness checklist

(one line per slide confirming all seven fields are populated; any slide with an empty field is pulled per chapter 02 §"The decision-slide template")
```

### Step 2 — Fill Slide D-01 (T2 → T3 promotion)

Chapter 02 §"The decision-slide template" pins the seven fields and the order they appear in. Chapter 02 §"A worked decision slide (short form)" shows the reading-level shape on exactly this scenario (customer-support RAG assistant T2 → T3 with monitored ramp as the recommendation). Chapter 04 §"A worked short-form tier promotion — T2 to T3" pins the threshold-citation and rollback-trigger shape the `Evidence pointers` and `Rollback / reversibility` fields cite into.

Fill the slide (write it out in full so the shape is on the page, not in your head):

```markdown
## Decision slide D-01 — T2 → T3 promotion (HelpDesk)

**Decision requested:** Approve promotion of `HelpDesk` from T2 (limited external, human-in-the-loop) to T3 (general external availability, human-in-the-loop).

**Scope:** `HelpDesk` base variant, English-language cohort, EU + US customers, six-month promotion window (2026-09-24 → 2027-03-24). Excludes the agentic ticket-triage variant (handled in a separate T1 → T2 slide on the same council). No change to the HIPAA-adjacent flow carve-out (remains at T2 pending the mod-107 sub-flow re-evaluation on the 2026-Q4 council).

**Evidence pointers:**
- Register: rows `HR-CS-INJ-01`, `HR-CS-PII-04`, `HR-CS-FAIR-08`, `HR-CS-DRIFT-02`, `HR-CS-VEND-11` — all at or below appetite at the mod-103 Q3 aggregation (see `ART-HD-REG-2026-Q3` aggregate; linked in the pack).
- Red-team: `ART-HD-RED-Q3-01` v2.1 sha256:4e3a... — Q3 external partner engagement; 3 findings opened and closed; regression suite green.
- Guardrails: `ART-HD-GRD-Q3-01` v3.0 sha256:71bc... — retention corpus n=2,000; G-INJ-01 F1 0.93, G-PII-01 F1 0.96, G-TOXIC-01 F1 0.91 (all ≥ the chapter-04 §"A worked short-form tier promotion" T3 thresholds of 0.92 / 0.95 / 0.90).
- Fairness: `ART-HD-FAIR-Q3-01` v2.0 sha256:8acd... — `tenant_tier` per-slice parity delta 3.1 pts (within the chapter-04 T3 threshold of ≤ 4 pts).
- Privacy: `ART-HD-PRIV-Q3-01` v2.0 sha256:a21e... — canary extraction rate 0.03 percent (within the T3 threshold of ≤ 0.05 percent).
- Monitoring: `ART-HD-MON-2026` v1.3 sha256:3cfd... — `SC-HD-INJ-01` 30-day rolling 0.18 percent (within the T3 green threshold < 0.3 percent); no muted tripwires.
- Incidents: `ART-HD-INC-2026` — no open severity ≥ 2; post-mortem regression fixtures green.
- Cards: `ART-HD-CARD-2026-09` v3.0 sha256:5f08... — model / system / dataset cards current and published; residual-risk claim and invalidation triggers aligned to the T3 shape.

**Options and recommendation:**
- **A** — Approve as scoped (straight promotion to T3).
- **B** — Approve subject to a 60-day monitored ramp with revert at 5 percent traffic, pre-registered rollback triggers wired, and the Q4 quarterly council as the ramp-conclusion review.
- **C** — Defer to the 2026-Q4 council pending the ISO/IEC 42001 external audit conclusion.
- **Recommendation: B.** The Q3 evidence supports promotion but does not support straight T3 — `SC-HD-INJ-01` has been stable at 0.18 percent for the 30-day rolling window but has not been stressed against general-availability traffic volume. A 60-day monitored ramp with mechanically enforceable rollback retains the promotion but attaches the testable ramp chapter 04 §"2. Rollback criteria" names as the mechanical-enforcement shape. The ramp expires naturally into T3 at 2026-11-24 unless a rollback trigger fires or the Q4 council defers the final promotion step.

**Residual-risk claim:** After the ramp-wired safeguards, we assess residual as **Medium-low** on the English-language EU + US customer cohort for the 90-day ramp window. The claim holds subject to `SC-HD-INJ-01` remaining below 0.5 percent for any 3-consecutive-day rolling window, `SC-HD-FAIR-01` per-slice delta not widening beyond 5 points, and `SC-HD-PII-01` firing rate below the pre-registered threshold. Linked to the published card's residual section (`ART-HD-CARD-2026-09` §"Residual Risk").

**Escalation dependency:** None. Legal partner's outstanding opinion on children's data (Q2) does not apply to the English-language B2B cohort in scope. The 2026-Q4 ISO/IEC 42001 external audit conclusion is a *signal-slide* item, not a dependency on this decision; a non-conforming audit finding would trigger the chapter-04 §"2. Rollback criteria" "framework or obligation change" rollback shape ("hold at current tier until addendum published") which is pre-registered under the T3 contract.

**Rollback / reversibility:** Ramp revert to T2 is a config flip in the release pipeline (release-pipeline flag `helpdesk.deployment_tier=T2`; weekly-tested in the release-pipeline CI suite). Revert fires on any one of:
1. `SC-HD-INJ-01` injection-defeat rate above 0.5 percent for 3 consecutive days.
2. Any customer-visible incident at severity ≥ 2 (`ART-HD-INC-2026` entry).
3. Regression on the Q3 retention corpus > 5 percent on composite quality (`ART-HD-GRD-Q3-01` regression suite).
4. Mute of any T3+ tripwire monitor without risk-council approval (chapter 04 §"2. Rollback criteria" row; chapter 01 §"The decisions the programme owns" row).
5. `SC-HD-FAIR-01` per-slice delta widening beyond 6 points on any 7-day rolling window.

Revert does not require a human decision at trigger time; the config-flip is pre-authorised in the release pipeline. Decision authority on the promotion itself is risk council + `ai-evaluation-engineer` (L35) release-assurance sign-off.
```

Reading check: every one of the seven fields is populated; the `Evidence pointers` field cites IDs *and versions and hashes*, not descriptions; the `Options and recommendation` field names three options with the recommendation and ≤ 3 lines of rationale; the `Residual-risk claim` is tied to named monitors with thresholds; the `Escalation dependency` resolves ("none") rather than being left blank or marked "TBD"; the `Rollback / reversibility` field names five mechanically enforceable triggers, each with a specific metric, threshold, and duration guard — no trigger reads "if things get bad" or "at the council's discretion."

### Step 3 — Fill Slide D-02 (above-appetite acceptance, 90-day window)

Chapter 02 §"Common failure modes and their fixes" names **"silent above-appetite"** — above-appetite rows aggregated in the pre-read but no decision requested; the fix is "make above-appetite acceptance a *forced* decision slide when appetite is breached; no drift by silence." This slide is the exact instantiation of that forced-slide rule. Chapter 01 §"The decisions the programme owns" row — "Accept residual *above* appetite for a fixed window" — routes this through the risk council; chapter 02's `Rollback / reversibility` field on an above-appetite acceptance is **how the acceptance expires** (chapter-02 exercise 02 §"Starter guidance": "a date, a condition that revokes the exception, or an evidence event that closes the row"; '"we will revisit" is not a rollback').

Fill the slide:

```markdown
## Decision slide D-02 — above-appetite acceptance on `tenant_tier` parity (90-day window)

**Decision requested:** Approve above-appetite acceptance on register row `HR-CS-FAIR-08` (`tenant_tier` per-slice quality parity) for a 90-day fixed window (2026-09-24 → 2026-12-23), subject to the named conditions and auto-revocation triggers below.

**Scope:** `HelpDesk` base variant, English-language cohort, EU + US customers. The 3.1-point widening of the `standard`-vs-`enterprise` per-slice parity delta on the composite quality metric; register row re-scored above appetite at the mod-103 Q3 refresh. No change to any other register row's appetite status. The window ends on 2026-12-23 whether or not the mod-106 remediation work (prompt-template refresh and re-balance intake) has concluded; a request for extension beyond the window is a separate decision slide at the 2026-Q4 council.

**Evidence pointers:**
- Register: row `HR-CS-FAIR-08` v2026.09 — currently above appetite at the Q3 refresh; residual claim moved from "Medium-low" to "Medium" pending remediation.
- Fairness: `ART-HD-FAIR-Q3-01` v2.0 sha256:8acd... — Q3 per-slice measurement showing the 3.1-point parity delta; prior Q2 measurement was 1.9 points (at-appetite).
- Monitoring: `ART-HD-MON-2026` v1.3 sha256:3cfd... — `SC-HD-FAIR-01` fired on 2026-09-10 and is the signal that triggered this decision; other critical monitors (`SC-HD-INJ-01`, `SC-HD-PII-01`) remain green.
- Treatment plan: mod-106 remediation ticket `TREAT-HD-FAIR-2026-Q4-01` — prompt-template refresh (owner: ML lead, due 2026-11-01); re-balance intake composition (owner: data lead, due 2026-12-01). Both items on the chapter-01 bi-weekly working-session tracker.
- Legal exposure: `OPIN-HD-EU-FAIR-2026-09-18` — legal partner's preliminary note on EU AI Act Article 10 data-governance exposure; reservation text in the decision-log entry (see Section 2 of this pack); legal partner records a formal *reservation* at the council, not a dissent.

**Options and recommendation:**
- **A** — Approve the 90-day above-appetite acceptance as scoped, subject to the four conditions below.
- **B** — Reject and hold `HelpDesk` at current tier (T2) until the remediation closes the parity delta.
- **C** — Approve a shorter 45-day window with a mid-window checkpoint on 2026-11-07.
- **Recommendation: A.** The remediation plan is on a 90-day critical path with two named milestones on the bi-weekly tracker; the shorter window (option C) adds council overhead without changing the mechanism of remediation. Rejection (option B) blocks the T2 → T3 promotion (slide D-01) without proportionate risk reduction, because `HR-CS-FAIR-08` was in appetite for the full Q3 window prior to the 2026-09-10 widening and the remediation targets the underlying content-mix drift rather than the measurement.

**Residual-risk claim:** After the named conditions take effect, we assess residual on `HR-CS-FAIR-08` as **Medium** for the duration of the 90-day window, with a return to **Medium-low** on remediation close. The claim holds subject to no further widening beyond 5 points on `SC-HD-FAIR-01` and no severity ≥ 2 fairness-related incident on `ART-HD-INC-2026`.

**Escalation dependency:** None on this decision. The EU AI Act Article 10 legal-opinion question is held *open* on the register (it is not a council-decision question; chapter 01 §"The decisions the programme owns" row — "Legal-opinion decision (obligation compliance ruling) — *never owned here* — Legal counsel"). The legal partner's reservation is recorded in the decision-log entry, but the reservation does not block the acceptance decision.

**Rollback / reversibility:** The acceptance expires automatically on 2026-12-23 (90 days from the council date). The exception is auto-revoked before the expiry on any one of:
1. `SC-HD-FAIR-01` per-slice delta widening beyond 5.0 points on any 7-day rolling window (auto-triggered revocation; monitor wires to the release-pipeline flag `helpdesk.fairness_exception_active=false`; falls back to the pre-exception residual claim and holds the T2 tier).
2. A severity ≥ 2 incident tied to fairness-shaped harm on the `standard` tier (`ART-HD-INC-2026` entry; manual revocation fires on incident-ticket creation).
3. Any slip on either remediation milestone (`TREAT-HD-FAIR-2026-Q4-01` — prompt-template refresh by 2026-11-01; re-balance intake by 2026-12-01) that is not re-baselined with a named new milestone date by the next bi-weekly session.
4. A subsequent legal-partner opinion on EU AI Act Article 10 that elevates the exposure above the "reservation" level (the opinion is tracked on the register; the opinion itself is not a council decision, but its conclusion is a revocation trigger here).

Any request for extension beyond 2026-12-23 is a new decision slide at the 2026-Q4 council, not a renewal of this one. Decision authority: system owner + risk council co-signed (chapter 01 §"The decisions the programme owns" row).
```

Reading check: the slide is a decision slide, not a context slide — the "ask" is one imperative sentence at the top, not "the council should be aware that..." (chapter 02's read-out-session anti-pattern). Every one of the seven fields is populated. The `Rollback / reversibility` field is **how the acceptance expires** — a specific date (2026-12-23), four specific auto-revocation triggers each with a specific metric and threshold, and an explicit statement that extension is a *new* decision at the next council, not a renewal. "We will revisit" does not appear anywhere.

### Step 4 — Fill the readiness checklist

Chapter 02 §"The decision-slide template" pins the acceptance test: "**If any of the seven fields is empty, the slide is not ready for the council. Pull it, close the gap, and hold it for the next cycle.**" Make the check explicit on the slide pack so the pulled-slide decision is auditable.

```markdown
## Readiness checklist

| Slide | Decision requested | Scope | Evidence pointers | Options & rec. | Residual claim | Escalation dep. | Rollback / reversibility | Ready for council? |
|---|---|---|---|---|---|---|---|---|
| D-01 (T2 → T3 promotion) | yes | yes | yes | yes | yes | yes | yes | **Yes** |
| D-02 (above-appetite acceptance, 90-day window) | yes | yes | yes | yes | yes | yes | yes | **Yes** |

Both slides satisfy chapter 02 §"The decision-slide template" — all seven fields populated on every slide, no slide mixing context / decision / signal content, every `Evidence pointers` field citing IDs and versions, every `Rollback / reversibility` field citing mechanically enforceable triggers with named metrics and durations.
```

Reading check: the checklist is one row per decision slide; the acceptance test from chapter 02 is reflected as a per-field yes/no; a slide with any `no` cell is **pulled, not circulated**. Chapter 02 §"Common failure modes and their fixes" names this as the "ambush decision" fix — no decision may be added the day of the meeting; the readiness check runs at pre-read circulation time (5-7 days before the council).

### Step 5 — Final checkpoint

Re-read the two slides and the checklist. Confirm:

- **Slide D-01 (tier promotion)** is a decision slide with all seven fields populated; `Evidence pointers` cites artefact IDs *and* versions *and* hashes (not descriptions); `Options and recommendation` names three options with the recommendation in ≤ 3 lines of rationale; `Rollback / reversibility` names five mechanically enforceable triggers each with a specific metric, threshold, and duration guard; no trigger reads "if things get bad" or "at the council's discretion."
- **Slide D-02 (above-appetite acceptance)** is a decision slide with all seven fields populated; the slide is *forced* onto the agenda by chapter 02's silent-above-appetite rule even though the engineer could in principle have left the row un-discussed; `Rollback / reversibility` names **how the acceptance expires** (a specific date, not "we will revisit"), with four auto-revocation triggers; extension beyond the window is a new decision at the next council.
- **The checklist** runs the seven-field acceptance test as one row per slide. Any slide with an empty field is pulled, not circulated.
- No slide mixes context / decision / signal content. No slide reads "the council should be aware that..." in place of "approve X." Every field is populated with specific values tied to chapter-02 citations and specific scenario evidence; no cell reads "TBD" or "unclear."

## Expected output

`mod112-lab01/pre-read-decision-slides.md` — one populated document; two decision slides (one T2 → T3 promotion; one above-appetite acceptance) plus a readiness checklist; one to two pages total. The two decision slides are the load-bearing artefacts; the readiness checklist is the one an auditor reads first to confirm the seven-field discipline is operating.

## Checkpoint questions (self-check)

1. Slide D-01's recommendation is **B** (monitored ramp), not **A** (straight promotion). The Q3 evidence shows every threshold cleanly within the T3 bands. Why is the ramp-with-rollback the chapter-02 / chapter-04 shape rather than the straight promotion? (*Chapter 02 §"A worked decision slide (short form)" uses this exact scenario at the reading level and lands on option B for the same reason: the Q3 evidence supports promotion but has not been stressed against general-availability traffic volume. Chapter 04 §"2. Rollback criteria" pins the mechanical-enforcement shape — "rollback is a config flip in the release pipeline, tested weekly; if revert requires a human decision, the criterion has not been pre-registered properly." A monitored ramp with pre-registered rollback triggers retains the promotion commitment but attaches the testable, reversible form chapter 02 §"The decision-slide template" requires on field seven. Straight promotion (option A) is a decision without a mechanically enforceable reversibility clause — chapter 02's "missing rollback" failure mode — unless the T2 tier is itself a rollback destination with named triggers, which for a general-availability promotion at a six-month window is the shape option B explicitly commits to.*)

2. Slide D-02's `Rollback / reversibility` field reads **"The acceptance expires automatically on 2026-12-23"** with four auto-revocation triggers. Chapter 02 §"Common failure modes" says "missing rollback" is one of the seven failure modes. Why is an **expiry date** the right shape of rollback for an above-appetite acceptance, and what would make it a hedge? (*Chapter-02 exercise 02 §"Starter guidance" pins the shape: "On the above-appetite decision, the rollback field is *how the acceptance expires* — a date, a condition that revokes the exception, or an evidence event that closes the row. 'The exception is time-boxed to 90 days' is a rollback; 'we will revisit' is not." An expiry date is a rollback because the exception *does not persist* beyond the date without a new council decision — extension is a *new* decision slide at the next council, not a renewal of this one. The four auto-revocation triggers provide early termination on any of the named conditions. The field would be a **hedge** if it read "we will revisit in 90 days" without a hard expiry — the exception would then persist indefinitely in the absence of positive action to close it, which is the shape of a drift-to-permanence exception that chapter 02 §"Motivation" and chapter-02 §"Common failure modes" name as how programmes go silent on their appetite. Chapter 01 §"The decisions the programme owns" row — "Accept residual *above* appetite for a fixed window" — is explicit on the "fixed window" discipline for the same reason.*)

3. Slide D-02 exists at all. Chapter 02 §"Common failure modes and their fixes" names **silent above-appetite** as a failure mode. What is the forced-decision-slide rule, and what would the alternative look like? (*Chapter 02 §"Common failure modes and their fixes" pins the rule verbatim: "Make above-appetite acceptance a *forced* decision slide when appetite is breached; no drift by silence." The one-pager (chapter 02 §"The one-pager") carries "number of rows above appetite; delta since last council" as a required field; when the aggregate delta is non-zero, the pre-read cannot be circulated without a corresponding decision slide on each above-appetite row's status. The alternative (and the failure mode) is: the aggregate above-appetite count appears on the one-pager as a number, the pre-read moves on, and the row drifts for another quarter — no one asked the council and no one decided to accept. Six months later an auditor asks "who authorised the above-appetite acceptance on `HR-CS-FAIR-08`" and the answer is "nobody" because the decision was never put. D-02 exists because the silent-above-appetite rule forces the slide into the pack the moment the fairness-slice monitor fires and the register row re-scores above appetite. The slide also exists as the only shape chapter 01 §"The decisions the programme owns" and §"The sign-off routing graph" authorise — above-appetite acceptance is risk-council-countersigned, three-signature-minimum at the system level plus the council on top.*)

4. The `Evidence pointers` field on both slides cites IDs with versions and hashes rather than descriptions. Chapter 02 §"The decision-slide template" pins "Evidence pointers — Direct links to the register rows, evaluation reports, monitor extracts, red-team findings underneath the request." What property of a hashed evidence pointer does chapter 02 §"Minutes that survive audit" require that a prose description does not satisfy? (*Chapter 02 §"Minutes that survive audit" pins "the three properties that make minutes audit-durable": "(i) they *reference* the pre-read and one-pager by hash, (ii) they *distinguish* decisions from discussion, (iii) they carry *sign-off* from more than one attendee. Missing any one of the three tends to become a finding under ISO/IEC 42001 clause 9.3 (management review) or under EU AI Act Article 61-style post-market surveillance evidence expectations." §"Common failure modes and their fixes" names "**evidence-hash absence**" verbatim: "Minutes reference 'the pre-read' without a hash; fix: pre-read and one-pager are hashed and pinned in the minutes; if they change post-meeting, that is a corrigendum, not a silent edit." The property a hashed pointer satisfies that a prose description does not: **the pointer is to a specific *version* of the evidence, byte-identical and verifiable; a later refresh of the artefact does not retroactively change what the council considered**. A prose description — "the Q3 guardrail report" — points at whatever the folder contains *now*, which may have been refreshed after the decision; the decision log would then read as "approved on the basis of evidence X" where X is no longer the thing the council read. Chapter 05 §"Crosswalk shape" extends the same discipline: every artefact pointer on the crosswalk is **version-hash pinned**. The council-pack pointer shape and the crosswalk-row shape are two expressions of one evidence-integrity discipline.*)

## Why this lab matters

Exercise 02 asks you to assemble a full council packet — three decision slides, a one-pager, three decision-log entries, and minutes — for a mock quarterly session on your own primary running system. If the seven-field shape is new at exercise-02 authoring time, the budget blows on relitigating what "rollback" means in context, whether "we will revisit" counts, and whether the above-appetite row has to be a decision slide at all. This lab drills the atomic slide shape on a fixed fabricated system with two deliberately contrasting judgement calls: a straightforward T2 → T3 promotion where the discipline is the ramp-with-rollback refinement; and an above-appetite acceptance where the discipline is the hard-expiry-date shape plus the forced-slide rule. Getting the slide shape right here cascades through exercise 02 (whose three slides are exactly this shape) and into exercise 04 (whose tier-promotion block has the same evidence-citation and rollback-trigger shape at tier-document scope rather than council-slide scope). The slide is the atomic unit of the council interface; the lab is the drill that lets the full packet be assembled from already-rehearsed parts rather than from first principles.
