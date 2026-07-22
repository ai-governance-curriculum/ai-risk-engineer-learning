# Resources — mod-105 Adversarial ML and Robustness Engineering

Primary sources first. Every reference below is a standards body, a MITRE / OWASP / government project, an open-source tool project page, an original arXiv / conference paper, or a government AI safety institute. Anything less authoritative is either not listed or marked explicitly.

## Working adversarial-ML taxonomy

- **NIST AI 100-2 e2025 — "Adversarial Machine Learning: A Taxonomy and Terminology of Attacks and Mitigations"** — NIST CSRC — <https://csrc.nist.gov/publications/detail/nistir/8269/final>
- **MITRE ATLAS — Adversarial Threat Landscape for Artificial-Intelligence Systems** — MITRE — <https://atlas.mitre.org/>
- **OWASP Machine Learning Security Top 10** — OWASP — <https://owasp.org/www-project-machine-learning-security-top-10/>
- **OWASP Top 10 for LLM Applications** (cross-tag for dual-domain scenarios) — OWASP — <https://owasp.org/www-project-top-10-for-large-language-model-applications/>
- **AI Vulnerability Database (AVID)** — <https://avidml.org/>

<!-- needs-research: pin the exact publication identifier and revision date of NIST AI 100-2 e2025 at authoring time; the e2025 revision replaces the prior NISTIR draft. -->
<!-- needs-research: pin the current published version number of OWASP ML Top 10 and OWASP LLM Top 10 at authoring time; both are versioned lists. -->
<!-- needs-research: confirm active-maintenance status of AVID; if it has moved or paused, update or remove. -->

## Tooling — attack runners and defence libraries

- **Adversarial Robustness Toolbox (ART)** — Linux Foundation AI & Data — <https://github.com/Trusted-AI/adversarial-robustness-toolbox>
- **TextAttack** — QData / University of Virginia — <https://github.com/QData/TextAttack>
- **CleverHans** — CleverHans Lab — <https://github.com/cleverhans-lab/cleverhans>
- **Counterfit** — Microsoft (Azure Trustworthy ML) — <https://github.com/Azure/counterfit>
- **RobustBench** — standardised adversarial-robustness leaderboard and API — <https://robustbench.github.io/>
- **Foolbox** — Bethge Lab — <https://github.com/bethgelab/foolbox>

<!-- needs-research: verify current maintainer, latest release, and framework-support coverage for each tool at authoring time. -->
<!-- needs-research: confirm Counterfit is still actively maintained; if it has been archived, note the successor tool. -->

## Attack literature — evasion

- Goodfellow, I. J., Shlens, J., Szegedy, C. — "Explaining and Harnessing Adversarial Examples" (ICLR 2015 / arXiv:1412.6572) — <https://arxiv.org/abs/1412.6572> — FGSM.
- Madry, A., Makelov, A., Schmidt, L., Tsipras, D., Vladu, A. — "Towards Deep Learning Models Resistant to Adversarial Attacks" (ICLR 2018 / arXiv:1706.06083) — <https://arxiv.org/abs/1706.06083> — PGD, adversarial training.
- Carlini, N., Wagner, D. — "Towards Evaluating the Robustness of Neural Networks" (IEEE S&P 2017 / arXiv:1608.04644) — <https://arxiv.org/abs/1608.04644> — C&W attacks.
- Croce, F., Hein, M. — "Reliable evaluation of adversarial robustness with an ensemble of diverse parameter-free attacks" (ICML 2020 / arXiv:2003.01690) — <https://arxiv.org/abs/2003.01690> — AutoAttack.
- Carlini, N., Athalye, A., Papernot, N., Brendel, W., Rauber, J., Tsipras, D., Goodfellow, I., Madry, A., Kurakin, A. — "On Evaluating Adversarial Robustness" (arXiv:1902.06705) — <https://arxiv.org/abs/1902.06705> — the canonical guidance on adaptive-attack evaluation of defences.
- Jin, D., Jin, Z., Zhou, J. T., Szolovits, P. — "Is BERT Really Robust? A Strong Baseline for Natural Language Attack on Text Classification and Entailment" (AAAI 2020 / arXiv:1907.11932) — <https://arxiv.org/abs/1907.11932> — TextFooler.
- Garg, S., Ramakrishnan, G. — "BAE: BERT-based Adversarial Examples for Text Classification" (EMNLP 2020 / arXiv:2004.01970) — <https://arxiv.org/abs/2004.01970> — BAE.

## Attack literature — extraction, inversion, membership inference

- Tramèr, F., Zhang, F., Juels, A., Reiter, M. K., Ristenpart, T. — "Stealing Machine Learning Models via Prediction APIs" (USENIX Security 2016 / arXiv:1609.02943) — <https://arxiv.org/abs/1609.02943> — model extraction.
- Orekondy, T., Schiele, B., Fritz, M. — "Knockoff Nets: Stealing Functionality of Black-Box Models" (CVPR 2019 / arXiv:1812.02766) — <https://arxiv.org/abs/1812.02766>.
- Fredrikson, M., Jha, S., Ristenpart, T. — "Model Inversion Attacks that Exploit Confidence Information and Basic Countermeasures" (CCS 2015) — <https://dl.acm.org/doi/10.1145/2810103.2813677>.
- Zhu, L., Liu, Z., Han, S. — "Deep Leakage from Gradients" (NeurIPS 2019 / arXiv:1906.08935) — <https://arxiv.org/abs/1906.08935> — gradient-inversion.
- Shokri, R., Stronati, M., Song, C., Shmatikov, V. — "Membership Inference Attacks Against Machine Learning Models" (IEEE S&P 2017 / arXiv:1610.05820) — <https://arxiv.org/abs/1610.05820>.
- Yeom, S., Giacomelli, I., Fredrikson, M., Jha, S. — "Privacy Risk in Machine Learning: Analyzing the Connection to Overfitting" (CSF 2018 / arXiv:1709.01604) — <https://arxiv.org/abs/1709.01604>.
- Carlini, N., Chien, S., Nasr, M., Song, S., Terzis, A., Tramèr, F. — "Membership Inference Attacks From First Principles" (IEEE S&P 2022 / arXiv:2112.03570) — <https://arxiv.org/abs/2112.03570> — LiRA.
- Abadi, M., Chu, A., Goodfellow, I., McMahan, H. B., Mironov, I., Talwar, K., Zhang, L. — "Deep Learning with Differential Privacy" (CCS 2016 / arXiv:1607.00133) — <https://arxiv.org/abs/1607.00133> — DP-SGD.

## Poisoning attack and defence literature

- Gu, T., Dolan-Gavitt, B., Garg, S. — "BadNets: Identifying Vulnerabilities in the Machine Learning Model Supply Chain" (arXiv:1708.06733) — <https://arxiv.org/abs/1708.06733> — foundational backdoor attack.
- Shafahi, A., Huang, W. R., Najibi, M., Suciu, O., Studer, C., Dumitras, T., Goldstein, T. — "Poison Frogs! Targeted Clean-Label Poisoning Attacks on Neural Networks" (NeurIPS 2018 / arXiv:1804.00792) — <https://arxiv.org/abs/1804.00792>.
- Chen, B., Carvalho, W., Baracaldo, N., Ludwig, H., Edwards, B., Lee, T., Molloy, I., Srivastava, B. — "Detecting Backdoor Attacks on Deep Neural Networks by Activation Clustering" (SafeAI @ AAAI 2019 / arXiv:1811.03728) — <https://arxiv.org/abs/1811.03728> — activation-clustering defence.
- Tran, B., Li, J., Madry, A. — "Spectral Signatures in Backdoor Attacks" (NeurIPS 2018 / arXiv:1811.00636) — <https://arxiv.org/abs/1811.00636> — spectral-signature defence.
- Gao, Y., Xu, C., Wang, D., Chen, S., Ranasinghe, D. C., Nepal, S. — "STRIP: A Defence Against Trojan Attacks on Deep Neural Networks" (ACSAC 2019 / arXiv:1902.06531) — <https://arxiv.org/abs/1902.06531>.
- Wang, B., Yao, Y., Shan, S., Li, H., Viswanath, B., Zheng, H., Zhao, B. Y. — "Neural Cleanse: Identifying and Mitigating Backdoor Attacks in Neural Networks" (IEEE S&P 2019) — <https://ieeexplore.ieee.org/document/8835365> — trigger-reverse-engineering.

<!-- needs-research: confirm whether "Spectre" in the module description refers to the Tran/Li/Madry spectral-signatures line of work or a distinct method; if the latter, add the correct citation. -->

## Indirect-adjacent — LLM red-team literature (cross-reference to mod-104)

- Perez, F., Ribeiro, I. — "Ignore Previous Prompt: Attack Techniques For Language Models" (arXiv:2211.09527) — <https://arxiv.org/abs/2211.09527>.
- Greshake, K., Abdelnabi, S., Mishra, S., Endres, C., Holz, T., Fritz, M. — "Not what you've signed up for: Compromising Real-World LLM-Integrated Applications with Indirect Prompt Injection" (arXiv:2302.12173) — <https://arxiv.org/abs/2302.12173>.

## Government AI safety institute guidance

- **UK AI Safety Institute (UK AISI)** — <https://www.aisi.gov.uk/>
- **US AI Safety Institute (US AISI)** — <https://www.nist.gov/aisi>
- **NIST AI Resource Center** (AIRC — hosts the AI RMF, GenAI Profile, and adversarial-ML documents) — <https://airc.nist.gov/>

<!-- needs-research: pin any current AISI evaluation-methodology publications relevant to adversarial-ML robustness at authoring time. -->

## Related reading — from earlier modules, revisited here

- NIST AI RMF 1.0 (AI 100-1) — <https://nvlpubs.nist.gov/nistpubs/ai/NIST.AI.100-1.pdf>
- NIST AI RMF Generative AI Profile (AI 600-1) — <https://airc.nist.gov/AI_RMF_Knowledge_Base/AI_RMF/Uses/GenAI-Profile>
- ISO/IEC 23894:2023 — Guidance on risk management (AI) — <https://www.iso.org/standard/77304.html>
- ISO/IEC 42001:2023 — AI management system — <https://www.iso.org/standard/81230.html>

## Reading order for this module

If you are working through mod-105 for the first time, read in this order:

1. NIST AI 100-2 e2025 — full read; the six-category partition and the threat-model rubric are the module's spine (chapter 01).
2. MITRE ATLAS — walk the matrix once end-to-end and open two or three case-study techniques in detail (chapter 01).
3. OWASP ML Top 10 — full read; note the AppSec framing and the overlap / divergence from NIST (chapter 01).
4. Madry et al. 2018 (PGD) — read for the adversarial-training framing that anchors the evasion panel (chapter 02).
5. Carlini & Wagner 2017 — read for the defence-defeat framing (chapter 02).
6. Carlini et al. 2019 — "On Evaluating Adversarial Robustness" — read before you baseline any defence (chapter 02, chapter 05).
7. Gu, Dolan-Gavitt, Garg (BadNets) — read for the backdoor attack shape (chapter 03).
8. Chen et al. 2019 (activation clustering) and Tran, Li, Madry 2018 (spectral signatures) — read for the defence-baseline shape (chapter 03).
9. Tramèr et al. 2016 (extraction) and Shokri et al. 2017 (MIA) — read for the API-surface attack framing (chapter 04).
10. Abadi et al. 2016 (DP-SGD) — read at the survey level; mod-107 has the formal-accounting depth (chapter 04).

## Reading order — the paywalled and versioned sources

- Conference / IEEE / ACM proceedings (S&P, CCS, NeurIPS, ICML, ICLR, USENIX Security, ACSAC) are sometimes paywalled; most authors post arXiv preprints — cite the arXiv identifier alongside the conference identifier.
- NIST AI 100-2 is versioned — the e2025 revision replaces earlier drafts; cite the version you read.
- MITRE ATLAS is a living matrix; cite the technique IDs plus the release / snapshot date.
- OWASP ML Top 10 and OWASP LLM Top 10 are versioned; cite the version.
- ISO standards are paywalled through national bodies; access via national-body purchase, lead-implementer training, or the publicly available summaries on ISO's standard landing pages.
- Tool releases (ART, TextAttack, CleverHans) are versioned; pin the release / commit hash in the robustness-eval report methodology section.

Do not rely on unofficial re-postings of paywalled standards; those may be copyright violations and their fidelity to the current release is uncertain.
