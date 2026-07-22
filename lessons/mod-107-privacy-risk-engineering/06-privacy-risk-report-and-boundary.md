# The Privacy-Risk Report and the Boundary to Legal Counsel and Level 30

Everything in chapters 01-05 lands in one artifact: the **privacy-risk report** for one AI system. This chapter engineers that artifact. It fixes the report shape, maps each section to GDPR concepts (Article 22 in particular) and to the NIST Privacy Framework functions, states the residual-risk claim the report is entitled to make, and draws two boundaries clearly — what is out of scope to *legal counsel*, and what is out of scope to *`model-evaluation-engineer`* at level 30. A privacy-risk report that muddles either boundary either overreaches (giving a legal opinion the engineer is not qualified to give) or under-reaches (calling something audit-grade when it needs methodological calibration the peer role owns).

## Motivation

Two audiences read the privacy-risk report and neither wants what the other wants.

- **The engineering and governance internal audiences** — the risk council, the peer eval track, security, the product team — want numeric residual-risk evidence per scenario, credited-control lists, monitoring plans, and a clean set of open items.
- **The compliance / legal / regulator-facing audiences** — the Data Protection Officer, external counsel, the auditor, the supervisory authority — want the mapping to legal concepts (personal data, special category, profiling, automated decision-making), a statement of the DPIA position under GDPR Article 35, and the audit trail of the technical decisions that support each claim.

A single report has to serve both. Level-25 authorship is: produce a report that *both* audiences read as the same document with two entry points. This chapter engineers that shape.

## The report shape

A privacy-risk report for one AI system, at this level, has ten sections. The shape is deliberately similar to the harm-model shape (mod-102) and the register shape (mod-103) so downstream artifacts compose.

1. **System description.** One page. What the system does; who the subjects are (users of the product, third parties whose data is processed, upstream data suppliers); the personal-data categories processed; the deployment shape (classifier vs. LLM vs. RAG vs. fine-tuned LLM — chapter 01's applicability table).
2. **Scope of this report.** What is and is not in scope. Explicitly cites the mod-102 harm-model version and the mod-103 register version this report is aligned to.
3. **Attack evaluation summary.** One table row per attack in scope: MIA / LiRA, attribute inference, model inversion, LLM memorisation / extraction. For each: the threat model, the metric, the residual number, the reference to the underlying eval artifact.
4. **Defence summary.** One row per defence credited: DP-SGD parameters (ε, δ, σ, C, accountant, sampling); Presidio pipeline positions (ingest, retrieval, output) with precision / recall per entity type; retrieval-side ACL; output filters; access controls. For each defence: the credited-control evidence pointer, the false-negative / degradation notes, and the control-defeated variant.
5. **Residual-risk claim per scenario.** The register row for each scenario: inherent, residual, control-defeated per named control (chapter 04's shape).
6. **Slice-level and equity notes.** Per-attack slice reporting: outlier deciles for MIA, per-language recall for Presidio, per-demographic utility for DP-SGD. Cross-linked to mod-106.
7. **GDPR mapping.** The mapping table below.
8. **NIST Privacy Framework mapping.** The mapping table below.
9. **Monitoring and incident-response hooks.** How mod-109 monitors each control-defeated row; how mod-111 routes the incident if the row's residual number breaches.
10. **Open items and boundary notes.** What escalates to legal counsel; what escalates to `model-evaluation-engineer` at level 30; what escalates to the risk council.

The report is versioned in the risk-engineering repo. Every material change gets a diff and a review.

## Section 7 — GDPR mapping table (shape)

Every row of the register (section 5) carries at least one GDPR-concept tag. The mapping table in the report:

| Register row | Attack / scenario | GDPR concept invoked | Article | Notes |
|---|---|---|---|---|
| HR-CLS-MIA-01 | MIA on classifier trained partly on Article 9 population | Special-category personal data; profiling if the model output feeds a decision | Art. 9; Art. 4(4) | Membership in an Article-9-flavoured cohort is itself Art. 9 data |
| HR-CLS-AI-01 | Attribute inference of income band from features + model output | Personal data; profiling | Art. 4(1); Art. 4(4) | Inferred attribute is personal data about the subject |
| HR-VIS-INV-01 | Model inversion producing recognisable face for labelled identity | Biometric data; Art. 9 | Art. 4(14); Art. 9(1) | Facial reconstruction of an identified subject |
| HR-LLM-EXT-01 | Verbatim extraction of PII from fine-tuned LLM | Personal data; possibly processing without lawful basis if training data was scraped | Art. 4(1); Art. 6 | Extraction rate: chapter 03's canary curve |
| HR-DEC-A22-01 | The model output *is* an automated individual decision with legal / similarly significant effect | Automated individual decision-making, including profiling | Art. 22 | Human-review / contest mechanism required by Art. 22(3) |

**Article 22 in particular** — the automated-individual-decision-making article — is where this module's privacy report intersects most directly with the wider governance stack. If the AI system produces a decision that legally or similarly-significantly affects the subject, and that decision is produced without meaningful human intervention, Article 22 obligations attach. The report:

- States whether Article 22 applies (as an engineering read of the system's role, not a legal conclusion).
- If it applies, records the *human-in-the-loop* configuration — where the human sees the model output, what the human is empowered to change, and how contest / review requests are routed. This is engineering evidence for the legal claim; the legal claim itself is counsel's.
- If it applies, records the *safeguards*: transparency about the decision logic (to the extent required and possible), the subject's right to obtain human intervention, the ability to express a view, the ability to contest. Each of these is either an engineering artifact (a UI surface, an API, a workflow) or a policy artifact.

**Article 35** — DPIA — is the second GDPR anchor. The report states whether a DPIA is required (a governance decision that the engineering evidence supports); if it is, the privacy-risk report is *material* to the DPIA and is cited from it. The DPIA itself is not authored in this module — it is authored by the DPO / privacy office and by `ai-governance-analyst` at level 15.

**Articles 25 and 32** — data protection by design and default (25) and security of processing (32) — are what many of the credited controls are evidence for. The mapping is not one-to-one; state which controls the report treats as evidence for which article.

## Section 8 — NIST Privacy Framework mapping table (shape)

The NIST Privacy Framework's function / category / sub-category layer maps naturally onto this module's engineering artifacts:

| NIST PF Function | Category (representative) | Engineering artifact from this module |
|---|---|---|
| IDENTIFY-P | Inventory and mapping; data processing ecosystem risk assessment | Attack evaluation summary (section 3); residual-risk per scenario (section 5) |
| GOVERN-P | Governance policies, processes, and procedures | The report itself; the register linkage; the change-log |
| CONTROL-P | Data processing policies, processes, and procedures; data processing management | DP-SGD parameters; Presidio pipeline; retrieval ACL; output filter |
| COMMUNICATE-P | Communication policies, processes, and procedures | Sections 6 and 7 of the report; subject-facing transparency artifacts named in section 4 |
| PROTECT-P | Data protection policies; identity management, authentication and access control; data security; maintenance | DP-SGD training itself; Presidio pipeline; API access controls; monitoring hooks |

The NIST PF Core lists Category and Sub-Category codes (e.g., `ID.IM-P1`, `PR.DS-P5`); the report tags each credited artifact with the sub-category it addresses. Chapter 01's mapping and this section's mapping together give a governance reader the full trace from a legal / framework line to a residual-risk number.

## The residual-risk claim the report is entitled to make

The report's residual-risk claim is bounded on three axes. State the bounds explicitly.

- **Attack strength.** "Under attack A at configuration C, on our held-out evaluation dataset D, the residual signal is X." Not "the system is safe against MIA" — no eval supports that claim.
- **Time.** "As of release R, on training-data snapshot T, and PII-corpus snapshot P." Data drifts; models are re-trained. The claim's validity window is the release cadence.
- **Threat model.** "Under black-box query access with logit responses and unlimited query rate." Extraction claims under one threat model do not survive an API upgrade that exposes more.

An anti-pattern to avoid: the summary sentence *"the system is GDPR-compliant."* That is a legal conclusion, not an engineering claim, and it is not the engineer's to make. What the engineer can make is: *"the technical controls listed in section 4 close the empirical risks measured in section 3 to the residual levels in section 5; the mapping to GDPR concepts is in section 7; the compliance conclusion is counsel's."*

## Boundary 1 — what is out of scope to legal counsel

Legal opinions on GDPR / EU AI Act / sector-specific privacy law are counsel's, not the engineer's. The engineer produces:

- The technical evidence (residual-risk numbers, credited controls, false-negative rates).
- The mapping to legal concepts *as an engineering read* (not a legal conclusion).
- The concrete list of open items where a legal opinion is needed.

The engineer does not:

- Interpret ambiguous GDPR articles.
- Decide whether processing has a lawful basis under Article 6.
- Decide whether a specific category of data is or is not "special category" for an ambiguous case.
- Draft the DPIA (that is a joint DPO / L15 artifact this report feeds into).
- Draft the contract clauses with data-processing partners.

An "out of scope to counsel" list in section 10 of the report names each open legal question and the fact pattern that raises it. Example entries:

- *Whether the training-data collection process qualifies as legitimate-interest processing under Article 6(1)(f) for the population of subjects at [scenario X].* Route to DPO + counsel.
- *Whether the model output at [scenario Y] constitutes a decision with legal or similarly significant effect for the purposes of Article 22.* Route to DPO + counsel.
- *Whether the residual extraction rate at [scenario Z] triggers a subject-access-request obligation for subjects whose PII appears in the training corpus.* Route to DPO + counsel.

Each entry has (a) the fact pattern, (b) the concrete technical evidence attached, and (c) the escalation destination. Nothing is left as an unspecified "TBD legal."

## Boundary 2 — what escalates to `model-evaluation-engineer` (level 30)

The peer role at level 30 owns the *methodological calibration* that a level-25 privacy eval defers on. Examples of the escalation:

- **Confidence intervals on ROC curves and TPR-at-fixed-FPR numbers.** A level-25 report cites the point estimate at the run's shadow-model count; a level-30 review says "at this shadow-model count and this attack-training-set size, the 95% confidence interval around TPR@1%FPR is [a, b]."
- **Statistical-significance framing of slice-level differences.** A level-25 report cites the outlier-decile TPR as a worst-case; a level-30 review says whether the delta from the overall TPR is significant given the slice size.
- **Distribution-shift robustness of the eval itself.** A level-25 report gives the eval on the deployment-shaped evaluation set; a level-30 review characterises the eval's stability under distribution shift and under adversarial re-weighting.
- **Cross-eval comparability and meta-analysis.** A level-25 report gives the residual numbers for this release; a level-30 review compares across releases, across model families, and against published attack baselines with error accounting.
- **Attack-tuning as a discipline.** A level-25 report cites the strongest attack the engineer ran; a level-30 review says whether the attack landscape has moved and whether the residual claim needs re-basing.

The escalation is explicit. The report's section 10 names each item that needs level-30 methodological calibration and states the current level-25 position on it. This is not a weakness of the report — it is the shape of a shippable level-25 artifact, and it is how the peer track and this track compose without overlap.

## The role of `ai-governance-analyst` (level 15) as upstream

L15 owns the *impact-assessment* artifact (mod-101 chapter 03; ISO/IEC 42005). The privacy-risk report from this module is one of the inputs the impact assessment consumes. Downwards from L15, engineering evidence is treated as a black box ("the eval says X"). Upwards from L25, framework interpretation is treated as a black box ("counsel says Y"). The engineer's report holds the middle.

## Change-log discipline

The report is versioned. Each release records:

- What changed in the system (new capability, new integration, new data source).
- What changed in the evaluation (new attack, new attack configuration, larger shadow-model count, new canary set).
- What changed in the defence (new DP-SGD parameters, new Presidio recogniser, new retrieval-side ACL).
- The delta in each residual-risk number.
- The legal / counsel escalations that opened or closed since the last version.

A privacy-risk report without a change-log invites the auditor to reconstruct history from git — poorly. A change-log makes the trajectory of the residual-risk numbers legible.

## Reading order at review

For a reader coming to the report cold:

1. Section 1 (system) and section 2 (scope) — five minutes.
2. Section 5 (residual per scenario) — the argument in numbers.
3. Section 7 (GDPR mapping) and section 8 (NIST PF mapping) — the framework anchor.
4. Section 3 (attack summary) and section 4 (defence summary) — the evidence layer.
5. Section 10 (open items) — the escalations.

Section 6 (slices), section 9 (monitoring), and the change-log are what the auditor pulls when the top-line reading raises a question.

## Cross-references

- **mod-101** — the framework tie-in. RMF MEASURE, MANAGE; GenAI Profile risk on data privacy; ISO/IEC 42001 and 42005; EU AI Act Article 10 (data governance).
- **mod-102** — the report's scenario list is the mod-102 harm-model rows filtered to privacy scenarios.
- **mod-103** — the residual / control-defeated numbers are register rows.
- **mod-108** — DP-SGD-trained models and the Presidio pipeline are guardrail catalogue entries; the report is what shows the guardrails are doing the work claimed.
- **mod-109** — every control-defeated row has a monitoring hook; the mapping is in section 9.
- **mod-111** — the incident-response playbook consumes section 9 for routing.

## Summary

- The privacy-risk report is a ten-section artifact per AI system, versioned in the risk-engineering repo, aligned to specific mod-102 and mod-103 versions.
- Every scenario carries: attack evaluation number, credited defences, residual and control-defeated variants, GDPR-concept mapping, NIST Privacy Framework function mapping, monitoring hook.
- The engineering claim is bounded on attack strength, time, and threat model. It is not "GDPR-compliant"; that is counsel's.
- Two boundaries are drawn explicitly in section 10: what escalates to legal counsel (interpretation, lawful-basis decisions, DPIA authorship) and what escalates to `model-evaluation-engineer` at level 30 (methodological calibration, error bars, distribution-shift, cross-run meta-analysis).
- The report reads top-down as an argument in numbers, bottom-up as an evidence trail; the change-log makes trajectory legible across releases.
