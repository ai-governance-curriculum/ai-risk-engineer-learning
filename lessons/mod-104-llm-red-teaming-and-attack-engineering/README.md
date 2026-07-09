# mod-104-llm-red-teaming-and-attack-engineering: LLM and Agent Red-Team Engineering

> Scaffolded by `aicg org execute-plan`. Lecture chapters and exercise content are authored on subsequent autonomous cycles.

**Estimated effort:** 18 hours

## Learning objectives

- Author production-grade LLM / agent red-team programs — targets, techniques, coverage matrix, replay bundles — using garak, Microsoft PyRIT, Promptfoo red-team, and the Inspect (UK AISI) harness
- Engineer jailbreak evaluations at production scale — direct prompt injection, indirect prompt injection through retrieved context or tool output, encoding / obfuscation attacks, multi-turn refusal-bypass, role-play / persona-hijack, GCG-style adversarial suffixes, many-shot in-context — grounded in the OWASP GenAI Red-Team Guide, OWASP LLM Top 10, Do-Anything-Now taxonomy, and Anthropic Many-Shot Jailbreaking
- Engineer tool-abuse evaluations — unintended tool invocation, data-exfiltration through retrieval, unauthorised action chains — grounded in AgentDojo, InjecAgent, and enterprise threat models from Microsoft AI Red Team
- Adopt HarmBench, AIR-Bench 2024, and a policy-derived internal harm-eval suite as reference eval-sets and know how to keep harmful payloads out of the source repository
- Coordinate with `ai-eval-engineer` (peer, level 30) on the app-side safety measurement contract this role feeds; coordinate with `agentic-safety-engineer` (level 40) on where frontier-agent depth begins

## Structure

- `01-…md` … `0N-…md`: lecture chapters.
- `exercises/`: per-exercise prompts.
- `labs/`: long-form hands-on labs.
- `quizzes/`: knowledge checks.
- `resources.md`: external references.
