# Resources — mod-108 Guardrail Engineering and Effectiveness Measurement

Primary sources first. Every reference below is a standards body, an official tooling repository / documentation site, a classifier's model card or paper, or a vendor's own site. Anything less authoritative is either not listed or explicitly marked.

## Guardrail architecture guidance

- **OWASP Top 10 for LLM Applications** — <https://owasp.org/www-project-top-10-for-large-language-model-applications/>
- **OWASP GenAI Security Project — Guardrails and other GenAI security guidance** — <https://genai.owasp.org/>
- **Google Secure AI Framework (SAIF)** — <https://safety.google/cybersecurity-advancements/saif/> and <https://saif.google/>
- **MITRE ATLAS** — Adversarial Threat Landscape for Artificial-Intelligence Systems — <https://atlas.mitre.org/>
- **NIST AI RMF 1.0 (AI 100-1)** — background context, MEASURE / MANAGE functions — <https://nvlpubs.nist.gov/nistpubs/ai/NIST.AI.100-1.pdf>
- **NIST AI RMF Playbook** — day-to-day sub-category reference — <https://airc.nist.gov/AI_RMF_Knowledge_Base/Playbook>
- **NIST AI 600-1 (Generative AI Profile)** — the twelve GenAI risks used to check coverage — <https://airc.nist.gov/AI_RMF_Knowledge_Base/AI_RMF/Uses/GenAI-Profile>

<!-- needs-research: pin the exact current version identifier of OWASP LLM Top 10 (v2025 renumbering shifted several category codes vs. 2023 v1) and the current MITRE ATLAS matrix version at authoring time. -->

## Open-source rule / flow / PII tooling

- **NVIDIA NeMo Guardrails** — documentation site — <https://docs.nvidia.com/nemo/guardrails/>
- **NVIDIA NeMo Guardrails** — source repository — <https://github.com/NVIDIA/NeMo-Guardrails>
- **Colang** language reference — <https://docs.nvidia.com/nemo/guardrails/user_guides/colang-language-syntax-guide.html>
- **Guardrails AI** — documentation site — <https://docs.guardrailsai.com/>
- **Guardrails AI Validator Hub** — <https://hub.guardrailsai.com/>
- **Guardrails AI** — source repository — <https://github.com/guardrails-ai/guardrails>
- **Microsoft Presidio** — documentation site — <https://microsoft.github.io/presidio/>
- **Microsoft Presidio** — source repository — <https://github.com/microsoft/presidio>
- **Presidio Analyzer** recogniser reference — <https://microsoft.github.io/presidio/analyzer/>
- **Presidio Anonymizer** operator reference — <https://microsoft.github.io/presidio/anonymizer/>

<!-- needs-research: cite the current stable Colang version (Colang 2.x direction) and the current Guardrails AI API shape (the library has moved between RAIL XML and Pydantic-model-driven guards); confirm the currently supported entry point at authoring time. -->

## Classifier guardrails — models, model cards, papers

- **Meta Llama Guard** — model card + release notes on Meta's llama.com — <https://llama.com/> (locate current Llama Guard family release, model card, and taxonomy)
- **Meta Llama Guard** — Hugging Face model hub (community-hosted authoritative model cards, when official release is mirrored) — search `meta-llama` organisation on <https://huggingface.co/>
- **Meta Llama Guard 3** — paper: Inan et al., "Llama Guard: LLM-based Input-Output Safeguard for Human-AI Conversations" (Llama Guard 1 origin paper; read alongside successor model cards for taxonomy evolution) — <https://arxiv.org/abs/2312.06674>
- **Google ShieldGemma** — technical report / paper — Zeng, et al., "ShieldGemma: Generative AI Content Moderation Based on Gemma" — <https://arxiv.org/abs/2407.21772>
- **Google ShieldGemma** — model card and release on the Google AI / Gemma docs — <https://ai.google.dev/gemma/docs/shieldgemma>
- **OpenAI Moderation** — API documentation — <https://platform.openai.com/docs/guides/moderation>
- **OpenAI Moderation** — model overview — <https://platform.openai.com/docs/models> (locate the current `omni-moderation-*` / `text-moderation-*` model identifier)
- **Anthropic Constitutional Classifiers** — research publication "Constitutional Classifiers: Defending against Universal Jailbreaks across Thousands of Hours of Red Teaming" — <https://www.anthropic.com/research/constitutional-classifiers>
- **Anthropic — Constitutional AI (foundational)** — Bai et al., "Constitutional AI: Harmlessness from AI Feedback" — <https://arxiv.org/abs/2212.08073>

<!-- needs-research: pin the exact currently-released Llama Guard version (3 vs. 4 vs. multimodal variants), the current ShieldGemma release (1 vs. 2 with multimodal), the current OpenAI Moderation model identifier, and the current productisation state of Anthropic Constitutional-Classifier-derived filters at authoring time. -->

## Third-party vendor guardrail products (each vendor's own site is authoritative)

- **Lakera Guard** (Lakera) — <https://www.lakera.ai/> (product page: Lakera Guard)
- **Arthur AI** — <https://www.arthur.ai/> (product line: Arthur Shield / Arthur runtime guardrails)
- **Robust Intelligence** — <https://www.robustintelligence.com/> (note: acquired by Cisco in 2024; verify current productisation)
- **Calypso AI** — <https://calypsoai.com/>
- **HiddenLayer** — <https://hiddenlayer.com/>
- **Protect AI** — <https://protectai.com/> (product portfolio: `ModelScan`, `Guardian`, `Recon`, `LayerX`, `Radar`, others; some components open source)

<!-- needs-research: at RFP time, cite each vendor's current product name, deployment options (SaaS / VPC / on-prem), pricing model, and evaluation methodology publication. All of these evolve; do not rely on this list without a fresh check. -->

### Complementary vendor context

- **Protect AI ModelScan** (open source) — <https://github.com/protectai/modelscan>
- **HiddenLayer AI Detection & Response** overview — on the HiddenLayer site above
- **Robust Intelligence AI Firewall** overview — on the Robust Intelligence / Cisco site

## Adversarial-payload sources for measurement (used per mod-104 workflow, cited here for cross-reference)

- **garak** — LLM vulnerability scanner — <https://github.com/leondz/garak>
- **PyRIT** — Python Risk Identification Toolkit — <https://github.com/Azure/PyRIT>
- **AgentDojo** — indirect prompt-injection benchmark for tool-using agents — <https://agentdojo.spylab.ethz.ch/>

These live in mod-104's resources; they are referenced here because the effectiveness measurement in chapter 05 consumes their output.

## Adversarial-ML context (mod-105 overlap — cited for the classifier-degradation shape)

- **NIST AI 100-2 e2025 — Adversarial ML: A Taxonomy and Terminology of Attacks and Mitigations** — <https://csrc.nist.gov/publications/detail/nistir/8269/draft>

## Related reading — from earlier modules, revisited here

- **NIST AI RMF Playbook** — MEASURE 2.3 / 2.7 sub-categories are the ones this module's evidence maps to — <https://airc.nist.gov/AI_RMF_Knowledge_Base/Playbook>
- **ISO/IEC 42001:2023** — AI management system — <https://www.iso.org/standard/81230.html>
- **ISO/IEC 42005:2025** — AI system impact assessment — <https://www.iso.org/standard/44545.html>
- **Regulation (EU) 2024/1689 (EU AI Act)** — Articles 9 (risk management) and 15 (accuracy, robustness, cybersecurity) are the primary consumers of the evidence this module produces — <https://eur-lex.europa.eu/eli/reg/2024/1689/oj>

## Reading order

If you are working through mod-108 for the first time, read in this order:

1. **Architecture.** OWASP LLM Top 10 landing page + read the ten category descriptions (chapter 01).
2. **Architecture.** MITRE ATLAS matrix landing page + skim the tactics (chapter 01).
3. **Architecture.** Google SAIF site + the six pillars (chapter 01).
4. **Rule / flow.** NeMo Guardrails documentation home + one Colang example (chapter 02).
5. **Rule / flow.** Guardrails AI documentation home + one validator example (chapter 02).
6. **Classifier.** Llama Guard model card of the current release (chapter 03).
7. **Classifier.** ShieldGemma technical report and model card (chapter 03).
8. **Classifier.** OpenAI Moderation API documentation (chapter 03).
9. **Classifier.** Anthropic Constitutional Classifiers publication (chapter 03).
10. **PII.** Presidio documentation home + analyser recogniser list + anonymiser operators (chapter 04).
11. **Measurement.** NIST AI RMF Playbook MEASURE 2.3 / 2.7 (chapter 05).
12. **Vendor review.** Each of the six vendor sites; cite the current product page and deployment options at RFP time (chapter 06).

## Reading order — versioning and paywalled sources

- **OWASP LLM Top 10** is versioned (v1 2023, v2025 update). Cite the version.
- **NeMo Guardrails / Colang**, **Guardrails AI**, and **Presidio** are versioned software; pin the version you install in the exercise, and note that recogniser catalogues and validator lists move.
- **Llama Guard**, **ShieldGemma**, and **OpenAI Moderation** are versioned models; pin the checkpoint / model identifier alongside every measurement row.
- **ISO/IEC 42001, 42005** are paywalled through ISO national bodies (same options as earlier modules).
- **Vendor product pages** are volatile — evaluate against the current page at RFP time, not against a cached read.

Do not rely on unofficial re-postings of any tool documentation, model card, or vendor page. Model cards in particular carry the safety-and-usage disclaimers that are load-bearing for defensible use.
