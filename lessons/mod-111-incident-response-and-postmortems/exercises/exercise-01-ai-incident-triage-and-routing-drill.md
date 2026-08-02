# Exercise 01 — AI Incident Triage and Routing Drill

**Estimated effort:** 2 hours

## Objective

Drill the triage pipeline defined in chapter 01 against a mixed queue of candidate incidents. For each candidate, assign an incident class from the seven-class taxonomy, assign a severity on the harm-shape rubric (SEV1-4), make the three-axis routing decision (product IR / security IR / governance IR), cross-walk to the mod-102 harm-model row and the mod-103 register row, and — for candidates that come out at SEV1 — write a one-line preliminary reportability signal (full reportability packaging is exercise 05's scope). The output is a triage workbook that a real on-call incident lead could follow without reinterpreting the taxonomy. The exercise trains the pattern-recognition muscle that is the difference between a jailbreak filed as a "P3 model quality regression" and an incident correctly opened as a SEV1 with the reporting clock started.

## Prerequisites

- Chapter 01 read in full — the taxonomy table, the severity rubric, and the three-axis routing matrix are the reference shapes.
- Chapter 06 skimmed — the SecOps / DFIR boundary decides which candidates should be routed to classical security IR *without* going through this rubric at all.
- Access to the mod-102 harm model and the mod-103 risk register for the primary running system you have carried since mod-101; you will cite row IDs from both.
- A pointer to whoever owns the general SEV process at your organisation — the AI severity rubric plugs into the general SEV bands and the mapping has to be defensible (chapter 01 §"Interface with the general SEV process").
- If you have no real running system, adopt the mock customer-service RAG assistant used throughout the module (the one that appears in the chapter 01 mock triage walkthrough and in chapter 02's mock incident) as the running system for this exercise.

## Problem statement

A mixed queue of ten candidate incidents has landed on the incident-triage rotation during a single shift. Some are AI-specific and belong to the seven-class taxonomy; some are general software / security incidents that should be filtered out and routed to the classical process; some are ambiguous and require a documented judgement call. The queue is intentionally noisy — the drill is to run every candidate through the same pipeline and to record the decisions so an auditor can reconstruct why each candidate was routed as it was.

The running system is a customer-service RAG assistant: it retrieves from an internal knowledge base plus a public help-forum mirror, and it can call two tools (`lookup_order`, `escalate_ticket`) on behalf of the customer. The assistant is deployed in the EU and to US customers; some conversations touch account data that is PII under GDPR and (in a narrow set of health-insurance customer journeys) PHI under HIPAA.

### The candidate queue

Use exactly these ten candidates. If the description is ambiguous, that is the point — record the judgement and its evidence.

| # | Candidate |
|---|---|
| C1 | Monitor (mod-109) fires: "prompt-injection-detected count > threshold on retrieval path in the last hour." One sampled trace shows the retrieved KB chunk contains injection strings that induce a `send_email` tool call; the trace shows the tool call was executed with a customer email in the body. |
| C2 | A support engineer files a ticket: the assistant recommended cancelling an active insurance policy to a customer who had asked about "reducing my monthly bill." The customer acted on it and lost coverage before the escalation was caught. |
| C3 | The nightly app-eval run reports a 12-point drop on the "protected-group refund tone" fairness slice against the previous week's baseline. Live traffic is running the newer model. |
| C4 | An anonymous internal report claims the assistant "sometimes just makes up policy numbers." No trace ID; no reproducer; no user complaint attached. |
| C5 | The observability platform reports a 40-minute period during which the assistant's p95 latency was above SLO. Root cause on the incident-response ticket already lists it as a model-provider incident, resolved. |
| C6 | A security-alerting rule flags a burst of failed login attempts against the support-agent inbox from a foreign IP range; one succeeded. The compromised inbox is the one the assistant escalates to via `escalate_ticket`. |
| C7 | A red-team engineer files a report: a new many-shot jailbreak variant against the assistant reliably extracts the system prompt in a staging tenant with the shipped guardrails. No production traffic is running the variant yet. |
| C8 | A dataset engineer notices that a debug log stream is receiving raw user prompts including full names, addresses, and (in the health-insurance flow) policy numbers. The log stream is retained for 90 days and is readable by the platform team. |
| C9 | A customer files a complaint that the assistant told them, in French, that they were entitled to a refund that (per policy) they are not entitled to. The refund was not paid; the customer disagrees. |
| C10 | A monitor flags "tool-call-distribution drift" — the assistant is calling `escalate_ticket` 3× the baseline rate on a specific segment. Sampled traces show no obviously wrong behaviour, but the pattern is unfamiliar. |

## Requirements

Produce all five deliverables:

1. **The triage workbook** — a single table with one row per candidate and columns: candidate ID, one-line summary, class (from the seven-class taxonomy in chapter 01 or "not AI-specific"), severity (SEV1-4 or "not an incident"), product IR (yes/no/conditional), security IR (yes/no/conditional), governance IR (yes/no/conditional), harm-model row (mod-102 ID or "gap"), register row (mod-103 ID or "gap"), one-sentence justification. Every cell is filled — silent blanks are not acceptable.
2. **The "conditional" resolution log** — for every routing cell you wrote as "conditional," a short block naming the evidence that would flip the decision to "yes" and the evidence that would flip it to "no." A conditional cell that does not have a documented trigger is a broken decision (chapter 01 §"The routing decision — product IR vs security IR vs governance IR").
3. **The filtered-out list** — candidates that are *not* AI-specific incidents, with a one-paragraph justification per candidate stating where they should be routed instead (classical availability IR, classical security IR / SOC, none-of-the-above). This is where you defend the "not-an-AI-incident" call so the rubric does not get colonised by every ticket in the queue.
4. **The harm-model / register gap list** — every candidate that produced a "gap" cell in the workbook, with the harm-model row that *would* have covered it and a one-line proposed edit for mod-102 or mod-103. A missing cross-walk is itself a finding (chapter 01 §"The AI-specific incident taxonomy") and this list is what feeds back into those artefacts.
5. **The contested-decisions retrospective** — a two- to four-paragraph write-up naming the two or three candidates whose triage was hardest to defend, what made them hard, and what evidence you would want in place before the next shift to shorten the decision. This is the retrospective that stops the same argument recurring next month.

## Starter guidance

- **The class label drives everything downstream.** Assign the class first; do not choose the severity or the routing before the class is nailed. A mis-classed candidate produces a mis-routed incident even if every other step is done perfectly.
- **A single user's serious harm can be SEV1.** Chapter 01 pins this rule explicitly. Availability-shaped severity rubrics grade C2 or C9 as P3 or P4 by default. Do not let the classical rubric downgrade a harm-shape incident.
- **A confirmed regulator-reportable event is always at least SEV1.** Chapter 01 also pins this. If a candidate might trigger EU AI Act Article 73, SR 11-7 materiality, MDR, or HIPAA breach notification, resolve up to SEV1 — do not litigate reportability inside the SEV rubric.
- **"Conditional" is a decision, not a hedge.** Every conditional routing cell needs a written trigger. If you cannot state the trigger, the routing is either "yes" or "no" and the ambiguity is a signal your evidence is incomplete.
- **The SecOps boundary comes first for some candidates.** Chapter 06 draws the line: a compromised inbox (C6) is a classical SOC IR incident whose *interaction* with the assistant is the AI-side concern, not the whole thing. Route the SOC-side to SOC; open a separate AI-side sub-incident only if the assistant's behaviour needs an app-side response.
- **Not-reproducible is a triage state, not a resolution.** C4 (anonymous, no trace ID) should not become a SEV4 that never gets looked at again. The triage decision is a specific ask — investigation ticket with a named owner and a "raise to SEV3 on reproduction" trigger — not silence.
- **PII in a log is a PII leakage class incident even if no external actor touched it.** C8's exposure crossed a trust boundary (from user session to platform-team-readable retention). Chapter 01's PII leakage row applies; the routing includes security IR conditional on trust-boundary evidence.
- **Cite chapter 01 by table row when you write the justifications.** The taxonomy and rubric are the source of authority; if your justification cannot point at a specific row of a specific table in chapter 01, the taxonomy is not doing the work — you are.

## Acceptance criteria

The submission is accepted when:

- The triage workbook has all ten candidates and every cell is filled — no silent blanks, no "TBD," no "unclear."
- Every candidate has a defensible class assignment: either one of the seven classes from chapter 01 §"The AI-specific incident taxonomy" or an explicit "not AI-specific" call.
- Severity assignments respect the two hard rules from chapter 01: single-party serious harm can be SEV1; confirmed regulator-reportable events are at least SEV1. Deviations are called out with a specific justification.
- Every "conditional" routing cell has a written trigger in the conditional resolution log — evidence that flips it to yes, evidence that flips it to no.
- The filtered-out list argues explicitly why each excluded candidate does not belong to the AI-specific taxonomy and where it goes instead. "Not an AI thing" is not a justification.
- Every "gap" cell in the harm-model / register columns produces a corresponding entry in the gap list with a proposed mod-102 or mod-103 edit.
- The contested-decisions retrospective is written; naming the hard cases is a first-order deliverable, not an optional wrap-up.
- A reader with no prior context can open the workbook and, in under ten minutes, reconstruct how every candidate would be handled on shift.

## Stretch goals

- **Round-trip the triage.** Hand the workbook to a peer (a second L25 risk engineer, or a peer role like `ai-eval-engineer` L35) and have them independently re-triage three candidates without seeing your rows. Where their triage differs from yours, that disagreement is the effective challenge chapter 01 §"Interface with the general SEV process" implies your process should generate. Record the deltas and reconcile.
- **Wire the workbook into the general SEV dashboard.** Take the AI severity assignments and map them into the general SEV bands your organisation uses (or a placeholder mapping). Show that an AI SEV1 is visible on the enterprise IR dashboard, not living in a shadow rubric.
- **Author the AIID / OECD.AI compare-point on the two most incident-shaped candidates.** For C1 and C2, search the AI Incident Database and OECD.AI's AI Incidents Monitor for public incidents in the same class. Cite two or three matching incidents and note whether your candidate is a novel variant or a known pattern (this becomes fixture-strengthening evidence in exercise 04).
- **Draft the reportability check on the SEV1s.** For every candidate you assigned SEV1, write a one-line preliminary "is this Article 73 / SR 11-7 / MDR / HIPAA reportable?" answer with the evidence you would gather next. Do not author the packages here — that is exercise 05 — but do produce the trigger record so the clock is not accidentally started late.

## Deliverable format

A single markdown document `mod-111-ex01-triage-queue.md` that contains, in order:

1. *Summary* — one paragraph naming the running system, the shift context, and the queue size.
2. *Triage workbook* — the ten-row table described in requirement 1.
3. *Conditional resolution log* — one block per "conditional" cell, with the trigger conditions.
4. *Filtered-out list* — one paragraph per excluded candidate.
5. *Harm-model / register gap list* — one row per gap, with the proposed edit.
6. *Contested-decisions retrospective* — the two- to four-paragraph write-up.
7. *Cross-references* — pointers to the mod-102 rows, the mod-103 rows, and (for the SEV1s) the reportability-check drafts if you did the stretch goal.

## Submission

Attach the index markdown; if any candidate description had to be adapted to your running system, note the adaptation at the top of the document. Redact any real customer or tenant identifiers before submission — the discipline of the workbook is preserved regardless. The triage workbook becomes the input record for exercise 02 (whose mock incident, INC-2026-08-14-cs-rag-inj-01, corresponds to candidate C1 in this queue), so keep it accessible for the next exercise.
