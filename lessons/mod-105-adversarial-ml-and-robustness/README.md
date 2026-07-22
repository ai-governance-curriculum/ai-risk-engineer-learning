# mod-105 — Adversarial ML and Robustness Engineering

**Estimated effort:** 16 hours
**Position in the track:** fifth module; assumes mod-101 (frameworks & role scope), mod-102 (harm modelling), mod-103 (risk quantification), and mod-104 (LLM red-teaming) are done.

Where mod-104 teaches the *LLM red-team* — prompt-injection, jailbreaks, indirect-prompt-injection — this module teaches the *classical adversarial-ML eval*: evasion attacks on classifiers and language models, data / label poisoning, model extraction and inversion, and membership inference. You pin **NIST AI 100-2 e2025** as the working taxonomy, cross-tag every technique to **MITRE ATLAS** and **OWASP ML Top 10**, engineer the attacks with **Adversarial Robustness Toolbox** and **TextAttack** (with **CleverHans** as a compare-point), baseline the defence set, and file a shippable **robustness-eval report** that plugs into the risk register with a residual-risk update. The chapter set closes with the level-25 → level-35 handoff to the platform-security peer in the sibling `security-learning` / `ai-infra-security` track who owns the runtime hardening at the inference edge.

## Learning objectives

- Read **NIST AI 100-2 e2025** as the working adversarial-ML taxonomy — evasion, poisoning, extraction, inversion, membership inference, availability — and map every technique in the working suite back to a MITRE ATLAS tactic and OWASP ML Top 10 entry.
- Engineer evasion attacks on production classifiers and language models with the **Adversarial Robustness Toolbox (ART)** and **TextAttack**, using **CleverHans** as a compare-point for classical baselines; report perturbation-budget curves rather than single-point success rates.
- Engineer data / label poisoning attacks (label flipping, targeted, backdoor / BadNets, clean-label) and evaluate defence baselines (**activation clustering**, **spectral-signature detection**, **STRIP**) on realistic ML systems; measure detection rate *and* clean FPR, not just detection.
- Engineer model-extraction and model-inversion attacks — and enough membership-inference to establish baseline advantage — well enough to defend the deployed API surface with query-rate control, differential-privacy training, and output-perturbation baselines. Note the mod-107 handoff for formal privacy accounting.
- Author a **robustness-eval report** — attack success rate, transferability, perturbation budget, defence delta, residual-risk update — that plugs into the mod-103 register and nominates monitors and incident-response thresholds.
- Cite the boundary to `security-learning` / `ai-infra-security` (level 35) on platform-scale defence at the inference edge — this module builds the working eval, that peer owns the runtime hardening (gateway config, WAF rules, rate-limit enforcement, artifact signing, endpoint attestation).

## Chapters

1. [`01-nist-ai-100-2-taxonomy.md`](01-nist-ai-100-2-taxonomy.md) — NIST AI 100-2 e2025 as the working taxonomy; six attack categories, the three-axis threat-model rubric, and the NIST → MITRE ATLAS → OWASP ML Top 10 cross-tag table shape.
2. [`02-evasion-attacks-and-tooling.md`](02-evasion-attacks-and-tooling.md) — evasion attacks on classifiers (FGSM, PGD, C&W) with ART; text-domain evasion with TextAttack; CleverHans as classical compare-point; perturbation budgets and threat models.
3. [`03-poisoning-and-defence-baselines.md`](03-poisoning-and-defence-baselines.md) — data / label poisoning taxonomy (label flip, targeted, backdoor, clean-label); defence baselines (activation clustering, spectral-signature, STRIP); how a defence is *measured* — detection rate, clean FPR, clean-accuracy delta, latency delta.
4. [`04-extraction-inversion-and-membership-inference.md`](04-extraction-inversion-and-membership-inference.md) — model-extraction (Tramèr, Knockoff Nets); model-inversion (Fredrikson, gradient-inversion); MIA (Shokri, Yeom, LiRA); the shared defence set — rate control, DP-training, output perturbation — and the mod-107 handoff for formal privacy work.
5. [`05-robustness-eval-report.md`](05-robustness-eval-report.md) — the shape of a shippable robustness-eval report: methodology, attack success rates, transferability, perturbation-budget curves, defence deltas, residual-risk updates, boundary and handoffs; worked evasion example.
6. [`06-level-ladder-handoff-and-runtime-hardening.md`](06-level-ladder-handoff-and-runtime-hardening.md) — what this module owns (the working eval) vs. what `security-learning` / `ai-infra-security` at level 35 owns (inference-edge hardening); the specifier / implementer / operator / evidence contract.

## Exercises

- [`exercises/exercise-01-nist-ai-100-2-to-atlas-to-owasp-cross-tag.md`](exercises/exercise-01-nist-ai-100-2-to-atlas-to-owasp-cross-tag.md) — build the NIST → ATLAS → OWASP cross-tag sheet for the running system.
- [`exercises/exercise-02-evasion-attacks-with-art-and-textattack.md`](exercises/exercise-02-evasion-attacks-with-art-and-textattack.md) — run the evasion attack panel (FGSM, PGD, C&W, TextFooler, BAE) and produce the perturbation-budget curves.
- [`exercises/exercise-03-data-poisoning-and-detection-baselines.md`](exercises/exercise-03-data-poisoning-and-detection-baselines.md) — engineer at least one label-flipping and one backdoor attack; baseline activation clustering, spectral-signature detection, and STRIP with full detection-rate / clean-FPR reporting.
- [`exercises/exercise-04-extraction-inversion-and-query-rate-defence.md`](exercises/exercise-04-extraction-inversion-and-query-rate-defence.md) — engineer an extraction attack and an MIA attack; specify and measure the query-rate / output-perturbation defence set.
- [`exercises/exercise-05-robustness-eval-report-and-residual-risk-update.md`](exercises/exercise-05-robustness-eval-report-and-residual-risk-update.md) — author the shippable robustness-eval report and file the residual-risk update into the mod-103 register.

## Labs & quizzes

- [`labs/`](labs/) — a longer hands-on lab is scheduled here; see the lab README for the current status.
- [`quizzes/`](quizzes/) — knowledge-check quiz for the module.

## Resources

See [`resources.md`](resources.md) for the primary sources — every reference is a standards body, a MITRE / OWASP / government project, an open-source tool project page, an original arXiv / conference paper, or a government AI safety institute.

## How to work through the module

1. Pick **one AI system** — the running system from mods 101-104, or a fresh one with a real ML classifier / language-model surface. All five exercises are cumulative and use this system.
2. Read chapters 01-06 in order. Each pins one dimension of the eval you are building.
3. Do exercises 01-05 in order. They stack:
   - 01 produces the NIST → ATLAS → OWASP cross-tag sheet for the system,
   - 02 runs the evasion attack panel and produces the perturbation-budget curves,
   - 03 engineers poisoning attacks and baselines the three detection defences,
   - 04 engineers extraction / MIA and specifies the shared defence set,
   - 05 authors the robustness-eval report and files the residual-risk update into the mod-103 register.
4. Take the quiz.

By the end of the module you should be able to hand an auditor a robustness-eval report where every attack scenario ties back to a harm-model row (mod-102), a taxonomy row (NIST AI 100-2 + MITRE ATLAS + OWASP ML Top 10 tags), a defence-delta measurement, and a residual-risk update in the register (mod-103); and a level-25 → level-35 handoff contract that names, per co-owned control, who specifies, who implements, who operates, and where the evidence lives. If the report is missing the defence-delta or the handoff contract is missing an owner column, the eval is not shippable.
