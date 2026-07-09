# Reading NIST AI RMF 1.0 as an Engineer

The NIST AI Risk Management Framework (AI RMF 1.0, publication AI 100-1, released January 2023) is the anchor document for this track. Governance analysts read it top-down — this role reads it bottom-up. Every sub-category in the framework has to translate into a *test, threshold, guardrail, monitor, or evidence artifact* that a repo can hold. This chapter teaches that translation.

## Motivation

An analyst can hand you the sentence "MEASURE 2.7 — AI system security and resilience are evaluated and documented." That sentence does not deploy. What deploys is:

- an adversarial-robustness evaluation suite that runs on every model candidate,
- a threshold in that suite that must be met for release,
- a guardrail that catches the failure modes the suite finds,
- a monitor that fires when production behaviour drifts away from the tested envelope,
- an evidence artifact — a signed report, a scorecard, a manifest — that shows the auditor the above four exist and were run.

Your job at level 25 is to *engineer the translation*, and to keep the framework's language and the engineering language pinned to each other so an audit doesn't turn into a translation project.

## Structure of the framework

AI RMF 1.0 has two parts: the **core** and the **profiles**. The core is organised into four functions, each broken into categories and sub-categories:

| Function | What it asks of the organisation | Where you show up |
|---|---|---|
| **GOVERN** | Policies, roles, accountability, culture | You supply engineering-side evidence that a policy is testable and tested |
| **MAP** | Context, scope, use cases, stakeholders, impacts | You supply the harm model and system inventory technical shape (owned in mod-102) |
| **MEASURE** | Analyse, assess, benchmark, monitor | You *own* this — every sub-category is an evaluation, a threshold, or a monitor |
| **MANAGE** | Prioritise, treat, respond, communicate | You supply mitigation code, incident hooks, and evidence pipelines |

The document that lists every sub-category, its outcome statement, and a set of suggested actions is the **NIST AI RMF Playbook**, hosted on the AI Resource Center (`airc.nist.gov`). Treat the Playbook — not AI 100-1 itself — as your day-to-day reference. AI 100-1 is the treaty; the Playbook is the RFC.

## The translation table you actually build

Below is the shape of the translation table this module trains you to produce. It is populated for a handful of MEASURE and MANAGE sub-categories to show the pattern; you will complete it in `exercise-01`.

| RMF sub-category (short) | Concrete test | Threshold / pass rule | Guardrail | Monitor | Evidence artifact |
|---|---|---|---|---|---|
| **MEASURE 1.1** — Approaches and metrics chosen are documented | Traceability doc: metric → risk it addresses → owner | Every risk in the register has ≥1 measured metric | n/a | n/a | `risk-metric-map.md` versioned in the risk-engineering repo |
| **MEASURE 2.3** — AI system performance evaluated in conditions similar to deployment | Shadow-eval on production-shaped traffic replay | Metric delta vs. offline eval within a budget you defend | Fall back to canary if delta exceeds budget | Continuous shadow-eval job | Shadow-eval run report + config hash |
| **MEASURE 2.7** — AI system security and resilience evaluated | Adversarial-ML + LLM red-team suite (mods 104-105) | Attack-success-rate ≤ target on named suite | Input filters, output validators (mod-108) | Prompt-injection detection metric fed into risk register (mod-109) | Signed red-team report, suite version pinned |
| **MEASURE 2.11** — Fairness and bias evaluated | Fairlearn / AIF360 slice report on protected groups defined by the harm model | Group-fairness metrics within band; disparate-impact ≥ 0.8 (only where legally sound) | Reweighing / post-processing (mod-106) | Group-level accuracy drift monitor | Bias audit report; if NYC LL144 applies, the AEDT bias-audit summary |
| **MEASURE 3.2** — Risk-tracking approaches consider negative impacts and unforeseen impacts | Near-miss reporting hook in trace store | Every guardrail hit above severity S logged as candidate incident | n/a | Near-miss dashboard | Incident-candidate queue in incident-response repo (mod-111) |
| **MANAGE 2.2** — Mechanisms are in place to sustain the AI system's value | Post-deploy regression fixture regenerated from every confirmed incident | Fixture green on every release | Rollback hook on regression | Regression pass-rate over time | Fixture history in `regressions/` |
| **MANAGE 4.1** — Post-deployment monitoring plans are implemented | Drift + safety-monitor signal wired to risk register (mod-109) | Alert cadence and threshold per risk in register | Circuit-breaker on named guardrail failures | Signal-to-register pipeline | Post-market surveillance report (also serves EU AI Act Article 72 — see chapter 04) |

The point of the table is that **no sub-category is left as prose**. If you cannot fill the last four columns for a given sub-category — and the sub-category applies to the system in question — that sub-category is unowned engineering work. Flag it in the risk register.

## GOVERN vs. MAP vs. MEASURE vs. MANAGE for a level-25 engineer

Not every function is yours to own:

- **GOVERN** is owned by governance leadership (levels 50-60 — `senior-ai-governance-architect`, `head-of-ai-governance`). You *feed* it with control implementations and test-run evidence, but you don't author the policy taxonomy.
- **MAP** is where the harm model lives. The analyst (level 15, `ai-governance-analyst`) does system intake and inventory; you (level 25) author the harm model on top of that intake in mod-102. The Playbook's MAP 1.x and MAP 5.x actions are the ones you translate into a repo of harm-model files.
- **MEASURE** is *your* function. Every eval library taught in mods 104-108 exists to close a MEASURE sub-category.
- **MANAGE** is co-owned. You engineer the mitigations, monitors, and incident hooks (mods 108, 109, 111). The prioritisation and treatment-plan decisions escalate to the AI risk council (mod-112) and — for release-gate decisions — to `ai-evaluation-engineer` at level 35.

## What "reading it as an engineer" looks like in practice

An engineering read of a sub-category is *five sentences*:

1. **What the sub-category says.** One sentence quoting the outcome statement.
2. **What that means for this system.** One sentence naming the metric or test that would demonstrate the outcome for the specific AI system in scope.
3. **How we test it.** One sentence naming the tool, dataset, or eval that runs the test.
4. **What we treat as pass.** One sentence naming the threshold and where it is defined.
5. **What we file as evidence.** One sentence naming the artifact and where it lives.

If any of those five sentences comes out as "TBD" or "the vendor covers it," the sub-category is *open* — and open sub-categories are the first thing an auditor asks about.

## A minimal example — one sub-category, end to end

Take MEASURE 2.7 for a customer-support LLM assistant.

1. *Says:* AI system security and resilience are evaluated and documented.
2. *Means:* the assistant does not comply with attempts to disclose secrets from the RAG store, exfiltrate the system prompt, or execute prompt-injected instructions embedded in retrieved documents.
3. *Test:* a garak + PyRIT + AgentDojo (indirect prompt injection) suite pinned at a versioned commit hash, run on every model candidate and on every prompt / retriever change; the prompt-injection portion additionally borrows from OWASP LLM Top 10 categories LLM01 (prompt injection) and LLM06 (sensitive information disclosure).
4. *Pass:* attack-success-rate ≤ the target set in the risk register for this system; regression fixture from every confirmed jailbreak (mod-111) is green.
5. *Evidence:* signed suite report at `evidence/measure-2.7/<release-tag>/report.json`, plus the run's config hash and the commit hash of the suite, both referenced from the release's evidence manifest.

That five-line pattern is what the "obligation-to-artifact map" exercise trains.

## Where this maps into the rest of the module and track

- Chapter 02 — the NIST GenAI Profile (AI 600-1) as the 12-risk starting checklist for the harm model that MAP asks for.
- Chapter 03 — ISO/IEC 42001 (AIMS), 23894, 42005, and 25059 as the standards that consume this evidence and produce a certifiable management-system record.
- Chapter 04 — EU AI Act articles as the *regulatory* consumer of the same evidence.
- Chapter 05 — sector and geography flavours (SR 11-7, FDA GMLP + PCCP, Colorado AI Act).
- Chapter 06 — role scope and the deferral contract.

## Summary

- AI RMF 1.0 (AI 100-1) has four functions: GOVERN, MAP, MEASURE, MANAGE. You own MEASURE; you co-own MAP and MANAGE.
- The NIST AI RMF Playbook is your day-to-day reference — it enumerates sub-categories and their suggested actions.
- Every sub-category in scope must translate to a *test, threshold, guardrail, monitor, and evidence artifact*. Anything unfilled is open work.
- The engineering read of a sub-category is five sentences: says / means / test / pass / evidence.
- Exercise 01 puts you through that translation on a real system.
