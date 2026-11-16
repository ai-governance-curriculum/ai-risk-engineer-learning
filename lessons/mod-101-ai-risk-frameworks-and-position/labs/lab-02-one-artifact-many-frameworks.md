# Lab 02 — One Artifact, Many Frameworks

**Duration:** 60 minutes · **Format:** guided practice, markdown authoring

## Objective

File a *single* engineering artifact — a signed LLM red-team report — against five frameworks at once: NIST AI RMF, ISO/IEC 42001, ISO/IEC 23894, ISO/IEC 25059, EU AI Act Article 15, and the SR 11-7 "effective challenge" column. Chapter 03's artifact-to-standard table says this is the norm, not the exception; this lab makes you *do* it until it feels routine. The deliverable is a single-page cross-walk file an auditor can read without a translation layer.

## Prerequisites

- Chapters 01, 03, 04, and 05 read.
- Lab 01 completed (you are fluent in the five-sentence read).
- A terminal for creating files; no code runtime required.
- The following open in tabs for reference:
  - the NIST AI RMF Playbook entry for MEASURE 2.7,
  - the ISO/IEC 42001 landing page on `iso.org` (for the publicly-visible clause and Annex A control *titles* — standard text is paywalled, so you will *paraphrase* and cite the control identifier only),
  - the ISO/IEC 23894 landing page (Annex A risk-source names),
  - the ISO/IEC 25059 landing page (quality-characteristic names),
  - the EU AI Act consolidated text on `eur-lex.europa.eu`, Article 15,
  - SR 11-7 (`federalreserve.gov/supervisionreg/srletters/sr1107.htm`).

## The fixed input artifact

Assume this artifact already exists (lab-01 established the shape; you don't need to generate the report — you file it):

> **Artifact:** `evidence/measure-2.7/2026.04.r1/redteam-report.json`
> **Content:** LLM red-team suite results against PolicyPal (lab-01 system). Covers direct prompt injection, jailbreak, and indirect prompt injection (attacker-authored instructions embedded in a tampered HR handbook page). Suite: garak + PyRIT + AgentDojo-shaped indirect suite, all pinned to specific commit hashes. Signed by the release pipeline. Overall attack-success-rate 3.2 %, under the risk-register target of 5 %.
> **Signer:** release pipeline (CI job identity); counter-signed by the AI risk council on acceptance.

Your job is *not* to produce the report. Your job is to produce the **cross-walk file** that tells six different reviewers where in their framework this artifact lands.

## Steps

### Step 1 — Create the cross-walk file

```bash
mkdir -p mod101-lab02 && cd mod101-lab02
touch redteam-report-crosswalk.md
```

Paste this skeleton into `redteam-report-crosswalk.md`:

```markdown
# Cross-walk — PolicyPal LLM red-team report, release 2026.04.r1

Artifact path: evidence/measure-2.7/2026.04.r1/redteam-report.json
Signed by: release pipeline (CI job identity); counter-signed by AI risk council
Suite versions: garak <pin>; PyRIT <pin>; AgentDojo-shaped indirect suite <pin>
Headline result: attack-success-rate 3.2 % vs. target 5 % (risk-register row RR-PP-07)

## Framework file-against map

| Framework | Clause / article / sub-category | How this artifact discharges it | Reviewer who consumes it |
|---|---|---|---|
| NIST AI RMF 1.0 | ... | ... | ... |
| ISO/IEC 42001:2023 | ... | ... | ... |
| ISO/IEC 23894:2023 | ... | ... | ... |
| ISO/IEC 25059 | ... | ... | ... |
| EU AI Act (Reg. 2024/1689) | ... | ... | ... |
| US Federal Reserve SR 11-7 | ... | ... | ... |
```

### Step 2 — Fill the NIST AI RMF row

The sub-category to cite is **MEASURE 2.7** (AI system security and resilience evaluated and documented). Chapter 01 walks the five-sentence read for exactly this sub-category on a customer-support assistant — paraphrase it for PolicyPal.

Fill the row with:

- *Clause / sub-category:* MEASURE 2.7 (quote the outcome statement in a trailing sentence).
- *How this artifact discharges it:* one sentence naming the suite, the attack-success-rate metric, the pass rule ("≤ target per risk register"), and the version pinning.
- *Reviewer:* AI risk council on acceptance; `ai-evaluation-engineer` (L35) on release-assurance review.

### Step 3 — Fill the ISO/IEC 42001 row

Chapter 03 says that for 42001 you file control-implementation evidence against Annex A reference controls the SoA has marked "applicable." You do *not* paste the ISO control text (paywalled); you cite the control identifier only and paraphrase the control objective.

Use a *paraphrased* control objective in the row — the operational / evaluation-oriented control that an organisation's SoA would most commonly mark applicable to cover adversarial-robustness evaluation. Mark the row as `paraphrased — control text not reproduced` if you cannot cite the exact control identifier.

The cell should include:

- the clause bucket (Annex A operational / evaluation controls — chapter 03's table), and
- the clause-9 performance-evaluation evidence line (because a monitor / eval produces this, and clause 9 asks for performance-evaluation evidence).

Reviewer: the ISO/IEC 42001 lead auditor during the external audit cycle; internally, `senior-ai-governance-architect` (L50) reviews inclusion in the Statement of Applicability.

### Step 4 — Fill the ISO/IEC 23894 row

Chapter 03 says 23894 Annex A is a *catalogue of risk sources*. For a red-team report, the risk sources discharged are *security* and *safety* (and *autonomy* if the system has agentic tool-use — PolicyPal does not, so skip autonomy).

Fill the row with:

- *Clause / annex:* 23894 Annex A risk sources: security, safety.
- *How this artifact discharges it:* this artifact is *analysis + treatment evidence* for the security risk source; the residual-risk statement after this control is "≤ 5 % attack-success-rate on the pinned suite."
- *Reviewer:* risk register owner (you); AI risk council sign-off.

### Step 5 — Fill the ISO/IEC 25059 row

25059 is a *vocabulary* tool, not an audited standard. Chapter 03's row for red-team reports says 25059 reads this artifact as evidence of the **robustness** characteristic.

Fill the row with:

- *Clause / characteristic:* 25059 quality characteristic: robustness.
- *How this artifact discharges it:* the attack-success-rate under adversarial / injection conditions is the measurable sub-characteristic the model-card robustness section references.
- *Reviewer:* `senior-ai-governance-architect` (L50) — they use 25059 names to reconcile across the org's control library.

### Step 6 — Fill the EU AI Act row

Chapter 04 says Article 15 is where all your evaluation-engineering work lands. The cybersecurity sub-obligation of Article 15 is the one this artifact discharges.

Fill the row with:

- *Article:* Article 15 (accuracy / robustness / cybersecurity). The *cybersecurity* sub-obligation is the specific one (resilience against prompt injection, model extraction, jailbreak).
- *How this artifact discharges it:* this artifact is one of the inputs to the Article 15 traceability table — the red-team report + suite version + attack-success-rate + risk-register target. The accuracy declaration and the robustness report are separate inputs.
- *Reviewer:* `ai-evaluation-engineer` (L35) packages this for external conformity-assessment review; a notified body (if applicable) consumes it.

### Step 7 — Fill the SR 11-7 row

Chapter 05's crosswalk table maps the SR 11-7 concept **effective challenge** to the red-team suite output. "Effective challenge" is SR 11-7's organised, active questioning of the model.

Fill the row with:

- *Clause / concept:* SR 11-7 "effective challenge" + ongoing-monitoring pillar.
- *How this artifact discharges it:* the red-team suite is the *organised, active questioning* of the model under adversarial conditions; a pinned, repeatable suite is the artifact that lets a validator defend "we challenged the model."
- *Reviewer:* the independent model-validation function (if PolicyPal were inside an FI subject to SR 11-7); since PolicyPal is internal HR, this row is included *for the shape*, with a note: "SR 11-7 not applicable to PolicyPal — row included to show the crosswalk shape only."

### Step 8 — Add the "two things that are *not* the same" footnote

Chapter 05 names three things that genuinely differ across regimes: *sign-off routing*, *independence rules*, and *named jurisdictional artifacts*. At the bottom of your cross-walk file, add a short footnote listing which of these differ for this single artifact across the six frameworks:

- Sign-off routing: AI risk council (internal) vs. notified body (Article 15 high-risk) vs. independent validation function (SR 11-7) — same evidence, different signer.
- Independence: this artifact was produced by the release pipeline, not by an independent red-team. If PolicyPal moved into SR 11-7 or Article 15 notified-body scope, *independence of the red-team* would become a gating requirement — flag as a follow-up.
- Named artifacts: no Article 61 registration entry attached (PolicyPal is internal, not placed on the EU market); if that changes, the cross-walk grows a row.

### Step 9 — Final checkpoint

Verify:

- Every row has a *clause / article identifier*, a *one-sentence file-against*, and a *reviewer*.
- No row pastes paywalled standard text verbatim; ISO rows are paraphrased and marked.
- The headline-result line at the top of the file lets a reviewer skim without opening the underlying JSON.
- The risk-register row (RR-PP-07 in the skeleton) is cited at the top so every "pass" claim in the cross-walk is traceable back to one source of truth for the threshold.
- The footnote on sign-off / independence / named-artifacts is present and names at least one follow-up.

## Expected output

`mod101-lab02/redteam-report-crosswalk.md` — a single-page markdown file with:

- a four-column table filled across six rows (NIST, 42001, 23894, 25059, EU AI Act, SR 11-7),
- a top-of-file manifest (artifact path, signer, suite pins, headline result, risk-register row),
- a footnote on sign-off / independence / named-artifact differences.

## Checkpoint questions (self-check)

1. If the SoA deletes the Annex A control you cited, does this artifact still have an ISO/IEC 42001 home? (Only if you re-cite under a remaining applicable control; otherwise the artifact is still produced but no longer files against 42001 — which is a governance decision, not a technical one.)
2. If the organisation later becomes subject to SR 11-7 (acquisition by a bank, say), what changes in the file-against table? (The SR 11-7 row goes from *shape-only* to *actually consumed*; the independence follow-up footnote becomes a gating action.)
3. If the suite version pin drops off the top-of-file manifest, what breaks? (The entire cross-walk — a red-team result without a suite version is not reproducible, and an auditor will reject it. Pins are load-bearing.)
4. Which chapter's lesson most directly explains why this cross-walk is a single file instead of six separate files? (Chapter 03's "the point of the table is that a single engineering artifact usually files against *several* standards at once. That is on purpose.")

## Why this lab matters

Governance work loses a surprising amount of time to *re-filing the same evidence*. If you train the cross-walk reflex now, mod-112's governance-council briefings will take an hour instead of a day. Exercises 03 and 04 produce the clause-level and article-level maps; this lab produces the *artifact-level* dual, which is the shape that actually sits in an evidence repo.
