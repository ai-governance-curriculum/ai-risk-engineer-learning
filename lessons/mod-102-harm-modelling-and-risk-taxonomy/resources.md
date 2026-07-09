# Resources — mod-102 Harm Modelling and Risk Taxonomy Engineering

Primary sources first. Every reference below is a standards body, regulator, incident-corpus maintainer, ontology maintainer, or the original publication of a frontier-lab policy. Anything less authoritative is either not listed or marked as such.

## Incident and harm-model reference corpora

- **AI Incident Database (AIID)** — Responsible AI Collaborative — <https://incidentdatabase.ai/>
- **OECD.AI AI Incidents Monitor (AIM)** — OECD.AI Policy Observatory — <https://oecd.ai/en/incidents>
- **MIT AI Risk Repository** — MIT FutureTech — <https://airisk.mit.edu/>
- **AI Risk Ontology (AIRO)** — ADAPT Centre — <https://w3id.org/airo>
- **Vocabulary of AI Risks (VAIR)** — ADAPT Centre (sibling to AIRO) — <https://w3id.org/vair>

<!-- needs-research: confirm the current stable URI for VAIR and its version alignment with AIRO. -->

## Risk taxonomies and benchmarks

- **NIST AI RMF Generative AI Profile (AI 600-1)** — the twelve GenAI risks — <https://airc.nist.gov/AI_RMF_Knowledge_Base/AI_RMF/Uses/GenAI-Profile>
- **NIST AI 100-1 (AI RMF 1.0)** — background context — <https://nvlpubs.nist.gov/nistpubs/ai/NIST.AI.100-1.pdf>
- **AIR-Bench 2024** — Stanford CRFM, Zeng et al., "AIR-Bench 2024: A Safety Benchmark Based on Regulation and Policies Specified Risk Categories" — <https://crfm.stanford.edu/> (locate the current paper landing and dataset repository at the CRFM publications index)

<!-- needs-research: cite the exact arXiv identifier and the current version of the AIR-Bench 2024 category hierarchy at authoring time. -->

## ISO/IEC AI standards (the ones this module tags to)

- **ISO/IEC 23894:2023 — Guidance on risk management (AI)** — <https://www.iso.org/standard/77304.html>
- **ISO/IEC 42001:2023 — AI management system (AIMS)** — <https://www.iso.org/standard/81230.html> (harm-model records feed this)
- **ISO/IEC 42005:2025 — AI system impact assessment** — <https://www.iso.org/standard/44545.html> (harm model composes with this at the L15/L25 interface)
- **ISO 31000:2018 — Risk management (general)** — <https://www.iso.org/standard/65694.html> (23894 specialises 31000)

## Adversarial ML and AI security threat taxonomies (context)

- **NIST AI 100-2 e2025 — Adversarial ML: A Taxonomy and Terminology of Attacks and Mitigations** — <https://csrc.nist.gov/publications/detail/nistir/8269/draft>
- **OWASP Top 10 for LLM Applications** — <https://owasp.org/www-project-top-10-for-large-language-model-applications/>
- **OWASP Machine Learning Security Top 10** — <https://owasp.org/www-project-machine-learning-security-top-10/>
- **MITRE ATLAS** — Adversarial Threat Landscape for Artificial-Intelligence Systems — <https://atlas.mitre.org/>
- **AI Vulnerability Database (AVID)** — <https://avidml.org/>

<!-- needs-research: confirm active-maintenance status of AVID; if it has moved or paused, update or remove. -->

## Frontier-lab risk-tier framework publications

- **Anthropic Responsible Scaling Policy** (versioned; read the current published version and the version history) — <https://www.anthropic.com/rsp>
- **OpenAI Preparedness Framework** (versioned) — <https://openai.com/safety/preparedness/>
- **Google DeepMind Frontier Safety Framework** — <https://deepmind.google/discover/blog/introducing-the-frontier-safety-framework/>
- **Frontier Model Forum** (issue briefs, technical reports) — <https://www.frontiermodelforum.org/>

<!-- needs-research: pin the exact document versions of the RSP, Preparedness, and FSF that are current at authoring time; these evolve. -->

## Government AI safety institutes (context for frontier evaluation)

- **UK AI Safety Institute (UK AISI)** — <https://www.aisi.gov.uk/>
- **US AI Safety Institute (US AISI)** — <https://www.nist.gov/aisi>

## Indirect prompt injection and adjacent threat literature (cited from chapter 01)

- Greshake, K., Abdelnabi, S., Mishra, S., Endres, C., Holz, T., Fritz, M. — "Not what you've signed up for: Compromising Real-World LLM-Integrated Applications with Indirect Prompt Injection" — <https://arxiv.org/abs/2302.12173>
- Perez, F., Ribeiro, I. — "Ignore Previous Prompt: Attack Techniques For Language Models" — <https://arxiv.org/abs/2211.09527>
- OWASP LLM Top 10 — LLM01 (Prompt Injection) and LLM06 (Sensitive Information Disclosure) — <https://owasp.org/www-project-top-10-for-large-language-model-applications/>

## Related reading — from mod-101, revisited here

- NIST AI RMF Playbook — <https://airc.nist.gov/AI_RMF_Knowledge_Base/Playbook>
- Regulation (EU) 2024/1689 — the EU AI Act (Official Journal) — <https://eur-lex.europa.eu/eli/reg/2024/1689/oj>
- Federal Reserve SR 11-7 — Model Risk Management — <https://www.federalreserve.gov/supervisionreg/srletters/sr1107.htm>
- FDA / Health Canada / MHRA GMLP guiding principles — <https://www.fda.gov/medical-devices/software-medical-device-samd/good-machine-learning-practice-medical-device-development-guiding-principles>

## Reading order for this module

If you are working through mod-102 for the first time, read in this order:

1. AIID landing page + a random sample of five incidents in your deployment domain (chapter 02).
2. OECD.AI AI Incidents Monitor landing page and one sample record (chapter 02).
3. MIT AI Risk Repository landing page + causal and domain taxonomy spreadsheets (chapters 02, 03).
4. NIST AI 600-1 (already read in mod-101 ch 02; revisit §2 and §3) — chapter 03.
5. ISO/IEC 23894 Annex A (via licensed access) — chapter 03.
6. AIRO / VAIR landing pages + a scan of the core classes — chapter 03.
7. AIR-Bench 2024 paper and category hierarchy — chapter 03.
8. Anthropic RSP, OpenAI Preparedness, Google DeepMind FSF — read all three end-to-end before chapter 04.
9. Frontier Model Forum publications — sample the two most recent issue briefs — chapter 04.

## Reading order — the paywalled and versioned sources

- ISO/IEC 23894, 42001, 42005 are paywalled through ISO national bodies. Same options as mod-101: national-body purchase, lead-implementer training, or the publicly available summaries on ISO's standard landing pages.
- Frontier-lab documents (RSP, Preparedness, FSF) are versioned; pin the version you cite and note the publication date. Prior versions remain instructive as evolution studies.
- MIT AI Risk Repository is versioned; cite the release version alongside every row identifier.
- AIRO / VAIR are versioned URIs; cite the URI as published (`https://w3id.org/airo` resolves to the current version; historical resolutions may be available).
- AI 600-1 is dated (July 2024 at time of authoring); check for revisions on the NIST AI Resource Center before citing at review time.

Do not rely on unofficial re-postings of any standard or ontology text; those are copyright violations for paywalled standards and their fidelity to the current release is uncertain.
