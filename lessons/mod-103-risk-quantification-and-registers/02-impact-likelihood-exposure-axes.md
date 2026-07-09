# The Scoring Axes: Impact × Likelihood × Exposure

Chapter 01 said each row carries three states. This chapter says the score itself is a product of three axes — impact, likelihood, and exposure — each defined by a rubric that a reviewer with no prior context can apply and get roughly the same number. This is where "risk quantification" stops being a phrase and starts being a repeatable operation.

## Motivation

Two failure modes plague first-pass AI risk scoring:

- **Single-axis conflation.** A "5-point severity" score conflates impact with likelihood — a common-and-mild scenario and a rare-and-catastrophic scenario end up with the same rating and neither is treated appropriately.
- **Rubric-free scoring.** The row says impact = High but does not define what High means. Two reviewers score the same scenario at 3 and 5. The register becomes a Rorschach test.

The engineering fix is the same one that works everywhere: separate the axes, write the rubric down, and make the rubric quote-able. The AI-specific piece is the *exposure* axis, which classical risk registers usually collapse into likelihood but which for AI systems is materially different and cheaper to control.

## The three axes

| Axis | What it measures | Why it must be separated from the others |
|---|---|---|
| **Impact** | The severity of harm *if the scenario occurs once* — magnitude, reversibility, breadth per instance, sensitivity of affected parties | Impact is what determines whether the scenario belongs in the register at all; you cannot buy it down with an operational control |
| **Likelihood** | The probability the scenario occurs *given the current attack surface and threat model* over a defined observation window | Likelihood is what red-team results, incident-corpus base rates, and adversarial-ML robustness studies actually calibrate |
| **Exposure** | The scale of exposure — how many users, how much data, how many downstream consumers, for how long, in which regulated contexts — the scenario reaches when it does occur | Exposure is what most operational controls actually change (rate limits, canary populations, tier restrictions), and separating it lets you measure that lever |

The row's combined score is a function of these three (typically a product or an ordinal combination — see below). Each axis is scored using its own rubric, and the row cites the rubric level used.

## Impact — the severity of one occurrence

Impact scores what happens *when the scenario succeeds once*. Not "how often." Not "against how many." Just: how bad is one instance.

An impact rubric adequate for a level-25 risk engineer covers at least the following sub-axes and rolls them into an ordinal level:

| Sub-axis | What it captures |
|---|---|
| Magnitude | Financial loss, service disruption, physical injury, dignitary harm, discrimination outcome, safety incident severity |
| Reversibility | Can the harm be undone (retract a bad answer) or is it permanent (a decision was auto-executed, funds transferred, medication administered, content published) |
| Sensitivity of party | Is the affected party a data subject in a protected class, a minor, a patient, a regulated counterparty, a whole community |
| Breadth per instance | Even one instance can hit a single person or thousands (a single mass email, a single bulk decision batch) |
| Regulatory / reputational amplification | Does the instance trigger mandatory reporting, licensure consequences, or public disclosure |

A working five-point impact rubric:

| Level | Working definition |
|---|---|
| Negligible | Single occurrence is annoying, no lasting effect, no reporting duty |
| Minor | Single occurrence causes limited financial or service impact to one party; reversible; no reporting duty |
| Moderate | Single occurrence causes non-trivial harm to one party, or annoying harm to many; potentially reversible with effort; disclosure to the affected party expected |
| Major | Single occurrence causes serious harm — significant financial, discrimination outcome that shapes access, safety near-miss, sensitive data exposed; some occurrences are hard to reverse; mandatory reporting is possible |
| Severe | Single occurrence causes catastrophic, irreversible, or systemic harm — physical injury, denial of essential service to a protected class, loss of life, systemic financial impact; mandatory disclosure to a regulator is expected |

The rubric is *your* rubric — organisations differ. What matters is that the rubric is written, versioned, and cited on the row. Impact-level 3 means the row author says the scenario matches the "Moderate" language above, and a reviewer can look at that language and disagree with an argument.

## Likelihood — the probability of occurrence

Likelihood scores how often the scenario is expected to occur over a defined observation window. Two mistakes drown likelihood scoring:

- **No window.** A likelihood of "medium" over what? Per session? Per month? Per model release? Pick a window at the register level and hold every row to it.
- **No prior.** A likelihood of "low" *because it feels low*. The rubric should point at the actual priors an AI risk engineer has access to.

For a level-25 program, the priors that matter are:

- **Incident-corpus base rates** — how often does a scenario in this class actually get reported in AIID, OECD.AI's AI Incidents Monitor, MITRE ATLAS case studies, or the MIT AI Risk Repository (mod-102 ch 02)? This calibrates the "class of attack exists in the wild" prior.
- **Red-team and adversarial-ML attack-success rates** — for the classes of scenario that can be attacked in a lab, the attack-success-rate from mod-104 and mod-105 gives you a signal that is much better than a guess.
- **Guardrail effectiveness numbers** — the residual-scoring version of likelihood should credit guardrail effectiveness measured in mod-108.
- **Deployment shape** — a scenario is more likely against a public-facing system than an internal one, more likely at high volume than at low.

A working five-point likelihood rubric over an *annualised* observation window (adjust window to your organisation):

| Level | Working definition |
|---|---|
| Rare | Not observed in comparable systems; class of attack has one or zero documented reports; base rate essentially unknown |
| Unlikely | Observed in comparable systems but well below one occurrence per system per year |
| Possible | Comparable systems report ≈ 1 occurrence per system per year, or class of attack is documented and technically feasible against this system |
| Likely | Comparable systems report multiple occurrences per year, red-team consistently reproduces the scenario, attack-success-rate above a target |
| Almost certain | The scenario has been observed in *this* system, or its base rate in comparable systems is high enough to expect regular occurrence |

Each level should cite what evidence it is calibrated against — the incident corpus row, the red-team run identifier, the guardrail effectiveness number. "Likely" without a citation is a guess.

<!-- needs-research: cite specific published base-rate ranges from AIID / OECD.AI AIM once verified against the current corpus size and category counts — do not invent numeric ranges here. -->

### The residual-and-inherent split on likelihood

Chapter 01 said inherent, residual, and control-defeated share a rubric and differ in what is credited. Likelihood is the axis where that split shows up most strongly:

- Inherent likelihood: the probability *without* preventive controls — governed by the incident corpus base rate and the raw attack-success rate.
- Residual likelihood: the probability *with* the credited controls — the inherent likelihood modified by measured guardrail effectiveness and by monitoring / oversight that would catch the scenario before it lands.
- Control-defeated likelihood: for each named control, the residual likelihood recomputed as if that control's effectiveness were zero.

Impact typically does not change across the three states (the same event, whether prevented or not, produces the same harm when it lands). Exposure sometimes changes (a rate-limit control affects exposure, not likelihood). Likelihood changes in every state.

## Exposure — the scale reached when the scenario occurs

Exposure scores *how much of the system's operating surface* is reached by a single occurrence of the scenario. It is the axis classical IT risk usually collapses into likelihood; AI risk splits it out for three reasons:

- **Operational controls act on exposure, not on impact.** A canary population, a rollout ring, a tier restriction to internal users, a per-account rate limit — these do not reduce the impact per event and often do not reduce the likelihood per event; they cap the exposure per event.
- **Aggregation across systems needs an exposure axis.** Portfolio roll-ups (chapter 05) sum exposure to argue about *portfolio blast radius*. If exposure is hidden inside likelihood, the roll-up cannot say anything about how many users the portfolio's controls collectively cover.
- **Sector overlays care about exposure.** SR 11-7 materiality tiering, EU AI Act "significant harm" thresholds, and healthcare "patient population affected" reporting are all exposure-shaped questions.

A working five-point exposure rubric:

| Level | Working definition |
|---|---|
| Contained | Scenario reaches a small named test population (developers, a canary cohort); no external users; no regulated jurisdictions in scope |
| Limited | Scenario reaches a defined internal population, or a restricted external cohort (e.g., pilot customers); some regulated jurisdictions in scope but limited |
| Moderate | Scenario reaches a substantial external population — a single business line's customers, one region — including regulated jurisdictions |
| Broad | Scenario reaches the general external population — multi-region, multi-line-of-business, all jurisdictions where the system is deployed |
| Systemic | Scenario reaches multiple products, downstream integrators / partners, or the general public through amplification (media, virality, mass integration) |

Exposure is where sector overlays land. Systems under SR 11-7 will tie exposure to model-materiality tiering (mod-110); systems under the EU AI Act will tie exposure to whether the deployment is "high-risk" (mod-101 ch 04); systems under FDA GMLP will tie exposure to the patient population affected. Cite the sector definition in your rubric so the exposure column is directly attributable to the sector's language.

## Combining the axes

Two respectable ways to combine, each with tradeoffs:

### Ordinal matrix (5×5×5 or 3×3×3)

Score each axis on the same ordinal scale; combine using a matrix that maps `(impact, likelihood, exposure)` triples to a combined ordinal (Very Low / Low / Medium / High / Very High or 1..5). This is the shape ISO 31000 / 27005 organisations use and it composes cleanly with an enterprise ERM heat map.

Advantages: interpretable, uses the same ordinal language across the ERM; the risk council can read it without a legend.

Disadvantages: ordinal multiplication is *not* multiplication — a Very-High × Very-High × Contained can end up in the same "Medium" cell as a Moderate × Moderate × Moderate, and the two rows do not have the same treatment profile. Mitigate by publishing the matrix and explaining every cell.

### Anchored numeric

Score each axis on a numeric anchor (e.g., impact = expected loss in currency, likelihood = probability per annum, exposure = number of user-sessions per annum). Combine as `impact × likelihood × exposure` for an annualised expected-loss-shaped number.

Advantages: composes with insurance and financial-risk practices; supports cardinal reasoning across a portfolio.

Disadvantages: anchoring is expensive (you need real currency-denominated impact estimates); statistical uncertainty on likelihood becomes very visible (chapter 07 hands the calibration methodology to the model-evaluation-engineer at L30); false precision is a real risk.

The choice depends on your ERM's language. Most enterprises at level-25 maturity run the ordinal matrix and publish it; the anchored numeric shape shows up in finance and insurance. **Whichever you pick, publish the matrix or formula, and cite it on every row.**

<!-- needs-research: reference a specific published enterprise ERM matrix example (COSO ERM or a bank supervisor guidance) to anchor the ordinal-matrix style, once verified against a primary source. -->

## Anchoring the numbers — evidence per axis

A quality gate to apply to every row before publishing: each axis level cites what it is anchored on.

| Axis | Anchor evidence types |
|---|---|
| Impact | Regulatory language ("significant risk of harm"), sector loss studies, incident post-mortem financial estimates, physical-injury classification, dignitary-harm precedent |
| Likelihood | Incident-corpus base rate, red-team ASR, adversarial-ML robustness study, guardrail-effectiveness measurement, prior in-system observation |
| Exposure | System inventory (user counts, jurisdictions), deployment ring definition, sector materiality tier, downstream-integrator map |

If a row scores an axis and cites no anchor, the row is a draft. Chapter 05's aggregation reports run this quality gate as a filter and drop uncalibrated rows from the roll-up — otherwise the portfolio number becomes noise.

## An end-to-end scored row

Continuing HR-CS-INJ-01 (customer-support RAG, indirect prompt injection to `escalate_ticket`):

```yaml
row_id: HR-CS-INJ-01
scoring:
  rubric_version: v1.3-cs-rag-2026-q2
  window_annualised_days: 365
inherent:
  impact:
    level: 4  # Major — sensitive data exposure, mandatory reporting possible
    anchor: "operator-side loss study for account-identifier exfiltration; GDPR Art. 33 reporting duty"
  likelihood:
    level: 4  # Likely — attack class widely documented, ASR without controls high
    anchor: "OWASP LLM Top 10 LLM01; Greshake et al. indirect-prompt-injection paper"
  exposure:
    level: 4  # Broad — public-facing customer support with multi-region deployment
    anchor: "system inventory record inv-2025-014; multi-region deployment"
  combined: 5  # Very High per matrix v1.3
residual:
  credited_controls:
    - id: CTL-CS-RAG-PROV-01
      evidence: "provenance-filter FN-rate 0.02 on retrieved chunks — mod-108 report"
    - id: CTL-CS-RAG-ALLOWLIST-01
      evidence: "escalate_ticket allow-list config hash abc123 pinned in release"
    - id: CTL-CS-RAG-MON-01
      evidence: "destination-anomaly monitor firing dashboard link"
  impact:
    level: 4  # unchanged
  likelihood:
    level: 2  # Unlikely with credited controls
    anchor: "measured filter FN-rate + allow-list restricts destinations"
  exposure:
    level: 4  # unchanged
  combined: 3  # Medium per matrix v1.3
control_defeated:
  - control_id: CTL-CS-RAG-PROV-01
    likelihood_level: 3  # Possible
    combined: 4  # High
    monitor_ref: MON-CS-RAG-FN-01
  - control_id: CTL-CS-RAG-ALLOWLIST-01
    likelihood_level: 4  # Likely
    combined: 5  # Very High
    monitor_ref: MON-CS-RAG-ALLOWLIST-01
```

That row can be aggregated (chapter 05), tied to appetite bands (chapter 05), monitored (mod-109), and rehearsed against incident-response thresholds (mod-111). It reads defensibly because every score cites an anchor, and the three states are visible.

## Anti-patterns to watch for

- **Impact-likelihood inversion.** A "high-likelihood, low-impact" scenario is written up as "medium impact" because the scorer wants it to move up the priority list. If impact means "one instance," it means one instance — the volume argument belongs in exposure or likelihood.
- **Exposure hidden inside likelihood.** A rate-limited scenario is scored as "unlikely" because the rate limit is credited on likelihood. It should be credited on exposure, and the row's exposure state should track through inherent / residual / control-defeated distinctly from likelihood.
- **Rubric drift.** The rubric is edited without a version bump, and old rows are still cited against an obsolete rubric. Version the rubric like code.
- **No-window likelihood.** A likelihood level with no observation window. Pick a window and stick to it across the register.
- **Anchor-free scoring.** An axis level with no cited anchor. Filter these out of aggregation reports so they cannot pollute the portfolio number.

## Summary

- Every row scores three axes: impact (severity of one occurrence), likelihood (probability over a stated window), exposure (scale reached per occurrence).
- Split them because they behave differently under control — impact resists it, likelihood responds to prevention, exposure responds to operational controls.
- Each axis is scored on a written, versioned rubric; the row cites the level and the anchor.
- Combine via a published ordinal matrix (most common) or an anchored numeric formula; whichever you pick, publish it.
- The inherent / residual / control-defeated states from chapter 01 share the rubric and differ only in which controls are credited.
- Exercise 02 asks you to author the scoring scheme, publish the matrix, and demonstrate it on real rows.
