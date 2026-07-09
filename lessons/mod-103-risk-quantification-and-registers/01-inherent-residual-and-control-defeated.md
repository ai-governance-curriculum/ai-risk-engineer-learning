# Inherent, Residual, and Control-Defeated: The Three States Every AI Risk Row Carries

A risk register that carries a single score per row cannot be defended under audit. The register has to carry **three** states of the same underlying scenario — inherent, residual, and control-defeated — and the relationships between them have to be visible. This chapter defines the three states, says which one is used for which conversation, and pins the audit contract that makes the numbers stick.

## Motivation

An auditor, a risk council, and a regulator ask three different questions about the same harm scenario:

- The auditor wants to know how big the risk would be *if the controls you claim were not there*. That is the inherent question. It tests whether your control catalogue is doing the work you say it is.
- The risk council wants to know how big the risk is *right now, with the controls that are actually running and working*. That is the residual question. It sets treatment priority and appetite comparisons.
- The regulator, incident responder, and (again) the auditor want to know how big the risk becomes *when a control fails or is bypassed*. That is the control-defeated question. It sets your escalation thresholds, monitoring priorities, and post-incident learning loops.

A register that answers only the middle question ("what's residual now?") looks fine in a quarterly report and disintegrates the first time a control fails. A shippable register carries all three states per row, and the delta between them is where the argument lives.

## The three states

Every risk scenario row in a mod-103 register carries at least these three quantifications. The definitions below match ISO 31000 / ISO/IEC 27005 usage of *inherent* and *residual* risk and extend that pair with the control-defeated column that AI systems need because their controls degrade differently than classical IT controls.

| State | What it scores | When it is used |
|---|---|---|
| **Inherent** | The scenario's risk *before any risk treatment is applied* — no filters, no guardrails, no human oversight, no monitors | Audit defensibility ("what would the number be if we had none of what we claim to have?"); prioritising control investment |
| **Residual** | The scenario's risk *after all currently-operating controls are considered working as designed* | Steady-state risk-appetite comparison; register-level reporting; portfolio aggregation |
| **Control-defeated** | The scenario's risk *when a named control fails, is bypassed, or is degraded* — one control at a time, worst-case per control | Monitoring priority; incident-response thresholds; sign-off routing when a control is disabled or in maintenance |

The three columns share a single scoring rubric (chapter 02). What changes is what is *assumed present*.

### Inherent — the "no controls" score

Inherent risk answers "how bad is this if we did nothing?" It is a hypothetical — no organisation runs without controls — but the hypothetical is essential. Without it, you cannot show *what your controls buy you*.

Two disciplines around inherent risk that keep it defensible:

- **The "no controls" list is written down, not implied.** You state which controls you are counting as *absent* for the inherent number: input filters off, output validators off, human oversight off, monitor thresholds infinite, guardrail models absent, rate limits absent. If a reviewer disagrees, they can point at the list.
- **Inherent is not "raw model behaviour."** It is the risk to the *system in operation with no risk-treatment controls*. So user reach, data sensitivity, and integration surface are still what they are — you do not pretend the system does not exist. What you strip is the *risk treatment*.

Inherent scores rarely change once set. When they do change, it is because the system itself changed — new capability, new integration, new user population — not because you improved a control. Improving a control moves the residual number, not the inherent number.

### Residual — the "controls working" score

Residual risk answers "how bad is this now, on a normal day?" It assumes every control listed in the row's control set is present and operating as designed. It is the number that shows up in most conversations — appetite comparison, portfolio roll-up, monthly reporting.

Two disciplines around residual risk that keep it honest:

- **Every control credited in residual has evidence it is actually running.** A filter that exists in staging but is off in production is not credited. A monitor whose alert channel has been muted for two quarters is not credited. Credit requires a live evidence pointer — a test that passes on release, a monitor whose last-24h dashboard has data.
- **Residual is the score under design assumptions, not under worst-case degradation.** The delta between residual-as-designed and control-defeated is where the risk moves when a control breaks. That is a first-order artifact, not a footnote.

Residual is what gets aggregated in the portfolio report (chapter 05). If your register has one number, this is usually the one — which is why the *other two* are so important to keep in the row alongside it.

### Control-defeated — the "one control at a time" score

Control-defeated risk answers "how bad is this when one specific control fails?" Not all controls at once (that collapses to inherent). One control at a time, worst-case per control.

This state exists because AI-system controls degrade in ways classical IT controls do not:

- Prompt-injection filters are bypassed by novel attack patterns while the filter still returns 200 OK.
- A safety classifier drifts on a new distribution while the classifier's uptime is 100%.
- A retrieval provenance check is turned off for a debug session and never turned back on.
- A rate limiter is disabled for a known enterprise customer and the disabled setting persists across a redeploy.
- A guardrail model is replaced with a smaller model to reduce cost and the swap slips through change management.

Each of those is a *silent* control-defeat. The inherent score does not capture it (inherent assumes no controls). The residual score does not capture it either (residual assumes all controls working). The control-defeated score does — one row per named control.

Two disciplines around control-defeated scores:

- **One score per control that materially bounds the residual number.** If the row credits three controls that each carry a lot of the mitigation, you produce three control-defeated variants. Controls whose absence would not meaningfully change the number can be aggregated to one "minor controls" line.
- **Bypass modes are enumerated, not implied.** "Bypassed" means named modes: filter tuned to a lower sensitivity, monitor mute, disabled config, replaced model, alerting muted, human reviewer at capacity. Each mode gets a row.

The control-defeated column is what drives the *monitor* selection at mod-109 and the *incident response threshold* at mod-111. If a control's defeat would push residual over appetite, that control must have a monitor whose alert cadence catches the defeat within the tolerable window. If the row cannot answer "which monitor catches this?", the monitoring plan is incomplete.

## The two deltas that matter

Between the three states, two deltas carry the audit argument:

| Delta | What it argues |
|---|---|
| Inherent − Residual | How much risk reduction the current control set is buying you. This is the ROI of your control catalogue. If it is small, the controls are ineffective; if it is huge, you had better be able to evidence every credited control. |
| Control-defeated − Residual | The blast radius of a single control failing. This is what the monitor and incident-response threshold must be sized against. If it is large for a control, that control's monitoring is disproportionately important. |

Both deltas are traceable numbers. In a good register, both are visible on the row without an analyst having to compute them. Portfolio reports (chapter 05) roll both deltas up, not just the residual number.

## A worked example — one row across three states

Continuing HR-CS-INJ-01 from mod-102 (customer-support RAG assistant, indirect prompt injection through a retrieved KB article that triggers `escalate_ticket` to an attacker-controlled destination). Scoring rubric details are in chapter 02; the pattern here is what a shippable row looks like across the three states.

| Field | Inherent | Residual | Control-defeated: retrieval-provenance filter bypassed | Control-defeated: tool-call allow-list disabled |
|---|---|---|---|---|
| Impact score | High | High | High | Very high |
| Likelihood score | High (attack is documented, indirect prompt injection is a class of attack widely reported in AIID and OWASP LLM Top 10 LLM01) | Low (provenance filter drops most injection payloads; tool-call allow-list restricts destinations) | Medium (allow-list still restricts destinations even if the filter fails) | High (with no allow-list, injection can call `escalate_ticket` to arbitrary destinations) |
| Exposure score | High (all customer sessions) | High | High | High |
| Combined score | Very high | Medium | High | Very high |
| Credited controls | none | provenance filter on retrieved chunks; tool-call allow-list on `escalate_ticket`; per-session rate limiter; monitor on `escalate_ticket` destinations | (provenance filter absent) | (allow-list absent) |
| Monitor that catches the defeat | n/a | n/a | filter false-negative-rate monitor; injection-signature match on retrieved chunks | change-log diff on tool-call allow-list; anomaly on new `escalate_ticket` destinations |

The inherent number justifies why the controls exist. The residual number is what the risk council reads. The two control-defeated numbers are what the monitoring plan and incident-response playbook must be sized against. And the deltas — how much the controls buy, and how much a single control's failure costs — are the argument.

## Calibrating against enterprise risk appetite

Risk appetite is set at the program level (mod-101 ch 06, the risk council and higher). Appetite is stated as *bands* on the same scoring axis your row scores use. A shippable register lays each state against the appetite band and answers, per row, three yes/no questions:

- Is the residual score inside the appetite band? *(if no, the row needs a treatment plan)*
- Is any control-defeated score outside the appetite band? *(if yes, monitoring and incident-response thresholds must catch the defeat inside the tolerable window)*
- Would the inherent score be outside the appetite band even if you credited every proposed control? *(if yes, the treatment plan cannot land you in appetite — either escalate to the risk council for tolerance, or refuse the deployment)*

Chapter 05 walks the appetite calibration in detail. The point at this chapter is: without the three states, none of the three questions can be answered.

## Anti-patterns

Four anti-patterns that show up in first-pass registers:

- **One score per row.** A single "risk score" collapses all three states into one number. The auditor asks "with or without controls?" and there is no clean answer.
- **Residual scored by feel.** The residual number was assigned without an enumerated control list. The auditor asks "which controls did you credit?" and the answer is a paragraph of prose.
- **Inherent scored as raw-model behaviour.** The inherent column shows what the raw model does with no system around it. But there is a system around it — data, integration, deployment — that shapes the risk. Inherent is *system-with-no-treatment*, not *model-with-nothing*.
- **Control-defeated ignored.** The register carries inherent and residual; when a control fails in production, the row's residual number is no longer valid and the register does not know it. Sizing the monitor to catch the defeat is the discipline that keeps the register live.

## Interfaces with adjacent modules

- **Mod-102 (harm modelling)** feeds each scenario row in as the *scenario* being scored. If a mod-103 row exists with no matching mod-102 harm-model row, either the harm model needs an update or the register row is out-of-scope.
- **Mod-108 (guardrail engineering)** produces the *controls* whose presence collapses inherent to residual and whose absence produces the control-defeated variants. The control catalogue lives there; this chapter cites it.
- **Mod-109 (continuous monitoring)** must catch a control defeat within the tolerable window. The control-defeated column here drives monitor selection there.
- **Mod-111 (incident response)** reads the control-defeated column when routing an incident — the same table that drives the monitor drives the runbook branch.

## Summary

- Every register row carries three states: inherent (no controls credited), residual (all controls credited as working), control-defeated (one named control failing).
- The two deltas — inherent−residual and control-defeated−residual — carry the audit argument.
- Inherent is *system-with-no-treatment*, not *raw model output*. Residual credits only controls with live evidence. Control-defeated is scored one control at a time.
- Appetite calibration asks three questions per row; without the three states, none of them can be answered.
- Anti-patterns: single-score rows, feel-scored residual, raw-model-only inherent, missing control-defeated.
- Exercise 01 drills the three states on rows carried forward from mod-102.
