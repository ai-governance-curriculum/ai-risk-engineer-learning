# Lab 01 — NIST → ATLAS → OWASP Cross-Tag Row-Authoring Drill

**Duration:** 45 minutes · **Format:** guided practice, markdown authoring

## Objective

Internalise the chapter-01 pattern for authoring **one cross-tag row per NIST AI 100-2 e2025 category** on a fixed toy system — every column filled, every three-axis threat-model slot populated, every `covered`-style flag traced to a harm-model row (mod-102) and a register row (mod-103). The lab asks you to fill *six* rows (one per NIST category) end-to-end with checkpoints so that when exercise 01 asks for the full sheet, the row-authoring step is "add more rows" rather than "re-invent the shape."

Blank columns and untraceable rows are the two most common first-pass defects chapter 01 names. This lab drills the discipline that eliminates both.

## Prerequisites

- Chapter 01 read (specifically "The six categories in AI 100-2 e2025", "The threat model rubric AI 100-2 uses", and "The cross-tag table shape").
- Chapters 02–04 skimmed enough to recognise the concrete technique names (FGSM / PGD / C&W, label-flipping / BadNets, Tramèr / Knockoff Nets, Fredrikson, Shokri / LiRA, sponge examples).
- A terminal where you can create a working directory and author markdown files.
- No external access required; the system, the harm-model stubs, and the register stubs are all fixed below.

## The fixed example system

Use this system throughout the lab — do *not* substitute your own. Carrying a running-example anchor keeps the lab bounded to the cell-marking skill.

> **`SupportIntent`** — a fine-tuned **intent-classifier head** on an internal customer-support LLM assistant for a mid-sized SaaS company.
> Five-class classification on free-text customer messages: `{billing, cancellation, tech-support, account-access, general}`. BERT-base fine-tuned on a labelled corpus of ~120k past tickets; served from an internal inference API with per-caller auth. The classifier's label routes the message to the downstream handler team; the assistant's LLM then drafts a response using the routed handler's playbook. Free-text replies can be rated by the customer at the end of the session.
>
> Surfaces in scope for this lab:
> - **text inputs** — the customer's free-text message, which the classifier sees.
> - **inference API** — the internal endpoint; per-caller authenticated, returns the predicted label and a confidence score per class.
> - **training pipeline** — the labelled-ticket corpus and the fine-tuning pipeline that produces release candidates monthly.

Harm-model row stubs (fixed — do not add more):

| Harm-row id | One-line harm |
|---|---|
| `HM-SI-01` | Customer message intentionally crafted to route to the wrong handler team, causing SLA miss and customer-trust harm |
| `HM-SI-02` | Training-corpus contamination from a disgruntled labelling-vendor employee causes systematic mis-routing of a target customer segment |
| `HM-SI-03` | Competitor extracts a functional copy of the classifier to replicate SupportIntent's routing behaviour without licence |
| `HM-SI-04` | Adversary reconstructs training-ticket content from confidence outputs; training corpus contains customer PII |
| `HM-SI-05` | Adversary determines whether a specific customer's ticket was in the training set — privacy regression for named enterprise accounts |
| `HM-SI-06` | Attacker drives sustained high-cost inference traffic against the endpoint, degrading legitimate-caller latency and burning compute budget |

Register row stubs (fixed — do not add more):

| Register-row id | Scope line |
|---|---|
| `REG-SI-EVASION-01` | Adversarial-text evasion at inference time on the classifier head |
| `REG-SI-POISON-01` | Training-corpus / supply-chain poisoning of the classifier head |
| `REG-SI-EXTR-01` | API-surface extraction of the classifier head |
| `REG-SI-INV-01` | Training-ticket content reconstruction from classifier outputs |
| `REG-SI-MIA-01` | Membership inference against the training corpus |
| `REG-SI-AVAIL-01` | Availability / cost-harvesting attacks on the inference endpoint |

## Steps

### Step 1 — Create your working directory and the row template

```bash
mkdir -p mod105-lab01 && cd mod105-lab01
touch cross-tag-supportintent.md
```

Open `cross-tag-supportintent.md` and paste this scaffold. You will fill one row per NIST category, in order, using the running-system surfaces above.

```markdown
# SupportIntent — NIST → ATLAS → OWASP cross-tag sheet (lab 01)

System: SupportIntent (BERT-base intent classifier on a customer-support assistant; see lab brief)
Chapter reference: mod-105 ch 01 (cross-tag table shape, three-axis threat-model rubric)
Taxonomy pin: NIST AI 100-2 e2025 (authoritative)
Overlays: MITRE ATLAS; OWASP ML Top 10 (2023); OWASP LLM Top 10 (2025) — if dual-domain

## Rows

| Scenario id | NIST AI 100-2 category | Attack technique | Attacker knowledge | Attacker capability | Attacker goal | MITRE ATLAS tactic + technique id | OWASP ML Top 10 entry | OWASP LLM Top 10 entry (if applicable) | Harm-model row (mod-102) | Register row (mod-103) | Scope flag |
|---|---|---|---|---|---|---|---|---|---|---|---|

## Notes

(one-paragraph cover note explaining why the six NIST categories are *collectively* in scope for SupportIntent, or why a category is marked out-of-scope.)
```

The twelve columns map directly to chapter 01's cross-tag table. If you widen or rename columns, fix it before moving on — the chapter is explicit that the schema is the sheet.

### Step 2 — Author Row 1: evasion

Walk the columns in order. Target shape:

- **Scenario id:** `SI-EVAS-01`
- **NIST category:** `evasion`
- **Attack technique:** `TextFooler (word-swap + USE-cosine constraint)` — the SupportIntent head is a text classifier, so the evasion runner is TextAttack, not ART (chapter 02).
- **Knowledge:** `black-box` — the attacker queries the inference API, no weights.
- **Capability:** `active querier` — standard API access.
- **Goal:** `targeted` — route a billing complaint to `tech-support` instead of `billing`.
- **ATLAS:** `Reconnaissance + ML Attack Staging + Evade ML Model` (cite the ATLAS release snapshot; `<!-- needs-research: pin the current ATLAS technique ids at authoring time -->` is acceptable if you do not have the live matrix in front of you).
- **OWASP ML:** `Input Manipulation Attack`.
- **OWASP LLM:** `n/a` — the classifier head is not the generative LLM; evasion against the head is not an LLM-Top-10 scenario even though the assistant uses an LLM downstream.
- **Harm-row:** `HM-SI-01`.
- **Register-row:** `REG-SI-EVASION-01`.
- **Scope flag:** `in-scope-this-module` — runs in exercise 02 on this sheet's row.

Fill the row in the markdown table. Every column populated; no blanks.

### Step 3 — Author Row 2: poisoning

Target shape:

- **Scenario id:** `SI-POIS-01`
- **NIST category:** `poisoning`
- **Attack technique:** `targeted dirty-label poisoning via labelling vendor` — a disgruntled vendor employee flips labels on a named customer segment's tickets; chapter 03's targeted-data-poisoning family.
- **Knowledge:** `grey-box` — the attacker (insider at the labelling vendor) knows the corpus schema and the label set but not the model internals.
- **Capability:** `training-data contributor` — explicit per chapter 01's capability axis.
- **Goal:** `targeted` — mis-route the named segment's tickets.
- **ATLAS:** `Poison Training Data` (note the separate `ML Supply Chain Compromise` tactic covers the pretrained-checkpoint path, which is a *different* row — do not merge).
- **OWASP ML:** `Data Poisoning Attack`.
- **OWASP LLM:** `n/a`.
- **Harm-row:** `HM-SI-02`.
- **Register-row:** `REG-SI-POISON-01`.
- **Scope flag:** `in-scope-this-module`.

Fill the row.

### Step 4 — Author Row 3: extraction

Target shape:

- **Scenario id:** `SI-EXTR-01`
- **NIST category:** `extraction` (model stealing)
- **Attack technique:** `Tramèr-style query-based extraction` with confidence-score responses; optionally note `Knockoff Nets` as an alternative-technique cell in the Notes column.
- **Knowledge:** `black-box`.
- **Capability:** `active querier` — needs query access.
- **Goal:** `information extraction` (per chapter 01's goal axis; the surrogate model is the "information" extracted).
- **ATLAS:** `Reconnaissance + ML Model Access + Exfiltration via ML Inference API`.
- **OWASP ML:** `Model Theft`.
- **OWASP LLM:** `n/a`.
- **Harm-row:** `HM-SI-03`.
- **Register-row:** `REG-SI-EXTR-01`.
- **Scope flag:** `in-scope-this-module` (runs in exercise 04).

Fill the row.

### Step 5 — Author Row 4: inversion

Target shape:

- **Scenario id:** `SI-INV-01`
- **NIST category:** `inversion`
- **Attack technique:** `Fredrikson model inversion with confidence information` — the API returns per-class confidences, which is the input the Fredrikson attack exploits.
- **Knowledge:** `black-box (with confidences)` — this is the hinge of the attack; if confidences are redacted, the attack weakens.
- **Capability:** `active querier`.
- **Goal:** `information extraction` — recovering training-ticket attributes.
- **ATLAS:** `ML Model Access + Exfiltration via ML Inference API`.
- **OWASP ML:** `Model Inversion Attack`.
- **OWASP LLM:** `n/a`.
- **Harm-row:** `HM-SI-04`.
- **Register-row:** `REG-SI-INV-01`.
- **Scope flag:** `in-scope-this-module` with `handoff:mod-107` note (formal privacy-metric depth sits at mod-107 per chapter 04).

Fill the row.

### Step 6 — Author Row 5: membership inference

Target shape:

- **Scenario id:** `SI-MIA-01`
- **NIST category:** `membership inference`
- **Attack technique:** `Shokri shadow-model MIA` as the baseline-advantage attack, with a note that `LiRA` at low FPR is the deeper framing owned at mod-107 (chapter 04).
- **Knowledge:** `black-box`.
- **Capability:** `active querier` — plus the ability to train shadow models on similar distribution.
- **Goal:** `information extraction`.
- **ATLAS:** `ML Model Access + Exfiltration via ML Inference API`.
- **OWASP ML:** `Membership Inference Attack`.
- **OWASP LLM:** `n/a`.
- **Harm-row:** `HM-SI-05`.
- **Register-row:** `REG-SI-MIA-01`.
- **Scope flag:** `in-scope-this-module` with `handoff:mod-107` — baseline advantage here; LiRA calibration at mod-107.

Fill the row.

### Step 7 — Author Row 6: availability

Target shape:

- **Scenario id:** `SI-AVAIL-01`
- **NIST category:** `availability`
- **Attack technique:** `sponge examples / energy-latency attack` on the classifier head — long, pathologically-structured messages that drive per-inference cost up.
- **Knowledge:** `black-box` (or `grey-box` if the attacker has tokenizer knowledge; pick one and note).
- **Capability:** `active querier`.
- **Goal:** `availability` — degrade legitimate-caller latency and burn compute.
- **ATLAS:** `Impact — Denial of ML Service / Cost Harvesting`.
- **OWASP ML:** `Denial of Service (adapted)` — chapter 01 notes availability is less explicitly ranked in ML Top 10; cite the general availability entry and note the mapping.
- **OWASP LLM:** `n/a` on the head; if the assistant's LLM is also in scope of the attack, dual-tag with the LLM Top 10 availability-adjacent entry.
- **Harm-row:** `HM-SI-06`.
- **Register-row:** `REG-SI-AVAIL-01`.
- **Scope flag:** `handoff:sec-L35` — per chapter 06, inference-edge rate-limit enforcement and gateway-side cost controls sit at the peer track. This module *specifies* the rate value and the metric; the peer *enforces*.

Fill the row.

### Step 8 — Checkpoint on completeness and traces

Re-read the sheet. Confirm each of these holds for every row:

- **Every column is populated.** Blank columns are the audit finding chapter 01 names by name. If you have "white-box" in the Knowledge column with Capability and Goal blank, you have failed the three-axis rubric (quiz 01 question 2); fill both.
- **Every row traces to at least one harm-model row (mod-102) and at least one register row (mod-103).** A row with no harm-row anchor is either miscategorised or reveals a harm-model gap; a row with no register anchor means the register does not know about the risk. Chapter 01 is explicit: no orphan rows.
- **Scope flags are specific.** `in-scope-this-module`, `handoff:mod-107`, `handoff:mod-108`, `handoff:sec-L35`, or `deferred:<reason-or-ticket>`. Do not leave the flag blank or write `tbd`.
- **ATLAS and OWASP columns are not collapsed into the NIST column.** The three vocabularies answer different review questions (quiz 01 question 3); each gets its own column. If you merged them, split them.
- **`<!-- needs-research: ... -->` is used honestly** for ATLAS technique ids you cannot pin without the live matrix, or for OWASP version numbers whose precise revision you do not have. Do *not* invent ids; the chapters model this discipline themselves.

If any check fails on any row, rewrite before moving on.

### Step 9 — Write the one-paragraph cover note

Convert the sheet to English in two to four sentences. Target shape:

> "SupportIntent's cross-tag sheet carries one row per NIST AI 100-2 e2025 category — evasion, poisoning, extraction, inversion, MIA, availability — because every category has at least one plausible attack path on SupportIntent's surface set (text inputs, confidence-returning inference API, monthly fine-tuning pipeline). The classifier head is not a generative LLM, so OWASP LLM Top 10 is `n/a` on every row; the generative assistant downstream of the head is a mod-104 concern. Three rows flag handoffs: inversion and MIA both carry `handoff:mod-107` for formal privacy accounting; availability carries `handoff:sec-L35` for inference-edge enforcement of rate and cost controls. No category is `deferred`; this sheet is the input to exercise 02's evasion panel and exercises 03 and 04's engineered attacks."

That paragraph is the sheet's living voice; it is what a reviewer reads cold in place of staring at a wide six-row table.

### Step 10 — Final checkpoint

Re-read the sheet and the paragraph. Confirm:

- All six rows are present (one per NIST category).
- All twelve columns are populated on every row.
- Every row traces to a harm-model row *and* a register row.
- Every row has a scope flag and the flag is specific.
- Three overlays are visible as separate columns (NIST, ATLAS, OWASP ML; plus OWASP LLM where dual-domain).
- `<!-- needs-research: ... -->` appears only where the authoring context genuinely lacks the identifier; invented ids are nowhere.
- The cover note argues the six categories *collectively* and names the two handoff categories and the escalated row.

## Expected output

`mod105-lab01/cross-tag-supportintent.md` — one populated six-row sheet (all twelve columns per row), with every row tracing to both a harm-model row and a register row, every row carrying a scope flag, and a short cover paragraph that argues the six-category coverage. Roughly one page.

## Checkpoint questions (self-check)

1. On the extraction row, you wrote "black-box" under Knowledge and left Capability and Goal blank because "black-box says what the attacker can do." A reviewer flags it. Is the reviewer right? (Yes — chapter 01's rubric and quiz 01 question 2: knowledge, capability, and goal are three distinct axes. "Black-box" names only the knowledge axis. The attacker's capability — active querier? training-data contributor? supply-chain participant? — and goal — targeted? untargeted? information extraction? availability? — are separate columns; leaving them blank makes the attack unreviewable.)
2. On the availability row, you considered `handoff:mod-108` (guardrail engineering owns runtime detectors) instead of `handoff:sec-L35`. Per chapter 06, which is right? (`handoff:sec-L35` for the inference-edge rate-limit *enforcement* and gateway-side cost controls. Mod-108 owns the guardrail *catalogue* for runtime detectors like STRIP; it does not own API-gateway configuration. Chapter 06 pins gateway and WAF at the peer at level 35. Either handoff can be *additionally* noted in the row's Notes column if both are triggered, but the primary enforcement handoff for availability at the API is sec-L35.)
3. A junior engineer argues that the inversion row and the MIA row can share one cross-tag row because "they are both API-surface privacy attacks." Per chapter 01, what specifically breaks? (NIST AI 100-2 e2025 separates `inversion` and `membership inference` as distinct categories with distinct attacker objectives — inversion reconstructs *input attributes*, MIA decides *record membership*. Collapsing them hides one of the two categories from the auditor's six-category coverage check; the chapter's partition is by objective, not by surface. The two rows also feed different mod-107 workstreams downstream; a single row obscures that too.)
4. The sheet has six rows and all six are `in-scope-this-module`. Is that good? (Probably not for most systems — real sheets almost always carry at least one handoff or deferred row because the module's scope ends before some depths. SupportIntent's sheet as written has two `handoff:mod-107` rows and one `handoff:sec-L35` row for exactly this reason. An all-`in-scope` sheet suggests the handoffs have been under-noted, which is exactly the orphan-owned failure mode chapter 06 names.)

## Why this lab matters

Exercise 01 asks you to produce the full NIST → ATLAS → OWASP cross-tag sheet for the running system in two hours, with every row tracing to mod-102 and mod-103. If the row-authoring step consumes most of that budget because you re-derive the three-axis rubric and the overlay-column discipline from scratch, the exercise blows its budget before the harder cells (the "is this attack genuinely in scope for *our* surfaces?" cells) get authored. This lab drills the atomic skill — six rows end-to-end, every column filled, every row traced, every scope flag specific — until it is routine, so exercise 01 is *filling the sheet with real scenario-scoping decisions* rather than *re-inventing the row shape*. Labs 02 and 03 drill the evasion-eval output-shape and the poisoning-defence measurement-matrix shape respectively; together they make exercises 01–05 tractable within their nominal budgets.
