# DP-SGD as the Primary Defence: Opacus, TensorFlow Privacy, and Defending a Parameter Choice

Differentially private stochastic gradient descent (DP-SGD) is the primary defence a level-25 privacy engineer runs against membership-inference, attribute-inference, model-inversion, and (to a bounded degree) LLM memorisation. Two engineering libraries dominate the space: **Opacus** (Meta, PyTorch) and **TensorFlow Privacy** (Google, TensorFlow / JAX). This chapter engineers DP-SGD end-to-end: the (ε, δ) semantics your report cites, the Rényi Differential Privacy (RDP) accounting the libraries use under the hood, the noise-multiplier and clipping-norm knobs you tune, the privacy / utility trade-off you defend at review, and the operational failure modes that quietly invalidate the privacy claim.

## Motivation

An engineer who runs DP-SGD without understanding it produces a training-run log with an "ε = 5" line in it and calls the privacy claim closed. Under review two questions land:

- *"Why ε = 5, not ε = 1?"* — the parameter choice must be **defended**, not chosen for convenience.
- *"What does ε = 5 actually mean for our subjects?"* — the number must be **interpretable** to a governance reader.

Both questions have engineering answers. This chapter is those answers.

## What differential privacy claims

Differential privacy (Dwork, McSherry, Nissim, Smith, TCC 2006 — "Calibrating Noise to Sensitivity in Private Data Analysis") is a *formal property of a randomised mechanism*, not a property of an outcome. A randomised training algorithm A is (ε, δ)-differentially private if for any pair of neighbouring training datasets D and D' (differing in one record), and any measurable outcome set S:

Pr[A(D) ∈ S] ≤ e^ε · Pr[A(D') ∈ S] + δ

Interpretation for the level-25 engineer:

- **ε (epsilon)** upper-bounds the *multiplicative* difference in outcome probability between two neighbouring datasets. Smaller ε ⇒ stronger privacy. A subject's contribution to the model has bounded influence on any statistic anyone can compute from the model.
- **δ (delta)** allows a small failure probability of the ε bound. Convention: δ ≪ 1/N where N is the number of subjects; e.g., for N = 10^6, δ = 10^-6 is a common choice. δ must be cryptographically small — never 0.1 or 0.01.
- **The definition is per-example** (or per-user, if the mechanism accounts per-user). "Neighbouring" is defined at the record level in most DP-SGD implementations; user-level DP is a stricter grouping and needs a group-privacy adjustment.

Two conventions that matter for defending the parameter choice at review:

- **ε on its own is not comparable across mechanisms** without stating what "neighbouring" means. Record-level ε = 5 and user-level ε = 5 are different claims.
- **ε does not translate directly to "one-in-e^ε chance of leaking."** The correct interpretation is the multiplicative bound above; over-simplified probability statements ("ε = 3 means a 5% chance of…") are wrong and get flagged.

## What DP-SGD does mechanically

DP-SGD (Abadi, Chu, Goodfellow, McMahan, Mironov, Talwar, Zhang, CCS 2016 — "Deep Learning with Differential Privacy") modifies stochastic gradient descent with two additions:

1. **Per-example gradient clipping.** Compute the per-example gradient (rather than a batch-average gradient); clip each per-example gradient to L2 norm C.
2. **Gaussian noise addition.** Sum the clipped per-example gradients over the (Poisson-sampled) minibatch and add Gaussian noise with standard deviation σ · C, where σ is the *noise multiplier*.

The three knobs the practitioner sets:

| Knob | What it does | Effect on ε | Effect on utility |
|---|---|---|---|
| Noise multiplier σ | Standard deviation of the Gaussian added per step (in units of clip norm C) | Larger σ ⇒ smaller ε (more private) | Larger σ ⇒ noisier gradients ⇒ lower utility |
| Clipping norm C | Per-example gradient L2 cap | Does not appear in ε in a first-order way; affects the noise-to-signal ratio via C in the noise scale | Too-small C ⇒ heavy clipping ⇒ underfitting; too-large C ⇒ noise dominates the signal |
| Sampling / batch shape (Poisson sampling rate q, number of steps T) | Composition scale | Larger T ⇒ larger ε (privacy budget spent per step); larger q ⇒ larger ε | More steps generally better utility; but at fixed ε budget, batch shape trades off |

## Privacy accounting: RDP and the Gaussian mechanism

The DP-SGD (ε, δ) figure is not read off directly from σ. It comes from a *composition analysis* — a bound on the privacy loss of running the noisy-clipped-gradient step T times.

The state of practice in the two reference libraries is **Rényi Differential Privacy** (Mironov, CSF 2017 — "Rényi Differential Privacy"):

- RDP is a *smoother* notion of DP that composes cleanly under Gaussian noise addition.
- The DP-SGD training loop tracks an RDP accountant over the T steps.
- At reporting time the accountant is converted to (ε, δ) for a chosen δ.

Both Opacus and TensorFlow Privacy provide RDP accountants; Opacus also supports the tighter Gaussian Differential Privacy (GDP) accountant (Dong, Roth, Su, JASA 2022 — "Gaussian Differential Privacy") and the moments-accountant-derived PRV (Privacy Random Variable) accountant. For a defensible report:

- **Pin the accountant** — which one, which version — and record the resulting (ε, δ) alongside the mechanism (σ, C, q, T).
- **Convert consistently.** Do not compare an RDP-derived ε from run A to a moments-accountant-derived ε from run B without stating the accountant delta.

## Opacus — the PyTorch reference

**Opacus** (`opacus.ai`, Meta) wraps a PyTorch optimiser and model to produce DP-SGD training with a minimum of code change. Skeleton:

```
from opacus import PrivacyEngine

privacy_engine = PrivacyEngine()
model, optimizer, dataloader = privacy_engine.make_private(
    module=model,
    optimizer=optimizer,
    data_loader=dataloader,
    noise_multiplier=σ,
    max_grad_norm=C,
)

for epoch in range(E):
    for batch in dataloader:
        ...standard training loop...

epsilon = privacy_engine.get_epsilon(delta=δ)
```

Operational notes:

- **Not every module is DP-SGD-compatible out of the box.** Batch normalisation layers, LayerNorm on some paths, and some attention implementations require replacement with DP-compatible equivalents (Opacus provides `ModuleValidator` and `ModuleValidator.fix`).
- **Per-example gradient computation is expensive.** Micro-batch training helps memory; `BatchMemoryManager` from Opacus manages the trade-off.
- **Poisson sampling vs. deterministic batching.** DP-SGD's theoretical bound assumes Poisson sampling of the batch; Opacus's `DPDataLoader` implements this. Using a non-Poisson sampler is a real privacy-claim risk — the reported ε is only valid under the assumed sampler.

## TensorFlow Privacy — the TensorFlow / JAX reference

**TensorFlow Privacy** (`github.com/tensorflow/privacy`, Google) provides the parallel scaffolding for TensorFlow-based training: `DPKerasSGDOptimizer`, `DPKerasAdamOptimizer`, and equivalent for TF/JAX flows via `dp_accounting`. It shares the mechanism-and-accountant model with Opacus and interoperates with the `dp_accounting` library that both projects use.

Operational notes:

- **Verify the accountant** — `dp_accounting.rdp.RdpAccountant`, `dp_accounting.pld.PLDAccountant`, or the `dp_accounting.mechanism_calibration` helpers — is pinned in the report.
- **Vectorised per-example gradient support** is available and worth using; naive per-example loops are prohibitively slow on modern accelerators.

<!-- needs-research: confirm the most current recommended accountant and per-example gradient path in the latest TensorFlow Privacy release at authoring time. -->

## The privacy / utility trade-off — the curve you file

DP-SGD imposes a utility cost that is real and non-uniform. The engineering artifact for a review is a **utility / privacy trade-off curve** on the task metrics that matter for your system.

Reporting shape (worked example, image classifier):

| ε | δ | σ | C | Test accuracy | LiRA TPR @ 1% FPR | Notes |
|---|---|---|---|---|---|---|
| ∞ (non-DP baseline) | n/a | 0 | n/a | 0.914 | 0.22 | Reference; no DP |
| 8 | 1e-5 | 0.6 | 1.0 | 0.891 | 0.08 | Modest utility cost; noticeable MIA drop |
| 3 | 1e-5 | 1.1 | 1.0 | 0.862 | 0.03 | Larger utility cost; strong MIA suppression |
| 1 | 1e-5 | 2.0 | 1.0 | 0.821 | 0.01 | Sharp utility drop; near-random MIA |
| 0.5 | 1e-5 | 3.6 | 1.0 | 0.752 | 0.005 | Steep cost; residual MIA at noise floor |

Two disciplines a level-25 report follows:

- **Report task-level utility and MIA together, not each in isolation.** The point of the eval is the trade-off; a table that shows only accuracy or only ε hides the trade-off from the reviewer.
- **Report *slice-level* utility.** DP-SGD's utility cost is not uniform — under-represented groups typically pay a larger utility cost (Bagdasaryan, Poursaeed, Shmatikov, NeurIPS 2019 — "Differential Privacy Has Disparate Impact on Model Accuracy"). The fairness / bias chapters in mod-106 already frame this; the DP-SGD chapter here inherits it. Report per-slice accuracy alongside overall.

## Defending the (ε, δ) choice at review

Under review, the parameter choice must be defensible on three grounds:

1. **Threat-model calibration.** Why is this ε enough for the disclosure risk in your context? Point to a residual MIA / attribute-inference / extraction number under the chosen ε that the register accepts.
2. **Utility calibration.** Why is this ε acceptable for the task? Point to the trade-off curve and the residual utility on the metrics that gate release.
3. **Comparability.** Cite the community reference points where possible: what have published papers used at similar model / data scales? US Census 2020 used ε on the order of 19.61 across the products (a public benchmark for a national-scale statistical release). Apple has published per-day ε budgets for its private-data collection in the low single digits. Academic DP-image-classification tends to cite ε in single digits at δ ~ 10^-5.

<!-- needs-research: pin the current authoritative US Census 2020 privacy-loss budget line and any updates from the Census Bureau's Disclosure Avoidance System at authoring time. -->

Two anti-patterns to avoid in the defence:

- **"ε = 100 is still DP."** Formally true; practically useless. Very large ε provides essentially no meaningful privacy bound. If the utility cost cannot be borne at ε in single digits, the honest report says "DP-SGD does not meaningfully bound this scenario at achievable utility," not "we chose ε = 100 to keep utility."
- **"We used DP so MIA can't happen."** DP-SGD suppresses MIA empirically at meaningful ε, but the residual MIA number is still measured and reported. The formal (ε, δ) bound is a mathematical guarantee under the mechanism; the *empirical* MIA is what the report cites as evidence.

## User-level DP vs. record-level DP

A subtlety that matters when subjects contribute *many records*: standard DP-SGD as implemented gives *record-level* neighbouring — one differing record between D and D'. If a single user contributes multiple records, the effective *user-level* ε is larger (by a group-privacy factor).

For an LLM fine-tuned on customer conversations, where one customer produces thousands of tokens across many conversations, the record-level DP claim understates the leak. User-level DP-SGD (Federated Learning with User-Level DP, McMahan et al. 2018) is the tighter framing. The level-25 engineer knows to state which level the (ε, δ) claim is at, and — where it matters — either engineers user-level DP or reports the group-privacy adjustment.

## Operational failure modes that silently invalidate the privacy claim

Six failure modes that produce a training run *labelled* DP but not *actually* DP:

1. **Non-Poisson batching.** The accountant assumes Poisson sampling; deterministic batching invalidates the bound. Use the library's DP dataloader.
2. **Layer replacement not applied.** BatchNorm on a path Opacus did not fix bypasses per-example clipping. Run `ModuleValidator` and fix findings.
3. **Gradient accumulation across steps.** Accumulating gradients over K micro-batches and stepping once means the effective batch is K times larger — the accountant must be told this.
4. **Warm-starting from a non-DP checkpoint.** A model warm-started from a non-private pretrained checkpoint carries whatever the pretrained model memorised, regardless of the DP-SGD applied downstream. State this — the (ε, δ) applies only to the *fine-tune* step.
5. **Data-dependent hyperparameter selection.** Selecting the noise multiplier or learning rate by looking at the training data spends privacy budget that the accountant has not tracked. Either pre-select or use published values.
6. **Non-DP evaluation on training data.** Reporting metrics computed on the training set spends privacy in the report itself. Either use held-out data or report DP-noised metrics.

Each of these gets a checklist entry in the DP-SGD training-run's release artifact.

## Cross-references

- **mod-101** — GenAI Profile risks 3 (Data Privacy) and 12 (Value Chain and Component Integration) both consume DP-SGD evidence.
- **mod-102** — the harm scenarios DP-SGD closes are the model-caused privacy rows.
- **mod-103** — DP-SGD's (ε, δ) parameters and the resulting empirical MIA / extraction numbers land in the residual column; the noise-multiplier-reduced variant is a control-defeated row.
- **mod-105** — the adversarial-ML chapters describe MIA and extraction as attacks; this chapter's defence closes them under a formal bound.
- **mod-108** — DP-SGD-trained models are a *catalogued control* in the guardrail catalogue.
- **mod-109** — MIA-drift monitoring in production is the operational check that the DP-trained model's residual number does not silently drift.

## Summary

- DP-SGD (Abadi et al., 2016) is the primary defence: per-example gradient clip to C, Gaussian noise σ·C added to the sum, RDP-accounted over T Poisson-sampled steps.
- (ε, δ) is a *formal* property of the mechanism, not the outcome. State the accountant, the sampling assumption, and the "neighbouring" definition (record- or user-level).
- Knobs: σ (privacy / utility trade), C (clip; too small underfits, too large invites noise dominance), sampling rate q and steps T (composition).
- Opacus (PyTorch) and TensorFlow Privacy (TF/JAX) are the reference libraries; pin the version and the accountant.
- Utility / privacy trade-off is the artifact filed at review: overall + slice-level utility and MIA together across ε ∈ {∞, 8, 3, 1, 0.5} or similar.
- Defend the (ε, δ) choice on threat-model calibration, utility calibration, and comparability to published references.
- Six operational failure modes silently invalidate the DP claim — checklist them into the release artifact.
