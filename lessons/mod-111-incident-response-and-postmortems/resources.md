# Resources — mod-111 AI-Specific Incident Response and Post-Mortem Engineering

Primary sources first. Every reference below is a regulator, a standards body, an incident-corpus maintainer, a government AI safety institute, or the original publication of an engineering practice. Anything less authoritative is either not listed or marked as such.

## Regulatory anchors — serious-incident reporting and model-risk incident obligations

- **EU AI Act — Regulation (EU) 2024/1689** (Official Journal) — <https://eur-lex.europa.eu/eli/reg/2024/1689/oj>
  - Article 3 — definition of "serious incident".
  - Article 73 — serious-incident reporting by providers of high-risk AI systems (and deployers where applicable).
  - Article 72 — post-market monitoring by providers (the surveillance system that feeds Article 73).
  - Article 55 — obligations for providers of general-purpose AI models with systemic risk (adjacent reporting duties).
- **Federal Reserve SR 11-7 — Guidance on Model Risk Management** — <https://www.federalreserve.gov/supervisionreg/srletters/sr1107.htm>
- **OCC 2011-12 — Supervisory Guidance on Model Risk Management** (parallel to SR 11-7 for national banks) — <https://www.occ.treas.gov/news-issuances/bulletins/2011/bulletin-2011-12.html>
- **Federal Reserve SR 23-4 — Interagency Guidance on Third-Party Relationships: Risk Management** — <https://www.federalreserve.gov/supervisionreg/srletters/SR2304.htm>
- **FDA Medical Device Reporting (MDR)** — 21 CFR Part 803 — <https://www.ecfr.gov/current/title-21/chapter-I/subchapter-H/part-803>
  - FDA MDR overview and reporting portal (eMDR) — <https://www.fda.gov/medical-devices/medical-device-safety/medical-device-reporting-mdr-how-report-medical-device-problems>
- **FDA — Good Machine Learning Practice for Medical Device Development: Guiding Principles** (joint FDA / Health Canada / MHRA) — <https://www.fda.gov/medical-devices/software-medical-device-samd/good-machine-learning-practice-medical-device-development-guiding-principles>
- **FDA — Predetermined Change Control Plans (PCCP) for AI/ML-enabled devices** — <https://www.fda.gov/medical-devices/software-medical-device-samd/marketing-submission-recommendations-predetermined-change-control-plan-artificial-intelligencemachine>
- **HIPAA Breach Notification Rule** — 45 CFR Part 164 Subpart D — <https://www.ecfr.gov/current/title-45/subtitle-A/subchapter-C/part-164/subpart-D>
  - HHS Office for Civil Rights — Breach Notification portal — <https://ocrportal.hhs.gov/ocr/breach/breach_report.jsf>
- **GDPR Article 33 — Notification of a personal data breach to the supervisory authority** (72-hour clock; commonly concurrent with the obligations above) — <https://eur-lex.europa.eu/eli/reg/2016/679/oj>

<!-- needs-research: verify the current implementing acts under EU AI Act Article 73 that may specify the standardised reporting template and precise notification form at authoring time. -->

<!-- needs-research: pin the current version of the FDA MDR eSubmitter mapping used for AI / SaMD submissions; forms and codes evolve. -->

## Incident corpora — public compare-points

- **AI Incident Database (AIID)** — Responsible AI Collaborative — <https://incidentdatabase.ai/>
- **OECD.AI AI Incidents Monitor (AIM)** — OECD.AI Policy Observatory — <https://oecd.ai/en/incidents>
- **AI Vulnerability Database (AVID)** — <https://avidml.org/>
- **MITRE ATLAS — Adversarial Threat Landscape for Artificial-Intelligence Systems** — <https://atlas.mitre.org/> (case studies of confirmed incidents alongside the TTP taxonomy).
- **MIT AI Risk Repository** — MIT FutureTech — <https://airisk.mit.edu/> (used in mod-102; cited here for cause-taxonomy cross-walk in post-mortem section 6).

<!-- needs-research: confirm active-maintenance status of AVID; if it has moved or paused, update or remove. -->

## General incident-response guidance to reuse

- **NIST SP 800-61 Rev. 2 — Computer Security Incident Handling Guide** — <https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-61r2.pdf>
- **NIST SP 800-61 Rev. 3 (draft, if current)** — <https://csrc.nist.gov/publications/detail/sp/800-61/rev-3/draft>
- **ISO/IEC 27035 — Information security incident management** (multi-part) — <https://www.iso.org/standard/78973.html>
- **Google SRE Book — chapter on Postmortem Culture: Learning from Failure** — <https://sre.google/sre-book/postmortem-culture/>
- **Google SRE Workbook — chapter on Postmortem Culture: Learning from Failure** — <https://sre.google/workbook/postmortem-culture/>

## AI safety institutes — context on incident evaluation and disclosure practice

- **UK AI Safety Institute (UK AISI)** — evaluation and publication practice on frontier-model incidents — <https://www.aisi.gov.uk/>
- **US AI Safety Institute (US AISI)** — <https://www.nist.gov/aisi>

<!-- needs-research: cite specific UK AISI / US AISI published incident-review materials that are stable references at authoring time; publication cadence evolves. -->

## NIST AI-side taxonomies used in RCA and post-mortem authoring

- **NIST AI 100-1 (AI RMF 1.0)** — MANAGE function references — <https://nvlpubs.nist.gov/nistpubs/ai/NIST.AI.100-1.pdf>
- **NIST AI RMF Generative AI Profile (AI 600-1)** — the risk categories that appear in taxonomy tagging on the incident record — <https://airc.nist.gov/AI_RMF_Knowledge_Base/AI_RMF/Uses/GenAI-Profile>
- **NIST AI 100-2 e2025 — Adversarial ML: A Taxonomy and Terminology of Attacks and Mitigations** — <https://csrc.nist.gov/publications/detail/nistir/8269/draft>

## Guardrail / attack-family evidence used in RCA and fixture back-feed

- **OWASP Top 10 for LLM Applications** — <https://owasp.org/www-project-top-10-for-large-language-model-applications/>
- **OWASP Machine Learning Security Top 10** — <https://owasp.org/www-project-machine-learning-security-top-10/>
- **Greshake, K., Abdelnabi, S., Mishra, S., Endres, C., Holz, T., Fritz, M. — "Not what you've signed up for: Compromising Real-World LLM-Integrated Applications with Indirect Prompt Injection"** — <https://arxiv.org/abs/2302.12173>

## GRC-for-AI platforms (register-update side of the incident loop)

- **Credo AI** — <https://www.credo.ai/>
- **Holistic AI** — <https://www.holisticai.com/>
- **ModelOp** — <https://www.modelop.com/>
- **Monitaur** — <https://www.monitaur.ai/>

<!-- needs-research: platform-specific incident-workflow claims should be verified against current published product documentation at authoring time; feature sets change per release. -->

## Frontier-agent escalation reference (L40 boundary)

- **Anthropic Responsible Scaling Policy** — <https://www.anthropic.com/rsp>
- **OpenAI Preparedness Framework** — <https://openai.com/safety/preparedness/>
- **Google DeepMind Frontier Safety Framework** — <https://deepmind.google/discover/blog/introducing-the-frontier-safety-framework/>
- **Frontier Model Forum** — <https://www.frontiermodelforum.org/>

<!-- needs-research: pin the exact document versions of the RSP, Preparedness, and FSF that are current at authoring time; these evolve. -->

## Reading order for this module

If you are working through mod-111 for the first time, read in this order:

1. Google SRE post-mortem chapters (SRE Book + SRE Workbook) — the classical shape you are extending.
2. NIST SP 800-61 Rev. 2 — the classical IR lifecycle you are plugging into.
3. EU AI Act Articles 3, 72, 73 (Regulation (EU) 2024/1689 in the Official Journal) — the serious-incident reporting anchor.
4. SR 11-7 + OCC 2011-12 — the FI MRM incident shape (skim SR 23-4 for third-party context).
5. 21 CFR Part 803 + FDA MDR overview + PCCP guidance — the SaMD reporting shape.
6. HIPAA Breach Notification Rule (45 CFR 164 Subpart D) + HHS breach portal.
7. GDPR Article 33 (Regulation (EU) 2016/679) — concurrent 72-hour clock where personal data is involved.
8. AIID + OECD.AI AIM — sample five incidents in the module's class taxonomy; note the metadata each maintains.
9. AVID + MITRE ATLAS — the technical-taxonomy compare-points used in RCA and fixture back-feed.
10. NIST AI 100-1 MANAGE function + AI 600-1 risk categories + AI 100-2 adversarial-ML taxonomy — the AI-side taxonomies your post-mortem tags with.
11. Anthropic RSP, OpenAI Preparedness, DeepMind FSF — the frontier-agent escalation reference for the L40 boundary.

## Reading order — paywalled and versioned sources

- ISO/IEC 27035 is paywalled through ISO national bodies. Same options as prior modules: national-body purchase, lead-implementer training, or the publicly available summaries on ISO's standard landing pages.
- EU AI Act implementing acts, delegated acts, and Commission guidance evolve; verify at the Official Journal and the European AI Office publications page at authoring time.
- FDA MDR forms (Form 3500A, eMDR eSubmitter mappings) and PCCP guidance evolve; verify at the FDA landing pages cited above.
- HHS OCR breach-portal fields evolve; verify current submission fields before authoring.

Do not rely on unofficial re-postings of any standard or regulation text; those are copyright violations for paywalled standards and their fidelity to the current release is uncertain.
