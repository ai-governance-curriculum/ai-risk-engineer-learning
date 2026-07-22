# The Certifications Portfolio and Career Shape

This is the final chapter of the final module of the track. It closes two loops. First, it names the certifications and credentials that are load-bearing in enterprise AI-risk-engineering hiring, so a working risk engineer can plan a defensible portfolio rather than collecting badges opportunistically. Second, it names the career hand-offs into the level-35, level-40, and level-50 tracks — the natural next-up moves once the twelve-module scope of this role is in hand.

The chapter is deliberately less prescriptive than the others: certification programmes evolve, geographic and sector conventions vary, and the specific requirements of individual credentials change per cycle. What is stable is the *shape of the decision* — which credential answers which hiring question, and which credential clusters signal readiness for the next-up track.

## Motivation

Two failure modes in certification planning at level 25:

- **Badge collection.** The engineer holds five credentials that overlap in coverage and none in depth. On a hiring page the portfolio reads as breadth signalling with no evidence of applied depth.
- **Wrong-programme investment.** The engineer invests six months and several thousand pounds / dollars in a credential that turns out to be marginal to their target employer's hiring signal — often because they picked the credential most publicised in the year they started rather than the one enterprise hiring actually filters on.

The productive posture treats certifications as *hiring signals to specific employers*, and picks the two-to-four that are load-bearing for the target employer type. This chapter names the credential families, what each signals, and the decision table for "which one when."

## The five credential families relevant to level 25

There are more than five credentials in the AI-governance space; the five below are the ones that most consistently appear in enterprise AI-risk-engineering job descriptions and RFP responses at the moment of authoring.

### 1. IAPP AI Governance Professional (AIGP)

Issued by the International Association of Privacy Professionals. Programme covers AI governance principles across law, policy, and practice, with content spanning international frameworks (EU AI Act, NIST AI RMF, ISO/IEC 42001 at the awareness level), risk-management practices, and the AI lifecycle. Positioning: *breadth* credential for AI governance.

- Signals: familiarity with the full AI-governance surface across jurisdictions; often the first credential named in a governance job description.
- Renewal: continuing-education-credit model (IAPP model).
- Fit for level 25: high. AIGP is the credential most often listed as "required" or "strongly preferred" in enterprise AI-governance postings that expect risk-engineering depth.

<!-- needs-research: pin the current IAPP AIGP body of knowledge, exam format, cost, and renewal-credit requirements at authoring time; the programme evolves per cycle. -->

### 2. ForHumanity Independent AI Auditor

Issued by ForHumanity, a nonprofit that maintains audit criteria for algorithmic systems (including criteria aligned to specific regulations — NYC Local Law 144, the EU AI Act's high-risk conformity assessments, and others). The Independent AI Auditor credential focuses on the *audit* discipline: evidence review, control testing, opinion formation. There are subsidiary credentials for specific audit specialisations.

- Signals: capability to conduct or contribute to an independent audit of an AI system; particularly meaningful for engineers moving into third-party audit / assurance work.
- Renewal / recertification: ForHumanity credential-maintenance model.
- Fit for level 25: medium-high, especially for engineers whose employer is an audit firm, a large regulated enterprise with an internal-audit function, or a service provider positioning as an AI-assurance partner. For a purely internal risk-engineering role, this credential is a strong nice-to-have.

<!-- needs-research: pin the current ForHumanity credential structure, its named audit specialisations (bias audit, EU AI Act audit, etc.), the training-provider list, and renewal cadence at authoring time. -->

### 3. BABL AI Algorithm Bias Auditor

Issued by BABL AI, providing training and certification focused on algorithmic bias auditing (with specific coverage of jurisdictional requirements such as NYC Local Law 144, EU AI Act bias-and-fairness obligations, and adjacent regulations). Positioning: *depth* credential specifically in fairness / bias auditing.

- Signals: depth in fairness / bias assessment methodology; especially meaningful if the target employer must comply with a bias-audit regulation (NYC LL 144 is the prominent example).
- Renewal: per BABL AI's own cadence.
- Fit for level 25: medium in general; high if the employer or client base is subject to a bias-audit regulation.

<!-- needs-research: pin the current BABL AI credential name, curriculum, exam format, and renewal cadence; the programme evolves. -->

### 4. ISO/IEC 42001 Lead Implementer (LI) and Lead Auditor (LA)

Issued by accredited training organisations (PECB, BSI, TÜV, DNV, and others) against ISO/IEC 42001:2023. The Lead Implementer credential focuses on setting up and running an AIMS; the Lead Auditor credential focuses on auditing one. Positioning: *depth* credential in the standard the enterprise most commonly reports its AI-governance posture against.

- Signals: capability to design, implement, and audit an AI Management System per the international standard. In organisations that are ISO/IEC 27001- or 42001-certified, this is the credential that maps most directly to the AIMS operating model.
- Renewal: per accredited training body's cadence.
- Fit for level 25: high for engineers in ISO-mature organisations or in organisations targeting 42001 certification. For an AIMS lead implementer role or a lead-auditor career move, this credential is essentially the ticket.

<!-- needs-research: pin the current list of accredited training providers for ISO/IEC 42001 LI / LA, the exam formats, and any national-body-specific variants; the accreditation landscape expands with adoption. -->

### 5. IAPP CIPP/US or CIPP/E (privacy adjacency)

Issued by IAPP. CIPP/US covers US privacy law; CIPP/E covers European privacy law (GDPR primarily). Not an AI-governance credential per se; it is the *privacy adjacency* that many AI-risk-engineering roles require because privacy risk (mod-107) sits inside AI risk.

- Signals: working understanding of the applicable privacy regime; often required or preferred alongside AIGP for combined AI-and-privacy governance postings.
- Renewal: IAPP continuing-education-credit model.
- Fit for level 25: medium-high in organisations where the privacy team and the AI-governance team share people or reporting lines, and in jurisdictions where privacy regulators are also the AI regulators.

<!-- needs-research: confirm the current CIPP/US and CIPP/E bodies of knowledge and exam formats; both are stable but versioned. -->

## Which one when — a decision table

The decision table below pairs an employer / role profile with the credential portfolio that best matches. It is a starting point, not a rule.

| Employer / role profile | First credential | Second credential | Third (optional) | Rationale |
|---|---|---|---|---|
| Enterprise (financial services, healthcare, insurance), internal AI-governance / risk-engineering role, ISO/IEC 27001 / 42001 mature | AIGP | ISO/IEC 42001 LI | CIPP/E or CIPP/US | Breadth then depth in the standard the enterprise reports against; privacy adjacency for combined roles |
| Enterprise, internal AI-governance role, US-headquartered, less ISO exposure | AIGP | CIPP/US | (ForHumanity or BABL AI depending on jurisdictional obligations) | AIGP as baseline; CIPP/US for jurisdictional privacy; specialist audit credential where regulation demands |
| Audit firm or AI-assurance service provider, external-audit role | ForHumanity Independent AI Auditor | ISO/IEC 42001 LA | AIGP | Depth in audit discipline first; lead auditor for AIMS engagements; breadth for client-facing conversation |
| Consulting firm, mixed AI-governance advisory | AIGP | ISO/IEC 42001 LI | ForHumanity or BABL AI as specialisation | Breadth for advisory conversation; implementer depth for delivery; specialisation where the practice differentiates |
| Any role covering NYC Local Law 144 obligations for hiring / employment algorithms | BABL AI Algorithm Bias Auditor | AIGP | ForHumanity Independent AI Auditor | Regulation-specific depth first; breadth second; audit discipline for defensible reporting |
| Engineer targeting level-40 / agentic-safety-engineer move | AIGP | (ISO/IEC 42001 LI *or* deep in RSP / Preparedness / FSF reading — see the level-40 track) | — | AIGP as baseline; the level-40 track's depth is largely non-credentialled at authoring time |
| Engineer targeting level-50 / senior-ai-governance-architect move | AIGP | ISO/IEC 42001 LI *and* LA | CIPP/E or CIPP/US | Architect-scope roles typically require both implementer and auditor perspectives on the standard |

<!-- needs-research: employer-side hiring conventions vary widely and change per year; the table above reflects a current-state working recommendation, not a static rule. Verify against the target employer's recent postings before investing. -->

## What is *not* on the list

Several other credentials appear in the AI-risk / governance space (Certified AI Compliance Manager and variants from specific training bodies; various vendor-specific AI-safety programmes; certificate-of-completion programmes from AI-focused executive-education courses). Some are useful; most are supplementary rather than load-bearing in enterprise hiring at the moment of authoring. The five families above are the ones most likely to appear as a filter in an enterprise job description or an RFP response.

Two additional categories that are *not* certifications but that carry hiring signal:

- **Named engagement or contribution to a public evaluation programme.** Contributing to the MLCommons AI Safety benchmarks, or to a UK / US AISI evaluation, or to an open-source red-team programme, is a hiring signal that no certification replaces.
- **Publications and speaking at recognised venues.** A paper at NeurIPS / ICML / FAccT, a talk at RSA / DEF CON / Black Hat with an AI-safety angle, or a technical blog post referenced by a standards body are hiring signals that certifications complement rather than compete with.

<!-- needs-research: the evaluation-programme and venue names above are illustrative; the community's canonical publication and speaking venues for AI risk evolve, and the current top-signal venues should be verified per hiring cycle. -->

## The career hand-off to level 35 / 40 / 50

The twelve-module scope of this track is the engineering-craft scope of AI risk end-to-end at team scope. Three natural next-up moves from here, each in the same Governance family:

### To `ai-evaluation-engineer` (level 35, Governance)

Same family, one level up. Focus shifts from *risk-engineering across the full surface* to *release-assurance and audit-trail methodology depth*. What the level-25 role produced as artefacts, the level-35 role packages as evidence bundles for regulator engagement and external audit. Certifications that map well into the transition: ISO/IEC 42001 LA (lead auditor), ForHumanity Independent AI Auditor, IAPP AIGP.

Concrete cross-training from level 25 that eases the transition:

- Deep familiarity with the crosswalk (this module ch 05) and gap-disposition discipline.
- Ownership of a full audit cycle end-to-end, even if the audit is internal.
- Experience packaging evidence for a real regulator engagement (EU AI Act post-market surveillance packet; sector regulator response).

### To `agentic-safety-engineer` (level 40, Governance)

Same family, two levels up. Focus shifts from *enterprise-deployment AI risk* to *frontier-agent red-team methodology and dangerous-capability evaluation depth*. The level-40 role owns what mod-102 ch 04 and this module ch 04 named as escalation territory: frontier capability categories, frontier evaluation harnesses, RSP / Preparedness / FSF depth. This track has been designed to make the escalation contract explicit, so the level-40 move is a *specialisation* rather than a departure.

Certifications map less directly to level 40 at the moment of authoring — the depth signal is often research-programme contribution, publication, or engagement with AI safety institutes.

<!-- needs-research: track any emerging credential specifically targeted at frontier-safety / dangerous-capability evaluation depth; as of authoring time, the discipline is largely non-credentialled. -->

Concrete cross-training from level 25 that eases the transition:

- Deep experience with mod-104 (LLM red-team) and mod-105 (adversarial ML) beyond the module minimums.
- Contribution to a public red-team programme or an AISI-style evaluation.
- Reading of the RSP / Preparedness / FSF depth well beyond the shape-template scope of this track.

### To `senior-ai-governance-architect` (level 50, Governance)

Same family, three levels up. Focus shifts from *engineering craft for team-scope systems* to *control-library architecture, policy taxonomy, cross-jurisdiction reconciliation, and Statement of Applicability authorship at organisation scope*. This move requires breadth across sectors and jurisdictions that the level-25 role touches but does not own.

Certifications map well: ISO/IEC 42001 LI + LA, AIGP, CIPP/E + CIPP/US.

Concrete cross-training from level 25 that eases the transition:

- Contribution to enterprise crosswalk work beyond your systems (the analyst-facing crosswalks; the sector-overlay crosswalks).
- Exposure to a real Statement of Applicability authoring cycle for ISO/IEC 42001.
- Cross-jurisdiction reconciliation for at least one system operating in multiple regions.

### Sideways cross-training that stays inside level 25

Two peer-level moves that strengthen the role rather than promoting out of it:

- **`ai-eval-engineer` (peer, level 30, AI Engineering family)** — application-layer eval depth (traces, judges, RAG eval, eval-gated CI/CD). Cross-trains directly with mod-104 and mod-108.
- **`model-evaluation-engineer` (peer, level 30, ML Engineering family)** — statistical methodology depth. Cross-trains directly with mod-103 (register scoring), mod-105 (adversarial ML), and mod-106 (fairness).

## A working planning template

At the end of this module, a defensible five-year certification and career plan for a level-25 risk engineer typically looks like:

- **Year 1** — AIGP (breadth baseline); the twelve-module track work applied to at least one real system.
- **Year 2** — ISO/IEC 42001 LI *or* CIPP/E / CIPP/US (depth in the enterprise's most relevant standard).
- **Year 3** — audit-side depth (ForHumanity Independent AI Auditor, or ISO/IEC 42001 LA) if moving toward L35; agentic-safety depth (public red-team / AISI contribution) if moving toward L40; architect-side depth (SoA authorship experience) if moving toward L50.
- **Years 4-5** — target credential for the chosen next-up track; sideways cross-training in one of the peer families.

That cadence keeps investment modest (two significant credential efforts in the first three years, one for the next-up track after that) and matches the shape of enterprise-hiring signal.

## Interfaces to earlier modules

- **mod-101** — the framework mapping in ch 01-05 covers the RMF, ISO, and EU AI Act content that AIGP and ISO/IEC 42001 LI credentialing curricula overlap with; the deferral contract in ch 06 sets the level-35 / 40 / 50 boundary this chapter operationalises for career shape.
- **mod-102 through mod-111** — the engineering-craft depth that certifications complement rather than substitute for; hiring managers filter on credential *and* portfolio of engineering artefacts.
- **This module chapters 01-05** — the programme-scope work (charter, council packet, model cards, tiering, crosswalk) that is the portfolio the certifications are meant to signal readiness to produce.

## Summary

- Five credential families cover the enterprise AI-risk-engineering hiring surface at the moment of authoring: IAPP AIGP (breadth), ForHumanity Independent AI Auditor (audit-side depth), BABL AI Algorithm Bias Auditor (fairness-specific depth), ISO/IEC 42001 LI / LA (standard depth), and IAPP CIPP/US or CIPP/E (privacy adjacency).
- The "which one when" decision table pairs employer / role profile with a two-to-four credential portfolio; the portfolio choice is a hiring-signal decision to a specific employer type, not a badge-collection exercise.
- Certifications complement rather than replace engineering artefacts and public contributions; portfolio, publications, and public-evaluation engagement are load-bearing alongside credentials.
- The career hand-offs from level 25 are into `ai-evaluation-engineer` (L35, release-assurance depth), `agentic-safety-engineer` (L40, frontier / agent depth), and `senior-ai-governance-architect` (L50, architecture depth) — all in the same Governance family.
- Sideways cross-training into `ai-eval-engineer` (L30, AI Engineering family) or `model-evaluation-engineer` (L30, ML Engineering family) strengthens the level-25 role without promoting out of it.
- A working five-year plan invests in AIGP first, one depth credential in year two, and the target next-up track's credential in year three, with sideways cross-training over years four-five.
- Exercise 06 walks the certifications-portfolio planning template for the reader's own employer profile and target career trajectory.
