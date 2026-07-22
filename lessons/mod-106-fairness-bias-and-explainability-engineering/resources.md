# Resources — mod-106 Fairness, Bias, and Explainability Engineering

Primary sources first. Every reference below is a standards body, regulator, published paper (with arXiv ID or DOI), or an official project page for a tool this module names. Anything less authoritative is either not listed or marked as such.

## Fairness metric literature — the papers behind the definitions and impossibilities

- Kleinberg, J., Mullainathan, S., Raghavan, M. — "Inherent Trade-Offs in the Determination of Risk Scores" — arXiv:1609.05807 — <https://arxiv.org/abs/1609.05807>
- Chouldechova, A. — "Fair Prediction with Disparate Impact: A Study of Bias in Recidivism Prediction Instruments" — arXiv:1610.07524 — <https://arxiv.org/abs/1610.07524>
- Hardt, M., Price, E., Srebro, N. — "Equality of Opportunity in Supervised Learning" — arXiv:1610.02413 — <https://arxiv.org/abs/1610.02413>
- Dwork, C., Hardt, M., Pitassi, T., Reingold, O., Zemel, R. — "Fairness through Awareness" — arXiv:1104.3913 — <https://arxiv.org/abs/1104.3913>
- Kusner, M. J., Loftus, J. R., Russell, C., Silva, R. — "Counterfactual Fairness" — arXiv:1703.06856 — <https://arxiv.org/abs/1703.06856>
- Pleiss, G., Raghavan, M., Wu, F., Kleinberg, J., Weinberger, K. Q. — "On Fairness and Calibration" — arXiv:1709.02012 — <https://arxiv.org/abs/1709.02012>
- Nabi, R., Shpitser, I. — "Fair Inference on Outcomes" — arXiv:1705.10378 — <https://arxiv.org/abs/1705.10378>
- Salimi, B., Rodriguez, L., Howe, B., Suciu, D. — "Interventional Fairness: Causal Database Repair for Algorithmic Fairness" — <https://dl.acm.org/doi/10.1145/3299869.3319901>

## Mitigation-algorithm literature

- Kamiran, F., Calders, T. — "Data Preprocessing Techniques for Classification without Discrimination" — Knowledge and Information Systems 33(1) — <https://link.springer.com/article/10.1007/s10115-011-0463-8>
- Zemel, R., Wu, Y., Swersky, K., Pitassi, T., Dwork, C. — "Learning Fair Representations" — <https://proceedings.mlr.press/v28/zemel13.html>
- Feldman, M., Friedler, S., Moeller, J., Scheidegger, C., Venkatasubramanian, S. — "Certifying and Removing Disparate Impact" — arXiv:1412.3756 — <https://arxiv.org/abs/1412.3756>
- Calmon, F., Wei, D., Vinzamuri, B., Ramamurthy, K. N., Varshney, K. R. — "Optimized Pre-Processing for Discrimination Prevention" — NeurIPS 2017 — <https://papers.nips.cc/paper/2017/hash/9a49a25d845a483fae4be7e341368e36-Abstract.html>
- Agarwal, A., Beygelzimer, A., Dudik, M., Langford, J., Wallach, H. — "A Reductions Approach to Fair Classification" — arXiv:1803.02453 — <https://arxiv.org/abs/1803.02453>
- Zhang, B. H., Lemoine, B., Mitchell, M. — "Mitigating Unwanted Biases with Adversarial Learning" — arXiv:1801.07593 — <https://arxiv.org/abs/1801.07593>
- Kamishima, T., Akaho, S., Asoh, H., Sakuma, J. — "Fairness-Aware Classifier with Prejudice Remover Regularizer" — ECML PKDD 2012 — <https://link.springer.com/chapter/10.1007/978-3-642-33486-3_3>
- Celis, L. E., Huang, L., Keswani, V., Vishnoi, N. K. — "Classification with Fairness Constraints: A Meta-Algorithm with Provable Guarantees" — arXiv:1806.06055 — <https://arxiv.org/abs/1806.06055>
- Kearns, M., Neel, S., Roth, A., Wu, Z. S. — "Preventing Fairness Gerrymandering: Auditing and Learning for Subgroup Fairness" — arXiv:1711.05144 — <https://arxiv.org/abs/1711.05144>
- Kamiran, F., Karim, A., Zhang, X. — "Decision Theory for Discrimination-Aware Classification" — ICDM 2012 — <https://ieeexplore.ieee.org/document/6413831>

## Explainability literature

- Lundberg, S. M., Lee, S.-I. — "A Unified Approach to Interpreting Model Predictions" (SHAP) — arXiv:1705.07874 — <https://arxiv.org/abs/1705.07874>
- Lundberg, S. M., Erion, G., Chen, H., et al. — "From Local Explanations to Global Understanding with Explainable AI for Trees" (TreeSHAP) — arXiv:1905.04610 — <https://arxiv.org/abs/1905.04610>
- Ribeiro, M. T., Singh, S., Guestrin, C. — "'Why Should I Trust You?': Explaining the Predictions of Any Classifier" (LIME) — arXiv:1602.04938 — <https://arxiv.org/abs/1602.04938>
- Ribeiro, M. T., Singh, S., Guestrin, C. — "Anchors: High-Precision Model-Agnostic Explanations" — AAAI 2018 — <https://ojs.aaai.org/index.php/AAAI/article/view/11491>
- Wachter, S., Mittelstadt, B., Russell, C. — "Counterfactual Explanations without Opening the Black Box: Automated Decisions and the GDPR" — <https://arxiv.org/abs/1711.00399>
- Adebayo, J., Gilmer, J., Muelly, M., Goodfellow, I., Hardt, M., Kim, B. — "Sanity Checks for Saliency Maps" — arXiv:1810.03292 — <https://arxiv.org/abs/1810.03292>

## Tooling — official project pages

- **Fairlearn** — <https://fairlearn.org/> — user guide + API reference + `MetricFrame`, `ThresholdOptimizer`, `ExponentiatedGradient`, `GridSearch`
- **AIF360 (AI Fairness 360)** — LF AI & Data Foundation — <https://aif360.res.ibm.com/> and <https://github.com/Trusted-AI/AIF360> — algorithm catalogue for pre-, in-, post-processing mitigations
- **Holistic AI Open Source** — <https://github.com/holistic-ai/holisticai> — multi-dimensional trustworthiness sweep
- **SHAP** — <https://shap.readthedocs.io/> — KernelSHAP, TreeSHAP, DeepSHAP, LinearSHAP
- **Alibi** — <https://docs.seldon.io/projects/alibi/> — anchors, counterfactuals, integrated gradients, contrastive explanations
- **LIME** — <https://github.com/marcotcr/lime> — reference implementation
- **imbalanced-learn** — <https://imbalanced-learn.org/> — sampling utilities used as a mitigation adjunct

<!-- needs-research: confirm current maintenance status of AIF360 (activity, release cadence) and the current recommended installation path for Holistic AI's holisticai package. -->

## US employment-law fairness anchors

- **EEOC Uniform Guidelines on Employee Selection Procedures (UGESP)** — 29 CFR Part 1607 — <https://www.ecfr.gov/current/title-29/subtitle-B/chapter-XIV/part-1607>
- **EEOC — landing** — <https://www.eeoc.gov/> — enforcement guidance and Title VII framing
- **EEOC — Technical Assistance Document on the ADA and Use of Software, Algorithms, and Artificial Intelligence to Assess Job Applicants and Employees** — <https://www.eeoc.gov/laws/guidance/americans-disabilities-act-and-use-software-algorithms-and-artificial-intelligence>
- **US Equal Credit Opportunity Act (ECOA) / Regulation B** — 12 CFR Part 1002 — <https://www.consumerfinance.gov/rules-policy/regulations/1002/>
- **US Fair Housing Act (FHA)** — 42 U.S.C. §§ 3601 et seq. — <https://www.hud.gov/program_offices/fair_housing_equal_opp/fair_housing_act_overview>
- **HUD final rule on disparate impact standard (2013, amended)** — <https://www.hud.gov/press/press_releases_media_advisories/hud_no_23_046>

<!-- needs-research: confirm current state of HUD disparate-impact rule after 2023 rulemaking; note the ruling date on citation. -->

## Jurisdictional audit shapes

- **NYC Local Law 144 of 2021 — Automated Employment Decision Tools** — <https://www.nyc.gov/site/dca/about/automated-employment-decision-tools.page>
- **NYC DCWP Final Rule — 6 RCNY §5-300 et seq.** — <https://rules.cityofnewyork.us/> (search DCWP AEDT rule text)
- **Colorado AI Act (SB 24-205)** — Colorado General Assembly — <https://leg.colorado.gov/bills/sb24-205>
- **EU AI Act — Regulation (EU) 2024/1689** — Article 10 (data and data governance); Article 14 (human oversight); Article 15 (accuracy, robustness, cybersecurity) — Official Journal — <https://eur-lex.europa.eu/eli/reg/2024/1689/oj>
- **Illinois Artificial Intelligence Video Interview Act** — 820 ILCS 42/ — <https://www.ilga.gov/legislation/ilcs/ilcs3.asp?ActID=4015>
- **UK Equality Act 2010** — <https://www.legislation.gov.uk/ukpga/2010/15/contents>

<!-- needs-research: confirm effective date of Colorado AI Act and any amendments post-2025 that shift the audit or impact-assessment shape. -->

## Credentialing bodies for independent AI audit

- **ForHumanity — Independent AI Auditor (IAA)** — <https://forhumanity.center/>
- **BABL AI — Algorithm Bias Auditor** — <https://babl.ai/>

## Adjacent framework anchors — reused from mod-101 / mod-102 / mod-103

- **NIST AI RMF 1.0 (AI 100-1)** — MEASURE 2.11 (fairness evaluated) is discharged by this module — <https://nvlpubs.nist.gov/nistpubs/ai/NIST.AI.100-1.pdf>
- **NIST AI RMF Playbook** — MEASURE 2.11 sub-category guidance — <https://airc.nist.gov/AI_RMF_Knowledge_Base/Playbook>
- **NIST AI 600-1 (Generative AI Profile)** — risk category "Harmful Bias or Homogenization" — <https://airc.nist.gov/AI_RMF_Knowledge_Base/AI_RMF/Uses/GenAI-Profile>
- **ISO/IEC TR 24027:2021 — Bias in AI systems and AI-aided decision making** — <https://www.iso.org/standard/77607.html>
- **ISO/IEC TS 12791:2024 — Treatment of unwanted bias in classification and regression machine learning tasks** — <https://www.iso.org/standard/84110.html>

<!-- needs-research: confirm publication year and current text of ISO/IEC TS 12791; the standard is recent and the version referenced should match citation date. -->

## Reading order

If you are working through mod-106 for the first time, read in this order:

1. Kleinberg et al. (arXiv:1609.05807) and Chouldechova (arXiv:1610.07524) — the two impossibility papers. Read before touching any tool.
2. Hardt et al. (arXiv:1610.02413) — equal opportunity, and the reference paper for `ThresholdOptimizer`.
3. Fairlearn user guide — the `MetricFrame` walkthrough.
4. AIF360 landing + a tour of the algorithm catalogue.
5. Dwork et al. (arXiv:1104.3913) — individual fairness metric formulation.
6. Kusner et al. (arXiv:1703.06856) — counterfactual fairness.
7. Lundberg & Lee (arXiv:1705.07874) and TreeSHAP paper (arXiv:1905.04610) — SHAP.
8. Ribeiro et al. — LIME paper and Anchors paper.
9. Wachter et al. — counterfactual explanations under GDPR (motivates Alibi counterfactual usage).
10. EEOC UGESP §1607.4 and §1607.5 — the four-fifths / validity framing.
11. NYC DCWP AEDT final rule — the LL144 field list.
12. ForHumanity certification-scheme index and BABL AI course catalogue — the credentialed report shapes.

## Paywalled and versioned sources — notes on access

- ISO/IEC TR 24027 and ISO/IEC TS 12791 are paywalled through national bodies (BSI in the UK, ANSI in the US). The ISO standard landing pages carry the abstract and TOC; the normative text requires purchase.
- Frontier-lab and vendor blog posts are *not* listed as primary sources for this module; when a claim depends on one, cite the underlying paper or standard, not the blog.
- Regulation text for NYC LL144, Colorado AI Act, and EU AI Act is published; always cite the version-of-record on the official portal, not a summary.
- arXiv IDs above are the version at first submission unless otherwise noted; check for revised versions on arXiv before citation in a formal report.
