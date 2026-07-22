# The Robustness-Eval Report and the Residual-Risk Update

The output of everything in this module is one artifact: a **robustness-eval report** for the running system, with the numbers plugged into the mod-103 risk register. This chapter defines that artifact — what sections it has, what numbers each section carries, how the numbers pair with defence deltas, and how the register is updated in one round-trip. Exercise 05 has you author the report; this chapter tells you the shape.

## Motivation

A report that only reports attack success rates against one attack at one budget is a slide, not an audit-defensible artifact. A shippable robustness-eval report has to answer, per attack scenario:

- *What attack, at what threat model, at what perturbation budget or query budget?* — the methodology.
- *How successful was the attack, with what transferability?* — the attack numbers.
- *How did the attack numbers change under each named defence?* — the defence deltas.
- *What does the register look like now?* — the inherent / residual / control-defeated update.

If any of those four sections is missing, the report is either a marketing summary (attack-successful, defence-shipped) or a research paper (attack characterisation without the deployment consequences). The engineering artifact sits between the two.

## Sections of the report

The report is one document per system per evaluation cycle. Suggested section list — filenames and headings can vary, but every section has to be filled.

### 1. Methodology

The threat-model and budget table (chapter 02 for evasion; chapters 03 and 04 for the other categories). One row per attack in scope, with:

- attack name and NIST AI 100-2 category,
- MITRE ATLAS technique id, OWASP ML (or LLM) Top 10 entry,
- knowledge / capability / goal axes,
- perturbation norm and budget ε (evasion), poison-rate sweep (poisoning), query-budget sweep (extraction / MIA / inversion),
- runner / library version (ART, TextAttack, CleverHans, research-code fork with commit hash),
- clean baseline dataset and its provenance.

This is the section a reproducibility reviewer reads first.

### 2. Attack success rates

For each attack in the panel, the primary numbers:

| Category | Primary numbers |
|---|---|
| Evasion | Attack success rate at each ε on the budget curve; success rate vs. ε plotted |
| Poisoning | Attack success rate at each poison rate on the sweep; success on the target input (targeted attacks); trigger-input success (backdoor) |
| Extraction | Surrogate agreement rate at each query budget; agreement-vs-queries curve |
| Inversion | Reconstruction fidelity metric (task-specific); confidence-redaction sensitivity |
| Membership inference | Advantage; TPR at 0.1% and 1% FPR; ROC curve |

Every number reports a *confidence interval* — bootstrap, or shadow-run variance, depending on the attack — not a point estimate. A single-run number is not a claim.

### 3. Transferability

Adversarial examples crafted against your model — do they also fool a surrogate you did not craft against? Adversarial examples crafted against a public surrogate — do they fool your model? Transferability is the argument for whether the attack works in a black-box setting even when the attacker never sees your weights. It is also the argument for whether your model's robustness has an off-by-default guarantee.

The transferability sub-table:

| Source model | Target model | Attack | Success rate on source | Success rate on target | Transfer ratio |
|---|---|---|---|---|---|

Transferability rows are cheap to add once the panel exists; adding them is what separates a first-pass eval from a shippable one.

### 4. Perturbation-budget curves

The primary artifact for evasion — attack success rate as a function of ε, plotted for each attack in the panel, with clean-accuracy marked. The curve says more than any single-ε number:

- Where is the model "brittle" (steep drop in accuracy above a small ε)?
- Where does the model catastrophically fail (accuracy → chance at some ε)?
- How does the curve shift when a defence is enabled?

For poisoning, the equivalent is *attack success as a function of poison rate*. For extraction, *agreement as a function of query budget*. For MIA, *advantage as a function of training-set size and query count*. Each category has its own budget dimension; every category produces its curve.

### 5. Defence deltas

For each defence lever, the same numbers as section 2 recomputed with the defence enabled. The delta is what determines the residual-risk update.

The defence-delta table:

| Attack | Baseline number (no defence) | Defence | Defence number | Delta | Clean-accuracy cost of defence | Latency / cost of defence | Notes |
|---|---|---|---|---|---|---|---|

Two rules:

- **Every defence tested against every attack**, including attacks it is known to be weak against. That is how the failure-mode picture is built.
- **Defence deltas evaluated against *adaptive* attacks.** A defence tested only against the un-adapted attack is a defence with an inflated delta. Cite the Carlini et al. "On Evaluating Adversarial Robustness" guidance in the methodology section.

<!-- needs-research: reference the current best-practice guidance for adaptive-attack evaluation of defences (Carlini et al., 2019 "On Evaluating Adversarial Robustness" and any updates). -->

### 6. Residual-risk update (the register handoff)

Per attack scenario, the register update:

| Scenario id | Inherent (before) | Inherent (after; usually unchanged) | Residual (before) | Residual (after) | Control-defeated variants added/updated | Monitors added / adjusted | Incident-response thresholds added / adjusted |
|---|---|---|---|---|---|---|---|

The residual number moves because the eval has *quantified* the attack — before the eval, residual was estimated on the basis of a defence catalogue; after the eval, it is quantified with numbers. Similarly, the control-defeated variants are populated with actual attack-success-under-defence-absence numbers, not estimates.

For each defence in the panel, a monitor entry: *what production metric would tell us this defence's residual number is no longer valid?* An attack-success-rate probe. A confidence-redaction on-off audit. A DP-training epsilon-budget drift monitor. Each entry links to a mod-109 monitor spec.

### 7. Boundary and handoffs

Explicitly named. Where does this eval stop, and what does the peer own?

- Edge-enforcement of the query-rate control → `security-learning` / `ai-infra-security` (level 35).
- Formal DP-epsilon accounting for training → mod-107.
- Runtime backdoor detectors deployed in the serving path → mod-108 catalogue.
- Continuous production monitoring on the numbers this eval produced → mod-109.
- Incident-response runbook branches for each control-defeated variant → mod-111.

This section is what stops the report from being a black-box "we did the eval" statement and turns it into an *interface document*.

## Worked example — evasion on a customer-support intent classifier

Continuing the running system from earlier modules (customer-support LLM assistant with a fine-tuned intent-classifier head). Extract from the report:

### Methodology (evasion slice)

| Attack | Threat model | Norm | ε | Steps | Restarts | Runner |
|---|---|---|---|---|---|---|
| FGSM | White-box, active, untargeted | L∞ | 8/255 | 1 | 1 | ART 1.x |
| PGD | White-box, active, untargeted | L∞ | 8/255 | 40 | 5 | ART 1.x |
| PGD | White-box, active, untargeted | L∞ | 16/255 | 40 | 5 | ART 1.x |
| C&W | White-box, active, untargeted | L2 | (norm-minimising) | 1000 | 1 | ART 1.x |
| TextFooler | Black-box, active, untargeted | Word-swap; USE ≥ 0.84 | (constraint-bounded) | (search-bounded) | 1 | TextAttack 0.3.x |

<!-- needs-research: pin the exact ART and TextAttack versions at authoring time. -->

### Attack success

| Attack | Clean accuracy | Attack success rate | 95% CI |
|---|---|---|---|
| FGSM (ε=8/255) | 92.4% | 41.2% | ±1.8% |
| PGD (ε=8/255) | 92.4% | 78.6% | ±1.3% |
| PGD (ε=16/255) | 92.4% | 93.1% | ±0.9% |
| C&W (L2) | 92.4% | 99.7% | ±0.3% |
| TextFooler | 92.4% | 62.4% | ±2.1% |

### Defence deltas — adversarial training

| Attack | Baseline (no def) | Adv-trained model | Delta | Clean-accuracy cost | Notes |
|---|---|---|---|---|---|
| PGD (ε=8/255) | 78.6% | 34.2% | −44.4pp | −2.7pp | Adaptive PGD used for adv-training |
| PGD (ε=16/255) | 93.1% | 71.8% | −21.3pp | −2.7pp | Robustness drops off outside training-ε |
| C&W (L2) | 99.7% | 91.2% | −8.5pp | −2.7pp | Adv-training is weak against L2 optimisation |

### Register update (residual slice)

| Scenario id | Inherent (before) | Inherent (after) | Residual (before) | Residual (after) | Control-defeated: adv-training absent | Monitor added |
|---|---|---|---|---|---|---|
| ADV-CS-EVASION-01 | High | High | Medium (estimated) | Medium (78.6% → 34.2% under adv-training at ε=8/255) | High (78.6% attack success without defence) | PGD-probe on canary set, weekly |

The residual number moves from *estimated* to *quantified*, and the control-defeated variant is populated with a concrete number, not a guess. That transition — from estimate to quantification — is what the eval buys the register.

## Report cadence and versioning

The robustness-eval report is *not* a one-time artifact. It is regenerated on each of:

- a new model candidate (retraining, fine-tuning, base-model upgrade),
- a new attack that the community publishes and the risk council rates as material,
- a new defence being added (report the delta),
- a periodic (e.g., quarterly) rerun to catch drift in attack tooling.

Every regeneration is a versioned artifact. The register update timestamps against the report version. This is what turns "we did the eval once" into "we run the eval; here's the trend."

## What the report is *not*

- **Not a research paper.** The report's audience is a risk council and an auditor, not NeurIPS reviewers. Novelty is not the point; defensibility is.
- **Not a marketing safety card.** Safety cards summarise for external readers; the eval report is the internal artifact those summaries would be *derived from*.
- **Not a compliance checkbox.** The report has to carry real numbers with real defence deltas, or it does not do its job.
- **Not the register itself.** The report *produces* the register update; the register lives at mod-103 and is the ongoing artifact. The report is the round-trip evidence.

## Cross-references

- **mod-102 (harm modelling)** — every scenario in the report ties back to a harm-model row; if a scenario appears in the report and not in the harm model, either the harm model needs an update or the scenario is out of scope.
- **mod-103 (risk quantification)** — the report is the primary input to the residual-risk update and the control-defeated column.
- **mod-104 (LLM red-teaming)** — the LLM red-team report has a similar shape; dual-domain scenarios file into both.
- **mod-107 (privacy-risk engineering)** — the MIA / inversion sections of this report feed the privacy-metric depth work at mod-107.
- **mod-108 (guardrail engineering)** — defences with favourable delta and acceptable cost become candidate guardrails.
- **mod-109 (continuous monitoring)** — every register-update row nominates a monitor; the specs land at mod-109.
- **mod-111 (incident response)** — each control-defeated variant nominates an incident-response runbook branch.

## Summary

- The robustness-eval report has seven sections: methodology, attack success, transferability, perturbation-budget curves, defence deltas, residual-risk update, boundary and handoffs.
- Every number carries a confidence interval; every defence is evaluated against adaptive attacks; every scenario ties back to the harm model and forward to the register.
- The primary artifact is the *curve*, not the single number — success rate vs. budget, per attack, per defence.
- The register update moves scenarios from *estimated residual* to *quantified residual* and populates the control-defeated column with real numbers.
- The report is versioned and regenerated on model changes, new attacks, new defences, and periodic reruns.
- Exercise 05 has you author this artifact end-to-end for the running system.
