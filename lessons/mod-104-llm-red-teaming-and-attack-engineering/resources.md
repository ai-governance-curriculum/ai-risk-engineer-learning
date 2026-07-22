# Resources — mod-104 LLM and Agent Red-Team Engineering

Primary sources first. Every reference below is a standards body, tool project page, government AI safety institute, foundation-lab publication, or a peer-reviewed / arXiv paper. Anything less authoritative is either not listed or marked as such.

## OWASP and threat taxonomies

- **OWASP GenAI Red-Team Guide** — OWASP GenAI Security Project — <https://genai.owasp.org/> <!-- needs-research: pin the exact document title, version, and stable URL for the GenAI Red-Team Guide at authoring time. -->
- **OWASP Top 10 for LLM Applications** — <https://owasp.org/www-project-top-10-for-large-language-model-applications/>
- **OWASP Machine Learning Security Top 10** — <https://owasp.org/www-project-machine-learning-security-top-10/>
- **MITRE ATLAS** — Adversarial Threat Landscape for Artificial-Intelligence Systems — <https://atlas.mitre.org/>
- **NIST AI 100-2 e2025 — Adversarial ML: A Taxonomy and Terminology of Attacks and Mitigations** — <https://csrc.nist.gov/publications/detail/nistir/8269/draft> <!-- needs-research: confirm the current NIST publication identifier for the adversarial-ML taxonomy — the series has evolved from NISTIR 8269 to the AI 100-2 form. -->
- **AI Vulnerability Database (AVID)** — <https://avidml.org/> <!-- needs-research: confirm active-maintenance status; if the project has paused or moved, update. -->

## Tools and harnesses

- **NVIDIA garak — Generative AI Red-teaming and Assessment Kit** — <https://github.com/NVIDIA/garak> — documentation at <https://reference.garak.ai/>
- **Microsoft PyRIT — Python Risk Identification Tool for generative AI** — <https://github.com/Azure/PyRIT> — documentation at <https://azure.github.io/PyRIT/>
- **Promptfoo red-team** — <https://www.promptfoo.dev/docs/red-team/> <!-- needs-research: confirm the current landing URL for the Promptfoo red-team feature set. -->
- **Inspect — UK AI Safety Institute evaluation framework** — <https://inspect.aisi.org.uk/> — repository at <https://github.com/UKGovernmentBEIS/inspect_ai>
- **AgentDojo** — repository and paper — <https://github.com/ethz-spylab/agentdojo> <!-- needs-research: pin the paper — Debenedetti et al., "AgentDojo: A Dynamic Environment to Evaluate Prompt Injection Attacks and Defenses for LLM Agents" — with arXiv ID. -->
- **InjecAgent** — repository and paper — <https://github.com/uiuc-kang-lab/InjecAgent> <!-- needs-research: pin the paper — Zhan et al., "InjecAgent: Benchmarking Indirect Prompt Injections in Tool-Integrated Large Language Model Agents" — with arXiv ID. -->

## Attack literature (primary papers)

- Greshake, K., Abdelnabi, S., Mishra, S., Endres, C., Holz, T., Fritz, M. — "Not what you've signed up for: Compromising Real-World LLM-Integrated Applications with Indirect Prompt Injection" — arXiv:2302.12173 — <https://arxiv.org/abs/2302.12173>
- Perez, F., Ribeiro, I. — "Ignore Previous Prompt: Attack Techniques For Language Models" — arXiv:2211.09527 — <https://arxiv.org/abs/2211.09527>
- Zou, A., Wang, Z., Kolter, J. Z., Fredrikson, M. — "Universal and Transferable Adversarial Attacks on Aligned Language Models" (GCG) — arXiv:2307.15043 — <https://arxiv.org/abs/2307.15043> — reference implementation at <https://github.com/llm-attacks/llm-attacks>
- Shen, X., Chen, Z., Backes, M., Shen, Y., Zhang, Y. — "'Do Anything Now': Characterizing and Evaluating In-The-Wild Jailbreak Prompts on Large Language Models" — arXiv:2308.03825 — <https://arxiv.org/abs/2308.03825>
- Anil, C., Durmus, E., et al. — "Many-shot jailbreaking" — Anthropic — <https://www.anthropic.com/research/many-shot-jailbreaking> <!-- needs-research: confirm the paper's citation form and whether an arXiv identifier has been assigned; the reference at time of authoring is the Anthropic research post. -->
- Chao, P., Robey, A., Dobriban, E., Hassani, H., Pappas, G. J., Wong, E. — "Jailbreaking Black Box Large Language Models in Twenty Queries" (PAIR) — arXiv:2310.08419 — <https://arxiv.org/abs/2310.08419> <!-- needs-research: confirm arXiv ID and current version. -->
- Mehrotra, A., Zampetakis, M., Kassianik, P., Nelson, B., Anderson, H., Singer, Y., Karbasi, A. — "Tree of Attacks: Jailbreaking Black-Box LLMs Automatically" (TAP) — arXiv:2312.02119 — <https://arxiv.org/abs/2312.02119> <!-- needs-research: confirm arXiv ID and current version. -->

## Reference eval-sets

- **HarmBench** — Mazeika, M., Phan, L., et al. — "HarmBench: A Standardized Evaluation Framework for Automated Red Teaming and Robust Refusal" — <https://www.harmbench.org/> — repository at <https://github.com/centerforaisafety/HarmBench> <!-- needs-research: pin arXiv ID and current release version of the behaviour set and classifier. -->
- **AIR-Bench 2024** — Zeng, Y., et al. — "AIR-Bench 2024: A Safety Benchmark Based on Regulation and Policies Specified Risk Categories" — Stanford CRFM — <https://crfm.stanford.edu/> <!-- needs-research: pin arXiv ID, current version, and canonical dataset URL. -->

## Enterprise red-team writing

- **Microsoft AI Red Team — Lessons from red-teaming 100 generative AI products** — <https://www.microsoft.com/en-us/security/blog/> <!-- needs-research: pin the exact URL and publication date for the "Lessons from red-teaming 100 generative AI products" report / whitepaper. -->
- **Microsoft Security Development Lifecycle (SDL) — AI-specific threat-modelling guidance** — <https://www.microsoft.com/en-us/securityengineering/sdl/> <!-- needs-research: pin the exact URL for the AI-specific SDL threat-modelling extension. -->

## Government AI safety institutes / red-team guidance

- **UK AI Safety Institute (UK AISI)** — <https://www.aisi.gov.uk/> — publications and Inspect harness.
- **US AI Safety Institute (US AISI)** — <https://www.nist.gov/aisi> — including US AISI evaluation reports.
- **NIST AI Resource Center (AIRC)** — <https://airc.nist.gov/> — AI RMF Playbook and GenAI Profile references (revisit from mod-101).

## Foundation-lab policies (revisit from mod-102, cited from red-team context)

- **Anthropic Responsible Scaling Policy** — <https://www.anthropic.com/rsp>
- **OpenAI Preparedness Framework** — <https://openai.com/safety/preparedness/>
- **Google DeepMind Frontier Safety Framework** — <https://deepmind.google/discover/blog/introducing-the-frontier-safety-framework/>
- **Frontier Model Forum publications** — <https://www.frontiermodelforum.org/>

<!-- needs-research: pin the exact document versions of RSP, Preparedness, and FSF that are current at authoring time; these evolve. -->

## Incident and harm-model corpora (revisit from mod-102, used for triage anchoring)

- **AI Incident Database (AIID)** — <https://incidentdatabase.ai/>
- **OECD.AI AI Incidents Monitor (AIM)** — <https://oecd.ai/en/incidents>
- **MIT AI Risk Repository** — <https://airisk.mit.edu/>

## Reading order for this module

If you are working through mod-104 for the first time, read in this order:

1. OWASP LLM Top 10 landing page — the vocabulary the module uses (chapters 01, 04, 05).
2. OWASP GenAI Red-Team Guide — the shape of a red-team programme (chapter 01).
3. NVIDIA garak documentation — the scanner model (chapter 02).
4. Microsoft PyRIT documentation — the orchestrator model (chapter 03).
5. Greshake et al. 2302.12173 — the indirect-prompt-injection paper (chapter 04).
6. AgentDojo paper and repository — the tool-abuse environment (chapter 04).
7. InjecAgent paper and repository — the injection-payload corpus (chapter 04).
8. Shen et al. 2308.03825 — the DAN empirical characterisation (chapter 05).
9. Zou et al. 2307.15043 — the GCG paper (chapter 05).
10. Anthropic *Many-shot jailbreaking* — the many-shot class (chapter 05).
11. HarmBench paper and leaderboard — reference behaviour benchmark (chapter 06).
12. AIR-Bench 2024 paper and taxonomy — reference policy-category benchmark (chapter 06).
13. Inspect documentation — the internal-suite authoring harness (chapter 06).
14. Microsoft AI Red Team "Lessons from red-teaming 100 generative AI products" — the enterprise-side threat-model shape (chapter 04).

## Reading order — the versioned and moving sources

- garak, PyRIT, Inspect, AgentDojo, InjecAgent, HarmBench are all versioned software projects; pin the release you cite in your suite configuration and note the release version alongside every measurement.
- OWASP LLM Top 10 and OWASP GenAI Red-Team Guide are versioned documents; cite the version number in the coverage matrix.
- Frontier-lab policies (RSP, Preparedness, FSF) are versioned; note the version and date on any citation.
- Attack papers on arXiv often have v1 / v2 / v3 — cite the version you read.
- HarmBench and AIR-Bench are versioned benchmarks; the classifier / judge versions matter for comparability across runs.

Do not rely on unofficial re-postings of any paper, tool, or policy — the fidelity to the current release is uncertain, and (for foundation-lab policies especially) the version drift is meaningful.
