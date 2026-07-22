# SR 23-4 and Third-Party Model / API Due Diligence

Almost no production AI system at a modern FI is built from first principles by the FI. There is a foundation-model API in the middle, or a hosted embedding model, or a fine-tuning-service vendor, or an off-the-shelf classifier, or a specialised OCR provider, or a labelled-data supplier. Every one of those is a **third-party relationship** in the language of US banking supervision — and the shape you apply to them is set by the *Interagency Guidance on Third-Party Relationships: Risk Management* published in June 2023 (Federal Reserve SR 23-4, jointly with the OCC and FDIC). This chapter teaches how SR 23-4 lands on AI-specific third parties, what the due-diligence questionnaire looks like when the third party is a model vendor, what contractual controls actually work, and how you monitor a vendor's model on your own signal rather than trusting their status page.

## Motivation

Two failure modes to avoid:

- **Copy-paste SaaS due diligence.** The team hands the foundation-model vendor the standard SaaS questionnaire — SOC 2, availability SLAs, data-processing addendum — and calls it done. Nothing in the questionnaire asks about model-change notification, training-data provenance, adversarial-robustness posture, or safety-classifier behaviour. When the vendor silently rotates the underlying model, your evaluations invalidate overnight and nobody notices for three weeks.
- **Vendor status page as monitoring.** The team declares the vendor's own uptime dashboard sufficient monitoring. It is not. The vendor's dashboard measures *the vendor's* metrics (availability, latency); it does not measure your safety metrics, your calibration, or your fairness slices. When behaviour changes without an incident being declared, you find out from a customer complaint.

SR 23-4 was written before generative AI was mainstream, but the *shape* it prescribes — planning, due diligence, contract, ongoing monitoring, termination — maps cleanly onto AI third parties if you translate each stage.

## SR 23-4 in six sentences

The Interagency Guidance was jointly issued by the Federal Reserve, the OCC, and the FDIC in June 2023 to replace prior agency-specific third-party guidance (including OCC Bulletin 2013-29). It applies to *any business arrangement* between a banking organisation and another entity, by contract or otherwise, and it deliberately covers a broad set of relationships — cloud, SaaS, data brokers, fintech partnerships, joint ventures. It prescribes a **third-party risk-management lifecycle**: planning, due diligence and third-party selection, contract negotiation, ongoing monitoring, and termination. Governance and independent review sit above the lifecycle. The guidance is *risk-based* — the depth of due diligence scales with the criticality of the relationship — so a foundation-model API that decides customer outcomes gets a very different depth of review than a document-classification vendor used only internally. Nothing in the guidance is AI-specific, but everything in it applies to AI third parties.

## The five lifecycle stages, translated to AI vendors

### 1. Planning

Before you select any vendor, decide *what the third party will do*, *what data will cross the boundary*, *what decisions will be influenced by the output*, and *what the criticality tier is*. For AI vendors specifically, planning has to answer:

- Which harm-model scenarios (mod-102) does this vendor's component participate in? If the answer is "many high-severity rows," the criticality tier is high.
- Which mod-103 register rows will credit this vendor's controls? If any residual score depends on a vendor-side control, the vendor cannot be reviewed as "non-critical".
- Which regulator-visible surface does the vendor sit behind? Consumer-facing lending decisions and medical device outputs sit behind heavily regulated surfaces; internal search does not.

The tier decision drives the depth of due diligence in stage 2. Under SR 23-4, *critical third parties* get materially more scrutiny than others.

### 2. Due diligence and third-party selection

The due-diligence pass is the questionnaire. For AI vendors, the standard SaaS questionnaire needs a *model-and-data* extension. A working question set:

| Question area | Concrete questions to ask |
|---|---|
| Model identity and version | Which model versions are in scope? What is the versioning scheme? Which version am I on today, and how will I know when it changes? |
| Change notification | Are model changes (weights, prompts, fine-tunes, safety-classifier updates) notified to me in advance? On what cadence? Can I pin to a version? |
| Training-data provenance | Do you publish or share a data provenance summary (Datasheet, Data Card)? What data-source classes were used? Are there opt-out or exclusion mechanisms? |
| Safety-classifier / guardrail posture | What safety classifiers or guardrails run on top of the model? Are they configurable by me? What is the false-positive / false-negative posture? |
| Evaluation and red-team disclosure | Do you publish safety cards, system cards, or model cards? Do you share red-team findings under NDA? What is your policy on responsible disclosure of vulnerabilities I find? |
| Adversarial-robustness posture | Do you publish adversarial-evaluation results? Under what threat model? Is prompt-injection resistance evaluated? |
| Fairness / bias posture | Do you publish disaggregated performance across demographic slices where relevant? What corrective measures apply? |
| Privacy | Where does prompt / completion data live? For how long? Is it used for training your future models by default, and can I opt out? What is your response to a subject access request? |
| Incident disclosure | What is your definition of a "safety incident"? What is your notification SLA? Is post-incident detail shared? |
| Right-to-audit | Do I get contractual right-to-audit? Under what conditions? Does that include model-behaviour audit (as opposed to only controls audit)? |
| Sub-processors | Who are your sub-processors for compute, storage, and safety filtering? Are their model / data provenance disclosures available to me? |
| Termination / model portability | If I terminate, what is my exit path? Can I export prompt-completion history? Is there a portable-model alternative? |

This questionnaire is *itself* an artefact in the MRM packet (chapter 01, "Third-party components"). Version it, date it, and store the vendor's responses alongside your evaluation results.

Where the vendor cannot or will not answer a question, note it as a *disclosure gap*. Disclosure gaps do not automatically kill a relationship — they raise the residual risk on the mod-103 rows the vendor participates in, and they require *compensating monitoring* from your side.

### 3. Contract negotiation

The contract is where due-diligence answers become enforceable. For an AI vendor, the AI-specific clauses to press for:

| Clause area | What good looks like |
|---|---|
| Version pinning | You can pin to a named model version; the vendor cannot silently rotate the underlying model on the pinned tag |
| Change-notification SLA | Notice window (e.g., N days) before any change to a pinned or default model; content of the notice (what changed, expected behaviour delta, mitigations) |
| Safety-metric SLA | The vendor commits to a floor on named safety-classifier metrics (e.g., "toxicity-filter recall ≥ X on our benchmark"); breach triggers notification |
| Right-to-audit | Contractually specified right to audit vendor controls (SOC 2 baseline) plus an AI-specific *right to run your own evaluations* against the vendor's model without terms-of-service violation |
| Data use | Prompt / completion data not used to train the vendor's future models by default; if it is, an opt-out is available and documented |
| Incident notification | Named safety-incident categories with a notification SLA per category |
| Sub-processor disclosure | List of sub-processors published; change notice on additions |
| Termination assistance | Exit runway including access to prompt / completion history and reasonable transition support |
| Liability and indemnity | Standard commercial terms; AI-specific carve-outs (e.g., IP indemnity on model output) as applicable |

Not every vendor will sign every clause. For each clause the vendor refuses, you have three options: *walk*, *escalate to the risk council as a residual-risk acceptance*, or *engineer a compensating control on your side* (a monitor, an internal safety classifier, a version-freeze wrapper). The choice is a risk-council decision (mod-112), not an engineer's decision.

### 4. Ongoing monitoring

This is where SR 23-4 diverges most from typical SaaS practice. The vendor's own status page is not sufficient. You need to monitor the vendor's model *on your own signal*.

| Vendor-side monitoring signal you own | Where it lives |
|---|---|
| Behaviour-drift monitor: a fixed prompt / input set is run against the vendor's model on a cadence; outputs are hashed and compared to baseline | Mod-109 monitor spec |
| Safety-metric monitor: the safety metrics you care about (harmful-content rate, injection-resistance rate, calibration) are measured on a rolling sample of production traffic | Mod-109 |
| Fairness monitor on the vendor's outputs across your demographic slices | Mod-106 fairness signal wired into mod-109 |
| Cost / usage anomaly detection (a proxy for silent model change: cost per token spikes when the vendor swaps to a bigger model) | Ops / FinOps signal fed into mod-109 |
| Regression against a pinned red-team fixture: the mod-104 fixture set runs against the vendor's current version daily | Mod-104 fixture + mod-109 wiring |
| Vendor's own model card / system card version tracking (the vendor sometimes publishes model changes as card revisions before or after the API silently changes) | RSS or scraper against the vendor's docs surface |

The point of monitoring on your own signal is *catching silent change*. Every foundation-model vendor has, at some point, rotated an underlying model on a stable API tag with either no notice or notice that failed to reach the affected teams. Your monitoring is what makes that visible before a customer notices.

### 5. Termination

Termination planning is not optional. SR 23-4 expects a documented exit path *before* you enter the relationship. For an AI vendor:

- What is your fallback model if the vendor exits (goes down, is acquired, changes terms)?
- What is the runway on that fallback (a canary running the fallback on 5% of traffic; a wrapper that can switch providers on a config change)?
- What data must you retrieve at termination (prompt / completion history, fine-tune weights that you paid for, evaluation results)?
- What is your obligation to notify downstream consumers of an infrastructure change?

Termination readiness is a risk register row: if the vendor exits tomorrow, what is the residual risk to your service, and is it inside appetite?

## The vendor-and-you MRM crosswalk

The SR 23-4 file for a critical AI vendor lives in the MRM packet under the "Third-party components" heading. The internal structure of that file matches the SR 23-4 lifecycle:

| SR 23-4 lifecycle stage | Artefact you file |
|---|---|
| Planning | Tier decision memo, harm-model rows the vendor participates in, register rows the vendor is credited on |
| Due diligence | Vendor questionnaire (version dated) + vendor responses + disclosure-gap list |
| Contract | Contract summary or excerpt showing the AI-specific clauses; a table of clauses attempted vs won vs walked-away |
| Ongoing monitoring | Monitor spec (from mod-109) with cadence, threshold, and last-N runs; drift-monitor results |
| Termination | Fallback design, runway estimate, data-retrieval plan |

## Cross-references

- **mod-101 chapter 05** placed SR 23-4 alongside SR 11-7 and the FDA GMLP shapes.
- **mod-103** register rows carry the vendor-credited controls; when a vendor control is unavailable or contractually not enforceable, the control-defeated column shows the resulting exposure.
- **mod-108 chapter on build-vs-buy** feeds directly into SR 23-4 planning — the build-vs-buy decision *creates* the third-party relationship SR 23-4 governs.
- **mod-109** owns the vendor-side monitors that make ongoing monitoring more than trust-me.
- **mod-111** incident-response playbook must include a "vendor-side incident" branch that reads notification content, kicks off an internal impact assessment, and updates the register within a defined window.
- Chapter 01 of this module described the parent MRM packet this vendor file sits inside.
- Chapter 04 of this module produces the ML-BOM the vendor's components appear on and the SPDX / SLSA provenance for anything the vendor supplies as an artefact.

## Summary

- SR 23-4 (Interagency Guidance on Third-Party Relationships, June 2023) is the parent shape for third-party AI vendors at US FIs; its lifecycle is planning, due diligence, contract, ongoing monitoring, termination.
- Standard SaaS due diligence is insufficient for AI vendors; the questionnaire needs a model-and-data extension covering version pinning, change notification, provenance, safety-classifier behaviour, adversarial posture, and incident disclosure.
- Contract clauses that matter: version pinning, change-notification SLA, safety-metric SLA, right-to-audit including behaviour audit, data-use opt-out, sub-processor disclosure, termination assistance.
- Ongoing monitoring on your own signal — drift, safety, fairness, cost anomaly, red-team fixture regression — is what catches silent vendor-side change; the vendor's status page is not sufficient.
- Every disclosure gap and every clause you did not win raises the residual risk on the corresponding mod-103 rows and either requires escalation or requires a compensating control on your side.
- Exercise 02 runs this shape end-to-end on one third-party model / API vendor of your choice.
