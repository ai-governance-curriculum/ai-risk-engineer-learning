# SR 11-7 as the Parent MRM Shape for ML/AI at a Regulated FI

If you work at a US financial institution — a bank, a broker-dealer, an insurer with a US banking parent, a savings-and-loan, a credit union of a certain size — every quantitative model you deploy already lives inside a Model Risk Management (MRM) programme. That programme was built for logistic-regression credit scorecards and Basel capital models. When a machine-learning or LLM system enters production, the MRM programme reaches for it too, and it uses the same shape it has always used. This chapter teaches that shape and shows where your mod-104 red-team suite, mod-105 adversarial evaluation, mod-106 fairness audit, mod-107 privacy evaluation, mod-108 guardrail catalogue, and mod-109 monitoring wiring *plug in* — because they do plug in, and if you do not know where, MRM will tell you the model is "undocumented" while your evidence sits in a repo two clicks away.

## Motivation

Two failure modes are common when an ML/AI system meets an MRM shop for the first time:

- **The programme rejects the model as "un-validatable."** The reviewer expects an assumptions document, a conceptual-soundness narrative, an independent validator's report, and a back-test on out-of-time data. The team ships a notebook, a fine-tune script, and a Hugging Face model card. The reviewer cannot tell which fields match which, and the model is blocked at the validation gate.
- **The team ships a giant "AI evaluation report" that MRM cannot file.** The report contains every eval the team ran — red-team, adversarial, fairness, privacy — but organises them by *tool* (garak §, PyRIT §, Fairlearn §, Presidio §). MRM's document taxonomy has slots for *conceptual soundness*, *outcome analysis*, *ongoing monitoring*. Nothing in the report obviously belongs in any of them.

Both failures are packaging failures. The engineering is fine; the shape is wrong. This chapter fixes the shape.

## SR 11-7 in ten sentences

The US Federal Reserve issued **Supervision and Regulation Letter 11-7** ("Guidance on Model Risk Management") in April 2011. The OCC issued the companion **Bulletin 2011-12** the same day; the two documents are treated as functionally equivalent by supervisors and jointly cited as "SR 11-7 / OCC 2011-12". The guidance defines **model risk** as the potential for adverse consequences arising from decisions based on incorrect or misused model output. It prescribes a *lifecycle* — development, implementation, and use; validation; governance, policies, and controls — and it names four core pillars for the validation function to exercise across that lifecycle: **independence** of validation from development, **conceptual soundness** review, **ongoing monitoring** including outcome analysis and back-testing, and **effective challenge** as a cultural expectation on every model owner and reviewer. Everything else in the guidance — model inventory, model documentation, tiering, risk rating, sign-off routing — hangs off those pillars. The guidance is prescriptive on *shape* and *governance* but silent on *technique*; it does not tell you which metrics to compute. That silence is a feature: it means the same shape survives from linear regression to gradient boosting to a fine-tuned LLM behind a retrieval-augmented generation stack.

## The four pillars, translated to AI

### Independence of validation from development

The person who validates the model does not report — directly or indirectly — to the person who built it. At a large FI this is enforced by placing validation inside a separate Model Risk Management function that reports to Risk, not to the business line that owns the model. At smaller institutions the independence may be achieved through committee structure rather than org chart.

For AI, independence has two engineering consequences you have to get right:

- **Evaluation code is owned by validation, not development.** The mod-104 red-team suite, the mod-105 adversarial suite, the mod-106 fairness audit, the mod-107 privacy evaluation — the *suite definitions* (which tests run, which thresholds pass, which datasets are used) belong to validation. Development can *contribute* tests and *reproduce* runs, but it does not change the pass rule.
- **Evidence artefacts are signed by validation.** A red-team report that shows up in the release evidence packet must carry the validator's signature, not the developer's. In practice this means the CI system that runs the suite runs under a validation-owned service account and writes to a validation-controlled artefact store.

### Conceptual soundness

The validator asks *is the model design fit for purpose*, independent of whether the code runs. For a credit-scorecard this is a discussion of variable selection, monotonicity, and economic intuition. For an ML/AI system it is a broader question: is the *task specification* right, is the *training data* representative of production traffic, are the *evaluation metrics* the ones that would catch the harms the harm model names, are the *guardrails* aimed at the failure modes the adversary would actually attempt.

Your engineering inputs to this pillar:

| Conceptual-soundness question | Engineering artefact that answers it |
|---|---|
| Is the task specification right? | Task specification document + link to the mod-102 harm model rows this system's failure modes correspond to |
| Is the training / fine-tuning data representative of production? | Datasheet for Datasets (chapter 04) + drift report between training corpus and production sample |
| Are the evaluation metrics the ones the harm model needs? | Metric-to-harm mapping (from the mod-101 five-sentence translation) referenced by every eval report |
| Are the guardrails aimed at the right failure modes? | Guardrail catalogue (mod-108) row-by-row, each pointing at the harm-model scenario it mitigates |
| Are the human-oversight points designed for the human's actual workflow? | Human-oversight design document (mod-108) + human-factors evaluation notes |

If any of the right-hand cells are missing, the validator has no evidence for conceptual soundness and the model does not clear the pillar — regardless of how the metrics came out.

### Ongoing monitoring, outcome analysis, back-testing

Once the model is in production, MRM wants continuous evidence that it is still fit for purpose. For classical models this is back-testing predicted vs realised outcomes. For ML/AI it is broader: drift, calibration, safety-signal rate, guardrail firing rate, escalation rate, and — for LLMs — the rate of adversarial or unsafe patterns caught in real traffic.

Your engineering inputs come almost entirely from mod-109:

| MRM monitoring expectation | Mod-109 signal |
|---|---|
| Predictive performance stability | Rolling accuracy / calibration / drift metric per model |
| Population-stability across cohorts | Fairness monitor (mod-106) fed into mod-109 |
| Adverse-outcome surveillance | Guardrail-fire rate + safety-classifier-fire rate (mod-108) |
| Novel-attack surveillance | Injection-signature match rate on inputs (from mod-104 fixtures) |
| Vendor-model behaviour change | Vendor-side signal (see chapter 02 for SR 23-4 shape) |
| Post-market outcome sample | Complaint / dispute / near-miss stream (mod-111) |

Monitoring evidence is what turns a *point-in-time* validation into an *ongoing* validation, which is what SR 11-7 requires. A model without live monitoring cannot be credited as "in scope of ongoing validation" no matter how thorough the initial validation was.

### Effective challenge

Effective challenge is the cultural pillar. It says every model must be subject to organised, active questioning by informed parties who have the standing and the incentive to press hard. For AI this is *your* role at level 25 plus the AI risk council (mod-112) plus the L35 evaluation engineer who signs off on release.

Engineering-visible signs of effective challenge:

- The mod-104 red-team suite includes tests the *developer did not write*. If every test was written by the model team, the challenge is not effective.
- The mod-103 register includes rows the developer would not have volunteered. If the register was built by the developer alone, the challenge is not effective.
- The mod-111 post-mortems include a "what did we miss" section owned by someone other than the developer. If not, the challenge is not effective.

## MRM-shaped validation documentation for an ML/AI model

MRM documentation is a *packet*, not a document. Its top-level table of contents is roughly stable across FIs; below is a canonical shape and the engineering artefact that fills each slot for a modern ML/AI system.

| MRM document slot | AI engineering artefact that fills it |
|---|---|
| Model description and purpose | Model card (chapter 04) + task specification + intended-use statement |
| Model tiering / risk rating | Risk register row(s) (mod-103) with inherent / residual / control-defeated scores + appetite comparison |
| Data | Datasheets for Datasets / Data Cards (chapter 04) for training, fine-tuning, evaluation, and retrieval corpora |
| Methodology / conceptual soundness | Method write-up + citation of the mod-102 harm-model rows the design targets |
| Assumptions and limitations | Assumptions document, cross-referenced with the harm-model uncertainty column |
| Development testing | Held-out evaluation report(s); fairness slice (mod-106); privacy evaluation (mod-107); adversarial evaluation (mod-105); red-team suite output (mod-104) |
| Independent validation | Validation memo authored by the independent validator that consumes the above and states pass / fail per pillar |
| Implementation testing | Deployment-shape tests: shadow-eval, canary results, guardrail integration tests (mod-108) |
| Ongoing monitoring plan | Monitor spec (mod-109) — metric, threshold, alert cadence, action per row |
| Change management | Change protocol (for PCCP-eligible systems, this is the FDA PCCP; for others, the FI's own model-change policy) |
| Third-party components | ML-BOM (chapter 04) + SR 23-4 due-diligence file (chapter 02) for every third-party model / API / dataset |
| Governance sign-off | Signed approval memo from the AI risk council (mod-112) and the L35 release-authoriser |

An MRM reviewer will not accept a folder that ships "the evidence" without this packet. Conversely, an MRM reviewer *will* accept a folder that ships this packet even if the underlying evidence is a bit thin, because the shape gives them somewhere to write their findings. Packaging matters.

## Where AI-specific work files against MRM headings

The rest of this track produces the raw evidence. This module (and this chapter especially) is where you *name the heading* that each piece of evidence files under. The map:

| Track module | Evidence produced | MRM heading it files under |
|---|---|---|
| mod-101 | Framework-to-artefact translation, deferral contract | Governance policies + tiering criteria |
| mod-102 | Harm model | Conceptual soundness (which failure modes the design must resist) |
| mod-103 | Risk register (inherent / residual / control-defeated) | Model tiering / risk rating |
| mod-104 | LLM red-team suite output | Development testing + effective challenge |
| mod-105 | Adversarial-ML robustness evaluation | Development testing + conceptual soundness (robustness assumptions) |
| mod-106 | Fairness / bias / explainability evaluation | Development testing + assumptions-and-limitations (fairness caveats) |
| mod-107 | Privacy risk evaluation (membership inference, extraction, PII leakage) | Development testing + assumptions-and-limitations |
| mod-108 | Guardrail catalogue + effectiveness measurement | Implementation testing + controls-and-mitigation |
| mod-109 | Continuous monitoring wiring + register updates | Ongoing monitoring plan |
| mod-110 (this module) | Provenance package (ML-BOM, SPDX, SLSA, signatures, scan gates) | Data + third-party components + implementation testing |
| mod-111 | Incident-response playbook + post-mortem shape | Ongoing monitoring (incident channel) + change management (learnings) |
| mod-112 | Cross-functional programme record + risk-council minutes | Governance sign-off + change management |

If you can point at a filled cell in the right-hand column for every row, your MRM packet is complete. If a cell is empty, that is a gap the validator will surface.

## Cross-references

- **mod-101 chapter 05** placed SR 11-7 / OCC 2011-12 / SR 23-4 on the same map as NIST AI RMF, ISO/IEC 42001, and the EU AI Act. This chapter makes the SR 11-7 shape concrete.
- **mod-103** on inherent / residual / control-defeated risk populates the *model tiering / risk rating* section of the MRM packet.
- **mod-108** on guardrail engineering produces the *implementation testing* and *controls-and-mitigation* rows credited by MRM.
- **mod-109** on continuous monitoring is the *ongoing monitoring plan*.
- **mod-111** on incident response feeds *ongoing monitoring* (via the incident channel) and *change management* (via post-mortem-driven change requests).
- **mod-112** on the cross-functional programme is the *governance sign-off* forum where the packet is approved and where effective challenge is exercised.
- Chapter 02 of this module handles SR 23-4 for the third-party components row of the MRM packet.
- Chapter 03 of this module handles FDA GMLP + PCCP for the SaMD variant.

## Summary

- SR 11-7 (US Federal Reserve, 2011) and OCC Bulletin 2011-12 are the parent MRM shape at US regulated FIs; they are functionally equivalent and jointly cited.
- The four pillars are independence of validation from development, conceptual soundness, ongoing monitoring (including outcome analysis and back-testing), and effective challenge.
- MRM documentation is a packet with a stable table of contents; every slot is filled by an engineering artefact you already produce elsewhere in this track.
- The AI-specific packaging failure is not evidence quality — it is *heading assignment*. Name the MRM heading each evidence artefact files under, and the packet writes itself.
- Exercise 01 walks you through producing a shippable MRM packet for one of your models by consuming the artefacts from mods 102-109.
