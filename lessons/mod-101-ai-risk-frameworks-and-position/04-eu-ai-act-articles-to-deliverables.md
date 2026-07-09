# EU AI Act Articles → Engineering Deliverables

Regulation (EU) 2024/1689 — the EU AI Act — is the first horizontal AI regulation with direct engineering obligations. This chapter reads seven of its articles as *engineering deliverables*: Articles 9, 10, 14, 15, 55, 61 (registration), and 72. Legal opinion on the scope of the Regulation (is your system "high-risk"? is your model a "GPAI with systemic risk"?) is out of scope — that goes to counsel and to `senior-ai-governance-architect` at level 50. Your job is: once the classification is set, what do you *build*?

## Motivation

Article numbers get quoted as if they were verdicts ("this violates Article 15"). The actual text of the article is technical and, once decoded, is easier to comply with than to argue about. Every one of the seven articles below decomposes into a small number of engineering artifacts that you already produce for NIST AI RMF (chapter 01) and ISO/IEC 42001 (chapter 03). Your work is to make the *same* engineering evidence answer the EU-Act questions without reinventing it.

## Article 9 — Risk-management system for high-risk AI

Article 9 requires providers of high-risk AI systems to establish, implement, document, and maintain a *risk-management system* over the AI system's whole lifecycle. It requires:

- identification and analysis of known and reasonably foreseeable risks,
- estimation and evaluation of risks that may emerge under intended use and reasonably foreseeable misuse,
- adoption of appropriate and targeted risk-management measures,
- testing to verify that risk-management measures are effective,
- ongoing iterative process, updated as needed.

### What you build

- The **risk register** from mod-103, populated per the harm model from mod-102, is the artifact.
- The register's *risk sources* column is populated from ISO/IEC 23894 Annex A (chapter 03) plus the NIST GenAI Profile 12 risks (chapter 02).
- The register's *residual risk* column is closed by evidence from mods 104-108 (red-team, adversarial-ML, fairness, privacy, guardrails).
- Article 9 asks explicitly for *testing* — that testing is your evaluation suites (mods 104-107) plus the release-gate that reports them. Testing evidence is signed and version-pinned.

If your NIST AI RMF translation from chapter 01 is done, Article 9 is roughly a re-labeling exercise, not a re-implementation.

## Article 10 — Data and data governance

Article 10 requires that training, validation, and testing datasets meet quality criteria: they are relevant, sufficiently representative, and, to the best extent possible, free of errors and complete. It also requires:

- examination in view of possible biases likely to affect health, safety, or fundamental rights (paragraph 2(f)),
- appropriate measures to detect, prevent, and mitigate such biases (paragraph 2(g)),
- special-category data may be processed subject to appropriate safeguards where strictly necessary for bias detection and correction (paragraph 5).

### What you build

- **Dataset cards** (Datasheets-for-Datasets shape) documenting source, provenance, licence, collection methodology, cleaning, deduplication.
- **Bias-audit artifacts** (mod-106) for representativeness and disparate-impact across protected groups defined in the harm model.
- A **data-lineage manifest** for training / validation / test splits that a mod-110 supply-chain check can validate against ML-BOM.
- A **special-category-data safeguard** memo for any use of protected attributes in bias evaluation (mod-107 will teach the DP-SGD / access-control shape).

## Article 14 — Human oversight

Article 14 requires that high-risk AI systems be designed such that they can be effectively overseen by natural persons during the period they are in use. Human overseers must be able to:

- understand relevant capacities and limitations of the system,
- monitor its operation and be aware of automation bias,
- correctly interpret its output,
- decide not to use it or otherwise override, disregard, reverse, or intervene in it,
- interrupt the system through a stop button or similar procedure.

### What you build

- A **human-oversight design document** that shows where the human sits in the loop, what signals they see, and what they can override.
- **Intervention paths in code**: a documented API by which an operator can halt, roll back, or hand off to a human. This is what mod-108's guardrails and mod-109's monitors *escalate to*.
- An **automation-bias mitigation note** — often as simple as UI wording — vetted against user research or human-factors input where the system is safety-critical.
- Evidence that overseers are trained (out-of-scope for you as an engineer, but you are the source-of-truth for the *training material's technical content*).

## Article 15 — Accuracy, robustness, and cybersecurity

Article 15 requires high-risk AI systems to be designed and developed such that they achieve appropriate levels of accuracy, robustness, and cybersecurity throughout their lifecycle. It also requires:

- accuracy metrics to be declared in the instructions for use,
- robustness against errors, faults, or inconsistencies (including feedback loops and drift),
- resilience against attempts to alter use, outputs, or performance by exploiting vulnerabilities (data poisoning, model poisoning, evasion, prompt injection, model extraction, confidentiality attacks).

### What you build

Article 15 is where *all* your evaluation-engineering work lands:

- **Accuracy declaration** — the metrics in the instructions-for-use (or model card) matched to the eval-set they were measured on, with confidence intervals and slice reports (`ai-eval-engineer` peer at level 30 owns the methodology; you consume it).
- **Robustness report** — adversarial-ML suite output (mod-105), noise / distribution-shift results, drift-monitor spec (mod-109).
- **Cybersecurity report** — LLM red-team suite output (mod-104), NIST AI 100-2 mapping (evasion / poisoning / extraction / inversion / MIA / availability), MITRE ATLAS mapping, prompt-injection detection (mod-108/109).
- **Traceability table** from every Article-15 obligation to the eval file / suite version / evidence file that discharges it.

## Article 55 — Systemic-risk GPAI obligations

Article 55 imposes additional obligations on providers of *general-purpose AI models with systemic risk*: model evaluation with adversarial testing, tracking and reporting of serious incidents, and cybersecurity of the model and its physical infrastructure.

Whether a given model triggers Article 55 is a legal / regulatory determination — not yours. But if you are inside a provider organisation whose model *has* been classified GPAI-with-systemic-risk:

### What you build

- **Model evaluation for systemic risks** — the evaluation suite lives in mod-104 (LLM red-team) and mod-105 (adversarial-ML), often complemented by external red-teaming (Anthropic, OpenAI, Microsoft, Google DeepMind, and UK / US AISI publish evaluation methodology you cite).
- **Adversarial testing** — again mod-104 / mod-105; the difference at Article 55 scale is *external* red-team engagement and *pre-deployment* evaluation gates. This aligns with the practices in the EU AI Office **General-Purpose AI Code of Practice**.
- **Serious-incident tracking and reporting** — mod-111 owns this. Article 55(1)(c) uses the same *serious incident* concept as Article 73; you package the same incident data both ways.
- **Cybersecurity of the model** — model weights integrity, supply chain (mod-110: SLSA / Sigstore / ModelScan / picklescan).

At level 40, `agentic-safety-engineer` inherits the frontier-agent / dangerous-capability evaluation depth this article implies. You supply the *enterprise-shape* variant.

## Article 61 — Registration (in the EU database)

Article 61 requires providers of certain high-risk AI systems to register the system in the EU AI database before placing it on the market or putting it into service.

### What you build

- The **registration record** itself is a governance artifact (drafted by level 15, signed at level 35+). You supply the technical fields: intended purpose, algorithms and models used, computing / hardware resources needed, data used for training, foreseeable output range, cybersecurity controls, human-oversight measures.
- A **release-manifest field** in your evidence pipeline that ties each release to its registration entry, so a change requiring re-registration is caught at release-gate rather than post-launch.

*(Article 49 in the final published text uses adjacent registration-related language; check the consolidated OJ text and the latest EU AI Office implementation guidance if numbering ambiguity arises.)*

## Article 72 — Post-market monitoring

Article 72 requires providers to establish and document a *post-market monitoring system* that actively and systematically collects, documents, and analyses data on the system's performance throughout its lifecycle.

### What you build

- The **monitoring wiring** from mod-109 — drift, quality regression, safety-monitor signal, near-miss data, jailbreak / prompt-injection detection — feeding the risk register.
- A **post-market surveillance report** — the same report file you use for internal executive review is what serves Article 72 evidence; you tune the summary section for the audience.
- **Incident escalation hooks** to the Article 73 serious-incident reporting path (mod-111).

Article 72 and NIST AI RMF MEASURE 3.x + MANAGE 4.x are functionally the same requirement viewed from two governance systems. If mod-109 is done right, Article 72 is a formatting exercise.

## The article-to-artifact map you actually maintain

The artifact for exercise 04 has this shape:

| Article | Obligation (short) | Artifact you own | Where it lives in the repo | Signed by |
|---|---|---|---|---|
| 9 | Risk-management system | Risk register (mod-103) | `risk-register/` | AI risk council (mod-112) |
| 10 | Data governance | Dataset card + bias audit + data-lineage manifest | `data-cards/`, `evidence/mod-106/`, `bom/` | Data owner + AI risk council |
| 14 | Human oversight | Oversight design doc + intervention API + automation-bias note | `docs/oversight/` + `src/oversight/` | Product + AI risk council |
| 15 | Accuracy / robustness / cybersecurity | Accuracy declaration + robustness report + cybersecurity report + traceability table | `model-cards/`, `evidence/mod-104/`, `evidence/mod-105/` | AI risk council + `ai-evaluation-engineer` (level 35) |
| 55 | Systemic-risk GPAI | External red-team engagement + evaluation report + serious-incident tracker | `evidence/systemic/`, `incidents/` | Model-provider governance leadership (level 60) |
| 61 | Registration record | Technical fields of registration entry + release-manifest field | `docs/registration/` | Governance leadership |
| 72 | Post-market monitoring | Post-market surveillance report + monitor wiring | `reports/pms/`, `monitors/` | AI risk council |

## Interaction with the harmonised standards process

The Regulation delegates significant detail to *harmonised standards* that CEN-CENELEC JTC 21 is authoring. These will formalise how ISO/IEC 42001, 23894, 42005, 25059, and other AI standards discharge the Regulation's essential requirements. When those harmonised standards land, the concrete metrics and thresholds Article 15 asks for will be codified — and your engineering evidence will need to file against them by name. Track the CEN-CENELEC JTC 21 workplan and the EU AI Office's guidance releases via the analyst layer (level 15).

<!-- needs-research: as CEN-CENELEC JTC 21 publishes the harmonised standards supporting the AI Act (particularly for Article 15), backfill their reference numbers and status here. -->

## Summary

- Articles 9, 10, 14, 15, 55, 61, 72 are the seven articles this role's engineering evidence discharges.
- Article 9 = the risk register you already build. Article 10 = dataset cards + bias audit. Article 14 = oversight design + intervention API. Article 15 = accuracy + robustness + cybersecurity reports. Article 55 = systemic-risk-only, external red-team plus incident tracking. Article 61 = the technical fields of a registration entry. Article 72 = post-market monitoring.
- Every article maps to one or more engineering artifacts you already produce. The article-to-artifact map is the deliverable; exercise 04 walks it.
- Legal classification (is this system high-risk? is the model GPAI-with-systemic-risk?) is out of scope; you *comply* to whatever classification legal + governance issue.
