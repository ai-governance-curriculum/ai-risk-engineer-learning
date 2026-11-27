# Lab 03 — Crosswalk-Row and Gap-Disposition Drill (One Artefact Many Standards + Three-Disposition Gap + Version-Hash Pinning)

**Duration:** 45 minutes · **Format:** guided practice, markdown / YAML authoring on a fixed fabricated LLM application

## Objective

Internalise the chapter-05 **crosswalk-row shape** — one row per evidence artefact, columns for RMF MEASURE / MANAGE sub-cats, ISO/IEC 42001 Annex A controls, EU AI Act articles, sector overlays, owner, last-refresh; **many-to-many** citation; **version-hash pinned** — and the **three-disposition gap-register shape** — every uncovered framework citation marked **fill** (schedule the missing artefact), **waive with rationale** (dated and signed-off out-of-scope decision), or **escalate** (work above L25 with the mod-101 ch 06 contract cited). The lab drills the atomic row shape and the gap-disposition shape chapter 05 pins so that when **exercise 05** builds the full crosswalk across ten-plus artefacts on your real primary running system, the engineering budget goes into surfacing the actual gaps and dispositioning them, not into relitigating whether "TBD" counts as a disposition or whether the register counts as one artefact or fifty.

The chapter-05 anti-patterns the shape forecloses: the **prose crosswalk** (paragraphs instead of per-citation pointers; auditor cannot verify from the document alone); the **N-to-1 collapse** ("the register" cited against every sub-category with no per-citation granularity); the **1-to-1 duplication** (a separate artefact for every sub-category producing redundant drifting evidence stores); the **unmarked gap** (a framework citation with no filled artefact, no waiver, no escalation — "TBD" is not a disposition and a crosswalk with unmarked gaps is non-shippable to audit); and the **stale-hash row** (an artefact pointer whose hash no longer matches what the file contains — chapter 05 §"Maintenance discipline" wires this into monthly hash checks).

## Prerequisites

- Chapter 05 read — specifically "What the crosswalk covers" (RMF and ISO/IEC 42001 as the level-25 crosswalks; EU AI Act and sector overlays as overlay columns); "Crosswalk shape" (one row per evidence artefact with the many-to-many and version-hash-pinned properties); "The 'one artefact, many standards' pattern — worked" (a single guardrail report filing against seven distinct downstream uses); "Coverage gap detection" (the three dispositions — fill, waive with rationale, escalate — and the rule that unmarked gaps are non-shippable); "Handing off release-assurance work to `ai-evaluation-engineer` (level 35)" and "Handing off control-library architecture to `senior-ai-governance-architect` (level 50)" (the escalation-destination table); and "Maintenance discipline" (monthly hash check; per-framework-revision; per-system-change; per-audit regeneration; CI check).
- Mod-101 chapter 06 (deferral contract) re-skimmed — the escalation targets are `ai-evaluation-engineer` (L35), `agentic-safety-engineer` (L40), `senior-ai-governance-architect` (L50), and `head-of-ai-governance` (L60).
- Chapter 02 skimmed at §"The decision log" — the decision-log entry format (YAML with `decision_id`, `evidence_pack_hash`, `linked_obligations`) is the shape the crosswalk cites into; a council decision-log entry is itself a crosswalk-row candidate.
- A terminal where you can create a working directory and author a markdown file (with an embedded table).
- No external access or compute required; the system brief, the artefact extracts, and the framework-citation stubs are fixed below.

## The fixed example system

Use this system throughout the lab — do *not* substitute your own. Carrying a fixed scenario keeps the lab bounded to the row-and-gap authoring skill. The system matches the `HelpDesk` carried through the mod-111 labs and labs 01 and 02 of this module.

> **`HelpDesk`** — a customer-support RAG assistant for a mid-size B2B SaaS company. Serves English-language customers of a billing-and-provisioning product. Backed by an in-house fine-tune (`helpdesk-finetune` v1.4.0) of the vendor-hosted base `FoundationCo-LLM-3.5` v3.5.2.
>
> **Classification under the EU AI Act:** high-risk (Annex III use case affecting a provider's customers; account data in scope). Deployed to EU and US customers. The sponsoring organisation is a US bank subsidiary — SR 11-7 is the parent MRM shape; HIPAA applies on the narrow subset of health-insurance-adjacent customer-service journeys; GDPR Article 33 runs concurrently for EU-resident data subjects.
>
> **Current deployment tier:** T2 (limited external, human-in-the-loop). T3 promotion decision is in flight on the 2026-09-24 council (lab-01 slide D-01).
>
> **In-scope framework overlays for this programme:** NIST AI RMF 1.0 MEASURE / MANAGE; ISO/IEC 42001:2023 Annex A; EU AI Act (Articles 9, 10, 13, 14, 15, 72); SR 11-7 (parent FI MRM); HIPAA and GDPR Article 33 as *overlay columns* on the same crosswalk per chapter 05 §"What the crosswalk covers." Not in scope for this programme: Colorado AI Act; NYC LL 144 (no hiring-shaped flows); FDA GMLP + PCCP (not a medical device).

### Fixed artefact-row stubs (do not add more)

Every crosswalk row's `Artefact ID` is from this set. Chapter-05's §"Crosswalk shape" worked table uses the same shape. Where a field is marked **<fill in Step 2>**, you will author the citation columns.

| Artefact ID | Artefact type (module author) | Version | Hash | Owner | Last refresh |
|---|---|---|---|---|---|
| `ART-HD-RED-Q3-01` | Red-team run report (mod-104) — Q3 external partner engagement | v2.1 | sha256:4e3a... | risk-eng-L25 | 2026-09-01 |
| `ART-HD-GRD-Q3-01` | Guardrail effectiveness report (mod-108) — Q3 retention corpus | v3.0 | sha256:71bc... | risk-eng-L25 | 2026-09-15 |
| `ART-HD-MON-2026` | Monitor spec + Q3 digest (mod-109) | v1.3 | sha256:3cfd... | risk-eng-L25 | 2026-09-30 |
| `ART-HD-FAIR-Q3-01` | Fairness-slice report (mod-106) | v2.0 | sha256:8acd... | risk-eng-L25 | 2026-09-20 |
| `ART-HD-PRIV-Q3-01` | Privacy-risk report (mod-107) | v2.0 | sha256:a21e... | risk-eng-L25 | 2026-09-20 |
| `ART-HD-REG-2026-Q3` | Register aggregation report (mod-103 ch 05) | v1 | sha256:b4d2... | risk-eng-L25 | 2026-09-30 |
| `ART-HD-COUNCIL-Q3` | Council decision-log deltas 2026-Q3 (this module ch 02) | v1 | sha256:1e87... | risk-eng-L25 | 2026-09-24 |
| `ART-HD-CARD-2026-09` | Model / system / dataset cards (this module ch 03) | v3.0 | sha256:5f08... | risk-eng-L25 + comms | 2026-09-15 |
| `ART-HD-TIER-2026` | Deployment-tier contract (this module ch 04) | v2.1 | sha256:9a0c... | risk-eng-L25 | 2026-09-24 |
| `ART-HD-INC-2026` | Incident log + post-mortems (mod-111) | rolling | sha256:7c33... | risk-eng-L25 | rolling |
| `ART-HD-SBOM-2026` | Model-BOM / SBOM (mod-110) | v1.2 | sha256:2d59... | risk-eng-L25 + supply-chain | 2026-09-30 |

### Fixed framework-citation stubs (do not substitute)

Chapter 05 pins placeholder-identifier discipline — the specific identifiers vary by framework revision, but the *categories* and *article numbers* are stable. Where chapter 05 explicitly cites an identifier in its worked table, use the same citation here; where chapter 05 notes `needs-research` on the identifier, use the category and mark with the same marker.

- **RMF MEASURE sub-cats** (from chapter 05 §"The NIST AI RMF MEASURE and MANAGE functions" and §"The 'one artefact, many standards' pattern — worked"): `MEASURE-1.1` (risk measurement scope), `MEASURE-1.3` (methodology selection at the tiering level), `MEASURE-2.1` (applying measurement methods), `MEASURE-2.5` (adversarial robustness / prompt-injection defeat), `MEASURE-2.6` (measurement across risk categories), `MEASURE-2.7` (measurement across risk categories, continued), `MEASURE-2.9` (supply-chain), `MEASURE-2.10` (privacy), `MEASURE-2.11` / `MEASURE-2.12` (fairness), `MEASURE-3.1` / `MEASURE-3.2` / `MEASURE-3.3` (monitoring and tracking), `MEASURE-4.1` (gathering feedback), `MEASURE-4.2` (feedback to cards), `MEASURE-4.3` (incident-and-near-miss feedback).
- **RMF MANAGE sub-cats:** `MANAGE-1.1` (prioritisation), `MANAGE-1.2` (prioritisation at the tiering level), `MANAGE-1.3` (prioritisation on supply-chain), `MANAGE-1.4` (prioritisation on the register), `MANAGE-2.1` (risk-response strategy), `MANAGE-2.2` (strategies on identified risks), `MANAGE-2.3` (strategies on red-team findings), `MANAGE-2.4` (strategies on fairness and privacy), `MANAGE-3.1` (residual-risk documentation on the register), `MANAGE-3.2` (residual at council scope), `MANAGE-4.1` (incident response), `MANAGE-4.2` (incident communication), `MANAGE-4.3` (post-market communication / cards).
- **ISO/IEC 42001:2023 Annex A controls (category level; identifiers per chapter-05 §"ISO/IEC 42001:2023 — the controls the crosswalk covers" `needs-research` marker):** A.5 leadership (`A.5.2`, `A.5.3`); A.6 operation — AI system lifecycle (`A.6.2.2`, `A.6.2.3`, `A.6.2.4`, `A.6.2.5`, `A.6.2.6`, `A.6.2.7`, `A.6.2.8`); A.7 data — resources (`A.7.2`, `A.7.3`); A.9 third-party relationships; A.10 improvement (`A.10.1`).
- **EU AI Act articles (Regulation (EU) 2024/1689):** Article 9 (risk management), Article 10 (data and data governance), Article 13 (transparency), Article 14 (human oversight), Article 15 (accuracy / robustness / cybersecurity), Article 72 (post-market monitoring).
- **SR 11-7 attributes** (parent FI MRM shape for the sponsoring US bank subsidiary): `model-tier`, `third-party-risk`.
- **HIPAA / GDPR Article 33 overlay columns**: `HIPAA-security-rule` (for the PHI sub-flow carve-out); `GDPR-Art-33` (72-hour breach-notification clock on EU-resident data subjects).

### The two authoring targets (do not substitute)

Use exactly these two. Target T-01 drills the "**one artefact, many standards**" pattern on the artefact chapter 05 §"The 'one artefact, many standards' pattern — worked" worked explicitly: the guardrail-effectiveness report `ART-HD-GRD-Q3-01` cited against seven distinct framework rows. Target T-02 drills the three-disposition gap register on three specific uncovered framework citations the HelpDesk programme would plausibly surface at the 2026-Q3 refresh.

> **Target T-01 — author the crosswalk row for `ART-HD-GRD-Q3-01`.** Chapter 05 §"The 'one artefact, many standards' pattern — worked" enumerates the seven downstream uses of this exact artefact: MEASURE-2.5; MEASURE-2.7; MANAGE-2.2; ISO 42001 Annex A verification-and-validation control family; EU AI Act Article 15; the programme's internal deployment-tier contract (threshold input); the model / system card (`Evaluation` section input); and the council pre-read decision slides (evidence pointers). The crosswalk row cites the first five (framework citations); the last three are back-pointers to other crosswalk rows (the tier contract, the card, the decision-log deltas) and do not appear as framework-citation columns.
>
> **Target T-02 — author three gap-register entries, one per disposition class (fill, waive with rationale, escalate).** The three specific gaps to disposition:
>
> 1. **Gap G-01 — MEASURE-4.1 (gathering feedback) has no filled artefact.** The HelpDesk programme has not yet wired the formal "feedback channel from downstream consumers" artefact chapter 05's MEASURE-4.x cluster expects — the monitor spec covers MEASURE-3.x (monitoring and tracking) but not the channel-shape artefact under MEASURE-4.1. **Disposition: fill.** The gap corresponds to work the programme should be doing.
> 2. **Gap G-02 — EU AI Act Article 14 (human oversight) has no filled artefact at Q3.** The Article 14 coverage lives on the deployment-tier contract and the card (both of which name the human-in-the-loop requirement at T2) but is not pinned to a *separate* artefact. For this programme, the artefact is either the tier contract (already row-authored) or a sibling human-oversight artefact that chapter 05 does not force. The decision is whether to treat Article 14 as already filled (by the tier contract and the card) or as a gap requiring a separate artefact. **Disposition: waive with rationale** — the chapter-05 "one artefact, many standards" pattern is explicit that Article 14 can be a filled citation on multiple artefacts; a separate sibling artefact is not required at team scope.
> 3. **Gap G-03 — RMF MANAGE "release-assurance evidence packaging" (not a specific sub-cat; this is L35 scope).** An auditor asks "how is the evidence bundled for external regulator engagement?" The L25 artefacts individually file against RMF sub-cats, but the *bundle* discipline — chain of custody, timestamping, immutable log — is `ai-evaluation-engineer` (L35) scope per chapter 05 §"Handing off release-assurance work to `ai-evaluation-engineer` (level 35)." **Disposition: escalate** to L35 with the mod-101 ch 06 contract cited.

## Steps

### Step 1 — Create your working directory and the two files

```bash
mkdir -p mod112-lab03
cd mod112-lab03
touch crosswalk.md gaps.md
```

Open `crosswalk.md` and paste this scaffold:

```markdown
# HelpDesk — framework crosswalk (lab 03)

Programme: HelpDesk (customer-support RAG assistant; see lab brief)
Crosswalk version: v1, date: 2026-09-30, hash: sha256:<placeholder>
Chapter references: mod-112 ch 05 (crosswalk shape; one artefact, many standards; three-disposition gap register; L35 / L50 hand-off)
In-scope frameworks: NIST AI RMF 1.0 MEASURE / MANAGE; ISO/IEC 42001:2023 Annex A; EU AI Act (Arts 9, 10, 13, 14, 15, 72); SR 11-7 (parent FI MRM); HIPAA overlay (PHI sub-flow); GDPR Art 33 overlay (72-hour breach clock)
Not in scope: Colorado AI Act; NYC LL 144; FDA GMLP + PCCP

## 1. Crosswalk rows

(one row per evidence artefact; many-to-many framework citations; version-hash pinned; chapter 05 §"Crosswalk shape")

## 2. Coverage summary

(count of in-scope citations by framework family: filled / waived / escalated; chapter 05 §"Coverage gap detection" and §"Coverage summary — appended to `crosswalk.md`")

## 3. Maintenance discipline

(citation to chapter 05 §"Maintenance discipline"; monthly hash check cadence; per-framework-revision citation refresh; CI check wiring)
```

Open `gaps.md` and paste:

```markdown
# HelpDesk — gap register (lab 03)

Gap register version: v1, date: 2026-09-30
Chapter references: mod-112 ch 05 (three-disposition gap register — fill, waive with rationale, escalate); mod-101 ch 06 (deferral contract to L35 / L50 / L60)

## Gap entries

(one entry per uncovered in-scope framework citation; disposition: fill / waive / escalate; chapter 05 §"Coverage gap detection")
```

### Step 2 — Fill Target T-01 (the `ART-HD-GRD-Q3-01` crosswalk row)

Chapter 05 §"The 'one artefact, many standards' pattern — worked" enumerates the seven downstream uses of the guardrail-effectiveness report. The crosswalk row lifts the first five (framework citations) directly; the sixth and seventh (tier contract, card, decision-log deltas) are back-pointers to other crosswalk rows and do not appear here.

In `crosswalk.md`, fill Section 1. Author the row for `ART-HD-GRD-Q3-01` first; then add the other ten rows in the same shape (you can abbreviate the non-target rows for the lab; the full build is exercise-05 scope).

```markdown
## 1. Crosswalk rows

| Artefact ID | Artefact type | Version / hash | RMF MEASURE sub-cats | RMF MANAGE sub-cats | ISO/IEC 42001 Annex A | EU AI Act articles | SR 11-7 attributes | HIPAA / GDPR overlay | Owner | Last refresh |
|---|---|---|---|---|---|---|---|---|---|---|
| `ART-HD-GRD-Q3-01` | Guardrail effectiveness report (mod-108) — Q3 retention corpus n=2,000; G-INJ-01 F1 0.93, G-PII-01 F1 0.96, G-TOXIC-01 F1 0.91 | v3.0, sha256:71bc... | MEASURE-2.5, MEASURE-2.6, MEASURE-2.7 | MANAGE-2.2 | A.6.2.4 (V&V control family) <!-- needs-research: pin verbatim ISO/IEC 42001:2023 Annex A control identifier at authoring time --> | Art 15 (accuracy, robustness, cybersecurity) | — | — | risk-eng-L25 | 2026-09-15 |
| `ART-HD-RED-Q3-01` | Red-team run report (mod-104) — Q3 external partner | v2.1, sha256:4e3a... | MEASURE-2.5, MEASURE-2.6, MEASURE-3.1 | MANAGE-2.3 | A.6.2.4, A.6.2.6 | Art 15 | — | — | risk-eng-L25 | 2026-09-01 |
| `ART-HD-MON-2026` | Monitor spec + Q3 digest (mod-109) | v1.3, sha256:3cfd... | MEASURE-3.1, MEASURE-3.2, MEASURE-3.3 | MANAGE-4.1 | A.6.2.5, A.6.2.8 | Art 15, Art 72 | — | — | risk-eng-L25 | 2026-09-30 |
| `ART-HD-FAIR-Q3-01` | Fairness-slice report (mod-106) | v2.0, sha256:8acd... | MEASURE-2.11, MEASURE-2.12 | MANAGE-2.4 | A.6.2.4 | Art 10 | — | — | risk-eng-L25 | 2026-09-20 |
| `ART-HD-PRIV-Q3-01` | Privacy-risk report (mod-107) | v2.0, sha256:a21e... | MEASURE-2.10 | MANAGE-2.4 | A.7.2, A.7.3 | Art 10 | — | `HIPAA-security-rule` (PHI sub-flow carve-out); `GDPR-Art-33` (data-subject rights posture for EU-resident subjects) | risk-eng-L25 | 2026-09-20 |
| `ART-HD-REG-2026-Q3` | Register aggregation report (mod-103 ch 05) | v1, sha256:b4d2... | MEASURE-1.1, MEASURE-2.1 | MANAGE-1.1, MANAGE-1.4, MANAGE-3.1 | A.6.2.2, A.6.2.3, A.6.2.7 | Art 9 | `model-tier` | — | risk-eng-L25 | 2026-09-30 |
| `ART-HD-COUNCIL-Q3` | Council decision-log deltas 2026-Q3 (this module ch 02) | v1, sha256:1e87... | — | MANAGE-3.2, MANAGE-4.3 | A.5.2, A.5.3 (leadership) | Art 9 | — | — | risk-eng-L25 | 2026-09-24 |
| `ART-HD-CARD-2026-09` | Model / system / dataset cards (this module ch 03) | v3.0, sha256:5f08... | MEASURE-4.2 | MANAGE-4.3 | A.6.2.8 | Art 13 (transparency), Art 14 (human oversight — human-in-the-loop requirement named on the card) | — | — | risk-eng-L25 + comms | 2026-09-15 |
| `ART-HD-TIER-2026` | Deployment-tier contract (this module ch 04) | v2.1, sha256:9a0c... | MEASURE-1.3 | MANAGE-1.2, MANAGE-2.1 | A.6.2.4 | Art 15, Art 14 (human oversight — T2 human-in-the-loop) | `model-tier` | — | risk-eng-L25 | 2026-09-24 |
| `ART-HD-INC-2026` | Incident log + post-mortems (mod-111) | rolling, sha256:7c33... | MEASURE-4.3 | MANAGE-4.1, MANAGE-4.2 | A.6.2.5, A.10.1 (improvement) | Art 72 | — | `GDPR-Art-33` (72-hour breach-notification clock) | risk-eng-L25 | rolling |
| `ART-HD-SBOM-2026` | Model-BOM / SBOM (mod-110) | v1.2, sha256:2d59... | MEASURE-2.9 | MANAGE-1.3 | A.6.2.7, A.9 (third-party) | Art 10 | `third-party-risk` | — | risk-eng-L25 + supply-chain | 2026-09-30 |

<!-- The row for `ART-HD-GRD-Q3-01` is Target T-01 — authored first and used as the chapter-05 §"The 'one artefact, many standards' pattern — worked" example. The other ten rows follow the same shape at abbreviated depth; exercise 05 is where the full ten-plus rows get authored with full justification. -->
```

Reading check on the `ART-HD-GRD-Q3-01` row: the artefact has **five framework citations** (MEASURE-2.5, MEASURE-2.6, MEASURE-2.7, MANAGE-2.2, A.6.2.4, Article 15) — the "one artefact, many standards" pattern. Each citation is specific (sub-category ID or article number), not a paragraph. The hash is pinned (`sha256:71bc...`); any refresh of the underlying report will require the hash to be re-pinned. The ISO/IEC 42001 identifier carries a `needs-research` marker because chapter 05 explicitly notes that Annex A identifier pinning requires licensed access to the standard. The SR 11-7 and HIPAA / GDPR columns are empty because the guardrail-effectiveness report does not file against those obligations — Article 15 is the EU AI Act cell and that is the correct citation for guardrails as accuracy / robustness / cybersecurity controls.

### Step 3 — Fill the coverage summary (Section 2 of `crosswalk.md`)

Chapter 05 §"Coverage summary — appended to `crosswalk.md`" pins the summary shape: counts per framework family of `in-scope`, `filled`, `waived`, `escalated`. The counts are approximate for the lab; the discipline is that every in-scope citation has a disposition.

```markdown
## 2. Coverage summary

Counts are computed against the in-scope framework citations enumerated at the top of this crosswalk.

| Framework family | In-scope citations | Filled | Waived | Escalated |
|---|---|---|---|---|
| RMF MEASURE | ~15 sub-cats | 14 | 1 (MEASURE-4.1; see gap-register G-02 waiver for Article-14-shaped sibling-coverage reasoning, not applicable here — MEASURE-4.1 is gap-register G-01 with disposition `fill`) | 0 |
| RMF MANAGE | ~12 sub-cats | 11 | 0 | 1 (release-assurance evidence-bundle packaging; gap-register G-03 → `ai-evaluation-engineer` L35) |
| ISO/IEC 42001 Annex A | ~10 controls | 9 (identifiers carry `needs-research` markers per chapter 05 §"ISO/IEC 42001:2023 — the controls the crosswalk covers") | 1 (A.10.1 improvement — pending the mod-111 post-mortem back-feed; this programme has no completed improvement-cycle yet; waive with rationale pending the first full back-feed) | 0 |
| EU AI Act articles | 6 articles (9, 10, 13, 14, 15, 72) | 6 (Article 14 filled as "one artefact, many standards" on tier contract and card — gap-register G-02 waiver rationale) | 0 | 0 |
| SR 11-7 attributes | 2 (`model-tier`, `third-party-risk`) | 2 | 0 | 0 |
| HIPAA overlay | 1 (`HIPAA-security-rule` for PHI sub-flow) | 1 (filled on the privacy-risk report with PHI sub-flow carve-out) | 0 | 0 |
| GDPR Article 33 overlay | 1 (`GDPR-Art-33`) | 1 (filled on the privacy-risk report and the incident log) | 0 | 0 |

**Total in-scope: ~47; filled: 44; waived: 2; escalated: 1.** No unmarked gaps; every in-scope citation has a disposition. Audit-ready at the row-authoring layer; sign-off on waivers and the escalation is a downstream step (chapter 05 §"Coverage gap detection" — the waive disposition requires rationale, sign-off role, and date, authored in the gap register in Step 4).
```

Reading check: the counts resolve to a no-unmarked-gap state. The RMF MEASURE row's "waived (1)" references MEASURE-4.1 but is corrected to disposition `fill` per gap-register G-01 — the lab text is deliberately shaped so you cross-check the two documents. The EU AI Act Article 14 citation is filled on **two artefacts** (tier contract and card) rather than on a dedicated sibling artefact; this is the "one artefact, many standards" pattern read in reverse — one citation covered by multiple artefacts — and chapter 05 §"Crosswalk shape" names this as "by design."

### Step 4 — Fill the gap register (`gaps.md`)

Chapter 05 §"Coverage gap detection" pins the three-disposition discipline. Each entry names: (i) the framework citation; (ii) the disposition; (iii) for `fill`, which artefact needs to be produced, by whom, by when; for `waive`, the rationale, the sign-off role, and the date; for `escalate`, the destination role and the mod-101 ch 06 contract.

Author the three entries. In `gaps.md`:

```markdown
## Gap entries

### G-01 — MEASURE-4.1 (gathering feedback) · disposition: fill

**Framework citation:** NIST AI RMF 1.0 MEASURE-4.1 — gathering feedback from relevant AI actors on the measurement outcomes.

**Status:** no filled artefact at Q3 2026. The HelpDesk programme has the monitor spec (`ART-HD-MON-2026`) covering MEASURE-3.x (monitoring and tracking) and the incident log (`ART-HD-INC-2026`) covering MEASURE-4.3 (incident feedback), but there is no artefact covering the formal *feedback channel from downstream consumers* chapter 05 §"The NIST AI RMF MEASURE and MANAGE functions" names under MEASURE-4.1 (gathering feedback).

**Disposition: fill.** The gap corresponds to work the programme should be doing. The missing artefact is a *downstream-consumer feedback channel* — a documented mechanism for downstream integrators and end-user-facing teams to feed observations back into the measurement cycle (complaint channel; NPS / satisfaction signal; failed-escalation review loop).

**Artefact to produce:** `ART-HD-FEEDBACK-2026-Q4` — downstream-consumer feedback channel specification, including the channel's intake shape, the triage cadence (bi-weekly working session per chapter 01 §"The four-tier cadence"), and the back-feed path into the register.

**Owner:** risk-engineer-L25 (<name>) + product-owner (<name>).

**Target date:** 2026-11-15 (feeds the 2026-Q4 council pre-read).

**Back-pointer:** when authored, this artefact becomes a new row on the crosswalk citing MEASURE-4.1 (filled) alongside the existing MEASURE-4.2 (cards) and MEASURE-4.3 (incidents) coverage.

### G-02 — EU AI Act Article 14 (human oversight) · disposition: waive with rationale

**Framework citation:** EU AI Act Article 14 — human oversight.

**Status:** Article 14 is **not** carried on a dedicated sibling artefact; it is filled on two existing artefacts — the deployment-tier contract `ART-HD-TIER-2026` (which names the T2 human-in-the-loop requirement as a floor of the tier definition) and the model / system / dataset cards `ART-HD-CARD-2026-09` (which name the human-in-the-loop requirement in the `Uses — Direct Use` and `Residual Risk` sections).

**Disposition: waive with rationale.** The waive is *not* on Article 14 — the article is filled — but on the hypothetical sibling artefact a prescriptive crosswalk reading might expect. Chapter 05 §"Crosswalk shape" §"Many-to-many" is explicit: "One artefact points at multiple framework citations. One framework citation may be pointed at by multiple artefacts... That is by design." A separate "human oversight" sibling artefact would be the chapter-05 §"Motivation" **"1-to-1 duplication"** failure mode — redundant evidence drifting against the already-authoritative tier contract and card.

**Rationale:** the two existing artefacts (tier contract and card) are byte-identical on the T2 human-in-the-loop requirement; a third sibling artefact would drift against them within one refresh cycle. The "one artefact, many standards" pattern (chapter 05 §"The 'one artefact, many standards' pattern — worked") is the economic engine of the crosswalk; its reverse (one citation on multiple artefacts) is equally legitimate when the artefacts agree on the citation.

**Sign-off role:** risk council chair (countersign) + risk-engineer-L25 (<name>) as author.

**Date of waiver sign-off:** <target 2026-10-07; the next bi-weekly working session after the 2026-09-24 council>.

**Re-evaluation condition:** on any divergence between the tier contract and the card on the human-oversight requirement — the waiver is revoked and a sibling artefact is produced. The CI check in Section 3 of the crosswalk surfaces the divergence automatically.

### G-03 — RMF MANAGE "release-assurance evidence-bundle packaging" · disposition: escalate (→ L35)

**Framework citation:** Not a specific MANAGE sub-cat; a cross-cutting MANAGE-family concern. An ISO/IEC 42001 lead auditor asks "how is the per-artefact evidence bundled for external regulator engagement — chain of custody, timestamping, immutable log?" Chapter 05 §"Handing off release-assurance work to `ai-evaluation-engineer` (level 35)" pins this as L35 territory:

> | Level-25 responsibility | Level-35 responsibility |
> | Compute per-artefact hashes | Design the evidence-bundle integrity scheme (chain-of-custody, timestamping, immutable log) |
> | Reconcile RMF / 42001 for team-scope systems | Reconcile release-assurance evidence across systems and audit cycles |
> | File findings back into the register | Regulator-facing evidence discipline (post-market surveillance packet, external-audit response) |

**Disposition: escalate.** The gap is above L25 scope. The per-artefact hashes are L25 responsibility and are pinned on every row above; the *bundle integrity scheme* is L35 responsibility.

**Escalation destination:** `ai-evaluation-engineer` (L35), peer in the Governance family. Mod-101 ch 06 deferral contract cited: "release-assurance methodology depth" escalates to L35.

**Evidence bundle for the hand-off:** the current crosswalk itself (`crosswalk.md` v1), the artefact manifest with hashes (Section 1 rows), the current chain-of-custody discipline at team scope (one risk-engineer-L25 author + per-artefact owner sign-off per chapter 01 §"The sign-off routing graph"), the open question on immutable-log scheme (not yet designed at team scope).

**Escalation artefact:** hand-off packet filed as `handoff/L25-to-L35-2026-Q3-release-assurance.md` in the programme's L35-hand-off workspace.

**Owner of the hand-off:** risk-engineer-L25 (<name>) authors; `ai-evaluation-engineer` (L35) at the programme (<name>) receives.

**Target date:** 2026-10-15 (feeds the 2026-Q4 council signal slide and the L35 release-assurance workstream for 2027-Q1).

**L25 continuing responsibility:** per chapter 05 §"Handing off release-assurance work to `ai-evaluation-engineer` (level 35)," L25 **continues to maintain the citation** (hashes, refreshes) while the L35 work on the bundle integrity scheme proceeds; the escalation is a *pointer*, not a hand-off of maintenance.
```

Reading check: each entry has the three required components for its disposition. For `fill` (G-01): the missing artefact is named, the owner and target date are specified, the back-pointer to the eventual crosswalk row is explicit. For `waive with rationale` (G-02): the rationale is authored (not "out of scope" with no reason), the sign-off role is named, the date is specified, the re-evaluation condition is explicit (chapter 05's discipline that waivers are not permanent — they are time-boxed by condition). For `escalate` (G-03): the destination role is named (L35), the mod-101 ch 06 contract is cited, the evidence bundle for the hand-off is enumerated, the L25 continuing responsibility is explicit (the escalation is a pointer, not a maintenance hand-off).

### Step 5 — Fill the maintenance discipline (Section 3 of `crosswalk.md`)

Chapter 05 §"Maintenance discipline" pins the cadence and the CI check. Author Section 3:

```markdown
## 3. Maintenance discipline

Per chapter 05 §"Maintenance discipline":

- **Monthly** — hash-check every artefact pointer in Section 1. Any stale hash flags the row for refresh (owner re-publishes the artefact and updates the hash cell). Chapter 05: "CI check on the crosswalk file: every citation cell resolves; every referenced artefact exists at the pinned hash; no citation is left blank without a waiver entry."
- **Per framework revision** — when the AI RMF publishes a revision or the ISO/IEC 42001 Playbook / Annex A publishes an amendment, re-check every citation cell and report newly uncovered sub-categories to the gap register (chapter 05 §"Motivation" and §"Maintenance discipline").
- **Per system change** — when a system in scope changes scope or capability, re-check the citations on the affected artefact rows.
- **Per audit cycle** — regenerate the crosswalk into the audit-facing form with narrative per row (chapter 02's pre-read pack shape); the crosswalk delta since the last council packet is a signal slide at the quarterly (chapter 02 §"The pre-read deck — 'one decision per slide'").
- **Per council packet** — the coverage summary (Section 2) is the signal-slide content.

**CI-check wiring (to be implemented at the programme's crosswalk workspace):**
1. `check-hash.sh` — for every row in Section 1, resolve the artefact pointer and compare the file's SHA-256 against the `Version / hash` cell. Any mismatch fails the check.
2. `check-citation.sh` — for every citation cell (RMF, ISO 42001, EU AI Act, overlays), verify the cell contains a specific identifier (sub-cat, control, article) or `—` (empty-by-design) or a `<!-- needs-research: ... -->` marker. Any blank cell without a marker fails the check.
3. `check-gap-disposition.sh` — for every citation in the in-scope list that is not filled on any row, verify a corresponding entry exists in `gaps.md` with a disposition. Any unmarked gap fails the check.

**Next scheduled refresh:** 2026-10-31 (monthly hash check); 2026-12-15 (pre-2026-Q4-council regeneration).
```

Reading check: the maintenance section ties the row shape to the per-cadence refresh pattern. The CI-check wiring enumerates three specific scripts chapter 05 §"Maintenance discipline" names (the scripts themselves are exercise-05 stretch-goal shape; the lab just names the wiring).

### Step 6 — Final checkpoint

Re-read the two files. Confirm:

- **`crosswalk.md` Section 1** has one row per evidence artefact (eleven rows); every row has an artefact ID, version/hash (with `sha256:` prefix), framework citations in specific identifier form (not paragraphs), owner, and last-refresh date. The `ART-HD-GRD-Q3-01` row explicitly instantiates the "one artefact, many standards" pattern with five framework citations (MEASURE-2.5/2.6/2.7, MANAGE-2.2, A.6.2.4, Article 15).
- **`crosswalk.md` Section 2** has a coverage summary counting filled / waived / escalated per framework family. Total citations resolve to zero unmarked gaps.
- **`crosswalk.md` Section 3** pins the monthly / per-revision / per-system-change / per-audit-cycle maintenance cadence and names the three CI-check scripts.
- **`gaps.md`** has three entries — one per disposition class (fill, waive with rationale, escalate). Each entry has the chapter-05-required components for its disposition (artefact-and-owner-and-date for fill; rationale-and-sign-off-and-re-evaluation for waive; destination-and-evidence-bundle-and-mod-101-contract for escalate).
- No cell in the crosswalk reads "TBD" or "unclear" or is left blank without a `needs-research` marker. No waiver reads "out of scope" without a rationale. No escalation reads "escalate" without a destination role and the mod-101 ch 06 contract cited.

## Expected output

`mod112-lab03/crosswalk.md` and `mod112-lab03/gaps.md` — two populated documents; one crosswalk with eleven artefact rows and a coverage summary; one gap register with three entries covering the three disposition classes. Two to three pages total across both files. The crosswalk's row-shape and the gap register's three-disposition shape are the two load-bearing artefacts; the coverage summary is the one a council reads on the quarterly signal slide (chapter 02 §"The pre-read deck — 'one decision per slide'"); the maintenance section is the one an auditor reads to confirm the crosswalk is operating.

## Checkpoint questions (self-check)

1. The crosswalk row for `ART-HD-GRD-Q3-01` cites five framework citations — MEASURE-2.5, MEASURE-2.6, MEASURE-2.7, MANAGE-2.2, A.6.2.4, and EU AI Act Article 15. Chapter 05 §"The 'one artefact, many standards' pattern — worked" says the same artefact files against **seven** distinct downstream uses. What are the other two uses, and why are they not framework-citation columns on the crosswalk row? (*Chapter 05 §"The 'one artefact, many standards' pattern — worked" enumerates the seven uses of the guardrail-effectiveness report: (1) MEASURE-2.5; (2) MEASURE-2.7; (3) MANAGE-2.2; (4) ISO/IEC 42001 Annex A verification-and-validation; (5) EU AI Act Article 15; (6) the programme's internal deployment-tier contract as a threshold input; (7) the model / system card as an `Evaluation` section input; (and (8) the council pre-read decision slides as an evidence pointer, which chapter 05 adds as a bonus beyond seven). The last three (tier contract input; card input; council pre-read pointer) are **not framework citations** — they are **back-pointers to other crosswalk rows** (`ART-HD-TIER-2026`, `ART-HD-CARD-2026-09`, `ART-HD-COUNCIL-Q3`). Chapter 05 §"Crosswalk shape" pins the shape: "the table is many-to-many — one artefact points at multiple framework citations, and one framework citation may be pointed at by multiple artefacts" — but the artefact-to-artefact relationships are modelled as **separate crosswalk rows citing overlapping framework columns**, not as artefact-to-artefact columns. The downstream uses of an artefact are read off by looking at which other rows cite the same framework columns; this keeps the crosswalk shape mechanical and avoids the chapter-05 §"Motivation" **"1-to-1 duplication"** failure mode (an artefact-to-artefact matrix).*)

2. Gap G-02 (EU AI Act Article 14) is disposition **waive with rationale** even though Article 14 is in-scope for a high-risk system and the programme has a filled citation for it on two artefacts (tier contract and card). What is the waiver actually waiving, and what re-evaluation condition is specified? (*Gap G-02's waiver is **not** on Article 14 — the article is filled on two existing artefacts — but on the hypothetical sibling "human-oversight-only" artefact that a prescriptive crosswalk reading might expect. Chapter 05 §"Crosswalk shape" pins the many-to-many discipline: "one framework citation may be pointed at by multiple artefacts... That is by design." A separate sibling artefact would be the chapter-05 §"Motivation" **"1-to-1 duplication"** failure mode — redundant evidence drifting against the authoritative tier contract and card. The waiver is specifically waiving the sibling-artefact requirement; Article 14 itself is filled. The re-evaluation condition is explicit: "on any divergence between the tier contract and the card on the human-oversight requirement — the waiver is revoked and a sibling artefact is produced. The CI check in Section 3 of the crosswalk surfaces the divergence automatically." Chapter 05 §"Coverage gap detection" pins the discipline: every `waive` disposition has a rationale, a sign-off role, and a date; chapter 05 §"Maintenance discipline" wires the re-evaluation into the monthly / per-system-change cadence. Waivers are not permanent; they are time-boxed by a specific re-evaluation condition that the CI check surfaces.*)

3. Gap G-03 (release-assurance evidence-bundle packaging) is disposition **escalate** to `ai-evaluation-engineer` (L35), not to `senior-ai-governance-architect` (L50) or `head-of-ai-governance` (L60). What distinguishes the L35 scope from the L50 scope in chapter 05's hand-off tables, and why is this gap specifically L35? (*Chapter 05 §"Handing off release-assurance work to `ai-evaluation-engineer` (level 35)" pins the table: L25 computes per-artefact hashes; **L35 designs the evidence-bundle integrity scheme (chain-of-custody, timestamping, immutable log)**; L25 reconciles RMF / 42001 for team-scope systems; **L35 reconciles release-assurance evidence across systems and audit cycles; L35 does the regulator-facing evidence discipline (post-market surveillance packet, external-audit response)**. Chapter 05 §"Handing off control-library architecture to `senior-ai-governance-architect` (level 50)" pins a different table: L25 cites existing enterprise controls in the crosswalk; **L50 authors and versions the enterprise control library itself**; L25 reports gaps and duplications encountered in team-scope work; **L50 rationalises the control library across systems and jurisdictions; L50 designs the Statement of Applicability (SoA) for ISO/IEC 42001 at organisation scope**. The gap on release-assurance evidence-bundle packaging is **L35** because it is a *release-assurance methodology* question (how is the evidence bundled for regulator engagement?), not a *control-library architecture* question (which enterprise controls apply across systems and jurisdictions?). The sibling gap in the HelpDesk programme that would route to **L50** is "which enterprise control library applies and how is the SoA structured" — a different question with a different escalation destination. Chapter 05 §"What the crosswalk covers" row for "Enterprise ISO/IEC 27001 / SOC 2 / internal control library" is explicit: "Consumed as parent, not authored here; escalates to `senior-ai-governance-architect` (L50)." The deferral contract from mod-101 ch 06 draws both boundaries so the escalation is to the specific peer / senior role whose scope the gap actually sits in.*)

4. The coverage summary in Section 2 of `crosswalk.md` resolves to "no unmarked gaps." Chapter 05 §"Coverage gap detection" says "a crosswalk with unmarked gaps is not shippable to audit." What makes the summary a *mechanical* check rather than a *judgement call*, and how does the CI-check wiring in Section 3 enforce it? (*Chapter 05 §"Coverage gap detection" pins the check mechanically: "Enumerate every in-scope MEASURE / MANAGE sub-category the programme claims. Enumerate every in-scope ISO/IEC 42001 Annex A control the programme claims. Subtract the set of sub-categories / controls with at least one crosswalk row. Report the remainder as *gaps*." The `in-scope` set is enumerated at the top of the crosswalk; the `with at least one crosswalk row` set is read off Section 1; the subtraction is mechanical. §"Maintenance discipline" wires the mechanical check into a CI script (`check-gap-disposition.sh` in Section 3): "every citation in the in-scope list that is not filled on any row has a corresponding entry in `gaps.md` with a disposition. Any unmarked gap fails the check." The three-disposition discipline (fill / waive / escalate) converts the open-ended question "is this covered?" into a closed question "which disposition does this gap take?" — a crosswalk with every gap dispositioned is audit-ready; a crosswalk with any gap undispositioned fails the CI check and is not shippable. "TBD" is not a disposition — chapter 05 is explicit: "Every framework cell must be one of: filled, waived, or escalated." The mechanical check, the three-disposition closed-question shape, and the CI-check wiring together make the crosswalk an auditable artefact rather than a prose claim ("we address MEASURE 2.5 through our red-team programme") the auditor has to trust rather than verify.*)

## Why this lab matters

Exercise 05 asks you to build the full crosswalk across ten-plus artefacts on your own primary running system, author the gap register, and run coverage detection against the full in-scope framework-citation set. If the row shape is new at exercise-05 authoring time, the budget blows on relitigating whether "the register" counts as one artefact or many, whether a prose paragraph discharges a framework citation, or whether "TBD" is a disposition. This lab drills the atomic row shape and the three-disposition gap shape on a fixed fabricated system with the chapter-05 worked example (`ART-HD-GRD-Q3-01` filing against seven downstream uses) as the first row, and three gap entries covering all three disposition classes. Getting the row shape and the gap-disposition shape right here cascades through exercise 05 (whose main deliverable is exactly this shape at full programme scale) and connects back to lab 01 (whose decision-slide `linked_obligations` field cites into the crosswalk's framework columns) and lab 02 (whose card residual claim is itself a crosswalk-row artefact filing against MEASURE-4.2 and Article 13). The crosswalk is the single artefact an ISO/IEC 42001 lead auditor reads first on-site; the lab is the drill that lets the full crosswalk be assembled from already-rehearsed rows rather than from first principles.
