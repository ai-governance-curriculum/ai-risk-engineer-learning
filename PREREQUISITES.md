# Prerequisites — AI Risk Engineer track

**Role level:** 25 (hands-on engineering specialist, AI Governance family)
**Track:** `ai-risk-engineer-learning`

This track is a hands-on engineering specialty. It assumes the learner arrives with the operational analyst legwork already fluent, ML fundamentals in place, and engineering-craft basics muscle-memoried. Fundamentals are **not** re-taught here — the modules step directly onto harm modelling, adversarial-ML, guardrail engineering, and risk quantification.

## Assumed lower-level curriculum

Complete or be practically fluent in the equivalent of these lower-level tracks before starting:

- **[`ai-infra-junior-engineer-learning`](https://github.com/ai-infra-curriculum/ai-infra-junior-engineer-learning) — level 10** — engineering-craft prerequisites (Linux, Git, Python packaging, HTTP APIs, unit testing, logging, docker basics). If you cannot ship a small Python service and instrument it, start here.
- **[`ml-engineer-learning`](https://github.com/ml-engineering-curriculum/ml-engineer-learning) — level 20** — classical ML with scikit-learn, deep learning with PyTorch and Hugging Face Transformers, evaluation with standard sklearn / HF metrics, packaging with FastAPI + Docker, experiment tracking with MLflow / Weights & Biases. If you cannot ship a fine-tuned classifier and defend the eval, start here.
- **[`ai-governance-analyst-learning`](https://github.com/ai-governance-curriculum/ai-governance-analyst-learning) — level 15** — operational analyst work in the AI Governance family: AI use-case intake and triage; model / AI-system inventory maintenance; NIST AI RMF, ISO/IEC 42001, and EU AI Act framework crosswalks; first-draft AI risk assessments; algorithmic / fundamental-rights / data-protection impact assessments; model / system / dataset card authoring at the analyst tier; jurisdictional regulatory tracking; reading eval / red-team / audit evidence into governance records; control-tracking and evidence-collection cycles; participation in AI incidents at the analyst tier; council-support communication and policy drafting. This role *consumes* those artifacts and adds engineering depth on top — the track does not re-teach the intake questionnaire or the analyst-tier model-card walkthrough.

## Assumed working knowledge

Even without holding the equivalent role formally, you should be practically fluent in:

- **Python** — comfortable with packaging (pyproject.toml, uv / poetry, virtualenvs), unit testing (pytest, hypothesis), typed code (mypy / pyright), and CI (GitHub Actions).
- **Statistics & probability** — reading a confusion matrix, calibration curves, ROC/PR curves, bootstrap confidence intervals; comfortable enough with hypothesis-testing that a p-value / effect size discussion is not new. Deep statistical methodology depth (Bayesian priors, sequential testing, judge-vs-human calibration) is linked out to `model-evaluation-engineer` (peer, level 30).
- **PyTorch + Hugging Face Transformers** — enough to fine-tune a classifier, load a Hugging Face model, and run inference; enough to instrument a model with adversarial attack code from ART / TextAttack.
- **Data engineering basics** — pandas, SQL, columnar formats, dataset lineage. You should be able to author a labelled evaluation dataset and defend the sampling.
- **HTTP APIs + LLM providers** — comfortable calling OpenAI / Anthropic / Google / open-source (vLLM / TGI) inference APIs from Python; comfortable reading trace output from Arize Phoenix, Langfuse, or LangSmith at the analyst level.
- **Basic security literacy** — HTTPS / auth basics, secrets handling, OWASP Top 10 for web at the general awareness level. Deep application security is not required — this track adds AI-specific security on top of that baseline.
- **Reading the primary literature** — you should be able to open a paper on arXiv (e.g., Shokri MIA, Carlini LiRA, GCG jailbreak) and translate the method into runnable code within a working day.

## Not assumed, taught here

You do **not** need prior experience with:

- Any specific AI-risk framework — NIST AI RMF, ISO/IEC 23894, ISO/IEC 42001, ISO/IEC 42005, EU AI Act Articles 9-15, SR 11-7, FDA GMLP + PCCP, Colorado AI Act. Mod-101 covers the mapping to engineering artifacts.
- Any specific harm-model reference corpus — AI Incident Database, OECD.AI incidents, MIT AI Risk Repository, AIRO. Mod-102 covers harm-model authoring end-to-end.
- Any specific red-team tool — garak, PyRIT, Promptfoo red-team, Inspect. Mod-104 covers the working suite.
- Any specific adversarial-ML library — ART, TextAttack, CleverHans, Counterfit. Mod-105 covers the working suite.
- Any specific fairness / explainability library — Fairlearn, AIF360, Holistic AI, SHAP, Alibi, LIME. Mod-106 covers the working suite.
- Any specific privacy library — Opacus, TensorFlow Privacy, ML Privacy Meter, Presidio. Mod-107 covers the working suite.
- Any specific guardrail framework — NeMo Guardrails, Guardrails AI, Llama Guard, ShieldGemma, OpenAI Moderation, Constitutional Classifiers. Mod-108 covers the working suite.
- Any specific AI observability / GRC-for-AI platform — Credo AI, Holistic AI, Fiddler AI, Arthur AI, ModelOp, Robust Intelligence, WhyLabs, Evidently AI, Monitaur. Mod-103, mod-108, mod-109, and mod-112 cover the platforms at the risk-engineer level.
- SR 11-7 / FDA GMLP / EU AI Act Article 9 detail — mod-110 covers each shape.

## Recommended reading before starting

- [NIST AI Risk Management Framework 1.0 (AI 100-1)](https://nvlpubs.nist.gov/nistpubs/ai/NIST.AI.100-1.pdf) — front-to-back once.
- [NIST AI RMF Generative AI Profile (AI 600-1)](https://airc.nist.gov/AI_RMF_Knowledge_Base/AI_RMF/Uses/GenAI-Profile) — front-to-back once.
- [OWASP Top 10 for LLM Applications (2025)](https://owasp.org/www-project-top-10-for-large-language-model-applications/) — the ten entries and the "who is responsible" column.
- [Anthropic Responsible Scaling Policy](https://www.anthropic.com/rsp) and the [OpenAI Preparedness Framework](https://openai.com/safety/preparedness/) — one worked example of enterprise risk tiering language.
- A recent public system card (e.g., a Claude or GPT-family disclosure) — read it as an engineer would, tracing each claim to an evidence artifact.

## Not required, useful to have

- Prior work with an eval-tooling platform (Arize Phoenix, Langfuse, W&B Weave, Braintrust, Promptfoo, DeepEval, RAGAS, LangSmith) — the peer `ai-eval-engineer-learning` (level 30) owns this stack; a working feel is useful before mod-108 / mod-109.
- Prior work with an experiment-tracking platform (MLflow, W&B) — assumed at the `ml-engineer` level.
- Familiarity with a GRC platform (ServiceNow AI Control Tower, Credo AI, Holistic AI, ModelOp) — mod-103 and mod-112 walk the platforms.
- Certifications: [IAPP AIGP](https://iapp.org/certify/aigp/) is the most-cited certification in senior AI risk / governance postings; earning it is not required but is treated as a strong signal.

## Not in scope for this track (linked out)

- **Application-layer LLM/agent evaluation-engineering depth** — traces, judges, RAG eval, eval-gated CI/CD, online eval, cost / latency / quality — owned by [`ai-eval-engineer-learning`](https://github.com/ai-engineering-curriculum/ai-eval-engineer-learning) (peer, level 30, AI Engineering family).
- **Statistical / benchmark / judge-vs-human calibration methodology depth** — owned by [`model-evaluation-engineer-learning`](https://github.com/ml-engineering-curriculum/model-evaluation-engineer-learning) (peer, level 30, ML Engineering family).
- **Deep ML/AI security at platform scale** — inference-edge adversarial defence, model-extraction defence, judge supply-chain, eval-set exfiltration prevention, MLSecOps — owned by [`security-learning`](https://github.com/ai-infra-curriculum/security-learning) and [`ai-infra-security-learning`](https://github.com/ai-infra-curriculum/ai-infra-security-learning) (level 35).
- **Release-assurance / audit-trail / regulator-facing methodology depth** — owned by [`ai-evaluation-engineer-learning`](https://github.com/ai-governance-curriculum/ai-evaluation-engineer-learning) (peer / next-up, level 35, Governance family).
- **Frontier-agent red-team methodology and dangerous-capability evaluation depth** — owned by [`agentic-safety-engineer-learning`](https://github.com/ai-governance-curriculum/agentic-safety-engineer-learning) (level 40).
- **Control-library architecture, policy taxonomy, cross-jurisdiction reconciliation** — owned by [`senior-ai-governance-architect-learning`](https://github.com/ai-governance-curriculum/senior-ai-governance-architect-learning) (level 50).
- **Program leadership, board-level reporting, regulator engagement** — owned by [`head-of-ai-governance-learning`](https://github.com/ai-governance-curriculum/head-of-ai-governance-learning) (level 60) and [`chief-ai-officer-learning`](https://github.com/ai-governance-curriculum/chief-ai-officer-learning) (level 70).
- **Legal opinion** — out of scope; routed to counsel.
- **SOC-side incident handling** — out of scope; this role feeds AI-specific signal to SecOps.
