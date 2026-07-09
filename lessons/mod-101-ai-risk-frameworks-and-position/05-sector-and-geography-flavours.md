# Sector and Geography Flavours of the Same Shape

Once you know the NIST + ISO + EU AI Act shape (chapters 01-04), the sector-specific regimes stop being surprising: financial-services model-risk (SR 11-7 / OCC 2011-12 / SR 23-4), health medical-device ML (FDA GMLP + PCCP), and US state consumer-protection AI (Colorado AI Act) are *the same governance shape with different label sets, different sign-off routing, and one or two distinctive obligations each*.

## Motivation

If you can already produce a risk register, a harm model, evaluation reports, monitoring wiring, and post-mortems, moving into a regulated sector is largely a *packaging* exercise. This chapter teaches you the labels each regime uses and the one or two obligations that differ.

## Financial services — SR 11-7, OCC 2011-12, SR 23-4

Regulated US financial institutions (FIs) run *Model Risk Management* (MRM) programs. Even where the model in question is a modern ML/LLM system, the MRM shape is the one banking supervisors already speak. The three key documents:

### SR 11-7 (US Federal Reserve, 2011) + OCC 2011-12 (companion)

SR 11-7 defines **model risk** — the potential for adverse consequences from decisions based on incorrect or misused model outputs — and prescribes a lifecycle: development, implementation, and use; validation; governance, policies, and controls. The pillars it emphasises:

- **Independence** of validation from development.
- **Conceptual soundness** review — is the model design fit for purpose?
- **Ongoing monitoring** — including outcome analysis and back-testing.
- **Effective challenge** — organised, active questioning of the model.

The **OCC Bulletin 2011-12** is the OCC's companion supervisory guidance, functionally equivalent to SR 11-7 for OCC-regulated institutions.

### SR 23-4 (US Federal Reserve, 2023) — third-party risk

SR 23-4 is the *Interagency Guidance on Third-Party Relationships*. For AI, this is the shape you apply to third-party models, third-party datasets, and third-party evaluation vendors. It covers planning, due diligence, contract negotiation, ongoing monitoring, and termination.

### The MRM-to-your-shape crosswalk

| MRM concept | Your equivalent | Where it lives |
|---|---|---|
| Model documentation | Model card + dataset card + risk-engineering slice (mod-112) | `model-cards/` |
| Conceptual-soundness review | Harm model + assumptions review (mod-102) + risk register (mod-103) | `harm-model/`, `risk-register/` |
| Model validation (independent) | Independent evaluation runs, red-team, adversarial-ML, fairness, privacy (mods 104-107) | `evidence/` |
| Effective challenge | Red-team suite output + adversarial-ML suite output | `evidence/mod-104/`, `evidence/mod-105/` |
| Ongoing monitoring | mod-109 monitoring wiring | `monitors/` |
| Outcome analysis | Post-market surveillance report | `reports/pms/` |
| Model inventory | System inventory (owned at level 15) | Governance system, not your repo |
| Third-party risk (SR 23-4) | Vendor build-vs-buy matrix (mod-108), ML-BOM, provenance evidence (mod-110) | `bom/`, `evidence/mod-110/` |

**Independence** is the one thing that is *not* just a label swap. In an SR 11-7 MRM program, the validator cannot report to the developer. At level 25 you are usually inside the validation function or between it and development; the AI risk council (mod-112) is the neutral forum. You will typically hand release-assurance sign-off up the ladder to `ai-evaluation-engineer` (level 35).

## Health — FDA GMLP + PCCP

If the AI system is a *Software as a Medical Device* (SaMD), you are in FDA (or Health Canada, or MHRA) territory. Two documents:

### Good Machine Learning Practice (GMLP)

The FDA / Health Canada / MHRA **Good Machine Learning Practice for Medical Device Development: Guiding Principles** (October 2021) is ten principles that cover multi-disciplinary teams, security, dataset quality, independent test sets, human factors, transparency to users, monitoring after deployment, and re-training discipline.

### Predetermined Change Control Plan (PCCP)

The FDA **PCCP guidance** (finalised 2024) is the shape you use when the ML medical device *learns and changes over time*. It requires:

- a **Description of Modifications** — what will change, in what way,
- a **Modification Protocol** — the methodology by which modifications will be verified, validated, and monitored,
- an **Impact Assessment** for each type of change.

### The crosswalk

| GMLP principle | Your equivalent |
|---|---|
| Multi-disciplinary expertise | The cross-functional AI risk program (mod-112) |
| Good software engineering + security practices | Prerequisite (`ai-infra-junior-engineer`) + mods 108-110 |
| Clinical study participants + data representative of intended population | Dataset card + fairness slice on protected populations (mod-106) |
| Independent training vs test data | ML fundamentals (`ml-engineer`); you audit it |
| Reference-standard-quality labels | Data-quality documentation in dataset card |
| Model designed to fit clinical workflow | Human-oversight design (mod-108, adjacency to EU AI Act Article 14) |
| Focus on performance of the human-AI team | The oversight design document + human-factors evaluation |
| Testing demonstrates performance across intended conditions | Robustness eval (mod-105), stratified performance report |
| Users provided clear, essential information | Model card + user-facing labelling |
| Deployed models monitored + retraining risks managed | mod-109 monitoring + mod-110 supply-chain |

### PCCP-specific engineering

For a PCCP-eligible device, your artifact set adds:

- a **change taxonomy** — which modifications are inside the PCCP envelope and which require a new submission,
- a **verification-and-validation protocol** for each modification type — this is essentially a *release-gate specification* keyed to change type,
- a **post-market performance report** — the same monitoring wiring as EU AI Act Article 72 and NIST MEASURE 3.x, filed to FDA-required shape.

## US state consumer protection — Colorado AI Act (SB 24-205)

The **Colorado AI Act** (SB 24-205, signed May 2024, effective 2026) applies to *high-risk AI systems* used in consequential decisions (employment, education, financial or lending services, essential government services, health-care services, housing, insurance, legal services). It imposes duties on *developers* and *deployers* to use reasonable care to protect consumers from known or reasonably foreseeable risks of algorithmic discrimination.

Its named engineering-shaped requirements include:

- **Impact assessment** for high-risk AI systems (Colorado's version of the ISO/IEC 42005 shape).
- **Risk-management program** at the deployer — resembling NIST AI RMF or a recognised equivalent, which the statute names.
- **Notice to consumers** when a high-risk AI system is being used to make a consequential decision about them.
- **Notification to the Attorney General** of algorithmic discrimination discovered in deployment.

### What you build (that's not already built)

- A **notice / disclosure text** contributed to product wording.
- An **algorithmic-discrimination detection and reporting hook** on the fairness monitor (mod-106, mod-109) — the *legal* threshold is set by counsel; you supply the *technical* signal.
- The impact-assessment artifact — already covered under ISO/IEC 42005 (chapter 03).

Other US states have proposed or passed related laws — *always check the current status with the analyst layer before designing to them*. NYC Local Law 144 (AEDT) and California's ADMT-shaped regulations are two of the more mature adjacent regimes.

<!-- needs-research: track the current status and effective dates of Colorado AI Act delayed-implementation amendments and the analogous California ADMT regulations before finalising design assumptions. -->

## The "same shape, different labels" map

| Concept | NIST AI RMF | ISO/IEC 42001 | EU AI Act | SR 11-7 MRM | FDA GMLP + PCCP | Colorado AI Act |
|---|---|---|---|---|---|---|
| The risk process | GOVERN + MAP + MEASURE + MANAGE | Clauses 4-10 + Annex A controls | Article 9 | SR 11-7 lifecycle (dev / validation / use) | GMLP #7-#10 | RMP + IA |
| The impact assessment | MAP function | Guidance in 42005 | Article 27 (fundamental-rights impact assessment) + Article 61 | Model documentation | Clinical evaluation, PCCP impact section | IA required |
| The evaluation | MEASURE function | Annex A controls | Article 15 | Validation | GMLP #4 (independent test) + #8 (perf across conditions) | Reasonable-care duty |
| The monitoring | MEASURE 3.x / MANAGE 4.x | Clause 9 | Article 72 | Ongoing monitoring | GMLP #10 + PCCP post-market | RMP + AG notification |
| The incident | MANAGE 4.3 | Clause 10 | Article 73 | Model-issue escalation | MDR (medical-device reporting) | AG notification |
| The oversight | GOVERN 5.x | Leadership clause | Article 14 | Effective challenge | GMLP #6-#7 | Deployer duty of care |

That is the point of this chapter: *once you can produce the columns on the left, the columns on the right are packaging.*

## What is *not* the same shape

Three things do genuinely differ per regime:

- **Sign-off routing.** Who signs, in what order, before what release. This is the thing you have to learn on the ground per organisation, and it is largely owned above your level.
- **Independence rules.** SR 11-7 is unusually strict; FDA is regime-by-regime; EU AI Act imposes some independence via notified-body conformity assessment for high-risk systems.
- **Named jurisdictional artifacts.** Article 61 registration, Colorado's AG notification, FDA 510(k) or De Novo or PMA, EU AI Act Annex IV technical documentation — those are named documents you produce for a named authority, not just internal artifacts.

The list of named-artifacts-per-regime is where the analyst layer (level 15) does most of its work; you *populate* the named artifacts with your engineering evidence.

## Summary

- SR 11-7 / OCC 2011-12 / SR 23-4, FDA GMLP + PCCP, and the Colorado AI Act are sector / geography *flavours* of the NIST + ISO + EU AI Act shape.
- The main things that differ are label sets, sign-off routing, and one or two named jurisdictional artifacts per regime.
- Independence rules (particularly under SR 11-7) genuinely differ and shape how release-assurance is done.
- Exercise 05 grounds this by picking a target regime and mapping the label vocabulary.
