# Lab 02 — Per-Axis Rubric-Level-and-Anchor Drill

**Duration:** 45 minutes · **Format:** guided practice, markdown authoring

## Objective

Internalise the chapter-02 discipline of scoring an axis *with* a cited rubric level and a cited anchor evidence type. You will score three axes (impact, likelihood, exposure) on one fixed scenario on a fixed system, picking a level, quoting the rubric-level language your level matches, and naming the specific anchor evidence that justifies the level. The lab drills the atomic skill — one axis, one level, one rubric quote, one anchor — that exercise 02 then asks you to perform across a complete rubric and a complete three-row re-scoring.

## Prerequisites

- Chapter 02 read. Chapter 01 read is helpful but not required for this lab (we score only residual here).
- A terminal where you can create a working directory and author markdown files.
- No external access required; the rubric, the scenario, and the anchor evidence types are all fixed below.

## The fixed working rubric (use verbatim for this lab)

Use the chapter-02 working rubric. Do *not* substitute your own — the lab checkpoints assume these exact tables. Exercise 02 is where you author a system-specific rubric; the lab drills reading one.

**Impact rubric (5 levels, severity of one occurrence):**

| Level | Working definition |
|---|---|
| 1 — Negligible | Single occurrence is annoying, no lasting effect, no reporting duty |
| 2 — Minor | Single occurrence causes limited financial or service impact to one party; reversible; no reporting duty |
| 3 — Moderate | Single occurrence causes non-trivial harm to one party, or annoying harm to many; potentially reversible with effort; disclosure to the affected party expected |
| 4 — Major | Single occurrence causes serious harm — significant financial, discrimination outcome that shapes access, safety near-miss, sensitive data exposed; some occurrences are hard to reverse; mandatory reporting is possible |
| 5 — Severe | Single occurrence causes catastrophic, irreversible, or systemic harm — physical injury, denial of essential service to a protected class, loss of life, systemic financial impact; mandatory disclosure to a regulator is expected |

**Likelihood rubric (5 levels, annualised observation window):**

| Level | Working definition |
|---|---|
| 1 — Rare | Not observed in comparable systems; class of attack has one or zero documented reports; base rate essentially unknown |
| 2 — Unlikely | Observed in comparable systems but well below one occurrence per system per year |
| 3 — Possible | Comparable systems report ≈ 1 occurrence per system per year, or class of attack is documented and technically feasible against this system |
| 4 — Likely | Comparable systems report multiple occurrences per year, red-team consistently reproduces the scenario, attack-success-rate above a target |
| 5 — Almost certain | The scenario has been observed in *this* system, or its base rate in comparable systems is high enough to expect regular occurrence |

**Exposure rubric (5 levels, scale reached per occurrence):**

| Level | Working definition |
|---|---|
| 1 — Contained | Scenario reaches a small named test population (developers, a canary cohort); no external users; no regulated jurisdictions in scope |
| 2 — Limited | Scenario reaches a defined internal population, or a restricted external cohort (e.g., pilot customers); some regulated jurisdictions in scope but limited |
| 3 — Moderate | Scenario reaches a substantial external population — a single business line's customers, one region — including regulated jurisdictions |
| 4 — Broad | Scenario reaches the general external population — multi-region, multi-line-of-business, all jurisdictions where the system is deployed |
| 5 — Systemic | Scenario reaches multiple products, downstream integrators / partners, or the general public through amplification (media, virality, mass integration) |

**Anchor-evidence types (one per axis — chapter 02 table):**

| Axis | Anchor evidence types you may cite |
|---|---|
| Impact | Regulatory language ("significant risk of harm"), sector loss studies, incident post-mortem financial estimates, physical-injury classification, dignitary-harm precedent |
| Likelihood | Incident-corpus base rate (AIID / OECD.AI AIM), red-team ASR, adversarial-ML robustness study, guardrail-effectiveness measurement, prior in-system observation |
| Exposure | System inventory (user counts, jurisdictions), deployment ring definition, sector materiality tier, downstream-integrator map |

## The fixed scenario

Use this scenario — do not substitute. It is `HR-PP-RTV-01` carried forward from mod-102 lab 01 and mod-103 lab 01.

> **`HR-PP-RTV-01` on PolicyPal — indirect prompt injection via HR handbook content.**
> An HR editor pastes handbook content from a public help thread that carries a hidden instruction; the paragraph is indexed into PolicyPal's RAG store. In a later legitimate employee session, the retrieved chunk causes PolicyPal to execute the attacker's instruction, producing an answer outside the policy envelope (prior-turn context emitted, or compliance with attacker intent in place of the user's).

Score the row in its **residual state**, with the following four controls credited (same control set as lab 01):

- `CTL-PP-PROV-01` — retrieval provenance filter (stipulated residual false-negative rate ≈ 0.03 with 95% CI roughly 0.02–0.05 — for the lab, this is given and does not need to be computed)
- `CTL-PP-OUTVAL-01` — output validator
- `CTL-PP-RATE-01` — per-employee rate limiter (10 questions / 15 minutes)
- `CTL-PP-MON-01` — retrieval-anomaly monitor

Stipulated fixed facts for the lab:

- The hosted LLM provider is covered under a data-processing agreement; the RAG store contains no personal data by design.
- PolicyPal is deployed only in the company's single home jurisdiction (call it the EU for the lab, so GDPR applies).
- PolicyPal reaches the company's ~8,000 employees via the intranet.
- Indirect prompt injection via retrieved content is documented in OWASP LLM Top 10 (LLM01) and in Greshake et al. (2023, arXiv:2302.12173).
- The guardrail-effectiveness number above is the only one available for this lab; no bootstrap CI exercise is required.

## Steps

### Step 1 — Create your working directory and the template

```bash
mkdir -p mod103-lab02 && cd mod103-lab02
touch axis-levels-HR-PP-RTV-01.md
```

Open `axis-levels-HR-PP-RTV-01.md` and paste the following template.

```markdown
# HR-PP-RTV-01 — residual axis scoring (lab 02)

System: PolicyPal (internal HR-policy Q&A RAG assistant; see lab brief)
Row: HR-PP-RTV-01 — indirect prompt injection via HR handbook content
State scored here: RESIDUAL (all four credited controls present)
Rubric cited: chapter 02 working rubric (impact, likelihood annualised, exposure)
Observation window for likelihood: annualised

## Impact
- Level: <1..5>
- Rubric language I match (quoted from the chapter-02 table): "<paste exact phrase>"
- Anchor type (from chapter 02's anchor table): <type>
- Anchor evidence (specific): <one sentence; what the evidence is, where it would live>
- Argument (one sentence): <why this level, not the one above or below>

## Likelihood
- Level: <1..5>
- Rubric language I match: "<paste exact phrase>"
- Anchor type: <type>
- Anchor evidence (specific): <one sentence; cite AIID / Greshake / OWASP / guardrail-effectiveness number>
- Argument (one sentence): <why this level, not the one above or below; cite the credited controls that pulled it down>

## Exposure
- Level: <1..5>
- Rubric language I match: "<paste exact phrase>"
- Anchor type: <type>
- Anchor evidence (specific): <one sentence; system inventory or sector overlay>
- Argument (one sentence): <why this level, not the one above or below>

## Combined (under the chapter 02 ordinal-matrix convention)
- Combined level: <1..5>
- Combination rule applied: <one sentence; "max across axes" / "worst two averaged" / etc. Pick one and state it.>

## Anti-pattern self-check
- Did I score impact against "many occurrences" rather than "one occurrence"? If yes, rewrite impact.
- Did I credit a rate-limit-shaped control on likelihood rather than on exposure? If yes, rewrite likelihood and exposure.
- Does any axis have `Anchor evidence: <none>`? If yes, flag `staleness: uncalibrated` (chapter 02's quality gate).
```

### Step 2 — Score Impact

Score impact *once* (chapter 01: impact typically does not change across the three states; one-time landing severity).

Think through:

- *What is the harm per occurrence?* The retrieved chunk causes PolicyPal to execute the attacker's instruction inside a legitimate employee's session. Likely harms: (a) prior-turn context emitted outside policy envelope — potentially containing the employee's own HR question with personal context; (b) compliance with attacker intent in place of the user's — e.g., producing a non-HR response or dumping system-prompt content.
- *Which rubric level does this land under?* The affected party is a single employee per instance. The harm includes sensitive-data-adjacent exposure (the employee's own query context under GDPR is personal data). Reversibility is partial — a wrong answer can be retracted but prior-turn context once emitted is out. Mandatory reporting under GDPR Article 33 is *possible* (not automatic) depending on whether personal data was processed outside the agreed envelope.
- *Where does it sit — 3 (Moderate) or 4 (Major)?* The sensitive-data-exposed language and the mandatory-reporting-possible language in Major fits. Level 4 is defensible; a case for level 3 exists if you argue the prior-turn context is unlikely to contain personal data at scale.

Quote the rubric phrase you match verbatim. Cite an anchor — the GDPR Article 33 reporting-duty language is a fine impact anchor (regulatory language, from chapter 02's anchor-evidence table).

Example of the shape:

```markdown
## Impact
- Level: 4
- Rubric language I match (quoted): "Single occurrence causes serious harm — significant financial, discrimination outcome that shapes access, safety near-miss, sensitive data exposed; some occurrences are hard to reverse; mandatory reporting is possible"
- Anchor type: regulatory language
- Anchor evidence (specific): GDPR Article 33 mandatory breach-notification duty applies when personal data (an employee's prior-turn query context) is processed outside the data-processing agreement's envelope.
- Argument (one sentence): Level 4 over level 3 because the prior-turn context exposure crosses the "sensitive data exposed / mandatory reporting possible" threshold under GDPR; a case for 3 exists if the prior-turn context is argued to rarely contain personal data at scale, but the mandatory-reporting-possible language carries the level.
```

### Step 3 — Score Likelihood

Likelihood is annualised and scored for the residual state (all four controls credited).

Think through:

- *What is the prior?* Indirect prompt injection via retrieved content is documented (OWASP LLM01, Greshake et al.). In a system with *no* provenance filter, comparable RAG assistants have been shown to be regularly vulnerable — that is the inherent-likelihood prior (not what we are scoring here).
- *What do the credited controls buy on residual likelihood?* `CTL-PP-PROV-01` with a stipulated FN rate ≈ 0.03 drops ~97% of un-provenanced chunks. `CTL-PP-OUTVAL-01` catches responses matching known injection-success signatures. `CTL-PP-MON-01` catches novel provenance distributions before persistence. Likelihood falls substantially.
- *Which rubric level?* "Unlikely — observed in comparable systems but well below one occurrence per system per year" fits a FN rate ≈ 0.03 combined with the output validator. Level 2.
- *Anchor:* the guardrail-effectiveness number (0.03 FN rate) is the stipulated anchor. Under chapter 02's anchor types, this is a "guardrail-effectiveness measurement" anchor; chapter 07 says the risk engineer should prefer the *CI upper bound* for the adverse likelihood story — which would mean scoring against 0.05 rather than 0.03. For this lab's level, either picks the same rubric cell (both land in Unlikely).

Example shape:

```markdown
## Likelihood
- Level: 2
- Rubric language I match (quoted): "Observed in comparable systems but well below one occurrence per system per year"
- Anchor type: guardrail-effectiveness measurement
- Anchor evidence (specific): CTL-PP-PROV-01 residual FN rate ≈ 0.03 (95% CI roughly 0.02–0.05, stipulated for the lab); combined with CTL-PP-OUTVAL-01 signature backstop.
- Argument (one sentence): Level 2 over level 3 because at a 0.03 FN rate on the primary preventive control plus the output validator, the attack-success rate projected across 8,000 employees' annual session volume is expected to fall below one occurrence per year; the CI-upper-bound choice (0.05, per chapter 07's adverse-story convention) still lands in the Unlikely cell.
```

### Step 4 — Score Exposure

Exposure is per-occurrence reach. For a single successful injection, how much of PolicyPal's operating surface does it touch?

Think through:

- *Who is reached?* One employee in session per occurrence. The rate limiter (`CTL-PP-RATE-01`) caps repeated hits on the same session but does not change the per-occurrence reach — one occurrence is one session.
- *Jurisdictional reach?* GDPR applies (EU jurisdiction). Single business line (one organisation, internal tool).
- *Which rubric level?* "Limited — defined internal population, some regulated jurisdictions in scope but limited" fits. Level 2. Note the rate limiter affects aggregate exposure across the population (how many occurrences per window the system can experience) but not per-occurrence reach.
- *Anchor:* the system inventory record (naming PolicyPal as internal, ~8,000 employees, EU jurisdiction) is the exposure anchor.

Example shape:

```markdown
## Exposure
- Level: 2
- Rubric language I match (quoted): "Scenario reaches a defined internal population, or a restricted external cohort (e.g., pilot customers); some regulated jurisdictions in scope but limited"
- Anchor type: system inventory
- Anchor evidence (specific): system inventory records PolicyPal as internal-only, ~8,000 employees, single EU jurisdiction.
- Argument (one sentence): Level 2 over level 3 because the population is defined and bounded by the employee roster; GDPR is in scope but the exposure is single-jurisdiction and internal; Level 1 (Contained) would require a test / canary population only, which does not describe production deployment to all employees.
```

### Step 5 — Combine

Pick one combination rule and state it. For this lab the simplest defensible rule is **max across axes with a one-level correction when likelihood is at level 2 or lower**: a Major-impact / Unlikely-likelihood / Limited-exposure profile combines to Medium (3), not Major (4), because the Unlikely likelihood meaningfully pulls the combined score down. If you prefer a strict max rule, Major (4) is also defensible — *as long as you state the rule you applied*.

State the rule in one sentence. Fill the combined cell.

Example shape:

```markdown
## Combined
- Combined level: 3 (Medium)
- Combination rule applied: max across axes with a one-level correction when residual likelihood is level 2 (Unlikely) or lower, consistent with chapter 02's ordinal-matrix convention.
```

### Step 6 — Run the anti-pattern self-check

Walk the three checks at the bottom of the template:

- *Impact scored against "one occurrence"?* Impact at level 4 reads "single occurrence causes serious harm" — matches. If you had scored impact as Severe because "if this happened to all 8,000 employees it would be catastrophic," you would have conflated impact with exposure. Rewrite if so.
- *Rate-limit credit on exposure, not likelihood?* `CTL-PP-RATE-01` reduces aggregate reach, not per-occurrence probability of success; crediting it on likelihood would be the chapter-02 anti-pattern "exposure hidden inside likelihood." Confirm the rate limiter is credited on exposure.
- *Anchor-free axes?* Each axis block should have a non-empty `Anchor evidence` line. If any is empty, flag the row `staleness: uncalibrated` per chapter 02's quality gate — it would be excluded from exercise 04's portfolio aggregation.

### Step 7 — Final checkpoint

Verify:

- Every axis cites a rubric level, a verbatim rubric quote, an anchor type, an anchor-evidence specific pointer, and a one-sentence argument.
- Impact is scored once (not state-dependent).
- Likelihood cites a prior that the chapter-02 anchor table recognises (incident corpus, red-team ASR, guardrail effectiveness, or in-system observation). "It seems low" is not a prior.
- Exposure cites the system inventory or a sector overlay, not likelihood language.
- The combination rule is named explicitly, not inferred.
- No anti-pattern is unaddressed.

## Expected output

`mod103-lab02/axis-levels-HR-PP-RTV-01.md` — three axis blocks (impact / likelihood / exposure), one combined cell, one anti-pattern self-check, roughly one page.

## Checkpoint questions (self-check)

1. On impact, you cited GDPR Article 33 as a regulatory-language anchor. If the system were deployed outside the EU with no equivalent mandatory-reporting duty, what should change on the impact row? (The anchor would change — a different regulator's language, or, if no regulator is in scope, a sector loss study or post-mortem financial estimate. The *level* might also drop to 3 because the "mandatory reporting is possible" clause no longer carries. The impact cell depends on the citable regulator, not on the raw harm magnitude.)
2. On likelihood, chapter 07 says the default for an adverse scenario is to score against the CI upper bound (0.05 in this lab) rather than the point estimate (0.03). If using the upper bound pushed the score from Unlikely (2) to Possible (3), what would you do? (Score at Possible (3) and document the convention in the argument sentence. Chapter 07: "make the register hard to be surprised by.")
3. On exposure, the rate limiter did not move your level from 2 (Limited). Suppose a change proposal removed the rate limiter entirely. Which axis in the three-state table would you re-score, and what state? (You would re-score the exposure axis on the *residual* state because the credited-controls list changes. This is the chapter-06 event-triggered cadence — "control change." Residual likelihood may not move but residual exposure may.)
4. If your three axes land 4 / 2 / 2 but a reviewer insists the combined level should be High (4) rather than Medium (3), which chapter-02 artifact resolves the disagreement? (The combination rule — an ordinal matrix published in full, or an anchored numeric formula. Chapter 02: "Whichever you pick, publish the matrix or formula, and cite it on every row." Without a published rule, both of you have taste; with one, the disagreement is a review of the rule itself.)

## Why this lab matters

Exercise 02 asks you to author a complete per-axis rubric *and* re-score the exercise-01 rows against it, with every level citing its rubric language and anchor. If the per-axis read takes you 20 minutes of staring at the scenario, the exercise won't ship on time and your rubric will quietly drift into "feel." This lab drills the atomic read — one axis, one level, one rubric quote, one anchor, one argument — until it is five minutes per axis. Combined with lab 01's three-state discipline and lab 03's calibration-evidence discipline, you can go into exercise 02 with the hard part already automatic.
