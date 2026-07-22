# Resources — mod-110 Model Risk Management (MRM) for ML/AI and AI Supply Chain

Primary sources first. Every reference below is a regulator, standards body, tool project, or original publication. Anything less authoritative is either not listed or marked as such.

## US financial-services MRM anchors

- **Federal Reserve SR 11-7 — Guidance on Model Risk Management** (April 2011) — <https://www.federalreserve.gov/supervisionreg/srletters/sr1107.htm>
- **OCC Bulletin 2011-12 — Sound Practices for Model Risk Management** (companion to SR 11-7) — <https://www.occ.treas.gov/news-issuances/bulletins/2011/bulletin-2011-12.html>
- **Federal Reserve SR 23-4 — Interagency Guidance on Third-Party Relationships: Risk Management** (June 2023, joint with OCC and FDIC) — <https://www.federalreserve.gov/supervisionreg/srletters/SR2304.htm>
- **OCC Bulletin 2023-17 — Third-Party Relationships: Interagency Guidance on Risk Management** (companion to SR 23-4) — <https://www.occ.treas.gov/news-issuances/bulletins/2023/bulletin-2023-17.html>
- **FDIC FIL-29-2023 — Interagency Guidance on Third-Party Relationships: Risk Management** — <https://www.fdic.gov/news/financial-institution-letters/2023/fil23029.html>

<!-- needs-research: confirm the current form of any supplementary agency guidance (e.g., interagency supplements or FAQ documents) that has been published since SR 23-4 to clarify its application to AI/ML vendor relationships. -->

## US health-AI anchors

- **FDA / Health Canada / MHRA — Good Machine Learning Practice for Medical Device Development: Guiding Principles** (October 2021) — <https://www.fda.gov/medical-devices/software-medical-device-samd/good-machine-learning-practice-medical-device-development-guiding-principles>
- **FDA — Marketing Submission Recommendations for a Predetermined Change Control Plan for Artificial Intelligence-Enabled Device Software Functions** (final guidance, December 2024) — <https://www.fda.gov/regulatory-information/search-fda-guidance-documents/marketing-submission-recommendations-predetermined-change-control-plan-artificial-intelligence>
- **FDA — Artificial Intelligence and Machine Learning (AI/ML) Software as a Medical Device Action Plan** (January 2021) — <https://www.fda.gov/medical-devices/software-medical-device-samd/artificial-intelligence-and-machine-learning-software-medical-device>
- **FDA — Artificial Intelligence-Enabled Medical Devices (list of authorised devices)** — <https://www.fda.gov/medical-devices/software-medical-device-samd/artificial-intelligence-enabled-medical-devices>
- **IMDRF — Software as a Medical Device (SaMD): Key Definitions** (December 2013) — <https://www.imdrf.org/documents/software-medical-device-samd-key-definitions>
- **IMDRF — "Software as a Medical Device": Possible Framework for Risk Categorization and Corresponding Considerations** (September 2014) — <https://www.imdrf.org/documents/software-medical-device-possible-framework-risk-categorization-and-corresponding-considerations>

<!-- needs-research: pin the current published version of the FDA PCCP final guidance and note any post-finalisation supplementary Q&A the agency has issued. -->

## Provenance and SBOM standards — data

- **Gebru, T. et al. — "Datasheets for Datasets"** (2018, updated 2021 in Communications of the ACM) — <https://arxiv.org/abs/1803.09010>
- **Pushkarna, M. et al. (Google Research) — "Data Cards: Purposeful and Transparent Dataset Documentation for Responsible AI"** (2022) — <https://arxiv.org/abs/2204.01075>
- **Data Cards Playbook (Google) — practitioner-oriented Data Cards guidance** — <https://sites.research.google/datacardsplaybook/>

## Provenance and SBOM standards — models

- **Mitchell, M. et al. — "Model Cards for Model Reporting"** (2019) — <https://arxiv.org/abs/1810.03993>
- **Hugging Face — Model Card documentation and template** — <https://huggingface.co/docs/hub/model-cards>
- **Hugging Face — Model Card Guidebook** — <https://huggingface.co/docs/hub/model-card-guidebook>

## Provenance and SBOM standards — ML-BOM and SPDX

- **CycloneDX — official project site (OWASP)** — <https://cyclonedx.org/>
- **CycloneDX — Machine Learning BOM specification** — <https://cyclonedx.org/capabilities/mlbom/>
- **CycloneDX — current specification (JSON / XML schemas)** — <https://cyclonedx.org/specification/overview/>
- **SPDX — official project site (Linux Foundation)** — <https://spdx.dev/>
- **SPDX 3.0 specification** — <https://spdx.github.io/spdx-spec/v3.0/>
- **SPDX 3.0 AI Profile specification** — <https://spdx.github.io/spdx-spec/v3.0/model/AI/AI/>
- **SPDX 3.0 Dataset Profile specification** — <https://spdx.github.io/spdx-spec/v3.0/model/Dataset/Dataset/>

<!-- needs-research: confirm the current published minor version of SPDX 3.0 and any post-3.0 revisions to the AI profile before citing schema field names precisely. -->

## Provenance and SBOM standards — SLSA

- **SLSA — official project site (OpenSSF)** — <https://slsa.dev/>
- **SLSA v1.0 specification** — <https://slsa.dev/spec/v1.0/>
- **SLSA Build track levels** — <https://slsa.dev/spec/v1.0/levels>

<!-- needs-research: pin the current top-level SLSA specification version and its highest defined Build level at authoring time; levels have evolved between v0.1 and v1.0. -->

## Model signing — Sigstore and cosign

- **Sigstore — project site (OpenSSF)** — <https://www.sigstore.dev/>
- **cosign — GitHub repository (part of Sigstore)** — <https://github.com/sigstore/cosign>
- **Fulcio — the Sigstore certificate authority** — <https://github.com/sigstore/fulcio>
- **Rekor — the Sigstore transparency log** — <https://github.com/sigstore/rekor>
- **Sigstore documentation** — <https://docs.sigstore.dev/>
- **Model Signing project (Sigstore / OpenSSF)** — <https://github.com/sigstore/model-transparency>

<!-- needs-research: confirm the current name and scope of the Sigstore-adjacent model-transparency / model-signing project and cite its most recent release notes or documentation entrypoint. -->

## Malicious-artefact scanning for ML models

- **Protect AI ModelScan — GitHub repository** — <https://github.com/protectai/modelscan>
- **Protect AI ModelScan — project documentation** — <https://protectai.com/modelscan>
- **Hugging Face picklescan — GitHub repository** — <https://github.com/mmaitre314/picklescan>
- **Hugging Face — Pickle scanning documentation** — <https://huggingface.co/docs/hub/security-pickle>
- **safetensors — GitHub repository (Hugging Face safe tensor format)** — <https://github.com/huggingface/safetensors>
- **GGUF format specification (part of the llama.cpp project)** — <https://github.com/ggerganov/llama.cpp/blob/master/docs/gguf.md>

<!-- needs-research: confirm current maintainer and canonical repository for picklescan (it has moved between individual and organisational ownership); if a more current fork is the reference implementation, update. -->

## Commercial ML-security platforms (context)

- **HiddenLayer AISec Platform** — <https://hiddenlayer.com/aisec-platform/>
- **Protect AI Guardian** — <https://protectai.com/guardian>
- **Robust Intelligence AI Firewall / TrustSecurity products** — <https://www.robustintelligence.com/>

<!-- needs-research: confirm the current stable product names and scope for these commercial ML-security platforms; product lines and naming change frequently and any specific claims about coverage should be checked against the vendor's current documentation. -->

## Adjacent supply-chain frameworks referenced from the chapters

- **OpenSSF — Open Source Security Foundation** — <https://openssf.org/>
- **in-toto — supply-chain attestation framework** — <https://in-toto.io/>
- **NTIA — Software Bill of Materials primer** — <https://www.ntia.gov/page/software-bill-materials>
- **NIST SP 800-218 — Secure Software Development Framework (SSDF)** — <https://csrc.nist.gov/publications/detail/sp/800-218/final>
- **NIST SP 800-161 — Cybersecurity Supply Chain Risk Management Practices for Systems and Organizations** — <https://csrc.nist.gov/publications/detail/sp/800-161/rev-1/final>
- **CISA — Software Bill of Materials resource hub** — <https://www.cisa.gov/sbom>

## Related reading — from earlier modules, revisited here

- **NIST AI RMF Playbook** — <https://airc.nist.gov/AI_RMF_Knowledge_Base/Playbook> (for the MEASURE 2.7 and MANAGE 4.1 mapping used by chapters 05-06)
- **NIST AI 100-1 (AI RMF 1.0)** — <https://nvlpubs.nist.gov/nistpubs/ai/NIST.AI.100-1.pdf>
- **ISO/IEC 42001:2023 — AI management system (AIMS)** — <https://www.iso.org/standard/81230.html>
- **ISO/IEC 23894:2023 — Guidance on risk management (AI)** — <https://www.iso.org/standard/77304.html>
- **Regulation (EU) 2024/1689 — the EU AI Act (Official Journal)** — <https://eur-lex.europa.eu/eli/reg/2024/1689/oj>

## Reading order for this module

If you are working through mod-110 for the first time, read in this order:

1. **SR 11-7** (short — 21 pages) and skim **OCC 2011-12**. These frame chapter 01.
2. **SR 23-4** (the interagency guidance itself, plus the OCC and FDIC companion FILs). Frames chapter 02.
3. **FDA / HC / MHRA GMLP guiding principles** (short — ten principles, a few pages). Then **FDA PCCP final guidance** (longer; skim sections one and two first, then work through the modification-protocol section carefully). Frames chapter 03.
4. **Datasheets for Datasets** paper, then **Data Cards** paper, then **Model Cards** paper. Frames chapter 04 layer 1-2.
5. **CycloneDX ML-BOM specification** and **SPDX 3.0 AI Profile specification**. Frames chapter 04 layer 3-4.
6. **SLSA v1.0 specification and Build track levels**. Frames chapter 04 layer 5.
7. **Sigstore project overview** and **cosign quickstart**. Frames chapter 05.
8. **ModelScan documentation** and **picklescan repository README** and **Hugging Face pickle-scanning documentation**. Frames chapter 06.

## Reading order — the paywalled and versioned sources

- ISO/IEC 23894, 42001, 42005 are paywalled through ISO national bodies. Same options as prior modules: national-body purchase, lead-implementer training, or the publicly available summaries on ISO's standard landing pages.
- SR 11-7, SR 23-4, OCC bulletins, FDIC FILs are freely available at the source URLs above; treat the agency's own page as canonical rather than any secondary re-posting.
- FDA guidance documents are freely available and versioned; cite the *final guidance* date, not the draft, unless comparing draft to final for a historical argument.
- CycloneDX, SPDX, SLSA, and Sigstore are actively developed; pin the specification version you cite (e.g., "CycloneDX 1.6 ML-BOM", "SPDX 3.0.1 AI Profile", "SLSA v1.0") and note the publication date.
- Model Cards, Datasheets for Datasets, Data Cards, and the various academic papers cited above are stable — cite the arXiv identifier plus the venue if published.
- Tool repositories (ModelScan, picklescan, cosign) move; check for the current maintained repository and note the version you are using in every artefact you produce.

Do not rely on unofficial re-postings of any standard or guidance text; those are copyright violations for paywalled standards and their fidelity to the current release is uncertain.
