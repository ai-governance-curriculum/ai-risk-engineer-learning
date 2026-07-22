# mod-104 — LLM and Agent Red-Team Engineering

**Estimated effort:** 18 hours
**Position in the track:** fourth module; assumes mod-101 (frameworks & role scope), mod-102 (harm modelling), and mod-103 (risk quantification and register) are done.

The harm model (mod-102) named the scenarios; the register (mod-103) scored them. This module produces the *runnable proof* that the scenarios actually fire — and that the controls actually stop them — for LLM and agent systems. You author a production-grade red-team programme for one target using NVIDIA garak, Microsoft PyRIT, Promptfoo red-team, and the UK AISI Inspect harness. You engineer jailbreak evaluations across the eight technique classes named in the OWASP GenAI Red-Team Guide, OWASP LLM Top 10, the DAN empirical taxonomy, and Anthropic's Many-Shot Jailbreaking work. You engineer tool-abuse evaluations grounded in AgentDojo, InjecAgent, and Microsoft AI Red Team's enterprise threat models. You adopt HarmBench and AIR-Bench 2024 as reference eval-sets alongside a policy-derived internal suite, and you keep every harmful payload out of the source repository. The programme you produce here is what mod-108 (guardrails) consumes for effectiveness measurement, what mod-109 (monitoring) mines for detector logic, and what mod-111 (incident response) draws regression fixtures from.

## Learning objectives

- Author production-grade LLM / agent red-team programmes — targets, techniques, coverage matrix, replay bundles — using garak, Microsoft PyRIT, Promptfoo red-team, and the Inspect (UK AISI) harness.
- Engineer jailbreak evaluations at production scale — direct prompt injection, indirect prompt injection through retrieved context or tool output, encoding / obfuscation attacks, multi-turn refusal-bypass, role-play / persona-hijack, GCG-style adversarial suffixes, many-shot in-context — grounded in the OWASP GenAI Red-Team Guide, OWASP LLM Top 10, Do-Anything-Now taxonomy, and Anthropic Many-Shot Jailbreaking.
- Engineer tool-abuse evaluations — unintended tool invocation, data-exfiltration through retrieval, unauthorised action chains — grounded in AgentDojo, InjecAgent, and enterprise threat models from Microsoft AI Red Team.
- Adopt HarmBench, AIR-Bench 2024, and a policy-derived internal harm-eval suite as reference eval-sets and know how to keep harmful payloads out of the source repository.
- Coordinate with `ai-eval-engineer` (peer, level 30) on the app-side safety measurement contract; coordinate with `agentic-safety-engineer` (level 40) on where frontier-agent depth begins.

## Chapters

1. [`01-red-team-program-shape.md`](01-red-team-program-shape.md) — the five artifacts (target profile, coverage matrix, attack suite, replay bundle, triage log); cadence, thresholds, and the harmful-payload-out-of-repo discipline.
2. [`02-garak-as-the-baseline-scanner.md`](02-garak-as-the-baseline-scanner.md) — NVIDIA garak's probes / detectors / generators / harnesses; a per-commit smoke suite; the five-step triage that turns candidates into register updates.
3. [`03-pyrit-for-orchestrated-attacks.md`](03-pyrit-for-orchestrated-attacks.md) — Microsoft PyRIT's targets / converters / attack-strategies / orchestrators / memory / scorers; the attack-library composition; scorer calibration discipline.
4. [`04-indirect-prompt-injection-and-tool-abuse.md`](04-indirect-prompt-injection-and-tool-abuse.md) — indirect injection through retrieval and tool output; AgentDojo as environment; InjecAgent as payload library; unintended invocation, exfiltration, unauthorised action chains; enterprise threat models from Microsoft AI Red Team.
5. [`05-jailbreaks-gcg-and-many-shot.md`](05-jailbreaks-gcg-and-many-shot.md) — the eight-class jailbreak taxonomy; GCG replay + fresh-optimisation coverage against open-weight targets; many-shot in-context with shots-per-context sweeps; the payload discipline for the highest-risk classes.
6. [`06-reference-eval-sets-and-the-level-ladder-handoff.md`](06-reference-eval-sets-and-the-level-ladder-handoff.md) — HarmBench and AIR-Bench 2024 as reference eval-sets; the policy-derived internal suite authored on Inspect; boundaries with `ai-eval-engineer` (L30) and `agentic-safety-engineer` (L40).

## Exercises

- [`exercises/exercise-01-red-team-program-charter-for-one-target.md`](exercises/exercise-01-red-team-program-charter-for-one-target.md) — author the five artifacts of chapter 01 for the running system.
- [`exercises/exercise-02-garak-scan-and-triage-workflow.md`](exercises/exercise-02-garak-scan-and-triage-workflow.md) — configure the garak smoke suite, wire it into CI, and walk the five-step triage on real candidates.
- [`exercises/exercise-03-pyrit-attack-library-and-orchestrator.md`](exercises/exercise-03-pyrit-attack-library-and-orchestrator.md) — build the PyRIT attack library — targets, converters, attack strategies, scorers, orchestrators — for the running system.
- [`exercises/exercise-04-indirect-injection-suite-with-agentdojo-and-injecagent.md`](exercises/exercise-04-indirect-injection-suite-with-agentdojo-and-injecagent.md) — run AgentDojo as reference, seed with InjecAgent payloads, and extend to a tool-topology-specific internal environment.
- [`exercises/exercise-05-gcg-and-many-shot-attack-engineering.md`](exercises/exercise-05-gcg-and-many-shot-attack-engineering.md) — assemble the GCG replay set (and, for open-weight targets, a fresh-optimisation cadence) plus the many-shot demonstration corpus and shots-per-context sweep.
- [`exercises/exercise-06-inspect-based-red-team-eval-authoring.md`](exercises/exercise-06-inspect-based-red-team-eval-authoring.md) — author the policy-derived internal suite as Inspect tasks and integrate HarmBench + AIR-Bench reference runs.

## Labs & quizzes

- [`labs/`](labs/) — a longer hands-on lab is scheduled here; see the lab README for the current status.
- [`quizzes/`](quizzes/) — knowledge-check quiz for the module.

## Resources

See [`resources.md`](resources.md) for primary sources — every reference points at a standards body, tool project, foundation-lab publication, government AI safety institute, OWASP project, or peer-reviewed / arXiv paper.

## How to work through the module

1. Pick **one AI system** (the same running system you used for mod-101–103, or a fresh one). All six exercises are cumulative — every one uses this system.
2. Read chapters 01–06 in order. Each pins one dimension of the red-team programme you are building.
3. Do exercises 01–06 in order. They stack:
   - 01 produces the programme charter (five artifacts, target profile, coverage matrix, threshold, cadence),
   - 02 wires the garak smoke suite into CI and walks the triage workflow on real candidates,
   - 03 builds the PyRIT attack library that carries the multi-turn and converter-chained depth,
   - 04 stands up the indirect-injection / tool-abuse suite with AgentDojo + InjecAgent + a tool-topology-specific extension,
   - 05 assembles the GCG replay set, the fresh-optimisation loop (if any open-weight target), and the many-shot corpus and sweep,
   - 06 authors the Inspect-based internal policy-derived suite and integrates HarmBench + AIR-Bench reference runs.
4. Take the quiz.

By the end of the module you should be able to hand a reviewer a red-team programme for one target where (a) every technique-class row on the coverage matrix is either `covered:<suite-id>`, `n/a`, `deferred:<ticket>`, or `escalated:L40`; (b) every finding in the triage log has a disposition, a register-row link, and — where applicable — a regression fixture; (c) every harmful payload is stored out-of-repo and referenced by hash; and (d) the handoffs to `ai-eval-engineer` (L30) and `agentic-safety-engineer` (L40) are named contracts, not vague deferrals. If any of those four checks fails, the programme is not shippable.
