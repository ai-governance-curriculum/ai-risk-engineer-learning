# Lab 02 — Model-Card Residual-Risk-Section Authoring Drill (Three-Part Claim + Named Monitors + Pre-Registered Invalidation Triggers)

**Duration:** 45 minutes · **Format:** guided practice, markdown authoring on a fixed fabricated LLM application

## Objective

Internalise the chapter-03 **residual-risk-section three-part shape** — (i) the residual level on a specific population / cohort / use case; (ii) the monitoring conditions under which the claim holds (named monitor IDs plus named thresholds); (iii) the pre-registered invalidation triggers (which conditions revoke the claim; what the system reverts to) — by authoring the `Residual Risk` section plus the `Uses — Out-of-Scope Use` section of a Hugging Face-schema model card on a fixed fabricated running system (`HelpDesk`). The lab drills the two load-bearing card sections chapter 03 §"6. Residual-risk claim" and §"1. Intended and out-of-scope use" pin so that when **exercise 03** authors a full three-card set (model, system, dataset) with all seven risk-slice components on the running system, the engineering budget goes into extracting the upstream evidence into the five other components (safety eval, adversarial robustness, fairness, privacy, monitoring plan), not into relitigating whether a residual claim needs an invalidation trigger or whether "not for high-stakes decisions" counts as a specific out-of-scope use.

The chapter-03 anti-patterns the shape forecloses: the **marketing card** residual section ("we continue to monitor"; "residual limitations exist"; no numbers, no monitors, no triggers); the **duplicate-authoring card** (the card drifts from the register six months later; the auditor asks which is authoritative); the **residual without invalidation** ("we assess residual as Medium-low" with no named condition that revokes the claim — a hope, not a claim); and the **generic out-of-scope** disclaimer ("not for high-stakes decisions") in place of the specific exclusions chapter 03 §"1. Intended and out-of-scope use" requires.

## Prerequisites

- Chapter 03 read — specifically "The three card families" (model card / system card / dataset card and where the risk-engineering slice lives); "The Hugging Face model-card spec — the operational format" (the YAML metadata block and section layout); "The risk-engineering slice — anatomy" (the seven components); "1. Intended and out-of-scope use" (specific off-label uses, not generic disclaimers); "6. Residual-risk claim" (the three-part load-bearing section); and "A worked short-form model-card risk slice" (the shape at the reading level).
- Chapter 04 skimmed at "1. Pre-registered thresholds" and "2. Rollback criteria" — the invalidation triggers on the residual claim are the card-side expression of the tiering document's rollback-trigger shape.
- Chapter 02 skimmed at "The decision-slide template" — the rollback/reversibility field on a council slide and the invalidation triggers on a card are instances of the same reversibility discipline.
- A terminal where you can create a working directory and author a markdown file.
- No external access or compute required; the system brief, the evidence-pointer IDs, and the two upstream evidence extracts are fixed below.

## The fixed example system

Use this system throughout the lab — do *not* substitute your own. Carrying a fixed scenario keeps the lab bounded to the two-section authoring skill. The system matches the `HelpDesk` carried through the mod-111 labs and lab 01 of this module; the chapter-03 worked example (customer-support RAG assistant) is the same shape.

> **`HelpDesk`** — a customer-support RAG assistant for a mid-size B2B SaaS company. Serves English-language customers of a billing-and-provisioning product. Assembles context from a knowledge-base store (product docs + internal runbooks + a mirrored public help-forum corpus) and from the authenticated caller's own recent ticket history. Backed by an in-house fine-tune (`helpdesk-finetune` v1.4.0) of the vendor-hosted base `FoundationCo-LLM-3.5` v3.5.2 behind a FastAPI service wrapping a LangGraph agent. Two production tool calls: `escalate_ticket(destination, summary)` and `lookup_account(account_id)`.
>
> **Classification under the EU AI Act:** high-risk (Annex III use case affecting a provider's customers; account data in scope). Deployed to EU and US customers. The sponsoring organisation is a US bank subsidiary — SR 11-7 is the parent MRM shape; HIPAA applies on the narrow subset of health-insurance-adjacent customer-service journeys; GDPR Article 33 runs concurrently for EU-resident data subjects.
>
> **Current deployment tier:** T2 (limited external, human-in-the-loop). The card you are authoring publishes at the T2 shape (chapter 03 §"1. Intended and out-of-scope use" — "deployment tier explicitly named alongside the intended use").
>
> **Card consumer profile:** downstream integrators inside the sponsoring organisation; a redacted external-facing projection of the same card is published to the organisation's public model registry.

### Fixed upstream evidence extracts (do not change)

Every field in the two sections cites IDs and numbers from this set. These are the extracts the card's residual claim projects over — chapter 03 §"The card-as-evidence-source pattern" pins the discipline: "the card is a *view* over the register and an *artefact indexed by* the register" — no fresh numbers.

| Upstream artefact | Content extract | Citation |
|---|---|---|
| Register row `HR-CS-INJ-01` v2026.09 | Residual **Medium-low** on prompt-injection exposure after guardrail credit (defence in depth across input filter, retrieval filter, tool-invocation policy). Appetite: Medium-low. | mod-103 Q3 refresh |
| Register row `HR-CS-PII-04` v2026.09 | Residual **Low** on PII egress after absolute-rule Presidio layering at ingest / retrieval / output. Appetite: Low. Absolute-rule row — any breach invalidates the Low claim and the row re-scores to High pending incident response. | mod-103 Q3 refresh |
| Register row `HR-CS-FAIR-08` v2026.09 | Residual **Medium** on `tenant_tier` per-slice quality parity, currently above appetite on the 2026-09-10 widening; 90-day above-appetite acceptance in effect to 2026-12-23 (council decision D-02). Remediation in progress. | mod-103 Q3 refresh |
| Register row `HR-CS-DRIFT-02` v2026.09 | Residual **Medium-low** on covariate drift; monitor coverage green on top-20 input features and prompt-embedding distribution. Appetite: Medium-low. | mod-103 Q3 refresh |
| Guardrail effectiveness `ART-HD-GRD-Q3-01` | G-INJ-01 F1 0.93 on retention corpus n=2,000; G-PII-01 F1 0.96; G-TOXIC-01 F1 0.91. Known defeat modes: many-shot injection variants (addressed by mod-104 Q3 red-team programme). | mod-108 Q3 |
| Fairness `ART-HD-FAIR-Q3-01` | `tenant_tier` per-slice parity delta 3.1 points on composite quality; `language_variant` per-slice (within English-language cohort) delta 1.2 points. Caveats: measurement does not cover non-English dialects (out of scope for the card). | mod-106 Q3 |
| Privacy `ART-HD-PRIV-Q3-01` | Canary extraction rate 0.03 percent on retention canary set n=5,000. MIA attack success rate 51.2 percent (near-random). No linkage findings. DPIA sign-off 2026-07-01. | mod-107 Q3 |
| Red-team `ART-HD-RED-Q3-01` | Q3 external partner engagement; 3 high-severity findings opened and closed; no open findings at card-publication date. | mod-104 Q3 |
| Monitor spec `ART-HD-MON-2026` | `SC-HD-INJ-01` injection-defeat rate, 30-day rolling, threshold < 0.3 percent; `SC-HD-PII-01` PII-egress detector firing rate, per-call, hard floor zero (any firing is an incident); `SC-HD-FAIR-01` `tenant_tier` parity delta, 7-day rolling, +/-5-point band; `SC-HD-DRIFT-01` embedding-drift KL-divergence, daily, band 0-0.12. | mod-109 Q3 |
| Deployment tier contract `ART-HD-TIER-2026` | T2 (limited external, human-in-the-loop) at card publication. T3 promotion in flight on the 2026-09-24 council (lab-01 decision slide D-01) subject to the 60-day monitored ramp. | mod-112 ch 04 |

### The two out-of-scope exclusions and the two residual-claim cohorts (do not substitute)

Chapter 03 §"1. Intended and out-of-scope use" pins the specificity bar: "the out-of-scope section names *specific* off-label uses. 'Do not use for hiring decisions' beats 'not intended for high-stakes decisions.'" The two exclusions below are deliberately contrasting — one is a statutory-category exclusion chapter 03 explicitly cites as the pattern; the other is a sector-adjacent exclusion that chapter 03's worked example does not name but that the HelpDesk context forces.

> **Exclusion E-01 — statutory high-stakes categories.** HelpDesk is not licensed for use in: (a) hiring, promotion, or termination decisions; (b) credit, benefits, lending, or underwriting decisions; (c) medical, legal, or tax advice; (d) legally binding communications without human review; (e) end users known or likely to be minors. This is the chapter-03 §"A worked short-form model-card risk slice" pattern lifted directly.
>
> **Exclusion E-02 — HIPAA-adjacent flows.** HelpDesk operates on a narrow subset of customer-service journeys that touch HIPAA-regulated health-insurance-adjacent flows (tenant tier of the sponsoring B2B customer is a US health plan or a US provider). The current card does **not** license HelpDesk for direct response on PHI content; those flows run through a separate T1 internal-only variant pending the mod-107 sub-flow re-evaluation on the 2026-Q4 council. The exclusion is specific: "no outbound tool call, no RAG retrieval, and no response generation on flows where PHI is present in the caller's recent ticket history; those flows route to human-agent escalation."
>
> **Residual-claim cohort C-01 — English-language EU + US B2B customers.** The primary residual claim covers the English-language cohort on the EU + US B2B customer population (the T2 deployment slice). This cohort covers register rows `HR-CS-INJ-01`, `HR-CS-PII-04`, `HR-CS-DRIFT-02` cleanly at appetite and `HR-CS-FAIR-08` in the 90-day above-appetite window.
>
> **Residual-claim cohort C-02 — HIPAA-adjacent flows (excluded; T1 internal-only variant).** The HIPAA-adjacent flow is not covered by the primary residual claim; a separate residual claim on the T1 internal-only variant lives on the sibling card and is not in scope for this lab. The primary card acknowledges the exclusion and points at the sibling card (chapter 03 §"The card-as-evidence-source pattern" — "the card is an evidence source in two directions").

## Steps

### Step 1 — Create your working directory and the card file

```bash
mkdir -p mod112-lab02
cd mod112-lab02
touch helpdesk-model-card-residual-slice.md
```

Open `helpdesk-model-card-residual-slice.md` and paste this scaffold. You will fill the two sections in order, then the readiness checklist.

```markdown
---
language: en
license: proprietary
tags:
  - customer-support
  - rag
  - risk-engineering-card
model-index:
  - name: helpdesk-finetune
    results: []  # populated in exercise 03's full-card authoring; out of scope for this lab
datasets:
  - helpdesk-transcripts-2024-2026-q3
---

# HelpDesk — model card (risk-engineering slice; lab 02)

Card version: v1.0
Card date: 2026-09-20
Card authors: risk-engineer-L25 (co-author); ML-lead (co-author)
Card contact: risk-engineering-oncall@example.internal
Register row back-pointers: `HR-CS-INJ-01`, `HR-CS-PII-04`, `HR-CS-FAIR-08`, `HR-CS-DRIFT-02`
Published-card hash: sha256:<placeholder>
Chapter references: mod-112 ch 03 (risk-engineering slice; residual-risk claim three-part shape; specific out-of-scope exclusions); ch 04 (invalidation triggers as card-side expression of tiering rollback)

## Uses — Direct Use

HelpDesk is intended for English-language first-line customer support for the sponsoring organisation's billing-and-provisioning product, up to and including drafting responses that are reviewed by a human agent before send. Deployment tier: **T2** (limited external, human-in-the-loop).

## Uses — Downstream Use

HelpDesk is not licensed for redeployment. Any integration outside the described product scope requires a fresh harm-model pass (mod-102) against the intended context and a sign-off on the chapter-01 decision-matrix row "Publish a model / system / dataset card externally."

## Uses — Out-of-Scope Use

(the specific exclusions enumerated under exclusions E-01 and E-02; chapter 03 §"1. Intended and out-of-scope use")

## Residual Risk

(the three-part shape: level on named cohort(s); monitors with thresholds; invalidation triggers; chapter 03 §"6. Residual-risk claim")

## Readiness checklist

(one line per section confirming the chapter-03 acceptance criteria)
```

### Step 2 — Fill the `Uses — Out-of-Scope Use` section

Chapter 03 §"1. Intended and out-of-scope use" pins the two rules: (i) out-of-scope uses are **specific**, not generic ("Do not use for hiring decisions" beats "not intended for high-stakes decisions"); (ii) the section ties to the harm model's persona / capability-surface enumeration (mod-102 ch 01) and the deployment-tier contract (this module ch 04) — specific exclusions are often the explicit floor of the harm model rather than decorative language on the card.

Fill the section:

```markdown
## Uses — Out-of-Scope Use

HelpDesk is **not licensed** for use in any of the following scenarios. Each exclusion is a floor condition from the mod-102 harm model and the mod-112 chapter-04 deployment-tier contract; deploying HelpDesk in any of these scenarios invalidates the residual claim below and routes to the chapter-01 decision matrix "Publish a model / system / dataset card externally" sign-off plus legal counsel on the obligation ruling.

### E-01 — Statutory high-stakes categories

1. **Hiring, promotion, or termination decisions** — HelpDesk output must not be used as input to, or replacement for, any employment-related decision. (EU AI Act Annex III(4) scope; NYC Local Law 144 where applicable on US deployments.)
2. **Credit, benefits, lending, or underwriting decisions** — HelpDesk output must not be used as input to, or replacement for, any financial-eligibility decision. (EU AI Act Annex III(5) scope; SR 11-7 parent MRM shape on the sponsoring organisation's US bank subsidiary.)
3. **Medical, legal, or tax advice** — HelpDesk output is not professional advice and must not be presented to end users as such. (EU AI Act Article 52 transparency obligations on AI-system interaction.)
4. **Legally binding communications without human review** — the T2 tier mandates human-in-the-loop review before send; any flow that sends HelpDesk output directly to end users without human review is outside the current card.
5. **End users known or likely to be minors** — HelpDesk is not evaluated against child-safety considerations; no cohort covering minors has been scoped under the mod-106 fairness-slice measurement or the mod-107 privacy evaluation.

### E-02 — HIPAA-adjacent flows (US health-insurance-adjacent customer-service journeys)

HelpDesk is **not licensed** for direct response on customer-service journeys where protected health information (PHI) is present in the caller's recent ticket history, where the caller's tenant is a US health plan or a US provider, or where the generated response would reference specific medical, insurance-claim, or prior-authorisation content.

Specifically:
1. **No outbound tool call** (`escalate_ticket`, `lookup_account`) on flows carrying PHI; those flows route to human-agent escalation at the service edge.
2. **No RAG retrieval** against the mirrored public help-forum corpus for queries that match the mod-107 PHI classifier; the retrieval layer short-circuits to a fixed "route to human agent" response.
3. **No response generation** on flows where PHI is present in the caller's recent ticket history or in the retrieval context; the service returns the fixed route-to-human response.

PHI-carrying flows are handled by the T1 internal-only `HelpDesk-PHI` variant (sibling card `cards/helpdesk-phi-t1-internal-only.md`), pending the mod-107 sub-flow re-evaluation on the 2026-Q4 council. The T1 variant is not externally covered by this card; any evaluation evidence for the T1 variant lives on the sibling card.

### What the out-of-scope section does *not* say

The exclusions above are **specific statutory and sector categories**, not generic disclaimers. The card does **not** contain the disclaimer "not intended for high-stakes decisions" — chapter 03 §"1. Intended and out-of-scope use" pins that a generic disclaimer is not evidence; specific exclusions with statutory citations are. Downstream integrators consuming this card are expected to re-run the harm model (mod-102) for any capability surface introduced by their integration that is not in the T2 cohort C-01.
```

Reading check: every exclusion is specific — "no hiring, promotion, or termination decisions" rather than "not for employment contexts"; "no outbound tool call on PHI flows" rather than "not for medical settings." Every exclusion cites a statutory or sector anchor (EU AI Act Annex III, NYC LL 144, SR 11-7, EU AI Act Article 52). The sibling-card pointer for the HIPAA-adjacent T1 variant is explicit — chapter 03 §"The card-as-evidence-source pattern" — rather than silent. The section closes with an explicit statement of what the card does *not* say, pre-empting the generic-disclaimer reading.

### Step 3 — Fill the `Residual Risk` section

Chapter 03 §"6. Residual-risk claim" pins the three-part shape: "(i) the residual level on the named population / cohort / use case; (ii) the monitoring conditions under which the claim holds (named monitor IDs plus named thresholds); (iii) the pre-registered invalidation triggers (which conditions revoke the claim; what the system reverts to)." The working shape in the chapter:

> After the mitigations described above, we assess residual risk as **[level, per programme rubric]** on the [population, cohort, use case] evaluated. This claim holds subject to [named monitoring signals from mod-109] remaining within [named thresholds]. If any of the following triggers fire, the claim is invalidated and the system reverts to [tier / mitigated state]: [named triggers, per this module ch 04].

Fill the section:

```markdown
## Residual Risk

### Primary residual claim — Cohort C-01 (English-language EU + US B2B customers, T2 tier)

After the mitigations described in the `Evaluation` and `Bias, Risks and Limitations` sections of this card (full enumeration lives in the full-card authoring from exercise 03; the risk-slice extracts are cited above), we assess the residual position on Cohort C-01 as follows, per register-row residual claims v2026.09:

- **`HR-CS-INJ-01` (prompt injection via retrieved context induces unauthorised tool call):** residual **Medium-low** — defence in depth across the input filter, retrieval filter, and tool-invocation policy (`ART-HD-GRD-Q3-01` v3.0 — G-INJ-01 F1 0.93 on retention corpus n=2,000). Known defeat modes: many-shot injection variants (addressed on `ART-HD-RED-Q3-01` v2.1).
- **`HR-CS-PII-04` (PII egress):** residual **Low** — absolute-rule Presidio layering at ingest / retrieval / output and per-caller entitlement enforcement (`ART-HD-GRD-Q3-01` v3.0 — G-PII-01 F1 0.96). Any PII egress is an incident that invalidates the Low claim and re-scores the row (see invalidation triggers below).
- **`HR-CS-FAIR-08` (per-slice quality parity on `tenant_tier`):** residual **Medium** for the 90-day above-appetite window to 2026-12-23 (council decision D-02; `ART-HD-FAIR-Q3-01` v2.0 — 3.1-point parity delta on composite quality). Remediation in progress; residual returns to Medium-low on remediation close.
- **`HR-CS-DRIFT-02` (covariate / embedding drift):** residual **Medium-low** — monitor coverage green on top-20 input features and prompt-embedding distribution (`ART-HD-MON-2026` v1.3 — `SC-HD-DRIFT-01` within 0-0.12 KL band daily).

**Aggregate residual claim on Cohort C-01:** **Medium-low** for the T2 tier, with `HR-CS-FAIR-08` at **Medium** for the 90-day above-appetite window. The aggregate returns to Medium-low on remediation close.

### Monitoring conditions under which the claim holds

The claim above holds subject to the following monitor signals remaining within the pre-registered thresholds (per `ART-HD-MON-2026` v1.3):

1. `SC-HD-INJ-01` (injection-defeat rate, 30-day rolling) below **0.3 percent** (green); the tripwire band 0.3-0.5 percent escalates investigation; 0.5 percent is an invalidation trigger (see below).
2. `SC-HD-PII-01` (PII-egress detector firing rate, per-call) at **zero** — hard floor; any firing is an incident and an invalidation trigger.
3. `SC-HD-FAIR-01` (`tenant_tier` per-slice parity delta, 7-day rolling) within the **+/-5-point band**; crossing the band on the 90-day above-appetite window re-opens council decision D-02 for revocation.
4. `SC-HD-DRIFT-01` (embedding-drift KL-divergence, daily) within the **0-0.12 band**; crossing 0.12 is a tripwire; sustained drift beyond 0.20 for 7 consecutive days is an invalidation trigger.

### Pre-registered invalidation triggers

If any one of the following triggers fires, this residual claim is invalidated and the system reverts to the stated mitigated state. These triggers mirror the rollback triggers on the deployment-tier contract `ART-HD-TIER-2026` (chapter 04 §"2. Rollback criteria"), expressed here as the card-side face of the same reversibility discipline. Revert is **mechanical** — a release-pipeline config flip (`helpdesk.deployment_tier=T1`), not a human decision at trigger time.

1. **`SC-HD-INJ-01` above 0.5 percent for 3 consecutive days.** System reverts to T1 (internal-only). Card is re-authored before any subsequent re-publication.
2. **Any `SC-HD-PII-01` firing on production traffic.** System reverts to T1 pending incident response (mod-111); any PII egress is an absolute-rule breach on `HR-CS-PII-04`. Card residual on `HR-CS-PII-04` re-scores to **High** pending post-mortem.
3. **`SC-HD-FAIR-01` per-slice delta widening beyond 5.0 points on any 7-day rolling window during the 90-day above-appetite window.** Council decision D-02 is auto-revoked; `HR-CS-FAIR-08` residual re-scores to its pre-exception state; system reverts to T1 unless the Q4 council re-authorises.
4. **Any severity ≥ 2 customer-visible incident** on `ART-HD-INC-2026`. System reverts to T1 pending post-mortem; card is re-authored before subsequent re-publication.
5. **Regression on the Q3 retention corpus > 5 percent** on composite quality (`ART-HD-GRD-Q3-01` regression suite). System holds at current tier pending diagnosis; no further promotion until regression is diagnosed and remediated.
6. **Supply-chain capability delta from `FoundationCo-LLM-3.5` base-model version bump** (`HR-CS-VEND-11` tripwire). System holds at T2 pending re-evaluation on the retention corpus; no promotion to T3 until re-evaluation completes.
7. **New EU AI Act delegated act requiring transparency addendum.** Card is re-authored with the addendum before any subsequent re-publication; system holds at current tier until addendum publishes (chapter 04 §"2. Rollback criteria" — framework/obligation change).

### Reversibility is mechanical, not discretionary

Each invalidation trigger above fires automatically at the named threshold with the named duration guard; the revert is a release-pipeline config flip tested weekly in CI. **No trigger reads "at the risk engineer's discretion" or "if things look bad."** Chapter 04 §"2. Rollback criteria" pins the discipline: "if revert requires a human decision, the criterion has not been pre-registered properly." Decision authority on *returning* to T2 or progressing to T3 after a revert is risk council + `ai-evaluation-engineer` (L35) sign-off, recorded in the chapter-02 decision log.

### What this residual claim does *not* cover

- **Cohort C-02 (HIPAA-adjacent flows).** The T1 internal-only `HelpDesk-PHI` variant has a separate residual claim on its sibling card (`cards/helpdesk-phi-t1-internal-only.md`).
- **Non-English dialects within the English-language cohort.** The mod-106 fairness-slice measurement does not cover non-English dialects; downstream integrators deploying to non-English cohorts must re-run the mod-106 measurement (chapter 03 §"4. Fairness slice measurements" — "a card that reports fairness deltas without slice definitions is not useful evidence"). The caveats sheet on `ART-HD-FAIR-Q3-01` v2.0 enumerates the non-coverage.
- **Minors as end users.** No cohort covering minors has been scoped under mod-106 or mod-107. Deployment to minor-end-user cohorts requires a fresh harm-model pass and is covered by exclusion E-01 above.
```

Reading check: the section has the three-part shape — level on named cohort(s); monitors with named thresholds; pre-registered invalidation triggers that fire mechanically. Every residual level cites a register row and an evidence-pointer ID. Every monitor cites a monitor ID and the exact threshold with the duration guard where applicable. Every invalidation trigger cites a specific metric, a specific threshold, a specific duration, and the mitigated state the system reverts to. The "what this residual claim does *not* cover" subsection closes the scope explicitly — chapter 03's "card without residual claim" failure mode is not only about having a claim but about having an **honest-scoped** claim; deliberately naming what the claim does not cover is the same discipline as deliberately enumerating the invalidation triggers.

### Step 4 — Fill the readiness checklist

Chapter 03 §"6. Residual-risk claim" is explicit on the acceptance: "**the claim is testable, tied to monitoring, and reversible. A residual-risk claim without an invalidation condition is not defensible.**" Chapter 03 §"1. Intended and out-of-scope use" is explicit on specificity: "'Do not use for hiring decisions' beats 'not intended for high-stakes decisions.'" Run the explicit checklist so the card-author's acceptance test matches chapter 03's reading-level acceptance test.

```markdown
## Readiness checklist

| Check | Section | Status | Chapter 03 citation |
|---|---|---|---|
| Deployment tier named alongside intended use | `Uses — Direct Use` | **Yes** — T2 named | §"1. Intended and out-of-scope use" ("tier explicitly named alongside the intended use") |
| Out-of-scope uses enumerated specifically | `Uses — Out-of-Scope Use` | **Yes** — E-01 statutory categories; E-02 HIPAA-adjacent flows; each with statutory/sector anchor | §"1. Intended and out-of-scope use" ("'Do not use for hiring decisions' beats 'not intended for high-stakes decisions'") |
| Residual level stated per named cohort | `Residual Risk` | **Yes** — Cohort C-01 with per-register-row residual; aggregate claim stated | §"6. Residual-risk claim" (level on named population/cohort/use case) |
| Named monitors with named thresholds | `Residual Risk` | **Yes** — 4 monitors (`SC-HD-INJ-01`, `SC-HD-PII-01`, `SC-HD-FAIR-01`, `SC-HD-DRIFT-01`) with thresholds and duration guards | §"6. Residual-risk claim" (monitors from mod-109 with named thresholds) |
| Pre-registered invalidation triggers (mechanical revert) | `Residual Risk` | **Yes** — 7 triggers, each with metric + threshold + duration + mitigated-state | §"6. Residual-risk claim" (pre-registered triggers; chapter 04 §"2. Rollback criteria" mechanical revert) |
| No trigger reads "at discretion" or "if things look bad" | `Residual Risk` | **Yes** — explicit "reversibility is mechanical, not discretionary" subsection | chapter 04 §"2. Rollback criteria" ("if revert requires a human decision, the criterion has not been pre-registered properly") |
| Scope-not-covered items named explicitly | `Residual Risk` | **Yes** — Cohort C-02, non-English dialects, minors | §"4. Fairness slice measurements" (slice-definition discipline); §"6. Residual-risk claim" (honest-scoped claim) |
| Register row back-pointers populated | Card header | **Yes** — `HR-CS-INJ-01`, `HR-CS-PII-04`, `HR-CS-FAIR-08`, `HR-CS-DRIFT-02` | §"The card-as-evidence-source pattern" (card is a *view* over the register and an *artefact indexed by* the register) |

Both sections and the card header satisfy chapter 03's acceptance criteria. Any `No` cell above pulls the card from publication and files a gap; a card that fails any check is not shippable per §"6. Residual-risk claim" ("a residual-risk claim without an invalidation condition is not defensible").
```

### Step 5 — Final checkpoint

Re-read the two sections and the checklist. Confirm:

- **`Uses — Out-of-Scope Use`** has specific statutory exclusions (E-01) and a specific sector-adjacent exclusion (E-02) with explicit mechanics (no outbound tool call, no RAG retrieval, no response generation on PHI flows). No exclusion reads "not for high-stakes decisions" or "use with caution." Every exclusion cites a statutory anchor or a mod-102 harm-model row.
- **`Residual Risk`** has the three-part shape: level on named cohort(s); monitors with named thresholds and duration guards; pre-registered invalidation triggers that fire mechanically. The aggregate residual claim is stated (Medium-low, with `HR-CS-FAIR-08` at Medium for the 90-day window). The scope-not-covered items are named explicitly (Cohort C-02, non-English dialects, minors). Reversibility is mechanical — no trigger reads "at discretion."
- **The readiness checklist** runs the chapter-03 acceptance tests as per-field yes/no. No `No` cells. Register row back-pointers are populated — the card is a projection of the register (chapter 03 §"The card-as-evidence-source pattern"), not a fresh authoring pass.
- Every number on the card comes from an upstream artefact ID — no fresh numbers; no fabricated metrics; the card is a view over mods 103-109 evidence.

## Expected output

`mod112-lab02/helpdesk-model-card-residual-slice.md` — one populated document with the Hugging Face-schema YAML front-matter, the two load-bearing risk-slice sections (`Uses — Out-of-Scope Use` and `Residual Risk`), and a readiness checklist; one to two pages total. The `Residual Risk` section is the load-bearing artefact; the `Uses — Out-of-Scope Use` section is the one downstream integrators read first to know what scope the residual claim actually covers.

## Checkpoint questions (self-check)

1. The `Uses — Out-of-Scope Use` section does not contain the phrase "not for high-stakes decisions." Chapter 03 §"1. Intended and out-of-scope use" says generic disclaimers do not satisfy the section. Why is the specificity bar set at this level, and what is the downstream-integrator consequence of a generic disclaimer? (*Chapter 03 §"1. Intended and out-of-scope use" pins the rule verbatim: "'Do not use for hiring decisions' beats 'not intended for high-stakes decisions.'" §"Motivation" names the failure as the **"marketing card"** — "sections that should carry evaluation numbers carry paragraphs of language that could equally describe half the models on the Hub. No numbers, no citations, no evidence anchors." The downstream-integrator consequence: a generic disclaimer does not operationalise. An integrator reading "not for high-stakes decisions" has to perform a fresh risk assessment to decide whether their deployment is "high-stakes." An integrator reading "not for hiring, promotion, or termination decisions" with EU AI Act Annex III(4) citation has an actionable floor — they know on the first read whether their deployment is in or out of scope. Chapter 03 §"The card-as-evidence-source pattern" extends the discipline: "cards act as evidence in both directions — downstream for consumers and upstream as an audit-linkable artefact indexed by the register." A generic disclaimer is not evidence; a specific exclusion with a statutory anchor is.*)

2. The `Residual Risk` section contains **seven** pre-registered invalidation triggers, each with a specific metric, threshold, duration, and mitigated state. Chapter 03 §"6. Residual-risk claim" requires "pre-registered invalidation triggers" as the third part of the three-part shape. Why is the trigger shape **mechanical** rather than **at the risk engineer's discretion**, and how does this connect to chapter 04's rollback discipline? (*Chapter 03 §"6. Residual-risk claim" pins the three-part shape: "the claim is testable, tied to monitoring, and reversible. A residual-risk claim without an invalidation condition is not defensible." Chapter 04 §"2. Rollback criteria" extends the discipline: "Rollback is *mechanical* — a config flip in the release pipeline. If revert requires a human decision, the criterion has not been pre-registered properly." The card-side invalidation triggers are the public face of the tiering document's rollback triggers — the same reversibility expressed at two artefacts. The mechanical shape has three properties: (i) a release engineer can wire the trigger into the release pipeline without further specification; (ii) the trigger fires at the named threshold with the named duration guard, independent of who is on-call at trigger time; (iii) the auditor reading the card can verify the trigger's wiring by inspecting the release pipeline's CI configuration, not by trusting that the on-call will make the right call. A discretionary trigger ("at the risk engineer's discretion") fails all three — the pipeline cannot be wired; the trigger's firing depends on who is on-call; the auditor has to trust rather than verify. Chapter 02 §"The decision-slide template" field seven (rollback/reversibility) and chapter 04 §"2. Rollback criteria" and chapter 03 §"6. Residual-risk claim" are three expressions of one reversibility discipline across three artefacts.*)

3. The `Residual Risk` section has a "what this residual claim does *not* cover" subsection naming Cohort C-02 (HIPAA-adjacent flows), non-English dialects, and minors. Chapter 03 §"6. Residual-risk claim" does not explicitly require this subsection. Why is scope-not-covered part of the three-part discipline, and what failure mode does its absence trigger? (*The three-part shape chapter 03 §"6. Residual-risk claim" names is "level on cohort — monitors with thresholds — invalidation triggers." The "what does not covered" subsection is not an explicit fourth part, but it is the necessary complement of part one — a residual claim on "the English-language EU + US B2B customer cohort" is only defensible if the **scope of the claim** is explicit. Without the non-coverage subsection, a downstream integrator reads the Medium-low claim and may deploy the model to a minor-end-user cohort or a HIPAA-adjacent flow and assume the Medium-low claim covers them. Chapter 03 §"4. Fairness slice measurements" pins the sibling discipline — "report the slice definition alongside the number, and cite the caveat sheet honestly. A card that reports fairness deltas without slice definitions is not useful evidence" — which is the same shape applied to fairness. The failure mode the subsection prevents: the **over-claim residual** — a claim that reads broader than it is because the scope is not named, so downstream readers apply it to cohorts it was not evaluated against. Chapter 03 §"Internal templates vs the Hugging Face schema" names a related failure: an internal template that omits "Bias, Risks and Limitations" has a compliance gap unless the omission is justified in writing — the discipline is the same: deliberate scope-naming is part of the claim, not an addendum.*)

4. The card header has **register row back-pointers** (`HR-CS-INJ-01`, `HR-CS-PII-04`, `HR-CS-FAIR-08`, `HR-CS-DRIFT-02`). Chapter 03 §"The card-as-evidence-source pattern" says "register rows carry a `card_ref` field pointing at the card version and hash that publicly represents them." Why is the pointer explicitly bidirectional, and what audit question does the bidirectional pointer answer? (*Chapter 03 §"The card-as-evidence-source pattern" pins the bidirectional discipline: "the card is a *view* over the register and an *artefact indexed by* the register. Register rows carry a `card_ref` field pointing at the card version and hash that publicly represents them." The forward pointer (register → card) and the back-pointer (card → register row) answer two different audit questions that compose: (i) given a register row, where does the public world see this evidence? (the forward pointer answers); (ii) given a public card, which register rows is this card's residual claim a projection of? (the back-pointer answers). Without the back-pointer, a card reader cannot trace the claim back to the underlying rows — the card could be an orphan document authored independently of the register (chapter 03 §"Motivation" names this as the **"duplicate-authoring card"** failure: "the card and the register drift; six months later they disagree about the fairness delta or the memorisation rate, and the auditor asks which is authoritative"). The bidirectional pointer makes the register-card relationship explicit and verifiable, and chapter 05 §"Crosswalk shape" extends the discipline — the crosswalk table contains both the card (as an artefact row) and the register (as an artefact row) and the two row types point at each other through the framework citations. The three-way consistency — register, card, crosswalk — is what makes the evidence trail audit-durable.*)

## Why this lab matters

Exercise 03 asks you to author a full three-card set (model, system, dataset) with all seven risk-slice components on your own primary running system. If the residual-section three-part shape is new at exercise-03 authoring time, the budget blows on relitigating whether an invalidation trigger is required (chapter 03 §"6. Residual-risk claim" says it is), whether "we continue to monitor" counts as a claim (chapter 03 §"Motivation" says it does not), or whether a generic high-stakes disclaimer discharges the out-of-scope obligation (chapter 03 §"1. Intended and out-of-scope use" says it does not). This lab drills the two load-bearing sections on a fixed fabricated system with deliberately contrasting scope conditions — one cohort covered by the primary claim, one cohort explicitly excluded with mechanics; one above-appetite row that complicates the aggregate claim; one invalidation trigger per failure mode the mod-109 monitor spec surfaces. Getting the two sections right here cascades through exercise 03 (where the same shape appears on the model card, the system card, and — in slightly different form — the dataset card) and into exercise 04 (where the card-side invalidation triggers are the tiering-document rollback triggers at a different artefact scope) and exercise 05 (where the card, the register, and the deployment-tier contract appear as three artefact rows in the crosswalk pointing at overlapping framework citations). The residual section is the atomic unit of external-facing accountability; the lab is the drill that lets a shippable card be assembled from already-rehearsed parts.
