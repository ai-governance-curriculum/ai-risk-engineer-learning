# Extraction, Inversion, and Membership Inference — Attacking the Deployed API Surface

Once a model is behind an inference endpoint, three attack categories target it *through* that endpoint rather than through the training pipeline: **extraction** (steal a functional copy of the model), **inversion** (reconstruct training-data inputs from model outputs), and **membership inference** (decide whether a specific record was in the training set). All three treat the API as the attack surface, and all three are defended with a shared control set — query-rate control, differential-privacy training, and output-perturbation — that the deployment team owns. This chapter reads the attack literature well enough that you can engineer the attacks and specify the defence baselines; deeper privacy-metric work (DP-epsilon accounting, MIA advantage confidence intervals) sits in mod-107 and the boundary is called out where it applies.

## Motivation

Three failure modes an auditor spots when an inference API is deployed without any of these attacks in scope:

- **No extraction budget.** The team ships an endpoint with no per-caller rate limit and no anomaly on aggregate query counts. A well-funded competitor extracts a functional surrogate model in a few days and the intellectual-property control is defeated silently — the team learns from a customer, not from a monitor.
- **No inversion / privacy check.** The model was trained on customer records. An adversary uses model-inversion techniques to reconstruct plausible member records from the outputs. The team's data-protection commitment ("we do not expose training data") is not backed by any test.
- **No MIA baseline.** The model was trained on medical / financial / other regulated records. A published MIA achieves non-trivial membership advantage on the deployed model, and the regulator asks what the team's baseline MIA advantage is. There is no baseline to answer with.

Engineering the eval means running at least one attack per category against the actual deployed endpoint (or a faithful staging replica), reporting the numbers, and pairing each result with the defence baseline that ships alongside.

## Model extraction (model stealing)

**Objective:** obtain a functional copy of the deployed model — same input-output behaviour on the target distribution — with only query access.

### The reference attacks

| Attack | Threat model | What it needs | What it produces |
|---|---|---|---|
| **Tramèr et al., "Stealing ML Models via Prediction APIs" (USENIX Security 2016)** | Black-box, active querier | Query access; possibly access to prediction confidences | A surrogate model with high agreement on the target distribution |
| **Knockoff Nets (Orekondy, Schiele, Fritz, 2019)** | Black-box, active querier | Query access; a public "surrogate" dataset that overlaps in domain | A functional copy trained by using the target's labels as ground truth on the surrogate dataset |
| **Active-learning extraction (various)** | Black-box, active querier | Query access; query budget to invest in high-information samples | Higher agreement per query by picking samples the surrogate is uncertain on |

Extraction attacks scale with query budget. The Tramèr paper showed extraction of some model classes with query counts on the order of the number of parameters; modern deep models require more, but the shape holds — *if the attacker can query freely, extraction is a matter of time and cost, not feasibility*.

<!-- needs-research: cite the exact arXiv / USENIX identifier for Tramèr et al. 2016 and the CVPR / arXiv identifier for Knockoff Nets 2019. -->

### The defence baselines this module runs

- **Per-caller query-rate control.** Any per-authenticated-caller rate limit is an extraction-budget throttle. The number that matters is *queries-per-caller-per-day*, not global QPS.
- **Aggregate query-pattern anomaly.** Extraction attackers tend to query in patterns unlike legitimate users — uniform coverage of the input space, low query-repetition, high entropy on inputs. A monitor on those aggregates catches an extraction campaign in progress.
- **Output-perturbation baselines.** Return top-k class labels rather than full logits; round confidences to a lower precision; add small noise to non-argmax logits. Each degrades the extraction attack more than it degrades legitimate use.
- **Prediction-confidence redaction.** Where the API returns confidences, redacting to a fixed granularity (e.g., three buckets) prevents the confidences from carrying the fine-grained gradient signal extraction attacks exploit.

Every defence measured as a *delta* against the extraction attack: baseline agreement of the surrogate vs. agreement under defence. The number is what shows up as the residual-risk update in mod-103.

## Model inversion (attribute / feature reconstruction)

**Objective:** given only query access to a model, reconstruct plausible inputs or attributes of inputs used in training.

### The reference attacks

| Attack | Threat model | What it needs | What it produces |
|---|---|---|---|
| **Fredrikson et al., "Model Inversion Attacks that Exploit Confidence Information" (CCS 2015)** | Black-box (with confidences) or white-box | Query access with confidences | Plausible reconstructions of training-input attributes (e.g., recognisable face reconstructions from face-classifier confidences) |
| **Gradient-inversion (Zhu, Liu, Han, "Deep Leakage from Gradients," NeurIPS 2019)** | Federated / gradient-sharing setting | Access to gradient updates | Reconstruction of the input batch that produced the gradients |

Model-inversion attacks vary hugely in fidelity — the Fredrikson face-inversion result is famously "recognisable but not photographic"; the gradient-inversion setting is much more powerful because gradients carry far more input information than confidences. Whether an inversion attack is a *material* risk depends on the sensitivity of the training data (customer records vs. public data) and the API surface (confidences returned vs. redacted, gradients shared vs. not).

<!-- needs-research: cite Fredrikson et al. 2015 CCS identifier; cite Zhu, Liu, Han "Deep Leakage from Gradients" NeurIPS 2019 identifier and arXiv ID. -->

### The defence baselines this module runs

- **Confidence redaction / top-k-only outputs.** Same defence lever as extraction; here the argument is that inversion attacks lean on the confidence gradient.
- **Differential-privacy training (DP-SGD; Abadi et al., 2016).** Adds calibrated noise to gradients during training so the trained model provably bounds what any single training record can influence in the outputs. This is the strongest defence for both inversion and MIA; it is also expensive (utility hit, extra hyperparameters).
- **Output perturbation.** Same as extraction.

DP-training is where mod-105 and mod-107 overlap. This module baselines DP-training as *a defence lever* — enable, measure attack-success delta, measure utility delta, report. Mod-107 owns the formal-privacy accounting — the epsilon / delta budget, the RDP / GDP accountant, the privacy-utility tradeoff curve, and the audit-defensible claim of "this model was trained with (ε, δ) DP." The handoff is called out explicitly in each defence entry.

## Membership inference (MIA)

**Objective:** given only query access to a model, decide whether a specific record was in the training set.

### The reference attacks

| Attack | Threat model | What it needs | What it produces |
|---|---|---|---|
| **Shokri et al., "Membership Inference Attacks Against Machine Learning Models" (S&P 2017)** | Black-box | Query access; ability to train shadow models on similar distribution | Per-record membership prediction with reported advantage over random |
| **Loss-threshold MIA (Yeom et al., 2018)** | Black-box | Query access; loss / confidence output; threshold calibration | Simpler variant; often within a few points of shadow-model performance |
| **LiRA — Likelihood Ratio Attack (Carlini et al., 2022)** | Black-box | Query access; ability to train many shadow models per target record | Well-calibrated per-record likelihood ratios; state-of-the-art advantage; the current reference attack |

The MIA measurement discipline is *not* "attack success rate" as a headline. It is:

- **Attack advantage** = (true-positive-rate − false-positive-rate) at a chosen operating point.
- **Attack TPR at very low FPR** — the LiRA framing. A MIA that has 60% TPR at 60% FPR is trivial; a MIA that has 5% TPR at 0.1% FPR is a real privacy leak because it identifies specific members with high confidence.
- **Advantage curves** — TPR vs. FPR as a full ROC, not one operating point.

<!-- needs-research: cite Shokri et al. 2017 S&P identifier, Yeom et al. 2018 CSF identifier, Carlini et al. 2022 LiRA S&P identifier and arXiv ID. -->

The heavy-lift of proper LiRA-style MIA — training dozens or hundreds of shadow models per target record, calibrating the likelihood ratio properly, producing an advantage-curve at low FPR — is expensive. This module runs enough MIA to establish *baseline advantage* and *whether the model is materially vulnerable*; mod-107 (privacy-risk engineering) owns the deeper LiRA calibration and the DP-epsilon accounting that goes with it. If your running system's harm model rates MIA as material, plan the mod-107 depth work as a follow-up, not as an alternative.

### The defence baselines this module runs

- **Differential-privacy training (DP-SGD).** Bounds membership advantage as a function of the (ε, δ) budget. As above — this module measures the delta; mod-107 owns the formal accounting.
- **Output perturbation on confidences.** Reduces the loss-threshold and shadow-model attacks' signal-to-noise.
- **Model regularisation and early stopping.** Overfitting is the primary source of MIA vulnerability. A model that generalises well (small train-test gap) is naturally harder to MIA. This is a "hygiene" defence, not a formal one — cite as a contributing lever, not a primary defence.
- **Restricting queries per record.** Not a full defence (a determined attacker distributes queries across identities) but a rate-shaping lever.

## The shared defence set — query-rate, DP-training, output perturbation

Extraction, inversion, and MIA all lean on the same API-surface levers. This means one defence stack — properly designed — covers all three. The stack:

| Lever | Attacks it addresses | Cost / tradeoff | Owner |
|---|---|---|---|
| **Per-caller query rate limit** | Extraction (budget throttle); MIA / inversion (rate-shapes attack cost) | Legitimate high-volume callers need lifted limits with monitoring | This module specifies; `security-learning` / `ai-infra-security` (level 35) enforces at the edge |
| **Aggregate query-pattern anomaly** | Extraction campaigns; distributed MIA / inversion campaigns | False positives on legitimate crawler / batch-inference workloads | This module specifies the metric; mod-109 wires the monitor |
| **Output perturbation / top-k-only / redacted confidences** | All three | Utility hit on downstream consumers that need calibrated confidences | This module baselines; deployment team decides the redaction shape |
| **Differential-privacy training (DP-SGD)** | MIA (formally bounded advantage); inversion (bounded influence per record); poisoning (bonus) | Utility hit; expensive to train; hyperparameter-sensitive | This module baselines the delta; mod-107 owns the formal accounting |
| **Model-endpoint attestation and signed responses** | Prevents in-flight response tampering; complements the above | Deployment infrastructure work | `security-learning` / `ai-infra-security` (level 35) owns |

Each lever has to be measured in the *residual-risk update* — how much does enabling this move the attack numbers, and what does it cost. Chapter 05 walks the report shape.

## The reporting shape for this category

Extraction-inversion-MIA report rows differ from evasion / poisoning rows because the primary numbers are different:

| Attack category | Primary numbers |
|---|---|
| Extraction | Surrogate agreement rate on target distribution; queries required to reach the agreement threshold |
| Inversion | Reconstruction fidelity metric (task-specific — e.g., L2 distance in feature space, human-eval recognisability); sensitivity of attack to defence lever |
| Membership inference | Attack advantage; TPR at low-FPR (0.1% and 1%); full ROC |

Each of those has to be paired with the *defence-delta* number when a lever is engaged. The residual-risk update in the register (mod-103) reads from these.

## The privacy handoff to mod-107 — what stays here vs. what goes there

| Piece of work | Owner |
|---|---|
| Establishing *that* MIA / inversion is materially possible on the target system | This module (105) |
| Establishing baseline MIA advantage and reporting it in the register | This module (105) |
| Running DP-SGD as a defence baseline and measuring the delta | This module (105) |
| Formal (ε, δ) accounting for DP-training | mod-107 |
| LiRA-calibrated advantage curves at low FPR | mod-107 |
| Deciding the DP-epsilon target for the system given regulatory context | mod-107, with escalation to `ai-evaluation-engineer` (level 35) for release-gate decisions |
| Data-minimisation, PII-scrubbing, and consent-boundary controls at the pipeline | mod-107 |

If a reader is working through mod-105 stand-alone and the running system's harm model rates privacy risk as material, mark the mod-107 handoff explicitly in the register and revisit once mod-107 has been worked.

## Cross-references

- **mod-102 (harm modelling)** — extraction, inversion, and MIA scenarios must appear as harm-model rows before you run them; personas are typically "adversarial user" (external attacker with API access) or "downstream integrator" (partner querying the model beyond intent).
- **mod-103 (risk quantification)** — every attack result and every defence-delta is a register update.
- **mod-104 (LLM red-teaming)** — data-exfiltration jailbreaks that get the LLM to *print* training data overlap with inversion; treat as dual-tag rows.
- **mod-107 (privacy-risk engineering)** — the deeper privacy-metric and DP-accounting work.
- **mod-108 (guardrail engineering)** — output-perturbation and confidence-redaction defences you baseline here become candidate guardrails there.
- **mod-109 (continuous monitoring)** — query-rate and query-pattern anomaly monitors originate here.
- **`security-learning` / `ai-infra-security` (level 35)** — WAF-style edge enforcement of the rate limit, signed responses, endpoint attestation. Chapter 06 pins the handoff.

## Summary

- Three attack categories share an API surface: extraction (steal the model), inversion (reconstruct training-data inputs), membership inference (decide record membership).
- Reference attacks: Tramèr / Knockoff Nets (extraction); Fredrikson / gradient-inversion (inversion); Shokri / Yeom / LiRA (MIA).
- The MIA measurement discipline is TPR-at-low-FPR and full ROC advantage curves, not a single "attack success rate."
- Shared defence set: per-caller rate limit, aggregate query-pattern anomaly, output perturbation / confidence redaction, differential-privacy training, model-endpoint attestation.
- DP-training is baselined here as a defence lever; the formal (ε, δ) accounting and LiRA-calibrated curves are mod-107 territory.
- Edge enforcement of rate limits and signed responses is `security-learning` / `ai-infra-security` (level 35) territory; chapter 06 draws the handoff.
