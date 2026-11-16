# Lab 01 — Five-Sentence Sub-Category Translation Drill

**Duration:** 45 minutes · **Format:** guided practice, markdown authoring

## Objective

Internalise the chapter-01 pattern for reading a NIST AI RMF sub-category *as an engineer* — the five sentences: **says / means / test / pass / evidence**. You will produce five-sentence reads for four sub-categories against a tiny fixed example system, and verify each one against a checkpoint before moving on. This lab does *not* ask you to cover all sub-categories (that is exercise 01); it drills the atomic translation.

## Prerequisites

- Chapter 01 read.
- A terminal where you can create a working directory and author markdown files.
- The **NIST AI RMF Playbook** (`airc.nist.gov/AI_RMF_Knowledge_Base/Playbook`) open in a browser tab so you can quote the outcome statement verbatim and confirm the sub-category identifier.
- No ISO / EU AI Act / SR 11-7 knowledge required — those chapters come later.

## The fixed example system

Use this throughout the lab — do *not* substitute your own system here. Keeping the system fixed lets your five-sentence reads be checked against a single expected shape.

> **`PolicyPal`** — an internal HR-policy Q&A assistant for a mid-sized tech company.
> Read-only RAG over the company's HR handbook (vacation, benefits, parental leave, code of conduct) using a hosted LLM. No decisions made automatically; responses are text only, surfaced in the employee intranet. Employees can ask free-text questions and receive an answer with citation links back to the handbook. No PII inputs are expected; the assistant does not have access to employee records.

## Sub-categories you will translate

Four sub-categories spanning MEASURE and MANAGE, chosen to exercise the full pattern:

1. **MEASURE 2.3** — AI system performance is evaluated in conditions similar to deployment.
2. **MEASURE 2.7** — AI system security and resilience are evaluated and documented.
3. **MEASURE 2.11** — Fairness and bias are evaluated.
4. **MANAGE 4.1** — Post-deployment AI system monitoring plans are implemented.

The short-form titles above paraphrase for brevity. Open each one in the Playbook and transcribe the *outcome statement verbatim* as sentence 1 when you fill the template.

## Steps

### Step 1 — Create your working directory and the template

```bash
mkdir -p mod101-lab01 && cd mod101-lab01
touch rmf-five-sentence-reads.md
```

Open `rmf-five-sentence-reads.md` and paste the following template. You will fill one block per sub-category.

```markdown
# PolicyPal — RMF Sub-Category Five-Sentence Reads

System: PolicyPal (internal HR-policy Q&A RAG assistant; see lab-01 brief)
Playbook version cited: airc.nist.gov/AI_RMF_Knowledge_Base/Playbook (retrieval date: YYYY-MM-DD)

## MEASURE 2.3 — <verbatim outcome statement here>
1. Says: <one sentence; verbatim outcome statement>
2. Means: <one sentence; what the outcome demands of PolicyPal specifically>
3. Test: <one sentence; tool / dataset / eval that runs the test>
4. Pass: <one sentence; threshold and where the threshold lives>
5. Evidence: <one sentence; artifact name and path>

## MEASURE 2.7 — <verbatim outcome statement here>
...

## MEASURE 2.11 — <verbatim outcome statement here>
...

## MANAGE 4.1 — <verbatim outcome statement here>
...
```

### Step 2 — Fill MEASURE 2.3 (performance in conditions similar to deployment)

Think through:

- *Means:* PolicyPal reads the HR handbook and returns guidance to employees. A deployment-shaped condition is a shadow eval on *real employee question traffic* (anonymised), not just a synthetic eval-set.
- *Test:* a shadow-eval job that replays a sample of production prompts against a candidate model and compares groundedness / retrieval-alignment to the offline eval baseline.
- *Pass:* the metric delta vs. the offline eval is within a budget defined in the risk register for PolicyPal (threshold *lives in the register*, not in a free-text doc — this is important).
- *Evidence:* a shadow-eval run report with config hash and model hash.

Write your block. Keep each of the five lines to one sentence.

**Expected output (shape — not the exact words to copy):**

```markdown
## MEASURE 2.3 — AI system performance is evaluated in conditions similar to deployment.
1. Says: AI system performance is evaluated in conditions similar to deployment.
2. Means: PolicyPal's groundedness and retrieval-alignment must be measured on production-shaped traffic replays, not only on an offline curated eval-set.
3. Test: a shadow-eval job that replays an anonymised sample of real employee prompts through the candidate model and compares groundedness and retrieval-alignment against the offline baseline.
4. Pass: the delta between shadow-eval and offline-eval metrics is within the budget defined for PolicyPal in `risk-register/2026/policypal.yaml`.
5. Evidence: `evidence/measure-2.3/<release-tag>/shadow-eval-report.json` with run config hash and model hash recorded.
```

### Step 3 — Checkpoint on MEASURE 2.3 before continuing

Check your block against these pass rules (same rules every block must pass):

- Sentence 1 is the outcome statement verbatim from the Playbook (no paraphrase in sentence 1).
- Sentence 2 names a *concrete* metric or test that would demonstrate the outcome for PolicyPal.
- Sentence 3 names the *tool, dataset, or eval* that runs the test — not "we evaluate."
- Sentence 4 cites a threshold *and the place the threshold is defined* (the risk register path, or an upstream policy document).
- Sentence 5 cites an artifact name *and a path* — not "we file a report."

If any of the five fails the rule, rewrite before moving on. Resist the urge to leave sentence 4 or 5 vague — the whole point of chapter 01 is that "TBD" sentences mean *open engineering work*.

### Step 4 — Fill MEASURE 2.7 (security and resilience)

Use chapter 01's minimal example as the template. For PolicyPal the shape is:

- *Means:* PolicyPal must not disclose other users' queries from the RAG store, must not comply with jailbreak attempts, and must not execute attacker-authored instructions embedded in HR handbook pages (indirect prompt injection via a tampered policy document).
- *Test:* a pinned LLM red-team suite covering direct prompt-injection, jailbreak, and indirect prompt-injection vectors (e.g., garak + PyRIT + an AgentDojo-shaped indirect suite).
- *Pass:* attack-success-rate ≤ target defined in the risk register.
- *Evidence:* signed suite report with version-pinned suite hash.

Write the block, then re-run the Step 3 checkpoint rules on it.

### Step 5 — Fill MEASURE 2.11 (fairness and bias)

This is a *scoping* exercise: PolicyPal is internal, read-only, over HR handbook content. Decide — *with a one-sentence reason* — whether MEASURE 2.11 is in scope. Chapter 01 and chapter 02 both say that out-of-scope rulings still require a reason.

Two acceptable outcomes:

- You rule it *in scope* because answers about parental leave, bereavement, or disability accommodation could systematically differ in quality across protected-group-relevant topics — in which case your block names a slice evaluation across HR-topic categories and a groundedness-parity metric.
- You rule it *out of scope*, with the reason being that PolicyPal makes no decisions and does not take protected-attribute inputs; the block then explicitly files the "out" ruling and names what would change the ruling (e.g., "if PolicyPal starts generating recommendations based on employee profile data, re-rule as in-scope").

Either outcome is acceptable; the checkpoint is that the ruling has a *reason*.

### Step 6 — Fill MANAGE 4.1 (post-deployment monitoring)

For PolicyPal the shape is:

- *Means:* a monitor on groundedness regression, a monitor on jailbreak / prompt-injection attempts, and a near-miss queue feeding the incident-candidate backlog (mod-109 wiring).
- *Test:* the monitoring pipeline itself — not a one-time eval. The "test" in sentence 3 is the daily monitor-pipeline run and the weekly risk-register review that consumes the signal.
- *Pass:* alert cadence and threshold per risk as defined in the register; any alert above severity S becomes an incident-candidate row.
- *Evidence:* the post-market surveillance report (also serves EU AI Act Article 72 — forward-reference to chapter 04).

Write the block.

### Step 7 — Final checkpoint

Re-read all four blocks top to bottom. Confirm:

- All four blocks follow the says / means / test / pass / evidence order.
- Every sentence 1 is verbatim from the Playbook.
- Every threshold cites *where* it is defined (never the block itself — thresholds live in the risk register).
- Every evidence sentence cites an artifact *path*.
- The MEASURE 2.11 block has a *reason* (whether ruled in or out).
- Nowhere do you have "TBD" or "the vendor covers it."

## Expected output (summary)

`mod101-lab01/rmf-five-sentence-reads.md` — four sub-category blocks, each exactly five single-sentence lines plus the verbatim outcome-statement header, totalling roughly one page.

## Checkpoint questions (self-check)

Answer each in your head. If any is "I don't know," re-read the relevant chapter-01 passage.

1. For every block, can you name the sub-category's *owner* — who signs off on the evidence? (Hint: look at the governance-council chapter in mod-112 for the typical answer; if your org doesn't have one, the AI risk council.)
2. If sentence 4 cites a threshold that is *not* in the risk register, what have you accidentally done? (You have created a shadow source of truth. Thresholds live in the register.)
3. For MEASURE 2.11, if you ruled "out," what would trigger a re-ruling? Write the trigger down. (This is the discipline chapter 02 teaches for the GenAI-Profile applicability sheet — same pattern applies here.)
4. Which of your four blocks would *also* file as evidence against EU AI Act Article 15 (forward-reference to chapter 04)? Likely MEASURE 2.7 — and that's the point. One artifact, multiple frameworks.

## Why this lab matters

Every exercise in this module (and most of mod-102 through mod-112) assumes you can do this translation fluently. Exercise 01 asks for ≥ 10 sub-categories in two hours; this lab makes sure you can do *one* in under ten minutes before you try ten in two hours.
