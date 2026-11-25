# Lab 01 — MRM Packet Shape Authoring Drill (SR 11-7 Four Pillars + Heading-to-Artefact Map)

**Duration:** 45 minutes · **Format:** guided practice, markdown authoring on a fixed fabricated LLM application

## Objective

Internalise the chapter-01 SR 11-7 / OCC 2011-12 **MRM packet shape** — the twelve-heading table of contents, the four-pillar coverage rubric (independence, conceptual soundness, ongoing monitoring, effective challenge), the heading-to-artefact map, the independent-validation-memo disposition line, and the gap list — by authoring the packet `README.md` for a fixed fabricated running system (`HelpDesk`). The lab drills the artefact shape chapter 01 pins so that when **exercise 01** produces the real MRM packet for your own primary running system, the engineering budget goes into which evidence files which heading and which pillar is actually evidenced — not into inventing the packet format.

The chapter-01 anti-patterns the shape forecloses: the **"we already do all of this"** submission that cannot cross-reference principle to artefact; the **"giant AI evaluation report organised by tool"** that has no slot in the MRM document taxonomy; the **"Independence Attestation" document** that is a label rather than a practice; and the **blank cell** that the auditor reads first.

## Prerequisites

- Chapter 01 read — specifically "SR 11-7 in ten sentences" (the parent shape; silent on technique, prescriptive on shape), "The four pillars, translated to AI" (independence as a practice; conceptual soundness as harm-model mapping; ongoing monitoring as mod-109 wiring; effective challenge as disagreement-in-the-record), "MRM-shaped validation documentation for an ML/AI model" (the twelve-heading canonical TOC and the AI engineering artefact that fills each), and "Where AI-specific work files against MRM headings" (the mod-101-through-mod-112 crosswalk).
- Chapter 02 skimmed — the third-party components heading reserves a slot for the SR 23-4 vendor file.
- Chapter 04 skimmed — the data heading reserves slots for the Data Cards and the ML-BOM.
- A terminal where you can create a working directory and author markdown files.
- No external access or compute required; the system brief, the register row stubs, and the artefact pointers are fixed below.

## The fixed example system

Use this system throughout the lab — do *not* substitute your own. Carrying a fixed scenario keeps the lab bounded to the packet-shape authoring skill. The system is deliberately aligned with the customer-support RAG assistant the chapters use as the running example and matches the system `HelpDesk` carried through the mod-109 labs.

> **`HelpDesk`** — a customer-support RAG assistant for a mid-size B2B SaaS company. Serves English-language customers of a billing-and-provisioning product. Assembles context from a knowledge-base store (product docs + internal runbooks) and from the authenticated caller's own recent ticket history. Backed by an in-house fine-tune of an instruction-tuned LLM (the fine-tune is on internal support transcripts) behind a FastAPI service wrapping a LangGraph agent. Two production tool calls: `escalate_ticket(destination, summary)` and `lookup_account(account_id)`.
>
> **US regulatory context:** The sponsoring organisation is a US bank subsidiary of the SaaS company's parent — therefore SR 11-7 / OCC 2011-12 is the parent MRM shape, SR 23-4 is the shape for the foundation-model API the fine-tune depends on, and FDA GMLP + PCCP are **not** applicable (no medical device). The change-management stub will point at the FI's model-change policy, not at a PCCP.
>
> **Primary third-party dependency:** `FoundationCo-LLM-3.5` — the base model the in-house fine-tune is built on, consumed via a vendor API that also serves the fine-tune at inference time. Exercise 02 will populate the SR 23-4 vendor file for this dependency.

### Fixed register row stubs (do not add more)

Every MRM-packet citation points at a register row id that exists in this stub set. The register-side schema is from mod-103; the fields referenced below are `inherent`, `residual`, `control_defeated`, `evidence_pointer`.

| Register row id | Scope line |
|---|---|
| `HR-CS-INJ-01` | Prompt-injection control credit — defence in depth across input filter, retrieval filter, and tool-invocation policy. |
| `HR-CS-PII-04` | PII egress control credit — Presidio at ingest / retrieval / output and per-caller entitlement; absolute-rule row. |
| `HR-CS-FAIR-08` | Per-slice quality parity on the `tenant_tier` slice (standard vs enterprise); fairness-tagged row; mod-106 seam. |
| `HR-CS-QUAL-03` | Quality regression on the composite (faithfulness + coverage + safe-refusal) metric. |
| `HR-CS-DRIFT-02` | Covariate drift on top-20 input features and on prompt-embedding distribution. |
| `HR-CS-VEND-11` | Vendor silent-swap risk on the `FoundationCo-LLM-3.5` base model — residual credited to the own-signal monitors and the pinned-version clause; exercise-02 vendor file is the evidence anchor. |
| `HR-CS-PROV-12` | Supply-chain tampering risk on the fine-tune weights — residual credited to signing + scanning gate; exercise 05-06 are the evidence anchors. |

### Fixed artefact pointers (do not revise)

Use these artefact identifiers verbatim; they match the exercise-01 reserved slots and the mod-101-through-mod-112 crosswalk.

| Pointer | Source |
|---|---|
| `mod-102/harm-model/intake-INT-HD-001.md` | Harm model for HelpDesk (mod-102) |
| `mod-103/register/HR-CS-*.md` | Register rows (mod-103) |
| `mod-104/red-team/suite-hd-v1.4/report-2026-10.md` | Red-team suite output (mod-104) |
| `mod-105/adversarial/adv-hd-v0.7/report-2026-10.md` | Adversarial evaluation (mod-105) |
| `mod-106/fairness/hd-slice-tenant-tier/report-2026-10.md` | Fairness evaluation on `tenant_tier` (mod-106) |
| `mod-107/privacy/hd-mia-pii-v0.3/report-2026-10.md` | Privacy evaluation (mod-107) |
| `mod-108/guardrails/catalogue-hd-v2.1.md` | Guardrail catalogue (mod-108) |
| `mod-109/monitors/helpdesk-signal-contract-v0.2.md` | Monitor spec (mod-109; the lab-01 output of the prior module) |
| `mrm-packet/HelpDesk/v0.1/11-third-party-components/FoundationCo-LLM-3.5/` | Reserved for the exercise-02 vendor file |
| `provenance/helpdesk-finetune/v1.3.0/` | Reserved for the exercise-04 provenance package |
| `gates/registry-ingestion-policy.rego` | Reserved for the exercise-05/06 gate policy |
| `mod-112/risk-council/minutes-2026-Q3.md` | Governance sign-off minutes (mod-112) |

### Fixed validator identity

The *independent validator* for this lab is **`L35-eval-eng-alex` (role: evaluation engineer, peer level L35)** — explicitly not the developer; they run the suite under the validation-owned CI service account `svc-helpdesk-validation` and sign evidence from a validation-controlled artefact store.

## Steps

### Step 1 — Create your working directory and the packet README scaffold

```bash
mkdir -p mod110-lab01/mrm-packet/HelpDesk/v0.1 && cd mod110-lab01/mrm-packet/HelpDesk/v0.1
touch README.md
```

Open `README.md` and paste this scaffold. You will fill the sections in order.

```markdown
# HelpDesk — MRM validation packet (lab 01)

System: HelpDesk (customer-support RAG assistant; see lab brief)
Packet version: v0.1
Register version bound to: mod-103 register v2026.11
Chapter references: mod-110 ch 01 (SR 11-7 / OCC 2011-12 parent shape; four pillars; MRM packet TOC; AI-specific files-against-headings map)
Independent validator: L35-eval-eng-alex

## 1. Packet summary

(one paragraph: system, packet version, register version, regulatory context, validator)

## 2. Four-pillar coverage table

(chapter 01 §"The four pillars" — one row per pillar; each row cites the specific file(s) in the packet that evidence it and the practice that makes it credible)

## 3. Heading-to-artefact map

(chapter 01 §"MRM-shaped validation documentation for an ML/AI model" — one row per heading in the twelve-heading TOC; each row cites the file path inside this packet and the upstream mod-10N artefact it consumes; empty cells appear in the gap list)

## 4. Validator memo disposition

(one-paragraph excerpt from 07-independent-validation-memo.md — scope, pillar-by-pillar assessment, disposition line)

## 5. Gap list

(table: MRM heading × missing artefact × owner × target date; every gap has a corresponding mod-103 register row id)

## 6. Open items

(what is still draft; what the exercise-01 run picks up on the real system)
```

### Step 2 — Fill Section 1 (Packet summary)

Pin the system, the packet version, the register binding, the regulatory context (SR 11-7 applicable; SR 23-4 applicable for the vendor file; FDA GMLP not applicable), and the validator identity. Chapter 01 §"MRM-shaped validation documentation for an ML/AI model" names the slots.

```markdown
## 1. Packet summary

HelpDesk v1.3.0 (fine-tune of `FoundationCo-LLM-3.5` on internal support transcripts) packaged for US bank MRM review under SR 11-7 / OCC 2011-12. Packet version v0.1 bound to mod-103 register v2026.11. Third-party-components heading reserved for the exercise-02 SR 23-4 vendor file on `FoundationCo-LLM-3.5`. FDA GMLP / PCCP not applicable (not a medical device); change-management heading points at the FI's model-change policy. Independent validation authored by `L35-eval-eng-alex` under the validation-owned CI service account `svc-helpdesk-validation`.
```

Reading check: five concrete pointers; every value is a specific identifier (packet version, register binding, vendor name, validator identity) rather than "TBD"; regulatory context is named explicitly (SR 11-7 applicable, FDA not applicable).

### Step 3 — Fill Section 2 (Four-pillar coverage table)

Chapter 01 §"The four pillars, translated to AI" names the four pillars and the engineering-visible evidence for each. Each row cites the specific file(s) that evidence the pillar *and* the practice that makes the evidence credible — the pillar is evidenced by the *practice*, not by a document whose title claims it.

```markdown
## 2. Four-pillar coverage table

| Pillar | File(s) in this packet | Practice that makes it credible |
|---|---|---|
| Independence of validation from development | `07-independent-validation-memo.md` (signed by `L35-eval-eng-alex`); `06-development-testing/*` (suite definitions owned by validation; mod-104/105/106/107 reports written from the validation-owned CI service account `svc-helpdesk-validation`) | The developer did **not** author the red-team suite definitions or the validation memo. The CI that runs the suite is a validation-owned service account; the artefact store is validation-controlled; the pass rule is set by validation. Independence is a **practice**, not a document title. |
| Conceptual soundness | `04-methodology-and-conceptual-soundness.md` (design rationale cites the mod-102 harm-model rows the design targets); `03-data.md` (Datasheet / Data Card evidence of training-corpus representativeness); `05-assumptions-and-limitations.md` (uncertainty columns from mod-102 cross-referenced); `06-development-testing/fairness-eval-report.md` (metric-to-harm mapping) | Every design choice in the methodology file traces to a specific mod-102 harm-model row it targets; fairness, privacy, and robustness evaluations are explicitly aimed at the failure modes the harm model names, not at generic benchmarks. |
| Ongoing monitoring (incl. outcome analysis / back-testing) | `09-ongoing-monitoring-plan.md` (mod-109 signal contract v0.2 — the lab-01 output of mod-109 — with per-row cadence, threshold, routing, register effect, evidence pointer) | The monitoring file **is** the mod-109 signal contract (named metric, threshold, cadence, routing, action-on-fire, evidence pointer per row) — not a screenshot of Grafana. Live monitoring is what turns a point-in-time validation into an ongoing validation; a model without live monitoring cannot be credited as "in scope of ongoing validation." |
| Effective challenge | `06-development-testing/red-team-report.md` (fixtures contributed by the eval-engineer peer that the model team did **not** write); `02-tiering-and-risk-rating.md` (register rows surfaced by validation, not volunteered by the developer — e.g., `HR-CS-VEND-11`, `HR-CS-PROV-12`); mod-111 post-mortem "what did we miss" section owned by `L35-eval-eng-alex` | Disagreement is present in the record: the suite includes fixtures the developer did not author; the register includes rows the developer would not have volunteered; the post-mortem has a non-developer owner on "what did we miss." If every artefact was written by the developer, the pillar fails regardless of memo wording. |
```

Reading check: four rows; every pillar cites at least one file by path; every row's practice column names a specific engineering-visible sign (CI service account identity, harm-row cross-reference, signal-contract row shape, fixture authorship). No pillar is evidenced by "we have a document called X" — every pillar is evidenced by *what the document is*.

### Step 4 — Fill Section 3 (Heading-to-artefact map)

Chapter 01 §"MRM-shaped validation documentation for an ML/AI model" names the twelve headings; §"Where AI-specific work files against MRM headings" names the mod-10N artefact each consumes. Fill one row per heading.

```markdown
## 3. Heading-to-artefact map

| MRM heading | File path in this packet | Upstream artefact consumed |
|---|---|---|
| Model description and purpose | `01-model-description-and-purpose.md` | `provenance/helpdesk-finetune/v1.3.0/model-card.md` (chapter 04; reserved for exercise 04) + mod-101 intended-use statement |
| Tiering and risk rating | `02-tiering-and-risk-rating.md` | `mod-103/register/HR-CS-*.md` — specifically `HR-CS-INJ-01`, `HR-CS-PII-04`, `HR-CS-FAIR-08`, `HR-CS-QUAL-03`, `HR-CS-DRIFT-02`, `HR-CS-VEND-11`, `HR-CS-PROV-12` with inherent / residual / control-defeated scores |
| Data | `03-data.md` | `provenance/helpdesk-finetune/v1.3.0/datasets/*.datacard.yaml` (chapter 04; reserved for exercise 04) — Data Card per training / fine-tuning / evaluation / retrieval corpus |
| Methodology and conceptual soundness | `04-methodology-and-conceptual-soundness.md` | `mod-102/harm-model/intake-INT-HD-001.md` rows the design targets; method write-up |
| Assumptions and limitations | `05-assumptions-and-limitations.md` | mod-102 uncertainty columns; mod-105 robustness-assumption notes |
| Development testing | `06-development-testing/{red-team,adversarial,fairness,privacy}-eval-report.md` | `mod-104/red-team/suite-hd-v1.4/report-2026-10.md`; `mod-105/adversarial/adv-hd-v0.7/report-2026-10.md`; `mod-106/fairness/hd-slice-tenant-tier/report-2026-10.md`; `mod-107/privacy/hd-mia-pii-v0.3/report-2026-10.md` |
| Independent validation | `07-independent-validation-memo.md` (signed by `L35-eval-eng-alex`) | Validator reads all of §06 and §09 and produces a pillar-by-pillar disposition |
| Implementation testing | `08-implementation-testing.md` | `mod-108/guardrails/catalogue-hd-v2.1.md` guardrail integration tests + canary results; signing/scanning gate pointer at `gates/registry-ingestion-policy.rego` (reserved for exercises 05-06) |
| Ongoing monitoring plan | `09-ongoing-monitoring-plan.md` | `mod-109/monitors/helpdesk-signal-contract-v0.2.md` — the fifteen-column signal contract |
| Change management | `10-change-management.md` | FI's model-change policy (FDA PCCP not applicable for HelpDesk); points to the FI's internal `change-policy-v3.md` |
| Third-party components | `11-third-party-components/FoundationCo-LLM-3.5/` | Reserved for exercise-02 vendor file; chapter 02 lifecycle sub-files |
| Governance sign-off | `12-governance-sign-off.md` | `mod-112/risk-council/minutes-2026-Q3.md` plus the L35 release-authoriser signature from `L35-eval-eng-alex` |
```

Reading check: twelve rows; every row points at a specific file path in the packet and at the specific upstream artefact (with mod-10N provenance) that fills it. Reserved slots (vendor file, provenance package, gate policy) are named as reserved and cross-reference the exercise that will populate them — they are not silent blanks.

### Step 5 — Fill Section 4 (Validator memo disposition)

The independent-validation memo lives at `07-independent-validation-memo.md`. Chapter 01 §"Independence of validation from development" and exercise 01 §"Requirements" name the structure: scope, evidence reviewed, pillar-by-pillar assessment (pass / conditional / fail with reason), material findings, validator's disposition (approve / conditional / reject). Here we paste only the disposition paragraph so the packet `README.md` has the one-line answer at the top; the full memo lives in the separate file.

```markdown
## 4. Validator memo disposition

Validator: `L35-eval-eng-alex` (role: evaluation engineer, peer L35; validation-owned CI service account `svc-helpdesk-validation`). Scope: HelpDesk v1.3.0 as submitted in packet v0.1 against mod-103 register v2026.11. Pillar assessment: **independence — pass**; **conceptual soundness — conditional** (methodology file cites harm-model rows; the data-representativeness argument depends on the exercise-04 Data Cards which are not yet in place — see gap G-01); **ongoing monitoring — pass** (mod-109 signal contract v0.2 in place with named routing and register effects); **effective challenge — pass** (red-team suite includes six fixtures contributed by eval-engineering; register rows `HR-CS-VEND-11` and `HR-CS-PROV-12` were surfaced by validation, not developer-volunteered). Material findings: three (see §5 gap list; G-01 Data Cards missing, G-02 vendor file reserved for exercise 02, G-03 signing/scanning gate reserved for exercises 05-06). **Disposition: conditional approve** — the packet shape is acceptable; the material findings must close by target dates below, with register-side evidence pointers updated as each gap closes.
```

Reading check: the disposition line is explicit (approve / conditional approve / reject); the pillar-by-pillar assessment names each of the four pillars; material findings cross-reference the gap list; the memo is authored by the named validator, not by the developer.

### Step 6 — Fill Section 5 (Gap list)

Every empty cell in the heading-to-artefact map or the pillar coverage table resolves to a gap row. Chapter 01 §"MRM-shaped validation documentation for an ML/AI model" names the rule: "Deferred is a legitimate status; silent is not." Each gap carries the responsible register row.

```markdown
## 5. Gap list

| Gap id | MRM heading | Missing artefact | Owner | Target date | Register row carrying the risk |
|---|---|---|---|---|---|
| G-01 | Data | Data Cards for training / fine-tune / eval / retrieval corpora at `provenance/helpdesk-finetune/v1.3.0/datasets/*.datacard.yaml` | exercise 04 author (data-platform peer) | 2026-12-15 | `HR-CS-PROV-12` (provenance evidence pending) |
| G-02 | Third-party components | SR 23-4 vendor file for `FoundationCo-LLM-3.5` at `11-third-party-components/FoundationCo-LLM-3.5/` | exercise 02 author (risk engineer) | 2026-12-05 | `HR-CS-VEND-11` (vendor file pending) |
| G-03 | Implementation testing | Signing + scanning gate at `gates/registry-ingestion-policy.rego`; signed Model Card, ML-BOM, SLSA attestation, ModelScan + picklescan reports in the provenance package | exercises 05-06 author (risk engineer) + L35 security platform | 2026-12-22 | `HR-CS-PROV-12` (gate evidence pending) |
| G-04 | Change management | FI's model-change policy reference confirmed by compliance; stub pending | compliance liaison | 2026-12-10 | `HR-CS-QUAL-03` (change-control coverage pending) |
```

Reading check: four rows; every gap names an owner, a target date, and a mod-103 register row id that carries the risk of the gap until it closes; no gap is silent. Target dates are specific dates, not "Q4".

### Step 7 — Fill Section 6 (Open items)

```markdown
## 6. Open items

- All four gap rows (G-01..G-04) are cross-referenced to exercises 02 / 04 / 05 / 06 and to the compliance liaison — the follow-up work is scoped and owned, not loose.
- `HR-CS-VEND-11` residual stays at `conditional` until the exercise-02 vendor file lands and the own-signal monitors established on `SC-HD-INJ-01` and `SC-HD-QUAL-01` have at least two weeks of data showing no silent-swap-shaped drift.
- The register-side back-link (the stretch goal from exercise 01) is not populated here — once G-01..G-04 close, every mod-103 row will carry an `mrm_packet_pointer` field to this packet at `v0.1` (or later versions as they are cut).
- The packet is `v0.1` — all four gap closures are expected to land before `v1.0`; `v1.0` is the shape the validator will re-sign and that the risk council will approve at the next quarterly meeting.
- The packet is unsigned at `v0.1`; signing `packet.manifest.json` with cosign per chapter 05 is the exercise-05 cross-reference and is tracked as part of G-03.
```

### Step 8 — Final checkpoint

Re-read the six sections. Confirm:

- **Section 1 (summary)** names system, packet version, register binding, regulatory context (SR 11-7 applicable; FDA GMLP not applicable with justification), and the validator identity.
- **Section 2 (four-pillar coverage)** has one row per pillar; each pillar cites at least one specific file *and* names the practice that makes the evidence credible. No pillar is evidenced by a document whose title claims the pillar (no "Independence Attestation" row).
- **Section 3 (heading-to-artefact map)** has one row per heading in chapter 01's twelve-heading TOC; every row names a specific file path in the packet and a specific upstream mod-10N artefact; reserved slots name the exercise that will populate them.
- **Section 4 (validator memo disposition)** has an explicit disposition line (approve / conditional / reject), a pillar-by-pillar assessment, material findings cross-referenced to §5, and the validator identity on the signature line.
- **Section 5 (gap list)** has one row per empty or deferred cell; every row names owner, target date, and the mod-103 register row id carrying the risk. No gap is silent.
- **Section 6 (open items)** names what closes at `v1.0`, what register-side back-links appear after close, and what signing cross-reference is pending on exercise 05.

## Expected output

`mod110-lab01/mrm-packet/HelpDesk/v0.1/README.md` — one populated six-section document; one to two pages. Section 2 (four-pillar coverage) and Section 3 (heading-to-artefact map) are the load-bearing artefacts; Section 5 (gap list) is the one an auditor reads first after the summary.

## Checkpoint questions (self-check)

1. The validator memo disposition reads "**conceptual soundness — conditional**" even though all mod-102 / mod-105 / mod-106 reports are in place. Why *conditional* rather than *pass*? (*Chapter 01 §"Conceptual soundness" lists five engineering-artefact cells — task specification + harm-model mapping; training-data representativeness via Datasheet for Datasets; metric-to-harm mapping; guardrail aim; human-oversight design. "If any of the right-hand cells are missing, the validator has no evidence for conceptual soundness and the model does not clear the pillar — regardless of how the metrics came out." The Data Cards cell is pending until exercise 04 — the training-data representativeness argument therefore has no supporting evidence file — so the pillar is **conditional** on G-01 closing, not pass.*)

2. The pillar-coverage table's "Effective challenge" row credits specific register rows (`HR-CS-VEND-11`, `HR-CS-PROV-12`) as evidence of challenge rather than any document called "effective-challenge-memo.md". Why? (*Chapter 01 §"Effective challenge" says "every model must be subject to organised, active questioning by informed parties who have the standing and the incentive to press hard" and names three engineering-visible signs — tests the developer did not write, register rows the developer would not have volunteered, post-mortem "what did we miss" sections owned by someone other than the developer. The pillar is evidenced by **disagreement in the record**, not by a document whose title claims the pillar. `HR-CS-VEND-11` and `HR-CS-PROV-12` are register rows validation surfaced that the developer would not have opened — the fact that those rows exist and were accepted into the register is the evidence of challenge.*)

3. Gap G-02 (vendor file) is a chapter-02 artefact, and gap G-03 (signing / scanning gate) is a chapter 05-06 artefact. Why do they appear on *this* packet's gap list rather than being deferred until their own modules are complete? (*Chapter 01 §"MRM-shaped validation documentation for an ML/AI model" pins the twelve-heading table of contents as the packet's canonical shape — the third-party-components and implementation-testing headings exist on the packet regardless of which chapter produces their content. Chapter 01 is the parent shape; chapters 02 and 05-06 produce artefacts that **file into** specific headings of that parent shape. If the parent packet omits the slots, the auditor reads the omission as "no third-party review was done" rather than "the vendor file is in a sibling folder you have not been shown." The packet reserves the slot and names the exercise that will fill it; that is what makes deferral legitimate rather than silent.*)

4. The pillar-coverage table's "Ongoing monitoring" row credits the mod-109 signal contract rather than a screenshot of Grafana. Why is the file shape load-bearing? (*Chapter 01 §"Ongoing monitoring, outcome analysis, back-testing" names what ongoing monitoring means for MRM — rolling accuracy / calibration / drift; fairness monitors; guardrail-fire rate; novel-attack surveillance; vendor behaviour-change signal; post-market outcome sample — and says the engineering inputs "come almost entirely from mod-109." The mod-109 signal contract (the fifteen-column artefact from mod-109 lab 01) carries the metric name, threshold, cadence, routing, action-on-fire, and evidence pointer per row — the five conditions chapter 01 of mod-109 pins for "a monitor." A Grafana screenshot satisfies none of the five. A model whose packet points at a screenshot cannot be credited as "in scope of ongoing validation" no matter how thorough the initial validation was — the file shape is what turns point-in-time into ongoing.*)

## Why this lab matters

Exercise 01 asks you to assemble the MRM packet for your real primary running system — the twelve-heading folder, the four-pillar coverage table, the heading-to-artefact map, the independent-validation memo, the gap list, and the change-management stub. If the packet shape is new at exercise-01 authoring time, the budget blows on inventing the folder layout, remembering which mod-10N artefact fills which heading, and arguing about whether a pillar is evidenced by a document or by a practice. This lab drills the top-level `README.md` shape on a fixed fabricated system so that exercise-01 time goes into the real engineering judgement — which register rows carry the risk of each gap, which validator can credibly sign, which pillar is genuinely evidenced by the team's practice — not into the format. The shape it drills is also what exercises 02 and 04 extend (vendor file; provenance package) and what exercises 05 and 06 sign and gate. Getting the top-level shape right here cascades through the module.
