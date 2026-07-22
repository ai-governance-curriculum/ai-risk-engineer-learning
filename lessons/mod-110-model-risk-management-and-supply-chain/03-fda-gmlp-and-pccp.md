# FDA GMLP and the Predetermined Change Control Plan

If the AI system you support is a *Software as a Medical Device* (SaMD) — a diagnostic classifier, a clinical decision-support model, a triage prioritiser, a radiology segmentation model — it is regulated as a medical device by the FDA in the US (and by Health Canada, MHRA, EU MDR / IVDR bodies, PMDA in Japan, and so on). The regulator's expectations shape the artefact set you deliver. This chapter teaches two of those anchors: the **Good Machine Learning Practice for Medical Device Development: Guiding Principles** jointly published in October 2021 by the FDA, Health Canada, and the UK MHRA; and the FDA's **Marketing Submission Recommendations for a Predetermined Change Control Plan for AI-Enabled Device Software Functions**, finalised in December 2024. The engineering slice — the part you own — is smaller than the overall submission, but it is highly structured.

Even if your system is not a medical device, the shapes below are instructive. Consumer-protection regulators (Colorado, EU AI Act Article 15) are converging on similar expectations around lifecycle change control, and the FDA's language of *locked* vs *adaptive* is a useful way to think about *any* ML system that continues to learn after deployment.

## Motivation

Two failure modes when a device team meets a real FDA submission cycle for the first time:

- **"We already do all of GMLP."** The team says yes to every principle in the abstract but has no cross-reference from principle to artefact. Under review, the reviewer asks *where GMLP principle 4 is evidenced* and the team scrambles across a wiki, a Confluence page, and a departed contractor's laptop.
- **The PCCP is treated as a paperwork exercise.** The team drafts a PCCP that authorises "improvements to the model" — vague enough that no reviewer will accept it. A good PCCP is *narrow, specified, and verifiable* — it authorises a *particular class of change* under a *particular verification protocol* with a *particular impact bound*. Broad wording is a rejection accelerator.

## SaMD, GMLP, and PCCP — the anchors in five paragraphs

The FDA regulates *devices* under the Federal Food, Drug, and Cosmetic Act; software that is intended to be used for one or more medical purposes and that performs those purposes without being part of a hardware medical device is *Software as a Medical Device* (SaMD), a term coined by the International Medical Device Regulators Forum (IMDRF) in 2013 and adopted by the FDA. SaMD is classified by the *significance of information provided by the SaMD to the healthcare decision* and by the *state of the healthcare situation or condition* the SaMD is intended for; the classification drives the regulatory pathway (510(k), De Novo, or PMA in the US).

The FDA / Health Canada / MHRA **Good Machine Learning Practice for Medical Device Development: Guiding Principles**, published October 2021, is a ten-principle statement that sits above any specific submission. It is not itself a regulation — it is a joint statement of expectations that the three regulators use as scaffolding when reviewing an AI/ML SaMD. The ten principles cover multi-disciplinary teams, security and software engineering practices, dataset representativeness, independence of test data, reference-standard label quality, model design fit to clinical workflow, human-AI team performance, testing across intended conditions, user-facing clarity, and post-deployment monitoring with disciplined retraining.

The **Predetermined Change Control Plan (PCCP)** is a mechanism by which an FDA-authorised device can be updated — including the underlying ML model — without a new submission for each update, provided the update fits within a change plan that was reviewed and authorised as part of the original submission. The FDA guidance *Marketing Submission Recommendations for a Predetermined Change Control Plan for AI-Enabled Device Software Functions* was issued in draft in April 2023 and finalised in December 2024. A PCCP has three components: a *Description of Modifications* (what changes are authorised), a *Modification Protocol* (how each type of modification will be verified, validated, and controlled), and an *Impact Assessment* (analysis of the risks and benefits of the authorised modifications).

The engineering community distinguishes *locked* algorithms — models whose function does not change between updates — from *adaptive* algorithms that continue to learn from deployment data. A PCCP-authorised device may be either; the PCCP is what makes controlled adaptation legally coherent under FDA review. It is not a licence to retrain silently; every change must fit inside the PCCP envelope, and changes outside the envelope require a new marketing submission.

Below, GMLP and PCCP are treated together because in practice they compose — the GMLP principles inform the *shape* of the modification protocol inside the PCCP, and the PCCP is where GMLP principle 10 ("Deployed models are monitored for performance and re-training risks are managed") is operationalised.

## The GMLP principles, translated to engineering artefacts

The ten principles are numbered and titled as published; the engineering artefacts that evidence each principle are yours to produce.

| GMLP # | Principle (short) | Engineering artefact(s) that evidence it |
|---|---|---|
| 1 | Multi-disciplinary expertise is leveraged throughout the total product lifecycle | Programme charter (mod-112) naming clinical, engineering, statistical, human-factors, and risk-engineering roles + meeting cadence + decision log |
| 2 | Good software engineering and security practices are implemented | Software development lifecycle documentation (prerequisite `ai-infra-junior-engineer`); IEC 62304 / ISO 13485 evidence; mod-108 guardrails; mod-110 supply-chain provenance (chapters 04-06) |
| 3 | Clinical study participants and datasets are representative of the intended patient population | Datasheet for Datasets / Data Card (chapter 04) with population characteristics + fairness slice per mod-106 + representativeness rationale |
| 4 | Training datasets are independent of test sets | Data-split documentation with reproducibility hash; leakage checks; a review sign-off that the split respects site, patient, and time boundaries |
| 5 | Selected reference datasets are based upon best available methods | Reference-standard specification (who labelled, on what protocol, with what adjudication) inside the Data Card |
| 6 | Model design is tailored to the available data and reflects the intended use of the device | Design rationale document citing task, data, and intended use; harm-model (mod-102) rows the design targets |
| 7 | Focus is placed on the performance of the Human-AI Team | Human-factors evaluation results; oversight design (mod-108) with role-and-workflow diagram; usability evaluation per IEC 62366 as applicable |
| 8 | Testing demonstrates device performance during clinically relevant conditions | Evaluation report stratified by clinically relevant conditions; robustness eval (mod-105) under expected distributional shifts; edge-case set |
| 9 | Users are provided clear, essential information | User-facing labelling (Indications for Use, contraindications, warnings) + the model card / user information sheet exposing performance and limitations to the clinician |
| 10 | Deployed models are monitored for performance and re-training risks are managed | Post-market surveillance plan (mod-109 monitors filed to FDA-required shape) + PCCP if authorised adaptation is in scope |

Every cell in the right-hand column must resolve to a file with an owner and a revision history. Empty cells are open work; hand-waved cells fail review.

## The PCCP: description, protocol, impact — in detail

### Description of Modifications

The *Description of Modifications* enumerates the types of change the manufacturer intends to make to the authorised device without a new submission. Each modification type is specified narrowly enough that a reviewer can bound it. Examples of modification types (illustrative, not exhaustive):

- Re-training on additional data drawn from named sites and patient populations, within a stated size envelope, with the model architecture and inference pipeline held fixed.
- Adjustment of a specified decision threshold within a stated range, driven by a specified triggering condition (e.g., calibration drift beyond a threshold).
- Update of a specified pre-processing step to accept a new imaging device model, with the model itself held fixed.
- Retraining under a stated hyperparameter search space, provided the resulting model passes the modification-protocol acceptance criteria below.

Broad wording — "improvements to the model" — is not a description of modifications. It is a request for unlimited change and it will not clear review.

### Modification Protocol

The *Modification Protocol* is the *verification, validation, and monitoring plan* that will be run *every time* a modification within the description is applied. In engineering terms, it is a *release-gate specification* keyed to the modification type. For each modification type in the description, the protocol names:

| Modification-protocol element | Engineering content |
|---|---|
| Data-management plan | Data sources allowed, inclusion / exclusion criteria, quality checks, data-split strategy, provenance requirements (chapter 04) |
| Re-training / re-tuning plan | Method, hyperparameter search bounds, compute reproducibility requirements, seed control, model-signing requirement (chapter 05) |
| Performance evaluation plan | Metrics measured, minimum acceptable values (the *acceptance criteria*), stratification (site, subgroup, condition), comparator to the previously authorised version |
| Update procedures | How the change is packaged, signed, released; how prior versions are retained; rollback procedure |
| Cybersecurity assessment | Supply-chain scans (chapter 06), signature validation (chapter 05), threat re-assessment |
| Post-modification monitoring | The mod-109 monitor set that will run after the modification lands, with thresholds and alert cadence |

The protocol must be tight enough that a modification can be adjudged in-envelope or out-of-envelope on inspection. If the answer is ambiguous, the modification is out-of-envelope and requires a new submission.

### Impact Assessment

The *Impact Assessment* analyses, per modification type, the *benefits* the modifications offer, the *risks* they introduce or exacerbate, and the *mitigations* that keep the residual risk inside the previously authorised envelope. This is where the mod-102 harm model, the mod-103 register (three-state — inherent, residual, control-defeated), and the mod-108 guardrail catalogue are cited in the submission itself. The impact assessment argues:

- for each risk elevated by the authorised modifications, which controls (existing or added) contain the elevation;
- for each control credited, which monitor detects its failure and how quickly;
- how the modification protocol's acceptance criteria ensure that the residual risk remains inside the envelope the original submission established.

A PCCP whose impact assessment does not close each of those three lines will not be authorised.

## The PCCP as a release-gate specification

Read the modification protocol *as an engineer*, not as a regulatory document. It is your release-gate spec, in FDA vocabulary. Each modification type has:

- an *acceptance criterion set* — the pass rule (matches mod-101's "what we treat as pass" line);
- a *test set* — the datasets and evaluations that produce the metric (matches "how we test it");
- a *monitor set* — the ongoing monitoring that catches post-release drift (matches mod-109);
- an *evidence artefact* — the signed report the FDA will inspect (matches "what we file as evidence").

A team that already runs the mod-101 five-sentence translation on every AI RMF sub-category will find the PCCP protocol easy to fill in. A team that treats releases as informal will find the PCCP a forcing function — it turns release process into a signed protocol.

## Locked vs adaptive — the one distinction that shapes everything

A *locked* algorithm behaves identically every time it is given the same input. It does not learn from deployment data; updates happen only through explicit re-training that is either inside the PCCP envelope or the subject of a new submission.

An *adaptive* algorithm changes its behaviour based on deployment data. Continuous online learning, session-level fine-tuning, and reinforcement-learning-from-clinical-feedback are all adaptive.

The FDA has been comfortable with locked algorithms since the earliest AI/ML authorisations. Adaptive algorithms require a PCCP that specifies exactly *how* they adapt, *what* they can adapt to, and *what* stops them adapting outside the envelope. Adaptive without a PCCP is not permitted in a marketed device.

The engineering consequence: if your device is adaptive, the adaptation path itself is a control that must be evidenced. The training-data ingestion pipeline is a controlled surface; the retraining trigger is a controlled surface; the newly trained model must pass the modification-protocol gate before it can replace the incumbent. Every step in that adaptation loop is an artefact.

## Cross-references

- **mod-101 chapter 05** placed FDA GMLP + PCCP alongside SR 11-7 as sector flavours of the same shape.
- **mod-102** harm model rows are cited in the PCCP impact assessment.
- **mod-103** register — inherent / residual / control-defeated — is the arithmetic the impact assessment depends on.
- **mod-106** fairness / bias evaluation is the disaggregated performance report GMLP principle 3 and 8 want.
- **mod-108** guardrails and human oversight are the human-AI team design GMLP principle 7 wants.
- **mod-109** monitoring is GMLP principle 10 and the post-modification monitoring block inside the PCCP.
- **mod-111** incident response includes the medical-device reporting (MDR) branch that reads a safety event and files to FDA within regulatory windows.
- **mod-112** cross-functional programme carries the GMLP principle 1 multi-disciplinary team charter.
- Chapters 04-06 of this module supply the supply-chain provenance and signing that GMLP principle 2 and PCCP cybersecurity assessment require.

## Summary

- FDA / Health Canada / MHRA GMLP guiding principles (October 2021) is a ten-principle statement that scaffolds AI/ML SaMD review; each principle resolves to a concrete engineering artefact.
- The FDA PCCP guidance (finalised December 2024) authorises predetermined change to an AI-enabled device across three components: description of modifications, modification protocol, impact assessment.
- The modification protocol *is* a release-gate specification in regulator vocabulary; a team that runs disciplined release gates has most of it already.
- Locked vs adaptive is the distinction that drives PCCP scope; adaptive without a PCCP is not permitted in a marketed device.
- The impact assessment cites the harm model, the three-state register, and the guardrail catalogue; without those, the PCCP will not be authorised.
- Exercise 03 drafts the risk-engineering slice of a PCCP for one SaMD-shaped system; do the exercise even if your system is not a medical device — the discipline transfers.
