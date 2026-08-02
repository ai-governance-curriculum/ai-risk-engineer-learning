# Resources — mod-103 Risk Quantification, Scoring, and Enterprise AI Risk Registers

Primary sources first. Every reference below is a standards body, regulator, framework maintainer, incident-corpus author, or the vendor's own published product documentation. Where a standard is paywalled through ISO national bodies, only the standard landing page is linked — the standard text itself is licensed.

## The parent frame: ISO/IEC 27005, ISO 31000, and ISO/IEC 23894

- **ISO/IEC 27005:2022 — Guidance on managing information security risks** — <https://www.iso.org/standard/80585.html>
- **ISO 31000:2018 — Risk management — Guidelines** (the general standard that both 27005 and 23894 specialise) — <https://www.iso.org/standard/65694.html>
- **ISO/IEC 23894:2023 — Artificial intelligence — Guidance on risk management** — <https://www.iso.org/standard/77304.html>
- **ISO Guide 73:2009 — Risk management vocabulary** (definitional grounding for the shared terms) — <https://www.iso.org/standard/44651.html>

<!-- needs-research: verify the exact clause identifiers for the 27005:2022 seven-step process and the ISO/IEC 23894:2023 annex-based mappings once the standard text has been consulted through a national body. -->

## The scoring dimensions: ISO/IEC 25059 (SQuaRE) and ISO/IEC 25010

- **ISO/IEC 25059:2023 — SQuaRE — Quality model for AI systems** — <https://www.iso.org/standard/80655.html>
- **ISO/IEC 25010:2023 — SQuaRE — Product quality model** (the general software quality model that 25059 extends) — <https://www.iso.org/standard/78176.html>
- **ISO/IEC 25012:2008 — Data quality model** (referenced from 25059 for data-quality characteristics) — <https://www.iso.org/standard/35736.html>

## NIST AI RMF and the MEASURE function

- **NIST AI 100-1 — AI Risk Management Framework 1.0** — <https://nvlpubs.nist.gov/nistpubs/ai/NIST.AI.100-1.pdf>
- **NIST AI RMF Playbook** (MEASURE sub-categories with outcome statements) — <https://airc.nist.gov/AI_RMF_Knowledge_Base/Playbook>
- **NIST AI 600-1 — Generative AI Profile** — <https://airc.nist.gov/AI_RMF_Knowledge_Base/AI_RMF/Uses/GenAI-Profile>
- **NIST AI Resource Center (AIRC)** — <https://airc.nist.gov/>

<!-- needs-research: pin the exact current NIST AI RMF MEASURE sub-category identifiers (e.g., MEASURE-2.7, MEASURE-2.11, MEASURE-2.3, MEASURE-3.2) and their outcome statements against the Playbook at authoring time; NIST revises the Playbook periodically. -->

## Management system: ISO/IEC 42001 and companions

- **ISO/IEC 42001:2023 — AI management system (AIMS)** — <https://www.iso.org/standard/81230.html>
- **ISO/IEC 42005:2025 — AI system impact assessment** — <https://www.iso.org/standard/44545.html>
- **ISO/IEC 38507:2022 — Governance implications of the use of AI by organisations** — <https://www.iso.org/standard/56641.html>

## Enterprise-ERM references (COSO, general)

- **COSO ERM — Enterprise Risk Management — Integrating with Strategy and Performance** — Committee of Sponsoring Organizations of the Treadway Commission — <https://www.coso.org/enterprise-risk-management>
- **COSO ERM ↔ ISO 31000 comparison discussion** — see COSO publications page for the current comparative paper — <https://www.coso.org/publications>

## Sector overlays (materiality tiering and appetite equivalents)

- **Federal Reserve SR 11-7 — Model Risk Management** — <https://www.federalreserve.gov/supervisionreg/srletters/sr1107.htm>
- **OCC 2011-12 — Sound Practices for Model Risk Management** — <https://www.occ.treas.gov/news-issuances/bulletins/2011/bulletin-2011-12.html>
- **SR 23-4 — Interagency Guidance on Managing Risks Associated with Third-Party Relationships** — <https://www.federalreserve.gov/supervisionreg/srletters/sr2304.htm>
- **FDA / Health Canada / MHRA — Good Machine Learning Practice (GMLP) Guiding Principles** — <https://www.fda.gov/medical-devices/software-medical-device-samd/good-machine-learning-practice-medical-device-development-guiding-principles>
- **FDA — Predetermined Change Control Plans (PCCP) for AI/ML-Enabled Device Software Functions** — <https://www.fda.gov/regulatory-information/search-fda-guidance-documents/predetermined-change-control-plans-artificial-intelligence-enabled-device-software-functions>
- **Regulation (EU) 2024/1689 — the EU AI Act (Article 9 risk-management system; Article 15 accuracy, robustness, cybersecurity)** — <https://eur-lex.europa.eu/eli/reg/2024/1689/oj>
- **Colorado Consumer Protections for Artificial Intelligence (SB24-205)** — <https://leg.colorado.gov/bills/sb24-205>
- **New York City Local Law 144 (2021)** — bias audits for automated employment decision tools — <https://www.nyc.gov/site/dca/about/automated-employment-decision-tools.page>

## GRC-for-AI platforms (vendor documentation is primary — feature sets change per release)

- **Credo AI** — product documentation — <https://www.credo.ai/>
- **Holistic AI** — product documentation — <https://www.holisticai.com/>
- **ModelOp** — product documentation — <https://www.modelop.com/>
- **Monitaur** — product documentation — <https://www.monitaur.ai/>

<!-- needs-research: verify each vendor's current product-documentation URL and feature-set claims at authoring time; product landing pages sometimes reorganise and feature sets evolve. -->

## Adjacent enterprise-ERM tooling (for the AI-register-plugs-into-ERM story)

- **ServiceNow GRC** — <https://www.servicenow.com/products/governance-risk-and-compliance.html>
- **RSA Archer** — <https://www.archerirm.com/>
- **MetricStream** — <https://www.metricstream.com/>
- **OneTrust GRC and AI Governance** — <https://www.onetrust.com/>

These are the *security-side / enterprise-side* GRC systems the AI register composes with under chapter 03's parent-frame alignment; the risk engineer's job is to make the AI register readable by these systems rather than to select them.

## Incident and evidence corpora (already cited in mod-102, revisited here for likelihood-anchoring)

- **AI Incident Database (AIID)** — Responsible AI Collaborative — <https://incidentdatabase.ai/>
- **OECD.AI AI Incidents Monitor (AIM)** — OECD.AI Policy Observatory — <https://oecd.ai/en/incidents>
- **MIT AI Risk Repository** — MIT FutureTech — <https://airisk.mit.edu/>
- **MITRE ATLAS** — Adversarial Threat Landscape for Artificial-Intelligence Systems — <https://atlas.mitre.org/>
- **OWASP Top 10 for LLM Applications** — <https://owasp.org/www-project-top-10-for-large-language-model-applications/>
- **AI Vulnerability Database (AVID)** — <https://avidml.org/>

<!-- needs-research: confirm active-maintenance status of AVID and cite the current release version at authoring time. -->

## Government AI safety institutes (evaluation-methodology anchor)

- **UK AI Safety Institute (UK AISI)** — <https://www.aisi.gov.uk/>
- **US AI Safety Institute (US AISI)** — <https://www.nist.gov/aisi>

## Calibration methodology (chapter 07's L30 handoff — the risk engineer *reads*, does not derive)

Statistical-methodology references. The risk engineer needs literacy to consume these; the model-evaluation engineer at L30 owns their derivation.

- **Efron & Tibshirani — An Introduction to the Bootstrap** (Chapman & Hall / CRC, 1993) — the foundational bootstrap-methodology reference.
- **Cohen, J. — "A coefficient of agreement for nominal scales"** (Educational and Psychological Measurement, 1960) — Cohen's κ.
- **Fleiss, J. L. — "Measuring nominal scale agreement among many raters"** (Psychological Bulletin, 1971) — Fleiss' κ.
- **Krippendorff, K. — Content Analysis: An Introduction to Its Methodology** (Sage) — Krippendorff's α (multi-rater, mixed-scale).
- **Landis, J. R., & Koch, G. G. — "The Measurement of Observer Agreement for Categorical Data"** (Biometrics, 1977) — often-cited κ-interpretation thresholds (caveat: thresholds are heuristics, not universal).
- **Zheng, L. et al. — "Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena"** — <https://arxiv.org/abs/2306.05685> — LLM-as-a-judge methodology and human-agreement calibration.

<!-- needs-research: cite the current-best-practice paper on judge-vs-human calibration for safety / risk grading (as opposed to preference / quality grading) once verified against the literature. -->

## Adversarial-ML and AI security threat taxonomies (already cited in mod-102, revisited for likelihood-anchoring)

- **NIST AI 100-2 e2025 — Adversarial ML: A Taxonomy and Terminology of Attacks and Mitigations** — <https://csrc.nist.gov/publications/detail/nistir/8269/draft>
- **OWASP Machine Learning Security Top 10** — <https://owasp.org/www-project-machine-learning-security-top-10/>

## Reading order for this module

If you are working through mod-103 for the first time, read in this order:

1. Skim ISO 31000:2018 landing page and the vocabulary standard ISO Guide 73:2009 for the shared definitions (chapter 03).
2. Read ISO/IEC 27005:2022 landing page and the process-shape summary; if you have licensed access, read the seven process-step clauses end-to-end (chapter 03).
3. Read ISO/IEC 23894:2023 landing page and Annex A / Annex B via licensed access to see the ISO 31000 ↔ AI-content mapping (chapter 03).
4. Read the NIST AI RMF Playbook's MEASURE section and pin the sub-category identifiers you will cite (chapter 04).
5. Read ISO/IEC 25059:2023 landing page and any accessible summary of its AI-specific quality characteristics; if licensed, read clause 5 (chapter 04).
6. Read one COSO ERM paper for the enterprise-ERM shape and one bank-supervisor risk-appetite discussion (chapter 05).
7. Read the current published product documentation for one GRC-for-AI platform end-to-end — Credo AI, Holistic AI, ModelOp, or Monitaur — before exercise 05 (chapter 06).
8. For chapter 07 depth, read Efron & Tibshirani chapters 1-3 (bootstrap fundamentals) and Cohen 1960 / Landis & Koch 1977 (κ thresholds and interpretation). These are the L30 peer's core; the risk engineer needs the literacy to *read* them.

## Reading order — the paywalled and versioned sources

- ISO/IEC 27005, 23894, 25059, 25010, 42001, 42005 are paywalled through ISO national bodies. Options: national-body purchase, lead-implementer training, or the publicly available summaries on ISO's standard landing pages.
- The NIST AI RMF Playbook is versioned; pin the sub-category identifiers and outcome statements as you cite them, and re-check on major Playbook revisions.
- GRC-for-AI platform features evolve; pin the product-documentation version cited and re-verify at review time. Do not cite marketing pages as evidence of feature availability.
- Sector-overlay documents (SR 11-7, FDA GMLP, EU AI Act, Colorado AI Act, NYC LL 144) update as regulators publish amendments. Cite the version and date on each row.

Do not rely on unofficial re-postings of any paywalled standard; those are copyright violations and their fidelity to the current release is uncertain.
