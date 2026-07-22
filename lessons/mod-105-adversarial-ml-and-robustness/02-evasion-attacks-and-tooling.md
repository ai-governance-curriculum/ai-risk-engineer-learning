# Evasion Attacks and the Working Tooling Stack

Evasion is the category of adversarial attack most engineers meet first — a small perturbation on an input at inference time causes the model to misclassify or mis-generate. It is also the category with the largest, most mature open-source tooling stack. This chapter takes you through the attack families you will actually run in exercise 02, the tools that run them (Adversarial Robustness Toolbox for classifier evasion, TextAttack for text-domain evasion, CleverHans as the classical compare-point), and the threat-model and budget discipline that makes the evaluation defensible.

## Motivation

Two failure modes an auditor spots in a first-pass adversarial evaluation:

- **A one-attack eval.** The team ran FGSM once at one epsilon and shipped a "robustness score." FGSM is a single-step attack from 2015; a system that only survives FGSM survives nothing that a real adversary would use. The auditor asks *"did you test PGD? C&W? adaptive attacks?"* and there is no answer.
- **An unspecified perturbation budget.** The report cites "attack success rate 12%" without saying what perturbation size was allowed. That number is uninterpretable — success rate at ε=1/255 and ε=32/255 differ by orders of magnitude. The auditor cannot compare across models, across time, or across teams.

Engineering the evaluation means picking an *attack panel* — a curated set of attacks with named threat models — and running each at a *named budget* against a *named baseline*. The output is a table with axes the auditor can read.

## The three classifier-evasion attacks that anchor the panel

Every evasion evaluation on a classifier in this module runs at least these three, in this order. Each represents a distinct step in the adversarial-optimisation story.

### FGSM — Fast Gradient Sign Method (Goodfellow et al., 2014)

`x_adv = x + ε · sign(∇_x L(f(x), y))`

Single-step, gradient-sign perturbation bounded by ε in the L∞ norm. FGSM is fast, cheap, and *weak*. Its role in the panel is as a floor — if the model fails FGSM meaningfully, it will fail everything harder. A model that survives FGSM only tells you that the model has some gradient obfuscation, not that it is robust.

### PGD — Projected Gradient Descent (Madry et al., 2017)

`x_{t+1} = Π_{B(x, ε)}(x_t + α · sign(∇_x L(f(x_t), y)))`

Iterative FGSM with projection back into the ε-ball at each step. PGD with enough steps is the reference *first-order white-box* attack for the L∞ threat model. Madry et al.'s work is why the community treats PGD as the minimum bar for a "meaningful robustness claim." If you cite robustness, you cite it under PGD at a stated ε, stated step count, stated step size, and stated number of random restarts.

### C&W — Carlini & Wagner (Carlini & Wagner, 2016)

An optimisation-based attack that minimises the perturbation norm subject to a misclassification constraint. C&W finds *tighter* perturbations than PGD at the cost of more compute per example. It is the attack that famously *broke* the early defensive-distillation results — a lesson in why the panel needs to include attacks that were designed to defeat known defences.

### The panel discipline

- **Always PGD at a stated ε, steps, step size, restarts.** ε on L∞ commonly reported as fractions of 255 for image domains (e.g., ε=8/255, ε=16/255). L2 attacks report ε in raw L2.
- **Add C&W where you claim robustness.** C&W success as a defence-defeat check.
- **Add FGSM as a floor.** Cheap, useful as regression against gradient-masking regressions.
- **State the norm.** L∞ vs. L2 vs. L0 vs. patch attacks each give a different rank ordering across models.
- **State restarts.** A robustness number under one random restart is not the same number as under twenty.

The panel above is a minimum. Any defence claim (adversarial training, certified robustness, input transformations) has to be evaluated against *adaptive* attacks — attacks specifically constructed to defeat the defence — following the Carlini et al. "On Evaluating Adversarial Robustness" guidance. Non-adaptive evaluations of defences have a long track record of being overturned.

<!-- needs-research: cite the current best-practice adaptive-attack guidance from Carlini et al. and the RobustBench evaluation protocol at authoring time. -->

## The Adversarial Robustness Toolbox (ART) as the runner

**ART** (formerly IBM ART, now under the Linux Foundation AI & Data umbrella) is the working evasion-attack runner for classifier evaluations in this module. Reasons for the pick:

- **Wide attack coverage** — FGSM, PGD, C&W, DeepFool, Auto-PGD, boundary attacks, patch attacks, sponge attacks, and more, all with a common API.
- **Framework-agnostic wrappers** — PyTorch, TensorFlow, scikit-learn, XGBoost, and others via a common `Classifier` abstraction.
- **Defence library** — adversarial training, input transformations, feature squeezing, certified defences (randomised smoothing, IBP) live alongside the attacks in the same repo, which lets you baseline the defence-delta on the same runner.
- **Actively maintained** — releases on a regular cadence, which matters for the auditor question "when was this last updated?"

<!-- needs-research: confirm the current maintainer, latest release, and defence-catalogue coverage of ART at authoring time. -->

The exercise-02 code path for classifier evasion follows this shape:

1. Load the model as an ART `Classifier` (framework-appropriate wrapper).
2. Load a clean evaluation set that mirrors production distribution (not just a canonical test set).
3. Instantiate each attack in the panel — `FastGradientMethod`, `ProjectedGradientDescent`, `CarliniLInfMethod` / `CarliniL2Method` — with the panel budget.
4. Generate adversarial examples on the clean set.
5. Compute per-attack success rate against the clean-set accuracy.
6. Log per-example perturbation norms so you can plot the perturbation-budget curve (chapter 05).

Every step here is a piece of the report you build in chapter 05 and exercise 05.

## TextAttack as the text-domain runner

Classifier evasion in the text domain is not "add ε to the pixels." Language is discrete; the perturbation vocabulary is transformations — synonym substitution, character-level typos, whitespace inserts, back-translation, embedding-space swaps. **TextAttack** is the runner that packages these transformations as composable attack recipes.

TextAttack's core abstractions:

| Concept | What it holds |
|---|---|
| **Goal function** | The success criterion — untargeted classification flip, targeted classification, minimum-cosine-similarity constraint on outputs |
| **Constraints** | Semantic / grammatical / edit-distance constraints on how the adversarial text may differ from the original |
| **Transformations** | The candidate-generation step — WordSwapEmbedding, WordSwapWordNet, character-level swaps, homoglyph swaps |
| **Search method** | Greedy, beam, particle-swarm, genetic — how the transformation space is searched |

Attack *recipes* combine those four axes to reproduce published attacks — TextFooler (Jin et al.), BAE (Garg & Ramakrishnan), DeepWordBug (Gao et al.), PWWS (Ren et al.). The exercise-02 pattern is to run at least two recipes (TextFooler and BAE, typically) against a text classifier or a fine-tuned LLM used as a classifier.

<!-- needs-research: confirm the current TextAttack recipe catalogue and the maintenance / release cadence at authoring time. -->

Two disciplines specific to text-domain evaluation:

- **Constraint sanity checks.** A high attack success rate at zero semantic constraint is meaningless — you have proved that changing all the words changes the classification. The constraint set (e.g., Universal Sentence Encoder cosine ≥ 0.84) is *part of the threat model*.
- **Human-readable adversarial examples.** For any text-domain adversarial example that will appear in a report, a human should be able to read the original and the adversarial side-by-side and agree the meaning is preserved. If they cannot, the constraint was too loose.

Text-domain evasion overlaps with LLM red-teaming (mod-104) when the target is a generative LLM rather than a classifier. Where the target is a *classification head* (safety classifier, PII detector, sentiment head), it is a mod-105 evasion attack. Where the target is *free-form generation* (jailbreak on a chat LLM, prompt injection into an agent), it is a mod-104 red-team. The boundary matters because the tools, the metrics, and the reporting shape are different.

## CleverHans as the classical compare-point

**CleverHans** predates ART and TextAttack; it was the first widely used adversarial-example library, released alongside the FGSM paper era, and it is the reference implementation many published results cite. Its role in this module is *compare-point*, not primary runner:

- When you reproduce a published result (Madry PGD numbers on CIFAR-10, for example), CleverHans is often the implementation the paper used. Reproducing on CleverHans first and then on ART is how you check that the ART configuration is equivalent.
- When the community reports "PGD success rate" without specifying the runner, CleverHans behaviour is a common implicit default. Knowing where CleverHans and ART differ in default hyperparameters (step size, restart count, projection order) prevents apples-to-oranges comparisons.

<!-- needs-research: confirm the maintenance status and current release of CleverHans; it has had periods of low activity and its role as reference implementation may have shifted. -->

You do not need to run CleverHans as your production runner. You do need to know what it is, why the literature cites it, and how to reconcile a CleverHans-reported number with your ART result.

## Threat models and perturbation budgets

The threat-model rubric introduced in chapter 01 (knowledge / capability / goal) has to be instantiated per attack in your panel. A worked template:

| Attack | Knowledge | Capability | Goal | Norm | Budget (ε) | Steps / iterations | Restarts | Notes |
|---|---|---|---|---|---|---|---|---|
| FGSM | White-box | Active querier | Untargeted | L∞ | 8/255 | 1 | 1 | Floor attack; regression sentinel |
| PGD | White-box | Active querier | Untargeted | L∞ | 8/255 | 40 | 5 | Reference first-order attack |
| PGD | White-box | Active querier | Targeted | L∞ | 8/255 | 40 | 5 | For targeted-misclassification scenarios only |
| C&W | White-box | Active querier | Untargeted | L2 | (norm-minimising) | 1000 | 1 | Tight-perturbation check; defence-defeat baseline |
| Auto-PGD | White-box | Active querier | Untargeted | L∞ | 8/255 | 100 | 5 | Step-size-schedule variant; RobustBench-style |
| TextFooler | Black-box | Active querier | Untargeted | (word-swap; USE ≥ 0.84) | (constraint-bounded) | (search-bounded) | 1 | Text-domain baseline |
| BAE | Black-box | Active querier | Untargeted | (BERT-based swap; USE ≥ 0.84) | (constraint-bounded) | (search-bounded) | 1 | Text-domain compare-point |

That table, filled in for the running system, is the *methodology* section of the robustness-eval report you author in chapter 05.

Two budget rules the module enforces:

- **Never report a robustness number without ε (or the text-domain equivalent constraint).** ε is part of the number, not a footnote.
- **The perturbation-budget curve is the primary output.** Attack success rate is a *function* of ε; the curve — success rate vs. ε at fixed steps and restarts — is what a reviewer actually needs to see. A single-point number is a summary slide, not a defence.

## Where evasion attacks meet the risk register

Every attack on the panel produces one or more of:

- an **attack success rate** at each ε on the budget curve,
- a **transferability rate** (does the adversarial example crafted against your model also fool a surrogate? and vice versa?),
- a **defence delta** — the change in the above numbers when a defence (adversarial training, input transform, certified smoothing) is enabled.

Those three numbers are the mod-105 → mod-103 handoff: each becomes a control-defeated-column entry, a residual-column update, or both. Chapter 05 walks the full report shape and exercise 05 has you file the register update.

## Cross-references

- **mod-104 (LLM red-teaming)** — text-domain evasion on generative LLMs (jailbreaks, prompt injection) is mod-104 territory; classifier-head evasion (safety classifier, PII detector) stays here.
- **mod-103 (risk quantification)** — every panel result becomes a residual-risk update or a control-defeated entry.
- **mod-108 (guardrail engineering)** — input-transform, feature-squeezing, and adversarial-training defences you baseline here become candidate guardrails there.
- **mod-109 (continuous monitoring)** — a production-time attack-success-rate probe originates from this panel.
- **`security-learning` / `ai-infra-security` (level 35)** — WAF-style filtering, inference-edge rate control, and endpoint attestation are runtime-hardening deliverables at that peer. Chapter 06 draws the handoff.

## Summary

- Evasion attacks come in a small panel: FGSM (floor), PGD (reference first-order), C&W (defence-defeat check), plus adaptive variants against any claimed defence.
- ART is the working classifier-evasion runner; TextAttack is the text-domain runner; CleverHans is the compare-point for reproducing published results.
- Every attack is named against the three-axis threat model (knowledge / capability / goal), the perturbation norm, the budget ε (or text-domain constraint), the step count, and the restart count.
- The primary output is the perturbation-budget curve, not a single success-rate number.
- Defence claims must be evaluated against adaptive attacks — non-adaptive evaluations of defences have a track record of being overturned.
- Every panel result feeds a residual-risk or control-defeated column in the mod-103 register.
