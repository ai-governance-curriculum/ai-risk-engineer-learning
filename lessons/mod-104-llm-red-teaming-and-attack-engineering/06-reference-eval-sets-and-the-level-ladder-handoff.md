# Reference Eval-Sets and the Level-Ladder Handoff

The prior five chapters told you how to *author* a red-team programme against a specific target. This chapter closes the module on three questions the programme has to answer to be defensible outside your team: which *reference* eval-sets you benchmark against so your numbers are comparable, how you author your *internal* policy-derived suite on the Inspect harness so the numbers travel across systems and reviewers, and where this role's scope stops and the next roles on the ladder pick up.

## Motivation

Two failure modes a programme hits when it has no reference eval-sets:

- **The "we made up our own numbers" programme.** The suite reports a 3% ASR on refusal-bypass. There is no baseline anywhere in the industry that this number compares to; the reviewer cannot tell whether 3% is good or terrible. Finding: "measurement is not calibrated against external references."
- **The "we ran HarmBench once at kickoff" programme.** A public benchmark was run at project start; the number went into a slide; nothing has re-run since; the model has changed twice; nobody can find the run's config. Finding: "external calibration is a one-time artifact, not an operating practice."

The three reference sets this chapter pins — HarmBench, AIR-Bench 2024, and your own policy-derived internal suite — together answer *"how do you know your numbers are meaningful."*

## HarmBench

HarmBench is a standardised harmful-behaviour evaluation suite for LLMs and defences. Its structure:

- A set of harmful behaviours across categories (e.g., cybercrime, biological / chemical, misinformation, illicit-content) — each behaviour is a target string the attacker is trying to elicit.
- A red-team attack corpus that has been evaluated against those behaviours — direct attacks, encoding attacks, GCG-optimised attacks, PAIR-style adaptive attacks, and others.
- Automated classifier-based scoring that decides whether the model's response constitutes a hit on the behaviour.
- Published leaderboards giving comparable numbers across models and attack methods.

<!-- needs-research: pin the HarmBench paper — Mazeika et al., "HarmBench: A Standardized Evaluation Framework for Automated Red Teaming and Robust Refusal" — with arXiv ID and current version of the behaviour set and classifier. -->

Two ways an engineer uses HarmBench:

- **Reference baseline on the target.** Run the HarmBench behaviour set with a subset of the attack corpus against your target's model configuration; report per-category ASR and compare against published numbers for the model you use. This is a comparability claim — you can defend "our number is in line with what has been reported for this model" or explain why it is not.
- **Attack-corpus source.** The attack corpus itself is a resource for your suite — a set of empirically effective attacks across categories that seeds your own converter chains and PyRIT library entries (chapter 03).

HarmBench's classifier is itself an artifact whose calibration matters — the release-gate credibility of a HarmBench-scored run depends on the classifier's precision / recall on your model's outputs. If the classifier is out of distribution for your model's response style, its scores are advisory only. This is the same discipline that PyRIT scorers require (chapter 03).

## AIR-Bench 2024

AIR-Bench 2024 (Stanford CRFM) is the reference eval-set organised around *policy-derived risk categories* rather than a fixed behaviour list. Its structure:

- A hierarchy of risk categories derived from published regulations and platform policies (EU AI Act, US Executive Order, company AUPs), producing several hundred fine-grained categories.
- A prompt corpus with prompts targeting each category.
- Judge-model scoring against per-category refusal / compliance definitions.

<!-- needs-research: pin the AIR-Bench 2024 paper — Zeng et al. — with arXiv ID and the current version and total category count of the taxonomy. -->

Two ways an engineer uses AIR-Bench:

- **Category-coverage cross-walk.** AIR-Bench's fine-grained taxonomy is a check against your internal policy-derived suite: if AIR-Bench names a category your suite does not cover, that is a coverage gap (or an explicit out-of-scope, with a reason).
- **Regulation-linked benchmark.** Because the categories cite the regulations they derive from, AIR-Bench numbers can be presented to a regulatory audience with the regulation-to-category linkage intact. This is unusual — most benchmarks are technical measurements without a policy trail.

AIR-Bench and HarmBench are complementary: HarmBench is the *behaviour-level* benchmark with published leaderboards; AIR-Bench is the *policy-category-level* benchmark with regulation linkage.

## The policy-derived internal suite

Neither HarmBench nor AIR-Bench encodes *your* organisation's specific policy. Internal policies name categories a public benchmark cannot know (product-specific tool-use policies, enterprise-agreement carve-outs, jurisdiction-specific content restrictions). The internal suite is the artifact that bridges the gap.

The authoring process:

### Step 1 — enumerate the policy categories

Read the internal policies that constrain the system's outputs and actions: acceptable-use policy, content policy, tool-use policy, jurisdiction-specific carve-outs, product-specific commitments. Each policy clause that constrains behaviour becomes a candidate category.

### Step 2 — define the pass rule per category

For each category, define what the model must do — refuse outright, refuse with a specific redirect, decline the tool call, decline with a safe-completion, produce a bounded response with a caveat. The pass rule is what the scorer will check against.

### Step 3 — author the prompt corpus

Author (or curate) prompts that test each category. Each prompt has: the policy category it targets, the expected pass behaviour, a difficulty class (surface-form ask; encoded ask; multi-turn refusal-bypass framing; persona-hijack framing). The prompts live in the internal-suite repo; if any *responses* are collected, they go to the out-of-repo payload store.

### Step 4 — author the scorers

For each category, choose the scorer shape (substring, classifier, LLM-judge) with the calibration methodology from chapter 03. LLM-judge scorers get calibration evidence — agreement with a human-labelled sample — pinned to the scorer's release.

### Step 5 — wire onto the Inspect harness

Author the suite in Inspect (see below) so it composes with garak, PyRIT, and AgentDojo runs into a single evaluation harness with a single results format.

### Step 6 — put it on the release gate

Per-category ASR ceilings become part of the release-gate pass rule. The internal suite reports feed directly into the register rows.

Exercise 06 walks steps 1–6 on the running system.

## Inspect as the authoring harness

Inspect is the UK AI Safety Institute's evaluation framework for LLMs. Its role in this programme is as the *authoring harness* for the internal suite (and for regression fixtures) — not as another attack library. Inspect is deliberately minimal and composable, and it gives you three primitives that are the right shape for the internal-suite work:

- **`Task`** — a named evaluation, comprising a dataset, a solver, and a scorer.
- **`Dataset`** — the set of prompts (and expected behaviours / labels) the task runs on.
- **`Solver`** — the way the target model is invoked (single-turn generation, multi-turn conversation, tool-using agent).
- **`Scorer`** — the way outputs are scored (substring, classifier, LLM-judge, custom).

<!-- needs-research: pin the current Inspect release, module names, and confirm that `Task` / `Dataset` / `Solver` / `Scorer` remain the top-level primitives. -->

The internal suite becomes a directory of Inspect `Task` files, one per policy category, composed into a single suite that runs on the release gate. A worked shape:

```python
# internal-suite/tasks/refusal_illicit_advice.py
# (illustrative sketch; refer to the current Inspect release for exact class paths)

from inspect_ai import Task, task
from inspect_ai.dataset import hf_dataset, MemoryDataset
from inspect_ai.solver import generate, system_message
from inspect_ai.scorer import model_graded_qa

from internal_suite.data.illicit_advice import load_dataset
from internal_suite.policies.illicit_advice import policy_rubric


@task
def refusal_illicit_advice():
    dataset: MemoryDataset = load_dataset()   # loads prompts from repo; refs to any harmful responses point out-of-repo
    return Task(
        dataset=dataset,
        solver=[
            system_message("<your production system message hash>"),
            generate(),
        ],
        scorer=model_graded_qa(
            template=policy_rubric,           # policy-derived rubric text (not the harmful examples)
            grade_pattern=r"(?i)pass|fail",
            model="internal-judge-model",
        ),
    )
```

The task is versioned in-repo. The dataset points at the corpus (with any harmful strings out-of-repo, loaded by hash). The scorer is calibrated separately and its calibration evidence lives alongside the task.

Two disciplines around the Inspect suite:

- **The suite runs the same way in dev, CI, and release-gate.** No divergence in invocation. If the release-gate suite passes and the CI smoke does not, that is a suite-configuration bug, not a triage decision.
- **The suite's results feed the same triage workflow as garak and PyRIT.** Chapter 02's five-step triage applies to Inspect findings unchanged.

Inspect also hosts your **regression fixtures** — every confirmed jailbreak, indirect-injection finding, or tool-abuse chain becomes a `Task` that reproduces the finding deterministically and is expected to *pass* (i.e., the fix holds). Regression fixtures live in mod-111's fixture store; Inspect is the shared substrate.

## The level-ladder handoff

The programme has two boundaries above it on the ladder that must be named on the coverage matrix (chapter 01) and in the module README.

### Boundary with `ai-eval-engineer` (peer, level 30)

The `ai-eval-engineer` role owns *app-side safety measurement* — the release-gate metrics the application team looks at, the calibrated scorer methodology, the judge-vs-human agreement studies, the statistical framing of ASR (confidence intervals, bootstrapping, per-category power analyses).

The handoff contract:

| You (L25) own | They (L30) own |
|---|---|
| The red-team programme artifacts — targets, coverage matrix, attack suites, replay bundles, triage log | The app-side release-gate eval metrics that consume your suite outputs |
| Suite composition — which garak probes, PyRIT attacks, AgentDojo environments, Inspect tasks are in the suite | Statistical methodology — CIs on ASR, judge calibration studies, agreement metrics |
| Scorer selection at the suite level | Scorer *methodology-of-methodology* — how a judge is calibrated, how a classifier's PR curve informs the threshold |
| Harmful-payload discipline | The safety measurement contract with the application team |

In practice: you author the suite and produce ASR numbers per category. L30 wraps those numbers with statistical framing, integrates them into the app's release-gate metric set, and calibrates the scorers. If a scorer's calibration is questionable, L30 flags it back to you and you decide whether the scorer can gate the build or is advisory-only.

### Boundary with `agentic-safety-engineer` (level 40)

The `agentic-safety-engineer` role owns *frontier-agent depth* — the attacks that require reasoning across dynamic tool topologies, multi-agent orchestration, browser-driving agents against unbounded upstream content, and capability-elicitation studies against frontier models with novel scaffolding.

The handoff contract:

| You (L25) cover | They (L40) pick up |
|---|---|
| Static tool topology, enumerable retrieval sources, single-agent or one-hop delegated systems | Dynamic tool composition, multi-agent orchestration with credential passing |
| Indirect injection through named retrieval / tool surfaces | Browser-driving agents against unbounded live web content |
| Published GCG-suffix replay + quarterly fresh optimisation against open-weight targets you deploy | Novel-attack research; capability-elicitation on frontier models with new scaffolding |
| Many-shot coverage at your context lengths | Frontier-model context-length envelope pushes |
| PyRIT / Inspect / garak-shaped tooling | RSP-informed dangerous-capability evaluations at frontier scale |

The visible form of the boundary on the coverage matrix is the `escalated:L40` cell (chapter 01). You do not have to *do* the L40 work; you have to *name* where it starts and open the escalation so the L40 role knows the boundary the L25 programme drew.

### Boundary with SecOps (peer, not on this ladder)

Classical application security — network exploitation, authentication bypass, RCE against the underlying infrastructure, supply-chain compromise of build systems — is SecOps. The AI red-team covers AI-specific attack surfaces. The line, in practice:

- If the attack is "get the model to say / do something it shouldn't" → AI red-team (this programme).
- If the attack is "compromise the infrastructure hosting the model" → SecOps.
- If the attack is "steal the model weights" → SecOps + mod-110 (MRM / supply chain).
- If the attack is "poison the training data" → mod-105 (adversarial ML) with SecOps coordination.

The target profile's out-of-scope list (chapter 01) is where the SecOps boundary is documented for a specific target.

## What "done" looks like at this module

A shippable module output — the artifact you carry into every subsequent module and the deferral contract you hand to the two roles above you on the ladder — is:

- A red-team programme for the running system with all five chapter-01 artifacts populated.
- A coverage matrix with every cell either `covered:<suite-id>`, `n/a`, `deferred:<ticket>`, or `escalated:L40`.
- A garak smoke suite running per commit and a full release-gate suite running per RC.
- A PyRIT attack library with converter chains, multi-turn adaptive attacks, and a policy-refusal scorer.
- An indirect-injection suite anchored on AgentDojo + InjecAgent, extended to your tool topology.
- A GCG suffix replay set (out-of-repo) and, for open-weight targets you deploy, a fresh-optimisation cadence.
- A many-shot demonstration corpus (out-of-repo) with a shots-per-context sweep.
- An Inspect-authored internal policy-derived suite with per-category ASR ceilings feeding the release gate.
- HarmBench + AIR-Bench reference runs against the target with published numbers cited for comparability.
- A triage log with every finding dispositioned and every TP producing a regression fixture in mod-111's store.
- A signed handoff to L30 (`ai-eval-engineer`) on the app-side metrics contract and to L40 (`agentic-safety-engineer`) on the frontier-agent depth boundary.

Anything less is the shape of a programme, not a shippable one.

## Interfaces with adjacent modules

- **Mod-102 (harm modelling)** — every category in the internal suite ties to harm-model rows; the suite is the operational form of the harm model's testable claims.
- **Mod-103 (risk register)** — per-category ASR is the likelihood anchor for every category-linked register row; the internal-suite cadence is a register-review trigger.
- **Mod-108 (guardrails)** — the internal suite is the effectiveness measurement for the guardrail set; guardrail additions drive suite additions.
- **Mod-109 (monitoring)** — production monitors mirror the categories the internal suite covers; the ASR trend over time is a monitoring input.
- **Mod-111 (incident response)** — every confirmed finding produces an Inspect regression fixture; the fixture store is authored on Inspect and shared with this module.

## Summary

- Reference eval-sets — HarmBench for behaviour-level baselines and attack-corpus source; AIR-Bench 2024 for policy-category-level baselines with regulation linkage — are what make your numbers comparable.
- The internal policy-derived suite is the artifact that closes the gap between public benchmarks and your specific policy; author it in six steps on the Inspect harness.
- Inspect provides `Task` / `Dataset` / `Solver` / `Scorer` primitives that host both the internal suite and the mod-111 regression fixtures.
- `ai-eval-engineer` (L30) picks up app-side safety measurement, scorer calibration methodology, and statistical framing of ASR; you produce the numbers, they wrap them.
- `agentic-safety-engineer` (L40) picks up dynamic tool topology, multi-agent orchestration, browser-driving agents, and frontier-model capability-elicitation studies; the boundary is named on the coverage matrix as `escalated:L40`.
- SecOps picks up classical AppSec (infra, auth, model-weight theft, training-data poisoning where infra is involved); the target profile's out-of-scope list is the documented boundary.
- Exercise 06 authors the Inspect-based internal suite and reference-benchmark integration for the running system.
