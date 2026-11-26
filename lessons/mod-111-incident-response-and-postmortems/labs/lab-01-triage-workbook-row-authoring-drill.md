# Lab 01 — Triage Workbook Row Authoring Drill (Class / SEV / Three-Axis Routing / Harm-Model + Register Cross-Walk)

**Duration:** 45 minutes · **Format:** guided practice, markdown authoring on a fixed fabricated LLM application

## Objective

Internalise the chapter-01 **triage-workbook row shape** — class (from the seven-class taxonomy), severity (SEV1-4 on the harm-shape rubric), the three-axis routing decision (product IR / security IR / governance IR with "conditional" as a documented decision), the harm-model row cross-walk (mod-102), the register row cross-walk (mod-103), and the one-sentence justification per cell — by authoring two complete triage rows on a fixed fabricated running system (`HelpDesk`). The lab drills the atomic row shape chapter 01 pins so that when **exercise 01** runs it against a mixed queue of ten candidates on your real primary running system, the engineering budget goes into which candidates are AI-specific, which are not, and where the hard-to-defend judgement calls sit — not into inventing the row format or re-arguing what each column means.

The chapter-01 anti-patterns the shape forecloses: the **miscategorised-as-a-defect** row (a biased-outcome regression filed as a product-quality P3 with no incident channel and no post-mortem); the **miscategorised-as-a-breach** row (a jailbreak with retrieval exfiltration routed to SOC-only, who look for a network vector and never open the trace store); the **conditional-as-a-hedge** row (a "conditional" routing cell with no written trigger to flip it to yes or no); and the **silent gap** row (a missing harm-model or register cross-walk treated as an empty cell rather than a finding).

## Prerequisites

- Chapter 01 read — specifically "The AI-specific incident taxonomy" (the seven-class floor table with class labels, primary evidence surface, and typical routing); "Severity rubric — a rubric your SEV process can consume" (the four SEV bands grading on harm shape; the two hard rules — single-party serious harm can be SEV1; a confirmed regulator-reportable event is always at least SEV1); "The routing decision — product IR vs security IR vs governance IR" (the three independent axes, the working routing matrix, and "conditional is a documented decision at triage time, not a default"); and "Mock triage walkthrough — a jailbreak with retrieval exfiltration" (the five-step walkthrough shape — signal, class assignment, evidence quick-look, severity assignment, routing).
- Chapter 06 skimmed at the escalation triggers — a candidate with frontier-agent signals routes to L40 at triage time, not after RCA.
- A terminal where you can create a working directory and author a markdown file.
- No external access or compute required; the system brief, the register row stubs, and the two candidate incidents are fixed below.

## The fixed example system

Use this system throughout the lab — do *not* substitute your own. Carrying a fixed scenario keeps the lab bounded to the row-authoring skill. The system matches the `HelpDesk` carried through the mod-109 and mod-110 labs; the chapter-01 and chapter-02 running examples are built on the same shape.

> **`HelpDesk`** — a customer-support RAG assistant for a mid-size B2B SaaS company. Serves English-language customers of a billing-and-provisioning product. Assembles context from a knowledge-base store (product docs + internal runbooks + a mirrored public help-forum corpus) and from the authenticated caller's own recent ticket history. Backed by an in-house fine-tune (`helpdesk-finetune` v1.3.0) of the vendor-hosted base `FoundationCo-LLM-3.5` v3.5.2 behind a FastAPI service wrapping a LangGraph agent. Two production tool calls: `escalate_ticket(destination, summary)` and `lookup_account(account_id)`.
>
> **Classification under the EU AI Act:** high-risk (Annex III use case affecting a provider's customers; account data in scope). Deployed to EU and US customers. The sponsoring organisation is a US bank subsidiary — SR 11-7 is the parent MRM shape; HIPAA applies on the narrow subset of health-insurance-adjacent customer-service journeys; GDPR Article 33 runs concurrently for EU-resident data subjects.
>
> **Protected-attribute slice in scope:** tenant tier (`standard` vs. `enterprise`), used as a per-slice proxy for coverage / quality parity monitoring on `HR-CS-FAIR-08`.

### Fixed register row stubs (do not add more)

Every triage row cites a `register_row_id` that exists in this stub set. The register-side schema is from mod-103.

| Register row id | Scope line |
|---|---|
| `HR-CS-INJ-01` | Prompt-injection control credit — defence in depth across input filter, retrieval filter, and tool-invocation policy. |
| `HR-CS-PII-04` | PII egress control credit — Presidio at ingest / retrieval / output and per-caller entitlement; absolute-rule row. |
| `HR-CS-FAIR-08` | Per-slice quality parity on the `tenant_tier` slice (standard vs enterprise); fairness-tagged row; mod-106 seam. |
| `HR-CS-QUAL-03` | Quality regression on the composite (faithfulness + coverage + safe-refusal) metric. |
| `HR-CS-DRIFT-02` | Covariate drift on top-20 input features and on prompt-embedding distribution. |
| `HR-CS-VEND-11` | Vendor silent-swap risk on the `FoundationCo-LLM-3.5` base model. |
| `HR-CS-PROV-12` | Supply-chain tampering risk on the fine-tune weights. |

### Fixed harm-model rows (do not add more)

The triage-row cross-walk to mod-102 uses this stub set. Add a gap row if a candidate matches no entry here — "gap" is itself a finding (chapter 01 §"The AI-specific incident taxonomy") and feeds the harm-model / register gap list.

| Harm-model row id | Scenario shape |
|---|---|
| `HR-CS-INJ-01` | Indirect prompt injection via retrieved context induces an unauthorised tool call. |
| `HR-CS-PII-04` | PII egress on an output, a log, or a downstream store. |
| `HR-CS-FAIR-08` | Disparate per-slice outcome on the `tenant_tier` slice (standard tier systematically disadvantaged). |
| `HR-CS-HAL-05` | Hallucinated policy or entitlement statement surfaced to a user with a downstream real-world action. |
| `HR-CS-MI-09` | Membership-inference / training-data reproduction via inference-time probe. |

### The two candidates (do not substitute)

Use exactly these two. Each is deliberately shaped to force one judgement — candidate L-01 forces the "harm-shape SEV1 even with 100% availability" reading and the three-axis routing; candidate L-02 forces the "conditional with a written trigger" shape and the harm-model gap-row shape.

> **Candidate L-01** — At 2026-11-20 14:03Z the mod-109 monitor `SC-HD-FAIR-01` fires: `tenant_tier` per-slice quality parity moved 14 points on the `standard` tier against the `enterprise` tier week-over-week; the metric sits well outside its +/-5-point band. Live traffic has been running `helpdesk-finetune` v1.4.0 since 2026-11-18; the v1.3.0 shadow-eval that cleared release passed the parity band on both tiers. A customer-success manager has flagged two `standard`-tier complaints citing refused refund escalations where comparable `enterprise`-tier cases were processed. HelpDesk is deployed in the EU; `standard` tier is disproportionately individual customers of small member-state resellers.
>
> **Candidate L-02** — A red-team engineer files a report at 2026-11-24 09:40Z: in a *staging* tenant with shipped guardrails and shipped tool allow-list, a many-shot variant against HelpDesk reliably causes the assistant to emit a text string that looks like a verbatim 50-token span from the training corpus (`helpdesk-transcripts-2024-2026-q3`). The emitted span contains what appears to be a real customer name and a real ticket reference. No production traffic is running this variant yet. No PII has reached a customer. The red-team report asks whether this is a membership-inference-shaped incident or a hallucination that coincidentally looks like training data.

## Steps

### Step 1 — Create your working directory and the workbook scaffold

```bash
mkdir -p mod111-lab01 && cd mod111-lab01
touch helpdesk-triage-workbook.md
```

Open `helpdesk-triage-workbook.md` and paste this scaffold. You will fill the sections in order.

```markdown
# HelpDesk — triage workbook (lab 01)

System: HelpDesk (customer-support RAG assistant; see lab brief)
Chapter references: mod-111 ch 01 (taxonomy; severity rubric; three-axis routing; harm-model + register cross-walk; routing matrix); ch 06 (frontier-agent escalation triggers skimmed at triage time)
Shift window: 2026-11-20 00:00Z through 2026-11-24 23:59Z
Register version bound to: mod-103 register v2026.11
Running-system harm model: mod-102/harm-model/intake-INT-HD-001.md

## 1. Triage workbook

(ten-column row per candidate: candidate id | summary | class | SEV | product IR | security IR | governance IR | harm-model row | register row | one-sentence justification)

## 2. Conditional-trigger log

(one block per "conditional" cell: evidence that would flip it to yes; evidence that would flip it to no)

## 3. Harm-model / register gap list

(one row per "gap" cell in the workbook: proposed mod-102 row / mod-103 row edit; owner; target date)

## 4. Reportability signal (SEV1 only)

(one line per SEV1: Article 73 / SR 11-7 / HIPAA / GDPR Article 33 preliminary answer with the evidence you would gather next)

## 5. Open items

(what is deferred to exercise 01 time; what is a judgement call for the retrospective)
```

### Step 2 — Fill Section 1, row 1 (candidate L-01)

Chapter 01 §"The AI-specific incident taxonomy" gives the seven classes; the *biased-outcome escalation* row is the one that pins "a protected-group slice's outcome metric moves outside its documented band, or a specific complaint chain shows disparate impact." Chapter 01 §"Severity rubric" pins the SEV1-forcing rule for confirmed regulator-reportable events; the EU AI Act Article 3 sub-category "infringement of Union law obligations intended to protect fundamental rights" is in scope when a protected-attribute slice is measurably disadvantaged in an EU-deployed high-risk system. The routing matrix row for biased-outcome escalation is `Yes / No / Yes` plus legal where disparate impact is actionable.

Fill the row (write it out in full so the shape is on the page, not in your head):

```markdown
## 1. Triage workbook

| # | Summary | Class | SEV | Product IR | Security IR | Governance IR | Harm-model row | Register row | Justification |
|---|---|---|---|---|---|---|---|---|---|
| L-01 | `tenant_tier` fairness-slice parity moved 14 pts on `standard` tier week-over-week after v1.4.0 rollout; two `standard`-tier complaints cite refused refund escalations where `enterprise`-tier comparable cases were processed; EU-deployed high-risk system | **Biased-outcome escalation** (ch 01 §"The AI-specific incident taxonomy" row 3) | **SEV1** — EU-deployed high-risk system with a measurably disadvantaged protected-attribute slice tripping its documented parity band plus a complaint chain; EU AI Act Article 3 sub-category "infringement of Union law obligations intended to protect fundamental rights" is in scope and ch 01 §"Severity rubric" pins "a confirmed regulator-reportable event is always at least SEV1" | **Yes** — user-facing outcome is wrong; app-side evidence chain (trace store + fairness slice report v2026-11-18) and immediate mitigation (model roll-back or fairness-shaped guardrail) sit here | **No** — no confidentiality, integrity, or availability surface is touched; routing matrix row "biased-outcome escalation" (ch 01) is `No` | **Yes** — Article 73 reportability check begins; residual on `HR-CS-FAIR-08` moves and the fairness liaison + legal are on routing | `HR-CS-FAIR-08` | `HR-CS-FAIR-08` (and a sibling-row candidate on `HR-CS-QUAL-03` if the parity delta is composed with a quality drop on the same tier) | Harm-shape rubric triggers SEV1 even though service is 100% available; three-axis routing is Yes/No/Yes; fairness liaison and legal are on the governance-IR routing per ch 01 §"The routing decision" |
```

Reading check: every column is filled with a specific value tied to a chapter citation; the SEV rationale cites the "harm shape, not outage minutes" rule and the regulator-reportable floor; the "security IR = No" is argued, not implicit; the governance-IR cell names the regulator the clock would start on; the harm-model and register cross-walks both land on existing rows.

### Step 3 — Fill Section 1, row 2 (candidate L-02)

Chapter 01 §"The AI-specific incident taxonomy" gives both membership-inference / training-data exposure (row 6) and hallucination-driven harm (row 4) as candidate classes. The judgement call is whether the emitted span is confirmed as a training-data reproduction or is a hallucinated look-alike — the evidence quick-look from ch 01 §"Mock triage walkthrough" is what the triage decision hangs on. The routing matrix row for membership-inference is `Yes / Yes / Yes + privacy office`; for hallucination-driven harm (single user, no downstream action) the row is `Yes / No / Yes`. "Conditional" on security IR becomes the correct answer when the evidence quick-look has not yet confirmed which class applies.

Fill the row:

```markdown
| L-02 | Many-shot variant in staging causes HelpDesk to emit a 50-token string resembling a real customer name + ticket reference from `helpdesk-transcripts-2024-2026-q3`; shipped guardrails and shipped tool allow-list did not block; no production traffic running the variant; no PII has reached a customer | **Membership-inference / training-data exposure** (ch 01 §"The AI-specific incident taxonomy" row 6) *pending* evidence quick-look — if the emitted span is confirmed verbatim against the training corpus, this is MI; if the span coincidentally resembles but does not match, this is **Hallucination-driven harm** (ch 01 row 4). Record the pending-class state on the row rather than guessing. | **SEV2** — a shipped safety control (guardrail + tool allow-list) was bypassed by a reproducible exploit in staging with no production traffic yet (ch 01 §"Severity rubric" SEV2 row — "jailbreak that bypasses a shipped safety control but has not yet been observed to cause harm"). Escalate to SEV1 the moment production traffic runs the variant or the span is confirmed as training-data reproduction with an identified named party. | **Yes** — reproduce in a controlled environment; the app-side evidence chain (red-team-fixture trace + training-corpus lookup) sits here | **Conditional** — flip to Yes if the quick-look confirms verbatim training-data reproduction (data-exposure chain begins); flip to No if the quick-look confirms the span is a hallucinated look-alike and no PII crossed a trust boundary. The trigger evidence is a byte-level comparison against `helpdesk-transcripts-2024-2026-q3` by the data-platform peer. | **Yes** — regardless of class, a bypassed shipped safety control is a governance-IR event; register residual on `HR-CS-INJ-01` and (if MI) `HR-CS-MI-09`-equivalent (see gap list) moves; the privacy office is conditionally on routing on the same trigger as security IR | `HR-CS-MI-09` (if MI is confirmed) OR `HR-CS-HAL-05` (if hallucinated look-alike is confirmed) — **both exist in the harm-model stub set** | `HR-CS-INJ-01` (shipped guardrail bypassed) and — **gap** — no register row currently carries the MI / training-data-exposure residual; proposed edit lands in Section 3 | Pending-class rows are legitimate when the evidence quick-look is not yet resolved; "conditional" routing cells carry the written trigger in Section 2; the register gap feeds Section 3 |
```

Reading check: the class is a pending judgement (not a guess) with the two candidate classes named and the quick-look that resolves between them cited; the SEV is SEV2 with an explicit escalation trigger to SEV1; the "conditional" security-IR cell is a documented decision, not a hedge, and its trigger evidence names a specific artefact (byte-level comparison against the training corpus); the register cell is "gap" with a Section-3 cross-reference rather than a silent blank.

### Step 4 — Fill Section 2 (Conditional-trigger log)

Chapter 01 §"The routing decision — product IR vs security IR vs governance IR" is explicit: `"Conditional" means a documented decision at triage time — not a default. The decision is recorded on the incident ticket and is itself audit evidence.` Every conditional cell from Section 1 needs a block here naming the trigger evidence in both directions.

```markdown
## 2. Conditional-trigger log

### L-02 · security IR = conditional

**Trigger to flip to Yes:** the data-platform peer completes a byte-level comparison of the emitted 50-token span against `helpdesk-transcripts-2024-2026-q3` and confirms the span is a verbatim reproduction of a corpus record with a real named party. Chapter 01 §"The routing decision" routing matrix row "Membership-inference / training exposure" is `Yes/Yes/Yes+privacy office`; confirmation forces security IR to Yes and adds the privacy office to governance IR. Residual on a new `HR-CS-MI-09`-equivalent register row (Section 3 gap) elevates.

**Trigger to flip to No:** the byte-level comparison completes and finds no match in the training corpus (the span is a hallucinated look-alike). The class resolves to *hallucination-driven harm (single user)*; chapter 01 §"The routing matrix" routing row is `Yes/No/Yes`; security IR closes; register residual lands on `HR-CS-HAL-05`.

**Due date for trigger evidence:** 2026-11-25 end-of-business (one working day from the red-team report filing on 2026-11-24 09:40Z). Owner: data-platform peer; artefact: byte-level comparison report filed as a ticket comment on the incident ticket (not a Slack thread; chapter 02 §"Anti-patterns" — "the SOC hand-off by chat" — forbids chat-only evidence hand-offs, and the same discipline applies to intra-app peer hand-offs).

### L-02 · governance IR · privacy office = conditional (sub-cell)

Same trigger shape as the security-IR cell above. Governance IR itself is unconditional Yes on L-02; the *privacy office* sub-routing is the conditional one, on the same byte-level-comparison trigger.
```

Reading check: every "conditional" cell from Section 1 has a matching block; each block names (i) the flip-to-Yes evidence, (ii) the flip-to-No evidence, (iii) the due date, and (iv) the named owner and the artefact that files the decision; no block reads as a hedge.

### Step 5 — Fill Section 3 (Harm-model / register gap list)

Chapter 01 pins that a missing cross-walk is itself a finding and feeds a mod-102 / mod-103 update through the incident-triggered path.

```markdown
## 3. Harm-model / register gap list

| Gap id | Workbook cell | Proposed edit | Owner | Target date |
|---|---|---|---|---|
| G-L02-R | Register row cell on L-02 (membership-inference branch) | Add a new row `HR-CS-MI-09` to mod-103/register/ covering membership-inference / training-data exposure on `helpdesk-finetune` v1.4.0; inherent / residual / control-defeated populated from the red-team report and from mod-107 privacy evaluation; evidence pointer to the incident record. The harm-model row `HR-CS-MI-09` (exists in mod-102 stub set) is the anchor. | Risk engineer (L25) / **<name>** | 2026-11-28 (within one week of the red-team report; chapter 03's incident-triggered register-update cadence applies on SEV2 at the same cadence as SEV1) |
```

Reading check: every "gap" cell in Section 1 produces a matching Section 3 row; every row names the specific proposed edit (not "investigate the gap"), the owner as an individual (placeholder name is fine; a role without a named individual is an un-owned edit), and the target date as a specific date, not a quarter.

### Step 6 — Fill Section 4 (Reportability signal — SEV1 only)

Chapter 01 pins: "A confirmed regulator-reportable event is always at least a SEV1. Do not litigate reportability inside the SEV rubric; that is the compliance track's decision (chapter 05)." The SEV1 row here is L-01 (biased-outcome escalation on an EU-deployed high-risk system). Chapter 05's packaging scope is exercise 05; here we produce only the preliminary reportability signal so the clock is not accidentally started late.

```markdown
## 4. Reportability signal (SEV1 only)

- **L-01 — EU AI Act Article 73**: *preliminary* reportable under Article 3 sub-category "infringement of Union law obligations intended to protect fundamental rights" — the EU-deployed high-risk system has a measurably disadvantaged protected-attribute slice tripping its documented parity band plus a complaint chain. Clock start: *awareness* at 2026-11-20 14:03Z (monitor fire plus first complaint read). Envelope: without undue delay, within 15 days (chapter 05 §"Timing envelopes summarised"). Evidence to gather next: the full fairness-slice report v2026-11-18 vs v2026-11-20; the shadow-eval report that cleared v1.4.0; the two customer-success complaint records with consent to cite. Submitting role: compliance liaison, not risk engineer (chapter 05 §"Practical authoring notes").
- **L-01 — GDPR Article 33**: *not triggered* on current evidence — no personal-data breach, only a disparate-outcome pattern. Re-evaluate on confirmation that the protected-attribute slice disadvantage manifests through a specific individual's data processing. If triggered, 72-hour clock runs concurrently.
- **L-01 — SR 11-7**: *conditional* on the FI's materiality threshold — the assistant touches account information on customer-decision flows; the MRM committee receives a memo on the committee's cadence regardless (SR 11-7 is guidance, not a hard-clock rule). Chapter 05 §"SR 11-7" pins the memo shape.
```

Reading check: every SEV1 row has a preliminary answer per obligation; each answer names the clock-start timestamp (not the mitigation timestamp — chapter 05 §"Motivation" pins "the clock starts at awareness, not at confirmation"); each answer names the evidence to gather and the accountable submitting role; a "not triggered" answer cites the criteria that were checked (chapter 01's "not reportable is a documented decision, not an omission" rule).

### Step 7 — Fill Section 5 (Open items)

```markdown
## 5. Open items

- L-02 remains pending-class until the byte-level-comparison trigger evidence arrives (2026-11-25 EOB); if confirmed as MI, the row re-grades to SEV1 and the register gap G-L02-R closes with the new `HR-CS-MI-09` row populated; the governance-IR reportability signal rows are then authored on the same cadence as L-01.
- Neither candidate matches the frontier-agent escalation triggers (chapter 06) — HelpDesk's deployment tier (mod-102 ch 04) is below the enterprise non-frontier threshold, no sub-agent spawning or credential-chain construction is observed, and no frontier-risk category applies. L40 escalation is *not* triggered on either row; the triage checklist entry confirms that the check was made.
- The workbook is the input to exercise 01 at scale (ten candidates on your own primary running system); the row shape and the conditional-trigger log shape drilled here are the atomic units exercise 01 produces ten of.
```

### Step 8 — Final checkpoint

Re-read the five sections. Confirm:

- **Section 1 (workbook)** has two rows; every one of the ten columns is filled per row; no cell reads "TBD" or "unclear"; one row is a pending-class call with the two candidate classes named and the quick-look trigger cited; both rows cite a specific chapter-01 table row in the class justification.
- **Section 2 (conditional-trigger log)** has one block per "conditional" cell from Section 1; every block names the flip-to-Yes evidence, the flip-to-No evidence, the due date, and the named owner and artefact.
- **Section 3 (gap list)** has one row per "gap" cell from Section 1; every row names a specific proposed edit, an individual owner, and a specific target date.
- **Section 4 (reportability signal)** has one line per obligation per SEV1 row; every line names the clock-start timestamp, the envelope, the evidence to gather, and the accountable submitting role.
- **Section 5 (open items)** closes the pending-class judgement on L-02 and records the frontier-agent-check negative result.
- No row is justified by "common sense" or "feels right" — every column has a specific chapter-01 citation and a specific piece of scenario evidence.

## Expected output

`mod111-lab01/helpdesk-triage-workbook.md` — one populated five-section document; one to two pages. Section 1 (triage workbook) and Section 2 (conditional-trigger log) are the load-bearing artefacts; Section 3 (gap list) is the one an auditor reads first after the workbook.

## Checkpoint questions (self-check)

1. L-01's SEV column reads **SEV1** even though the service is 100% available and no outage minutes were lost. Why does the SEV rubric return SEV1? (*Chapter 01 §"Severity rubric — a rubric your SEV process can consume" pins two hard rules: "A single user's serious harm can be a SEV1" and "A confirmed regulator-reportable event is always at least a SEV1." L-01 has a measurably disadvantaged protected-attribute slice on an EU-deployed high-risk system, which falls under the EU AI Act Article 3 sub-category "infringement of Union law obligations intended to protect fundamental rights" (chapter 05 §"EU AI Act — Article 73"). Availability-shaped SEV rubrics would grade this a P3 because no outage minutes accumulated; the harm-shape rubric grades it SEV1 on both rules. Chapter 01 §"Motivation" names this exact failure mode — "miscategorised as a defect" — and the rubric rule is what prevents it.*)

2. L-02's class column reads **pending** — membership-inference OR hallucination-driven harm — rather than one of the two. Why is "pending" a legitimate row state rather than a hedge that needs to be resolved before the row is written? (*Chapter 01 §"Mock triage walkthrough — a jailbreak with retrieval exfiltration" demonstrates the five-step triage shape — signal → class assignment → evidence quick-look → severity assignment → routing — and shows the evidence quick-look as a bounded step that can take a working day if the artefact (here, the byte-level comparison against the training corpus) is not yet in hand. Writing the row with a pending class and a named trigger that resolves it is the shape; writing the row with a guessed class is the anti-pattern — it leads to a miscategorised incident whose routing is wrong. The pending-class state is bounded by Section 2's trigger with a due date (2026-11-25 EOB); it is not open-ended.*)

3. L-02's security IR cell reads **conditional** rather than Yes or No. Chapter 01 says "conditional is a documented decision, not a default." What makes the conditional cell a decision in this workbook, and what would make it a hedge? (*Chapter 01 §"The routing decision — product IR vs security IR vs governance IR" says "Conditional means a documented decision at triage time — not a default. The decision is recorded on the incident ticket and is itself audit evidence." The conditional cell is a **decision** in this workbook because Section 2 names the flip-to-Yes evidence, the flip-to-No evidence, the due date, the named owner, and the artefact that files the resolution. It would be a **hedge** if Section 2 said "unclear whether security IR applies" without a named trigger — the ambiguity would signal the evidence is incomplete rather than the routing is genuinely two-sided. The L-02 case is genuinely two-sided on current evidence — both routings are correct answers to different versions of the facts — so "conditional" is the correct cell; the trigger converts it to Yes or No deterministically when the comparison report lands.*)

4. L-02's register-row cell is **gap** with a proposed edit in Section 3. Why not leave the cell blank and author the register row later as part of exercise 01? (*Chapter 01 §"The AI-specific incident taxonomy" is explicit: "Every AI-specific incident cross-walks to a harm-model row (mod-102) and a register row (mod-103); a missing cross-walk is itself a finding." A silent blank is the exact anti-pattern — the auditor reads the blank as "no cross-walk was attempted" rather than "a cross-walk was attempted and the row does not yet exist." The gap-list row in Section 3 is the record that (i) the cross-walk was attempted, (ii) the gap is known, (iii) the specific proposed edit is on a named owner with a specific due date. Deferring to exercise 01 would mean two separate records disagreeing about whether a cross-walk existed at triage time — chapter 03 §"Motivation" names this exact failure shape in the context of post-mortems ("the engineering-only post-mortem" that the regulator later contradicts), and the discipline is the same here.*)

## Why this lab matters

Exercise 01 asks you to run the triage pipeline against a mixed queue of ten candidates on your own primary running system — class, SEV, three-axis routing, harm-model + register cross-walk, and one-sentence justification per cell on each row. If the row shape is new at exercise-01 authoring time, the budget blows on re-arguing what "conditional" means, where the SEV1 floor sits, and whether a missing cross-walk is a blank or a finding. This lab drills the atomic row shape on a fixed fabricated system with two candidates deliberately shaped to force the hardest judgement calls (SEV1 on a 100%-available service; pending class with a named trigger; conditional routing with a documented trigger; register gap with a Section-3 edit). What it drills is also what exercises 02-05 consume — the L-01 row would open the RCA that exercise 02 runs; the L-02 row's register-gap edit would be the mod-103 patch that exercise 03's post-mortem reconciles. Getting the row shape right here cascades through the module.
