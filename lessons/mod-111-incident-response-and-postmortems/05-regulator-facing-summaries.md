# Regulator-Facing Incident Summaries — EU AI Act Article 73, SR 11-7, FDA MDR

The internal post-mortem (chapter 03) is the source of truth. What a regulator reads is a reshaping of that source into the format the regulator expects, on the clock the regulator sets. This chapter defines three of those reshapings — the EU AI Act Article 73 serious-incident summary, the SR 11-7-shaped FI model risk management incident report, and the FDA Medical Device Reporting (MDR) shape for AI / SaMD incidents — plus the "one incident, three regulator-facing packages" pattern that keeps the reshapings consistent, and the timing envelopes each obligation imposes.

## Motivation

Three failure modes for regulator-facing summaries authored in the field:

- **The re-authored post-mortem.** The regulator summary is written from scratch weeks after the internal post-mortem. Facts drift; the regulator submission and the internal file disagree on the timeline. The disagreement becomes the finding, not the incident.
- **The wrong-shape package.** The team submits the internal post-mortem verbatim to a regulator that expected a specific field structure (Article 73 categories, MDR Form 3500A fields, an SR 11-7 material-model-risk narrative). The submission is returned as non-conforming; the clock does not reset, and the organisation is late.
- **The clock miscount.** The clock starts at *awareness* of a serious incident, not at *confirmation*. The team treats the clock as starting on internal post-mortem sign-off. Days are lost. The subsequent regulator interaction opens with a lateness finding.

The engineering discipline is to define each obligation's format, start-of-clock, and duration once, and to generate the regulator package as an extract from the single post-mortem file (chapter 03) rather than re-authoring.

## The three obligation shapes and their timing envelopes

Named laws and rules as published. Direction of citation only; the source-of-truth timing and format are the published texts themselves — see the regulatory-anchors list in `resources.md`.

### EU AI Act — Article 73 serious-incident reporting

Regulation (EU) 2024/1689 Article 73 obliges providers of high-risk AI systems (and, subject to other articles, GPAI providers with systemic risk under Article 55) to report *serious incidents* to the market-surveillance authorities of the member states in which the incident occurred. The Act defines a serious incident (Article 3) as one that led to (or could have led to) any of the following: death or serious damage to a person's health; serious and irreversible disruption of critical infrastructure; infringement of Union law obligations intended to protect fundamental rights; or serious damage to property or the environment.

Timing envelope (as published; verify at authoring time against the Official Journal — see needs-research marker below):

- **Notification without undue delay** and, in any case, within **15 days** after the provider (or the deployer, where applicable) becomes aware of the serious incident.
- **Two days** for a widespread infringement or serious-and-irreversible disruption of critical infrastructure.
- **Ten days** in the event of a death.

Notification is followed by (a) an investigation, in cooperation with the market-surveillance authorities, and (b) further reporting and follow-up actions as the investigation proceeds.

<!-- needs-research: verify the exact wording and any implementing act updates to Article 73 timing at authoring time; the 15 / 2 / 10 day envelope is the shape published in the Official Journal 2024/1689 but has ongoing implementing acts. -->

Package shape (fields lifted from the post-mortem):

| Article 73 package field | Post-mortem source (chapter 03) |
|---|---|
| Provider / deployer identity, AI system identification | Header (section 1) + system inventory record (mod-102 ch 06) |
| Category of serious incident (per Article 3 sub-clause) | Impact statement (section 2) — regulatory classification |
| Date and place of the serious incident | Timeline (section 3) — first reproducible occurrence + affected geography |
| Facts of the incident | Impact statement (section 2) + timeline (section 3) |
| Immediate mitigating actions taken | Timeline (section 3) — mitigation cutovers |
| Corrective measures implemented or planned | CAP (section 7) — with owners and due dates |
| Preliminary root cause | Causal chain (section 4) — one paragraph |
| Post-market monitoring signal that surfaced the incident | Learning propagation (section 9) — monitoring signal reference to mod-109 |
| Follow-up reports | Timeline (section 3) — regulator notifications actually sent; update log |

The package is a *view* over the post-mortem, filled by extraction. If a field cannot be filled, it is filled explicitly as "not applicable" or "not yet known" — a blank field is a defect.

### SR 11-7 — supervisory guidance on model risk management (FI)

The Federal Reserve SR 11-7 (2011) and the OCC's parallel guidance (OCC 2011-12) set expectations for banks' model risk management. SR 11-7 is not a serious-incident reporting rule in the Article 73 sense — it is guidance on model-risk lifecycle. Model incidents (a model producing materially wrong outputs, a validation control failing, an unauthorised model change) fit into the guidance through:

- **Ongoing monitoring** — the incident is a monitoring event and updates the model's ongoing performance record.
- **Model inventory update** — the incident updates the model's status and any control credits in the model inventory.
- **Validation** — a material incident triggers a re-validation or a targeted validation on the affected control area.
- **Report to model risk management (MRM) governance** — an incident memo goes to MRM governance and, if material, to the model risk committee.

SR 23-4 (2023 update related to third-party risk management) is the parallel for third-party model providers.

<!-- needs-research: confirm any post-2024 supervisory updates that would change the SR 11-7 incident memo shape at authoring time; verify OCC 2011-12 revisions. -->

Package shape (SR 11-7-shaped MRM incident memo):

| SR 11-7-shaped memo field | Post-mortem source (chapter 03) |
|---|---|
| Model identifier, business use, model tier (materiality) | Header (section 1) + model inventory record |
| Incident date, discovery date, mitigation date | Timeline (section 3) |
| Description of the incident | Impact statement (section 2) + causal chain (section 4) |
| Impact on model output and downstream decisions | Impact statement (section 2) — including named-party effect and population-scale estimate |
| Root cause | Causal chain (section 4) |
| Ongoing-monitoring evidence supporting detection | Learning propagation (section 9) — monitoring reference (mod-109) |
| Controls affected; controls re-credited or dropped | Risk-model reconciliation (section 6) + CAP (section 7) |
| Validation impact (was a validation control defeated; is a re-validation required) | CAP (section 7) — re-validation actions; MRM governance sign-off |
| Reporting to MRM governance and committee | Signatures (section 10) — including MRM sign-off; council extract |

The memo is authored by the model owner and the risk engineer, reviewed by MRM validation, and signed off by MRM governance. For third-party models the SR 23-4 attributes travel with the memo (supplier identifier, contract-anchored control credits, supplier-side incident references).

### FDA MDR — Medical Device Reporting for SaMD / AI-enabled medical devices

The FDA's Medical Device Reporting regulation (21 CFR Part 803) obliges manufacturers to report to the FDA when they become aware of information reasonably suggesting a device may have caused or contributed to a death or serious injury, or has malfunctioned in a way that would be likely to cause or contribute to a death or serious injury if the malfunction were to recur. AI / SaMD (Software as a Medical Device) is covered where the device is regulated.

Timing envelope (as published):

- **30 calendar days** for events meeting the reporting criteria, from the date the manufacturer becomes aware.
- **5 working days** for events that require remedial action to prevent an unreasonable risk of substantial harm to the public health, or for other reporting events required by the FDA under 21 CFR 803.53.

<!-- needs-research: verify current form (Form 3500A / eMDR eSubmitter mapping) and any recent guidance updates specific to AI / SaMD reporting shapes at authoring time. -->

Package shape (Form 3500A / eMDR fields; the eMDR is the electronic submission mechanism):

| MDR field group | Post-mortem source (chapter 03) |
|---|---|
| Patient information | Impact statement (section 2) — named-party level (per privacy discipline) |
| Adverse event or product problem | Impact statement (section 2) + causal chain (section 4) — with the specific AI-behaviour description |
| Suspect medical device (identifier, model, software version) | Header (section 1) + model / software manifest |
| Manufacturer of the device | Provider identity (fixed field on the device) |
| Initial reporter | Timeline (section 3) — first-reporter role |
| Actions taken | Timeline (section 3) + CAP (section 7) |
| Manufacturer narrative | Causal chain (section 4) + impact statement (section 2) |
| Follow-up reports | Reporting log (section 8) |

For AI / SaMD, the pre-defined change-control-plan (PCCP) — where in place — is the mechanism through which planned model updates are pre-approved; an incident that requires a change *outside* the PCCP is a separate submission path.

### HIPAA breach notification (where PII / PHI is involved)

If an AI incident causes an impermissible use or disclosure of unsecured PHI, the HIPAA Breach Notification Rule (45 CFR Part 164, Subpart D) applies to covered entities and business associates. Notification obligations include: affected individuals; HHS (Office for Civil Rights) via the HHS breach portal; and, for breaches affecting 500+ residents of a state or jurisdiction, prominent media outlets.

Timing envelope: individual and media notification without unreasonable delay and **no later than 60 calendar days** after discovery; HHS notification for breaches ≥ 500 individuals within the same 60-day window (annual log for < 500).

<!-- needs-research: verify HHS breach portal URL and current submission fields at authoring time. -->

The HIPAA package shape is a separate submission; the post-mortem's impact statement (section 2), regulator classification (section 8), and CAP (section 7) supply the inputs.

## The "one incident, three regulator-facing packages" pattern

An AI incident affecting an EU-deployed high-risk AI system used by a US bank that also has SaMD components triggers *all three* obligations at once. The organisational failure mode is to have three teams reshape the post-mortem into three formats and then argue over the discrepancies. The engineering discipline is the opposite: a single post-mortem, a single evidence pack, and three declarative extract templates, each producing its regulator package by lift.

A working extract-generation shape:

```
regulator-packages/
  INC-2026-08-14-cs-rag-inj-01/
    post-mortem.md                         # single source of truth
    evidence-pack/                         # trace snapshots, manifests
    extracts/
      eu-ai-act-article-73-v1.md
      sr-11-7-mrm-memo-v1.md
      fda-mdr-3500a-v1.md
      hipaa-breach-notification-v1.md      # if triggered
    submission-log/
      2026-08-14-eu-ma-provisional.md      # what was submitted and when
      2026-08-16-fda-mdr-initial.md
      ...
    diff-checks/
      eu-vs-sr11-7-consistency.md          # confirms facts match across packages
      eu-vs-fda-consistency.md
```

Every extract is generated from the post-mortem, and a CI-style consistency check compares the facts across extracts (dates, causal-chain wording, impact counts). Where an extract must deviate (an SR 11-7 memo may have a broader "materiality" framing than an Article 73 categorisation), the deviation is documented, not silently authored.

## Timing envelopes summarised

| Obligation | Start of clock | Duration to notification |
|---|---|---|
| EU AI Act Article 73 — serious incident (general) | Awareness of the serious incident by provider / deployer | Without undue delay; within 15 days |
| EU AI Act Article 73 — widespread infringement / critical-infrastructure disruption | Awareness | Within 2 days |
| EU AI Act Article 73 — death | Awareness | Within 10 days |
| SR 11-7 MRM incident memo (Fed / OCC banks) | Discovery / material assessment | Not a supervised time-bound rule; internal governance cadence — typically committee cadence, ad-hoc for material events |
| FDA MDR — serious injury / death or malfunction likely to cause | Manufacturer awareness | 30 calendar days |
| FDA MDR — remedial-action-required event | Manufacturer awareness | 5 working days |
| HIPAA breach notification | Discovery of the breach | Without unreasonable delay; no later than 60 calendar days |

<!-- needs-research: cross-verify each duration and any concurrent obligations (e.g., GDPR Article 33 breach notification 72-hour clock where personal data of EU data subjects is involved) at authoring time. -->

The GDPR Article 33 personal-data-breach notification obligation (72 hours) commonly runs concurrently with HIPAA / EU AI Act obligations where personal data is involved and the data subject is in the EU/EEA. Concurrency handling — one incident, multiple obligations, multiple clocks — is the reason the post-mortem is a single artifact and the packages are extracts.

## Practical authoring notes

- **Precise language matters.** Article 73's "serious incident" categories, SR 11-7's "material" framing, and MDR's "serious injury" definition each have specific meaning in the source text. Use the language of the obligation, not a paraphrase, in the regulator-facing extract.
- **Named parties.** For MDR and HIPAA, patient / individual information travels with the report but subject to the privacy discipline of the obligation (de-identification for public sub-releases; identifiers to the regulator directly). Do not confuse the internal post-mortem's identifier-bearing evidence pack with the public-summary format.
- **Follow-up reports.** All four obligations expect follow-ups (progress reports, final reports, annual summaries). The submission-log directory in the extract-generation shape is where those live; they are versioned with the incident, not filed as separate incidents.
- **Legal / compliance sign-off.** Legal counsel and the compliance function are on the sign-off routing for every regulator-facing extract. The risk-engineer does not submit; the risk-engineer authors the extract and hands it to the accountable submitting role.

## Cross-references

- **Post-mortem file** — chapter 03 of this module is the single source of truth every extract lifts from.
- **Harm-model row and register update** — mod-102 and mod-103 supply the evidence linking the incident to the pre-registered risk shape; the regulator asks whether the risk was known.
- **Continuous monitoring** — mod-109's detection signal appears in every extract as the "how did you find out" answer.
- **Council reporting** — mod-112 receives the internal governance-council extract; regulator packages are separate outputs.
- **Level-40 escalation** — where the incident involves a frontier-agent capability at systemic scope, `agentic-safety-engineer` (L40) is on the sign-off routing; see chapter 06.

## Summary

- Regulator-facing summaries are *extracts* from the single post-mortem file (chapter 03), not re-authored documents. Consistency across regulators is engineered, not hoped for.
- EU AI Act Article 73 serious-incident timing envelope: without undue delay, within 15 days (2 days for critical-infrastructure disruption or widespread infringement; 10 days for death). Categories are defined in Article 3.
- SR 11-7 shape is a model-risk-management incident memo — model inventory update, ongoing-monitoring evidence, controls affected, validation impact, MRM governance sign-off; not a fixed-clock regulator notification.
- FDA MDR (21 CFR Part 803) for AI / SaMD: 30 calendar days for serious injury / death / malfunction; 5 working days for remedial-action-required events.
- HIPAA breach notification (45 CFR Part 164 Subpart D): individuals and HHS within 60 days; GDPR Article 33 (72 hours) commonly runs concurrently where EU personal data is involved.
- Every extract is generated from the post-mortem via declarative templates; consistency-check diffs across extracts catch drift.
- Exercise-05 authors the Article 73 and SR 11-7 extracts from a mock post-mortem and runs the consistency check.
