# Resources — mod-107 Privacy-Risk Engineering for ML and LLM Systems

Primary sources first. Every reference below is a standards body, regulator, tool maintainer, or the original publication of an attack, defence, or method. Anything less authoritative is either not listed or marked as such.

## Attack literature — membership inference

- Shokri, R., Stronati, M., Song, C., Shmatikov, V. — "Membership Inference Attacks Against Machine Learning Models" — IEEE S&P 2017 — <https://arxiv.org/abs/1610.05820>
- Carlini, N., Chien, S., Nasr, M., Song, S., Terzis, A., Tramèr, F. — "Membership Inference Attacks From First Principles" (LiRA) — IEEE S&P 2022 — <https://arxiv.org/abs/2112.03570>
- Yeom, S., Giacomelli, I., Fredrikson, M., Jha, S. — "Privacy Risk in Machine Learning: Analyzing the Connection to Overfitting" — IEEE CSF 2018 — <https://arxiv.org/abs/1709.01604>

## Attack literature — attribute inference and model inversion

- Fredrikson, M., Lantz, E., Jha, S., Lin, S., Page, D., Ristenpart, T. — "Privacy in Pharmacogenetics: An End-to-End Case Study of Personalized Warfarin Dosing" — USENIX Security 2014 — <https://www.usenix.org/conference/usenixsecurity14/technical-sessions/presentation/fredrikson_matthew>
- Fredrikson, M., Jha, S., Ristenpart, T. — "Model Inversion Attacks that Exploit Confidence Information and Basic Countermeasures" — ACM CCS 2015 — <https://dl.acm.org/doi/10.1145/2810103.2813677>
- Zhang, Y., Jia, R., Pei, H., Wang, W., Li, B., Song, D. — "The Secret Revealer: Generative Model-Inversion Attacks Against Deep Neural Networks" — CVPR 2020 — <https://arxiv.org/abs/1911.07135>

<!-- needs-research: confirm the current strongest published model-inversion baseline for face recognisers and its metric convention at authoring time. -->

## Attack literature — LLM memorisation and extraction

- Carlini, N., Tramèr, F., Wallace, E., Jagielski, M., Herbert-Voss, A., Lee, K., Roberts, A., Brown, T., Song, D., Erlingsson, Ú., Oprea, A., Raffel, C. — "Extracting Training Data from Large Language Models" — USENIX Security 2021 — <https://arxiv.org/abs/2012.07805>
- Carlini, N., Ippolito, D., Jagielski, M., Lee, K., Tramèr, F., Zhang, C. — "Quantifying Memorization Across Neural Language Models" — ICLR 2023 — <https://arxiv.org/abs/2202.07646>
- Nasr, M., Carlini, N., Hayase, J., Jagielski, M., Cooper, A. F., Ippolito, D., Choquette-Choo, C. A., Wallace, E., Tramèr, F., Lee, K. — "Scalable Extraction of Training Data from (Production) Language Models" — 2023 — <https://arxiv.org/abs/2311.17035>
- Lee, K., Ippolito, D., Nystrom, A., Zhang, C., Eck, D., Callison-Burch, C., Carlini, N. — "Deduplicating Training Data Makes Language Models Better" — ACL 2022 — <https://arxiv.org/abs/2107.06499>

## Differential privacy — foundations

- Dwork, C., McSherry, F., Nissim, K., Smith, A. — "Calibrating Noise to Sensitivity in Private Data Analysis" — TCC 2006 — <https://link.springer.com/chapter/10.1007/11681878_14>
- Dwork, C., Roth, A. — "The Algorithmic Foundations of Differential Privacy" — 2014 — <https://www.cis.upenn.edu/~aaroth/Papers/privacybook.pdf>
- Mironov, I. — "Rényi Differential Privacy" — IEEE CSF 2017 — <https://arxiv.org/abs/1702.07476>
- Dong, J., Roth, A., Su, W. J. — "Gaussian Differential Privacy" — JRSS-B / JASA 2022 — <https://arxiv.org/abs/1905.02383>

## Differential privacy — DP-SGD and disparate-impact literature

- Abadi, M., Chu, A., Goodfellow, I., McMahan, H. B., Mironov, I., Talwar, K., Zhang, L. — "Deep Learning with Differential Privacy" — ACM CCS 2016 — <https://arxiv.org/abs/1607.00133>
- McMahan, H. B., Ramage, D., Talwar, K., Zhang, L. — "Learning Differentially Private Recurrent Language Models" — ICLR 2018 (user-level DP) — <https://arxiv.org/abs/1710.06963>
- Bagdasaryan, E., Poursaeed, O., Shmatikov, V. — "Differential Privacy Has Disparate Impact on Model Accuracy" — NeurIPS 2019 — <https://arxiv.org/abs/1905.12101>

## DP training libraries

- **Opacus** (PyTorch DP-SGD) — Meta / opacus.ai — <https://opacus.ai/> and <https://github.com/pytorch/opacus>
- **TensorFlow Privacy** — Google — <https://github.com/tensorflow/privacy>
- **dp_accounting** (accountants used by both libraries) — Google — <https://github.com/google/differential-privacy/tree/main/python/dp_accounting>

<!-- needs-research: pin the current recommended accountant path and per-example-gradient path in the latest Opacus and TensorFlow Privacy releases at authoring time. -->

## PII tooling

- **Microsoft Presidio** (analyser, anonymiser, image redactor) — Microsoft — <https://microsoft.github.io/presidio/>
- **Presidio GitHub** — <https://github.com/microsoft/presidio>

<!-- needs-research: pin the complete current list of built-in Presidio recognisers per jurisdiction at authoring time; the set grows release to release. -->

## Working evaluation harness

- **ML Privacy Meter** (Shokri- and LiRA-style MIA harness) — NUS Data Security and Systems Group — <https://github.com/privacytrustlab/ml_privacy_meter>

<!-- needs-research: confirm the current maintained release of ML Privacy Meter and its LiRA-style attack implementation status at authoring time. -->

## Adversarial-ML privacy taxonomy (context — carried from mod-105)

- **NIST AI 100-2 e2025 — Adversarial ML: A Taxonomy and Terminology of Attacks and Mitigations** — <https://csrc.nist.gov/publications/detail/nistir/8269/draft>
- **OWASP Top 10 for LLM Applications** — <https://owasp.org/www-project-top-10-for-large-language-model-applications/>
- **MITRE ATLAS** — <https://atlas.mitre.org/>

## Regulatory anchors

- **GDPR — Regulation (EU) 2016/679** (Official Journal, consolidated text) — <https://eur-lex.europa.eu/eli/reg/2016/679/oj>
  - Article 4 — Definitions (personal data, processing, profiling, biometric data)
  - Article 6 — Lawfulness of processing
  - Article 9 — Processing of special categories of personal data
  - Article 22 — Automated individual decision-making, including profiling
  - Article 25 — Data protection by design and by default
  - Article 32 — Security of processing
  - Article 35 — Data protection impact assessment
- **European Data Protection Board (EDPB)** — guidelines and opinions — <https://www.edpb.europa.eu/edpb_en>
- **EDPB Opinion 28/2024 on certain data protection aspects related to the processing of personal data in the context of AI models** — <https://www.edpb.europa.eu/our-work-tools/our-documents/opinion-board-art-64/opinion-282024-certain-data-protection-aspects_en>

<!-- needs-research: check for further EDPB opinions on AI-model privacy that have been published subsequent to Opinion 28/2024 at authoring time. -->

- **NIST Privacy Framework v1.0** (2020) — <https://www.nist.gov/privacy-framework>
- **NIST SP 800-53 Rev. 5** (security and privacy controls; Appendix J for privacy control families) — <https://csrc.nist.gov/publications/detail/sp/800-53/rev-5/final>
- **NIST SP 800-188** (De-identifying Government Datasets) — <https://csrc.nist.gov/publications/detail/sp/800-188/final>
- **EU AI Act (Regulation (EU) 2024/1689)** — carried from mod-101 — <https://eur-lex.europa.eu/eli/reg/2024/1689/oj> (Article 10 on data governance is the principal AI-Act privacy anchor)

## Sector-specific privacy anchors (context)

- **HIPAA Privacy Rule** (US health) — <https://www.hhs.gov/hipaa/for-professionals/privacy/index.html>
- **CCPA / CPRA** (California) — <https://oag.ca.gov/privacy/ccpa>

<!-- needs-research: confirm current authoritative landing pages for each of the sector anchors and revise if the responsible agency has restructured its site at authoring time. -->

## Related reading — from mod-102 and mod-103, revisited here

- MIT AI Risk Repository — <https://airisk.mit.edu/> (domain "Privacy & Security")
- ISO/IEC 27701 — Privacy information management (extension to ISO/IEC 27001) — <https://www.iso.org/standard/71670.html>
- ISO/IEC 29134 — Privacy impact assessment guidance — <https://www.iso.org/standard/62289.html>

## Reading order for this module

If you are working through mod-107 for the first time, read in this order:

1. NIST Privacy Framework v1.0 (Core + Functions overview) — the vocabulary the report is written in.
2. GDPR Articles 4, 6, 9, 22, 25, 32, 35 — the regulatory anchor.
3. Shokri et al. 2017 (MIA) + Carlini et al. 2022 (LiRA) — the anchor attack pair, in order.
4. Fredrikson et al. 2014 + 2015 — attribute inference and model inversion.
5. Carlini et al. 2021 (Extracting Training Data), Carlini et al. 2023 (Quantifying Memorization), Nasr et al. 2023 (Scalable Extraction) — LLM memorisation and extraction, in order.
6. Dwork & Roth "Algorithmic Foundations" chapters 1-3 — DP foundations.
7. Abadi et al. 2016 (DP-SGD) + Mironov 2017 (RDP) — DP-SGD and accounting.
8. Bagdasaryan et al. 2019 — the disparate-impact caveat on DP-SGD; cross-link to mod-106.
9. Opacus documentation (Getting Started + accountant page) — the working library.
10. TensorFlow Privacy README + tutorials — the parallel library.
11. Presidio documentation (Analyzer + Anonymizer + Custom Recognizers pages) — the PII pipeline.
12. ML Privacy Meter README + example notebooks — the working attack harness.
13. EDPB Opinion 28/2024 — the current European regulatory read of AI-model privacy.

## Notes on paywalled and versioned sources

- The ISO standards (27701, 29134) are paywalled through ISO national bodies. Same purchase options as mod-101 and mod-102.
- The attack papers (Shokri, Fredrikson, Carlini, Nasr) are all posted on arXiv and are freely available.
- Opacus, TensorFlow Privacy, and Presidio release frequently; pin the exact version and commit hash cited in any privacy-risk report.
- ML Privacy Meter is research-grade; treat as living code and pin the version alongside the results.
- EDPB Opinion 28/2024 is the current European regulatory reading of AI-model privacy at authoring time; check for further opinions before citing.
- GDPR articles are stable; use the Official Journal consolidated text rather than secondary paraphrases.

Do not rely on unofficial re-postings of any standard or regulation; they are copyright violations for paywalled standards and their fidelity to the current text is uncertain.
