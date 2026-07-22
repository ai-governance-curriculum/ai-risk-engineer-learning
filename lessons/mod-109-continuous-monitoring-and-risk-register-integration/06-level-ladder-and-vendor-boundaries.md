# Level-Ladder and Vendor Boundaries — Where This Module Ends

Chapters 01-05 defined the artifacts the risk engineer owns at level 25 for continuous monitoring: the signal contract, the drift-and-quality signal family, the safety-and-application signal family, the Article 72 report, and the alert-routing / single-source-of-truth artifact set. This chapter draws the boundaries — what the commercial platforms actually give you, where the peer tracks (`ai-infra-mlops` at L25 and `ai-eval-engineer` at L30) pick up, and where the module's outputs flow into mod-111 (incident response) and mod-112 (governance council).

## Motivation

Two failure modes are common at the vendor-boundary conversation:

- **Vendor-shaped scope creep.** A team adopts Fiddler or Arthur or Holistic AI and, because the platform *can* do drift, quality, fairness, explainability, and reporting, gradually stops distinguishing what the platform provides from what the team owns. When the platform is later swapped or its output is contested, the team cannot separate its own opinion from the vendor's default.
- **Under-scoping the peer tracks.** The risk engineer treats MLOps monitoring and application-layer online eval as things they can build "if they need them." Both are entire disciplines at neighbouring levels; treating them as extension work rather than peer roles produces gaps.

The engineering fix is to write the boundaries down. This chapter does that.

## The commercial platforms — what they cover, and what they do not

Four commercial platforms sit near the space this module covers. Their published feature sets overlap; the goal here is to describe the *kind* of workload each is used for, not to rank them.

<!-- needs-research: platform capability descriptions below are drawn from published product pages; feature sets change per release. Verify each claim against the current published product documentation at authoring time. -->

### Fiddler AI

- ML / LLM observability platform: model performance monitoring, drift detection, explainability (SHAP-family, integrated-gradients variants), bias monitoring, alerting; LLM-side features cover prompt / response monitoring, safety and hallucination metrics, and tracing.
- Sits primarily in the *observability + explainability* corner of the space; used most naturally when the depth question is "why did the model predict this" alongside "did the model drift."
- Official product page: <https://www.fiddler.ai/>.

### Arthur AI

- ML / LLM monitoring platform: performance, drift, bias monitoring, explainability, model validation; Arthur Shield is a runtime guardrail-and-monitoring product for LLM applications (prompt-injection detection, PII, toxicity, hallucination detection).
- Sits at the intersection of *runtime monitoring* and *governance reporting*; the Shield product is one of the commercial packages of the safety-monitor signal family from chapter 03.
- Official product page: <https://www.arthur.ai/>.

### WhyLabs

- Commercial hosted platform on top of the open-source `whylogs` profiler; adds profile storage, drift dashboards, alerting, LangKit for LLM signal profiling, and monitoring configuration.
- Sits primarily in the *drift and data-quality* corner; the natural pair with the open-source `whylogs` at the library level.
- Official product page: <https://whylabs.ai/>.

### Holistic AI

- AI risk / governance platform with model inventory, risk assessment, bias / fairness assessment, and monitoring capabilities.
- Sits primarily in the *GRC-for-AI* corner (also referenced in mod-103 chapter 06 alongside Credo AI and ModelOp); the monitoring capabilities are typically consumed as one of several signal sources into the governance workflow.
- Official product page: <https://www.holisticai.com/>.

### Where each fits vs. open-source (chapter 02 recap)

| Workload | Open-source starting point | Commercial platforms that cover it |
|---|---|---|
| Tabular / feature drift, profile storage | `whylogs` | WhyLabs, Fiddler, Arthur |
| Batch data-drift / quality reports | Evidently AI | Fiddler, Arthur, WhyLabs |
| Multivariate / embedding / online drift | Alibi Detect | Fiddler, Arthur |
| LLM safety-monitor (prompt-injection, PII, hallucination detection) | mod-108 guardrails + custom | Arthur Shield, Fiddler LLM Observability, WhyLabs LangKit |
| Governance workflow / sign-off / reporting | Register in repo (mod-103) | Holistic AI, Credo AI, ModelOp, Monitaur |

The commercial platforms bundle several of the open-source pieces plus workflow, alerting, and reporting. At level 25, your job is the same regardless of stack: the signal contract and the register are yours; the pipeline underneath is bought or built by the MLOps track; the governance workflow around it is bought or configured by the L50/L60 leadership.

## The peer boundary to `ai-infra-mlops` at L25

The MLOps track owns the *monitoring pipeline as an engineering system*. Concretely:

| Owned by `ai-infra-mlops` at L25 | Owned by `ai-risk-engineer` at L25 |
|---|---|
| Metric emitters (log libraries, profile shippers) | Signal semantics (what the number means) |
| Metric-store integration (Prometheus, feature store, warehouse) | Register-field mapping (what the number updates) |
| Batch schedulers, streaming windows, backfill orchestration | Cadence choice (defended per row) |
| Detector configuration and tuning (thresholds tuned for pipeline noise) | Threshold semantics (defended per row against the harm) |
| Pipeline SLOs (detector uptime, false-positive-rate targets) | Signal contract sign-off and register effect |
| Integration with the incident-response tooling (PagerDuty, Opsgenie) at the plumbing level | Routing decisions (which audience gets what, per severity) |
| Data retention, storage, and access controls at the pipeline level | Evidence-pointer taxonomy and register-side hashing |

Two shared surfaces:

- **The monitor identifier.** MLOps owns the pipeline; risk owns the identifier that the signal contract cites. Neither can change the identifier without notifying the other.
- **The pipeline SLO.** MLOps owns the SLO; risk consumes SLO breaches as a control-availability signal (the pipeline going dark on a control that credits residual is itself a register-side event).

Cite the boundary in every affected signal contract row with `signal_source_owner` and `deferral_boundary` fields (chapter 01's schema).

## The peer boundary to `ai-eval-engineer` at L30

The eval-engineer track owns the *application-layer online evaluation loop*. Concretely:

| Owned by `ai-eval-engineer` at L30 | Owned by `ai-risk-engineer` at L25 |
|---|---|
| Session-level task evaluation methodology | The risk-side effect of task-evaluation outputs |
| LLM-as-judge harnesses, prompt design for judge, judge calibration loop | Golden-set alignment to harm-model coverage |
| Continuous quality experiments, A-B / bandit / eval infrastructure | Register effect of experimentation outcomes |
| Human-in-loop labelling protocol, inter-rater reliability, adjudication | False-positive review calibration loop for guardrail signals |
| Longitudinal user-outcome tracking | Signal contract rows that consume eval outputs |
| Adaptive-attacker / red-team-in-the-loop feedback | Register effect of red-team-in-the-loop findings |

Two shared surfaces:

- **The eval output.** Eval owns the harness; risk owns the signal contract row that consumes the harness's output.
- **The labelling calibration.** Eval owns the protocol depth; risk owns the wire from the calibrated labels back into the near-miss and detector-firing corrections in the register.

Cite the boundary in every affected signal contract row.

## The upstream wire from mod-108 (guardrails)

Every guardrail from mod-108 has three properties that this module reads:

- A stable identifier and a version.
- A stable claim about the register-row identifiers it credits (mod-108 chapter 06).
- A defined event schema for its fires (evaluated, flagged, blocked, severity).

Chapter 03 of this module turned those properties into signal contract rows. The wire runs one direction: guardrail change -> signal contract review -> register update. The reverse — the register asking a guardrail to change — is a mod-108 conversation, not a mod-109 one.

## The downstream wire to mod-111 (incident response)

The wire *out* of this module runs through the incident-candidate queue (chapter 05). Every S1 signal trip, every near-miss threshold breach, and every human-created candidate lands in the queue. Mod-111 owns the triage, containment, root-cause analysis, and post-mortem.

Signals flow back the other way too: a post-mortem often names an under-scored register row or an unmodelled signal. Mod-111 chapter (post-mortem) is the trigger to update the signal contract and the register — an incident-driven event on mod-103 chapter 06's event list.

## The downstream wire to mod-112 (governance council)

The governance-council digest (mod-112) is a reduction over the register and the tripped-signal records; chapter 05 named its content. The wire runs one direction from this module: the digest reads; the council decides; council decisions may drive changes back into the signal contract (raise a threshold, retire a signal, add a new signal). Those council-driven changes are governed by the same sign-off routing as any other contract change (chapter 01).

## Deferral contract — what this module does *not* own

A short list of what belongs outside this module's scope, with the correct owner:

- **The monitoring pipeline itself** (MLOps) — sensors, storage, dispatch. Peer L25 `ai-infra-mlops`.
- **The application-layer online eval loop** (Eval) — session-level scoring, judge calibration, longitudinal user outcomes. Peer L30 `ai-eval-engineer`.
- **The GRC platform selection** — Credo AI vs. Holistic AI vs. ModelOp vs. Monitaur. L50/L60 (`senior-ai-governance-architect`, `head-of-ai-governance`).
- **The commercial monitoring platform selection** — Fiddler vs. Arthur vs. WhyLabs. Above this role.
- **The incident response depth** — triage protocol, containment authority, post-mortem process. Mod-111 within this track.
- **The board / regulator packet composition** — external report authorship and delivery. Mod-112 within this track; and, for external submissions, the provider legal / compliance function.
- **Adversarial / frontier-agent monitoring depth** — beyond level 25. `agentic-safety-engineer` at L40 and above.

## A worked deferral clause for a signal contract row

To close the module, a worked example of the boundary fields for a single signal contract row (`SC-CS-INJ-01` from chapter 03):

```yaml
contract_id: SC-CS-INJ-01
register_row_id: HR-CS-INJ-01
register_field: [control_status, monitor_last_fired]
monitor_id: guardrail-injection-firing-rate

signal_source_owner: ai-infra-mlops-L25
signal_source_owner_person: <named individual>

deferral_boundary:
  - track: ai-infra-mlops (peer L25)
    scope: monitor pipeline plumbing, storage, SLOs, backfill; guardrail
      firing-event schema is joint with mod-108
    contact_ref: <team pointer>
  - track: ai-eval-engineer (peer L30)
    scope: LLM-as-judge calibration for the false-positive review loop that
      feeds the near-miss adjustment; the harness itself
    contact_ref: <team pointer>
  - track: mod-111 incident response (within ai-risk-engineer)
    scope: triage, containment, post-mortem when S1 trips are promoted to
      incidents
    contact_ref: <team pointer>

commercial_platform_notes:
  - platform: Arthur Shield (if adopted)
    scope: bundles the guardrail + monitoring surface; if adopted, the monitor
      identifier is the Shield identifier and this contract row cites it
```

Writing the deferral block on every row is the discipline that stops the ownership fog described at the start of the chapter.

## Interfaces with adjacent modules

- **mod-101 ch 06** — the role-scope / deferral-contract chapter this one closes the mod-109 specialisation of.
- **mod-102 ch 06** — the intake-shape and frontier-agent handoff at L40; the same shape applies here for the monitoring surface.
- **mod-103 ch 06** — GRC platform boundaries; commercial platforms named here overlap with Credo AI / Holistic AI / ModelOp / Monitaur named there.
- **mod-108 ch 06** — guardrail effectiveness measurement; the wire from guardrail fire to signal contract row.
- **mod-111** — the incident-candidate queue's downstream owner.
- **mod-112** — the governance-council digest's downstream owner.

## Summary

- Fiddler AI, Arthur AI, WhyLabs, and Holistic AI cover different corners of the observability / runtime monitoring / GRC space; the risk engineer's ownership is the signal contract and the register regardless of stack.
- `ai-infra-mlops` (peer L25) owns the monitoring pipeline itself; `ai-eval-engineer` (peer L30) owns the application-layer online eval loop; both boundaries are cited in every affected signal contract row.
- The upstream wire is mod-108 (guardrails); the downstream wires are mod-111 (incident response, via the incident-candidate queue) and mod-112 (governance council, via the digest).
- What this module does *not* own is a specific, short list — write it down on every row so ownership fog does not creep in.
- Exercises 01-05 have built the contract, wired the signals, produced the report, and set up the routing. The module ends here; mod-110 (MRM and supply chain) opens the next module.
