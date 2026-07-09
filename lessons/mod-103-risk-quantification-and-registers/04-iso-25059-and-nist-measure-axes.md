# The Scoring Dimensions: ISO/IEC 25059 Quality Model and NIST AI RMF MEASURE

Chapter 02 defined *how* to score (impact × likelihood × exposure). This chapter answers a different question: *along which quality dimensions* is a system being scored in the first place? The two source-of-truth catalogues at this level are **ISO/IEC 25059** (AI-specific quality model) and the **NIST AI RMF MEASURE function's** sub-categories. Reuse them as your scoring axes so an auditor can see *what* is being measured and cite the standard that defines the dimension.

## Motivation

An impact-likelihood-exposure scoring rubric can be applied to any scenario. The prior question is *which scenarios you should be scoring*. Two failure modes appear when the dimensions are not pinned to a standard:

- **The scoring covers what the team finds interesting** rather than what the system is being evaluated on. A robust security column and no fairness column. A rigorous accuracy column and no over-reliance column. The audit finding is "coverage is incomplete."
- **The dimensions are invented.** The team defines "trustworthiness" and "safety" as scoring axes with in-house definitions. An auditor cannot cite a standard, so the argument is by taste. The finding is "the scoring axes are not attributable."

The fix is to *reuse* two well-established dimension catalogues: ISO/IEC 25059 for the *quality* dimensions, NIST AI RMF MEASURE for the *evidence* dimensions. Both are indexed, both are cited-able, and both are widely enough understood to survive an audit read.

## ISO/IEC 25059 as the quality dimensions

**ISO/IEC 25059:2023 — Software engineering — Systems and software Quality Requirements and Evaluation (SQuaRE) — Quality model for AI systems** is the AI-specific member of the SQuaRE family. It sits alongside ISO/IEC 25010 (the general software quality model) and extends it with AI-specific quality characteristics.

At level-25 scope the useful pattern is:

- Treat ISO/IEC 25010 characteristics (functional suitability, performance efficiency, reliability, usability, security, maintainability, compatibility, portability) as the *shared-with-classical-software* baseline.
- Treat the ISO/IEC 25059 AI-specific characteristics — the ones covering trustworthiness aspects specific to AI systems — as the *AI-specific additions*.

The AI-specific characteristics include (working list — verify against the standard's clause text):

- **User controllability** — the degree to which the AI system allows the user to control its behaviour and outputs.
- **Transparency** — the degree to which information about the AI system is made available to relevant stakeholders.
- **Robustness** — the degree to which the AI system maintains its level of performance under any circumstances (adversarial and stress conditions).
- **Intervenability** — the degree to which humans can intervene in the AI system's behaviour.
- **Functional adaptability** — the degree to which the AI system can adapt its function to different environments or contexts.

<!-- needs-research: pin the exact list of AI-specific characteristics and sub-characteristics in ISO/IEC 25059:2023 clause 5 (verify clause identifier) against the standard text purchased through an ISO national body; do not invent items or definitions. -->

The pattern: for each system, publish a *quality profile* that names which ISO/IEC 25059 characteristics are in-scope and to what target. That profile is what your risk register scores *against*. Each risk-register row cites the quality characteristic it primarily degrades.

### A worked mapping — customer-support RAG assistant

For the running customer-support RAG assistant:

| ISO/IEC 25059 characteristic (shared or AI-specific) | Relevance | Scoring linkage |
|---|---|---|
| Functional suitability (25010) | Answer relevance to customer question | Register rows about *wrong-answer* harms score here |
| Reliability (25010) | Answer availability and consistency | Register rows about *drift* score here |
| Security (25010) | RAG store confidentiality, tool-call authorisation | HR-CS-INJ-01 primary linkage |
| User controllability (25059) | Customer can escalate to human at any turn | Register rows about *lock-in / over-reliance* score here |
| Transparency (25059) | User is aware they are talking to an assistant | Register rows about *human-AI configuration* score here |
| Robustness (25059) | Behaviour under adversarial input | HR-CS-INJ-01 secondary linkage; mod-104 / mod-105 evidence |
| Intervenability (25059) | Support agent can override / edit before send | Register rows about *irreversibility* score here |

If a register row does not link to at least one 25059 characteristic, the row is coverage-orphaned. Either the quality profile needs a new characteristic added, or the row is out-of-scope.

## NIST AI RMF MEASURE as the evidence dimensions

Mod-101 ch 01 established that the level-25 role owns the AI RMF **MEASURE** function — every MEASURE sub-category translates to a test, threshold, guardrail, monitor, and evidence artifact. Here we use the *sub-category identifiers themselves* as scoring dimensions on the register.

The AI RMF Playbook enumerates MEASURE sub-categories under a hierarchy that spans categories such as:

- MEASURE 1 — Appropriate methods and metrics are identified and applied.
- MEASURE 2 — AI systems are evaluated for trustworthy characteristics.
- MEASURE 3 — Mechanisms for tracking identified AI risks over time are in place.
- MEASURE 4 — Feedback about efficacy of measurement is gathered and assessed.

<!-- needs-research: pin the exact current MEASURE sub-category identifiers and their outcome statements from the NIST AI RMF Playbook at authoring time; the framework may have added or renumbered sub-categories. -->

Individual sub-categories such as MEASURE 2.7 (AI system security and resilience), MEASURE 2.11 (fairness and bias), MEASURE 2.3 (evaluation under deployment-similar conditions), and MEASURE 3.2 (unforeseen negative impacts) are the specific *dimensions* along which a register row is evaluated.

### How the register cites MEASURE

Each register row cites the MEASURE sub-categories it is bounded by. The sub-category answers "on what dimension is this row being measured?" and points at the evidence pipeline (mod-101 ch 01's five-column table).

Continuing HR-CS-INJ-01:

```yaml
row_id: HR-CS-INJ-01
iso_25059_characteristics:
  primary: security                 # 25010 characteristic
  secondary: [robustness, user_controllability]  # 25059 additions
nist_ai_rmf_measure:
  primary: MEASURE-2.7              # AI system security and resilience
  secondary: [MEASURE-2.3, MEASURE-3.2]
```

`MEASURE-2.7` is the sub-category the row is measured against. Its outcome statement — "AI system security and resilience are evaluated and documented" — is what the row's evidence pipeline (test / threshold / guardrail / monitor / artifact) is closing.

### Composing MEASURE with 25059

The two catalogues complement each other:

| Question | Answered by |
|---|---|
| What quality characteristic is at stake? | ISO/IEC 25059 |
| What measurement outcome closes it? | NIST AI RMF MEASURE |

A row with only a 25059 tag reads as "we care about robustness for this scenario" but doesn't say what evidence closes it. A row with only a MEASURE tag reads as "we ran evidence for MEASURE-2.7" but doesn't say which quality characteristic is being asserted. Both together read defensibly.

### A working coverage grid

Publish, per system, a coverage grid of 25059 characteristics × MEASURE sub-categories, and mark which register rows fill each cell. Empty cells are open work — either the characteristic isn't in-scope (mark it so), or the MEASURE sub-category isn't applicable (mark it so), or you have a coverage gap.

|                     | MEASURE-2.7 | MEASURE-2.11 | MEASURE-2.3 | MEASURE-3.2 |
|---|---|---|---|---|
| Robustness (25059)  | HR-CS-INJ-01 | | HR-CS-DRIFT-02 | HR-CS-INC-04 |
| Security (25010)    | HR-CS-INJ-01 | | | HR-CS-INC-04 |
| Transparency (25059)| | | | HR-CS-TRANS-06 |
| User controllability (25059) | | HR-CS-BIAS-03 | | |

Empty cells mean either the characteristic × MEASURE combination isn't relevant to this system (explicitly mark so) or you owe a row.

## Sector-overlay reuse of the same axes

Sector obligations from mod-101 ch 05 fold onto the same scoring dimensions rather than adding parallel columns:

- **SR 11-7 / OCC 2011-12 / SR 23-4** (US FI MRM) — model risk is scored on model *use appropriateness, conceptual soundness, ongoing monitoring*. Each is expressible as a 25059 × MEASURE pair. Cite the SR 11-7 dimension against the corresponding 25059 characteristic on the row.
- **FDA GMLP** — the GMLP guiding principles map to characteristics such as robustness, transparency, human factors, and lifecycle monitoring. Each cite composes with 25059 and MEASURE.
- **EU AI Act (Article 15, robustness / accuracy / cybersecurity)** — cites onto MEASURE-2.7 and the 25059 robustness characteristic; the Article 9 risk-management-system requirement folds onto the entire 27005-shaped register process (chapter 03).
- **Colorado AI Act / NYC LL 144 (bias audits)** — cite onto MEASURE-2.11 and the 25059 user-controllability / functional-suitability characteristics.

The pattern: sector obligations are *citations onto* the same scoring dimensions, not new dimensions of their own. That is how the register stays one file, not five.

## What the dimensions are *not*

- **Not a coverage checklist you tick.** Simply naming that MEASURE-2.7 applies is not a score. The row still has to carry the impact / likelihood / exposure scoring and the inherent / residual / control-defeated states. The dimension citation says *what* is being measured; the scores say *how much risk* is there.
- **Not immutable.** Both 25059 (as a standard) and the NIST AI RMF (as a versioned framework) are updated. Track the versions you cite and re-verify on framework revisions. Pin the version at row level so historical rows are still defensible against the version they were authored under.
- **Not exhaustive by themselves.** For agentic systems, MEASURE and 25059 may under-cover autonomy and tool-topology risk; those escalate to `agentic-safety-engineer` (L40) — chapter 07 pins that boundary.

## Anti-patterns

- **In-house dimension inventions.** Naming "trust" or "safety" as a scoring axis without citing 25059 or MEASURE. Reviewers cannot cite a definition and the register becomes taste-based.
- **Sub-category tags without an evidence pointer.** A row cites MEASURE-2.7 but the mod-101 ch 01 evidence pipeline is empty. The tag is a promise; the pipeline is the promise being kept.
- **Overlapping quality characteristics collapsed.** A row cites "security and robustness" as one dimension. Split them — security controls do not necessarily buy robustness under adversarial ML, and residual scores diverge.
- **Version-drift.** A row was scored against MEASURE-2.7 as it read in 2023 and the outcome statement has been revised. Diff the framework version and re-verify on refresh.

## Interfaces with adjacent modules

- **Mod-102** produces the harm-model rows; here each row picks up a 25059 characteristic and one-or-more MEASURE sub-categories.
- **Mod-104 (LLM red teaming)** produces evidence for MEASURE-2.7 (security / robustness) rows.
- **Mod-105 (adversarial ML)** produces evidence for MEASURE-2.7 (adversarial robustness) rows.
- **Mod-106 (fairness / bias / explainability)** produces evidence for MEASURE-2.11 (fairness / bias) rows and 25059 transparency / user-controllability characteristics.
- **Mod-107 (privacy risk)** produces evidence for MEASURE-2 sub-categories on data privacy and 25010 confidentiality.
- **Mod-108 (guardrails)** supplies the controls that swing residual scores.
- **Mod-109 (monitoring)** produces MEASURE-3 tracking evidence and closes the loop.

## Summary

- Reuse ISO/IEC 25059 (AI quality model) as the quality-characteristic axes and NIST AI RMF MEASURE sub-categories as the measurement / evidence axes.
- Every register row cites *at least one* 25059 characteristic and *at least one* MEASURE sub-category; both together say *what* is being scored.
- Publish a per-system coverage grid of 25059 × MEASURE with register rows in each occupied cell.
- Sector obligations (SR 11-7, FDA GMLP, EU AI Act, Colorado / NYC bias audits) fold onto the same axes as citations rather than parallel columns.
- Pin the framework versions cited; re-verify on updates.
- The dimensions are *what* is being scored; the impact / likelihood / exposure axes from chapter 02 are *how much* is being scored. Both are required.
