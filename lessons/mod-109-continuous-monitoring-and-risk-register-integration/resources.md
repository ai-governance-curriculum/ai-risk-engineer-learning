# Resources — mod-109 Continuous Monitoring, Post-Market Surveillance, and Risk-Register Integration

Primary sources first. Every reference below is a standards body, regulator, official project page, or the original publication. Vendor references point at the vendor's own product / documentation page. Drift-metric references point at the canonical statistical publication.

## Regulatory anchors

- **Regulation (EU) 2024/1689 — the EU AI Act (Official Journal)** — <https://eur-lex.europa.eu/eli/reg/2024/1689/oj>
  - Article 61 — post-market monitoring system for high-risk AI systems.
  - Article 72 — post-market monitoring by providers and post-market monitoring plan for high-risk AI systems.
  - Article 73 — reporting of serious incidents.
- **European Commission — AI Act implementing and delegated acts (index)** — <https://digital-strategy.ec.europa.eu/en/policies/regulatory-framework-ai>

<!-- needs-research: the Commission implementing act specifying the Article 72 report template is at consultation / adoption stages; verify status and align section headers to the adopted template where available. -->

- **FDA — Post-market Surveillance under 21 CFR Part 822** — <https://www.fda.gov/medical-devices/postmarket-requirements-devices/postmarket-surveillance-under-section-522-federal-food-drug-and-cosmetic-act>
- **FDA — Marketing Submission Recommendations for a Predetermined Change Control Plan (PCCP) for AI/ML-Enabled Device Software Functions (final guidance)** — <https://www.fda.gov/regulatory-information/search-fda-guidance-documents/marketing-submission-recommendations-predetermined-change-control-plan-artificial-intelligence>
- **FDA / Health Canada / MHRA — Good Machine Learning Practice (GMLP) guiding principles** — <https://www.fda.gov/medical-devices/software-medical-device-samd/good-machine-learning-practice-medical-device-development-guiding-principles>

<!-- needs-research: any post-market surveillance updates specific to AI/ML SaMD published since the initial GMLP guiding principles and the PCCP guidance — check the FDA CDRH AI/ML action-plan page for the most recent documents. -->

- **NIST AI RMF 1.0 (AI 100-1)** — <https://nvlpubs.nist.gov/nistpubs/ai/NIST.AI.100-1.pdf>
- **NIST AI RMF Playbook** — <https://airc.nist.gov/AI_RMF_Knowledge_Base/Playbook>
- **NIST AI RMF Generative AI Profile (AI 600-1)** — <https://airc.nist.gov/AI_RMF_Knowledge_Base/AI_RMF/Uses/GenAI-Profile>
- **ISO/IEC 42001:2023 — AI management system (AIMS)** — <https://www.iso.org/standard/81230.html>
- **ISO/IEC 23894:2023 — Guidance on risk management (AI)** — <https://www.iso.org/standard/77304.html>
- **ISO/IEC 42005:2025 — AI system impact assessment** — <https://www.iso.org/standard/44545.html>

## Open-source tooling — the working toolset

- **whylogs** (WhyLabs) — statistical data-logging library; official repository and documentation — <https://github.com/whylabs/whylogs>
- **whylogs documentation** — <https://whylogs.readthedocs.io/>
- **Evidently AI** — open-source ML / LLM evaluation and monitoring; project site — <https://www.evidentlyai.com/>
- **Evidently AI documentation** — <https://docs.evidentlyai.com/>
- **Evidently AI GitHub** — <https://github.com/evidentlyai/evidently>
- **Alibi Detect** (Seldon) — drift, outlier, and adversarial detection library; project site — <https://docs.seldon.io/projects/alibi-detect/en/stable/>
- **Alibi Detect GitHub** — <https://github.com/SeldonIO/alibi-detect>

## Commercial platforms

Named platforms below are cited from their published product pages. Feature sets change per release; verify against the current documentation.

- **WhyLabs** (commercial platform on top of `whylogs`) — <https://whylabs.ai/>
  - LangKit (LLM-side signal library) — <https://github.com/whylabs/langkit>
- **Fiddler AI** — ML / LLM observability and explainability — <https://www.fiddler.ai/>
- **Arthur AI** — ML / LLM monitoring; Arthur Shield runtime guardrail-and-monitoring product — <https://www.arthur.ai/>
- **Holistic AI** — AI risk / governance platform (also referenced in mod-103 chapter 06) — <https://www.holisticai.com/>
- **Credo AI** — AI governance platform (mod-103 cross-reference) — <https://www.credo.ai/>
- **ModelOp** — AI governance / model operations platform (mod-103 cross-reference) — <https://www.modelop.com/>

<!-- needs-research: confirm active-product status and pricing / tier information for each commercial vendor above at authoring time; product lines evolve. -->

## Drift-metric literature — canonical references

- **Kolmogorov, A. N.** — "Sulla determinazione empirica di una legge di distribuzione" (Giornale dell'Istituto Italiano degli Attuari, 1933) — foundational KS statistic paper. English secondary literature: Massey, F. J. — "The Kolmogorov-Smirnov Test for Goodness of Fit" (Journal of the American Statistical Association, 46(253), 1951) — <https://www.jstor.org/stable/2280095>.
- **Wu, D. (Naeem Siddiqi's tradition)** — Population Stability Index conventions in credit-risk practice; the widely cited convention bands (`< 0.1` no significant change, `0.1–0.2` moderate, `> 0.2` significant) originate in credit-scoring monitoring practice. Reference text: Siddiqi, N. — *Credit Risk Scorecards: Developing and Implementing Intelligent Credit Scoring* (Wiley) — publisher page: <https://www.wiley.com/en-us/Credit+Risk+Scorecards%3A+Developing+and+Implementing+Intelligent+Credit+Scoring-p-9780471754510>.

<!-- needs-research: PSI has no single canonical academic reference; if a more authoritative primary source is required for citation, note that the convention is practitioner-established and cite Siddiqi (or an equivalent credit-risk-monitoring text) plus the specific institution's internal convention. -->

- **Gretton, A., Borgwardt, K. M., Rasch, M. J., Schölkopf, B., Smola, A.** — "A Kernel Two-Sample Test" (Journal of Machine Learning Research 13, 2012) — foundational MMD paper — <https://jmlr.org/papers/v13/gretton12a.html>
- **Lin, J.** — "Divergence Measures Based on the Shannon Entropy" (IEEE Transactions on Information Theory, 37(1), 1991) — foundational Jensen-Shannon divergence paper — <https://ieeexplore.ieee.org/document/61115>
- **Gama, J., Žliobaitė, I., Bifet, A., Pechenizkiy, M., Bouchachia, A.** — "A Survey on Concept Drift Adaptation" (ACM Computing Surveys, 46(4), 2014) — concept-drift definitions and detection survey — <https://dl.acm.org/doi/10.1145/2523813>
- **Rabanser, S., Günnemann, S., Lipton, Z. C.** — "Failing Loudly: An Empirical Study of Methods for Detecting Dataset Shift" (NeurIPS 2019) — <https://arxiv.org/abs/1810.11953>

## Related adversarial-signal literature (context for chapter 03)

- **OWASP Top 10 for LLM Applications** — <https://owasp.org/www-project-top-10-for-large-language-model-applications/>
- **MITRE ATLAS** — Adversarial Threat Landscape for Artificial-Intelligence Systems — <https://atlas.mitre.org/>
- **NIST AI 100-2 e2025 — Adversarial ML: A Taxonomy and Terminology of Attacks and Mitigations** — <https://csrc.nist.gov/publications/detail/nistir/8269/draft>

## Cross-references — modules whose artifacts this module reads and writes

- mod-101 ch 01 — MEASURE 2.4, 2.7, 3.2 and MANAGE 4.1 close in this module; mod-101 ch 04 — the EU AI Act Article 61 / 72 / 73 obligations translated to engineering deliverables.
- mod-103 ch 03, ch 05, ch 06 — the register row schema, aggregation reports, and register operations that this module writes to.
- mod-108 — guardrail firing-rate and effectiveness measurement; the upstream wire into chapter 03 of this module.
- mod-111 — incident response; the downstream wire from the incident-candidate queue and Article 73 notifications.
- mod-112 — governance-council digest; the downstream wire from the register + tripped-signal records.

## Reading order for this module

If you are working through mod-109 for the first time, read in this order:

1. NIST AI RMF Playbook, MEASURE 2.4, 2.7, 3.2 and MANAGE 4.1 (revisit) — chapter 01.
2. EU AI Act Articles 61, 72, 73 in the Official Journal — chapters 01, 04.
3. `whylogs` documentation quickstart and profile concepts — chapter 02.
4. Evidently AI documentation — data-drift and target-drift report references and the test-suite pattern — chapter 02.
5. Alibi Detect documentation — the KS, chi-squared, MMD, and online / change-point detector references — chapter 02.
6. Gretton et al. 2012 (MMD paper) and Rabanser et al. 2019 (dataset-shift study) — chapter 02.
7. OWASP LLM Top 10 (revisit from mod-102 / mod-108) — chapter 03.
8. FDA post-market surveillance page + PCCP final guidance + GMLP guiding principles — chapter 04.
9. Fiddler, Arthur, WhyLabs, Holistic AI product pages — chapter 06 (comparison of what each covers vs. the open-source starting points).

## Reading order — the paywalled and versioned sources

- ISO/IEC 42001, 23894, 42005 are paywalled through ISO national bodies (as in mod-101 and mod-102).
- EU AI Act implementing acts under Article 72 evolve; pin the version you cite.
- FDA guidance documents (PCCP, GMLP guiding principles) are versioned; check the CDRH page for the current status.
- Commercial platform product pages evolve; pin the URL and access date when citing a specific capability claim.

Do not rely on unofficial re-postings of any standard or ontology text.
