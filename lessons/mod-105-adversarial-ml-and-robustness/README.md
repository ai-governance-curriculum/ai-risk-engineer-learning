# mod-105-adversarial-ml-and-robustness: Adversarial ML and Robustness Engineering

> Scaffolded by `aicg org execute-plan`. Lecture chapters and exercise content are authored on subsequent autonomous cycles.

**Estimated effort:** 16 hours

## Learning objectives

- Read NIST AI 100-2 (e2025) as the working adversarial-ML taxonomy — evasion, poisoning, extraction, inversion, membership inference, availability — and map every technique in the working suite back to a MITRE ATLAS tactic and OWASP ML Top 10 entry
- Engineer evasion attacks on production classifiers and language models with the Adversarial Robustness Toolbox and TextAttack, using CleverHans as a compare-point for classical baselines
- Engineer data / label poisoning attacks and evaluate defence baselines (Spectre / activation clustering / STRIP) on realistic ML systems
- Engineer model-extraction and model-inversion attacks well enough to defend the deployed API surface with query-rate control, differential-privacy training, and output-perturbation baselines
- Author a robustness-eval report — attack success rate, transferability, perturbation budget, defence delta — that plugs into the risk register with a residual-risk update
- Cite the boundary to `security-learning` / `ai-infra-security` (level 35) on platform-scale defence at the inference edge — this module builds the working eval, that peer owns the runtime hardening

## Structure

- `01-…md` … `0N-…md`: lecture chapters.
- `exercises/`: per-exercise prompts.
- `labs/`: long-form hands-on labs.
- `quizzes/`: knowledge checks.
- `resources.md`: external references.
