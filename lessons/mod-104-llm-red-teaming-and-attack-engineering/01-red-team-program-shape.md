# The LLM / Agent Red-Team Programme as an Engineering Artifact

A red-team programme for an LLM or agent system is not an event on a calendar. It is a versioned engineering artifact — a repo, a coverage matrix, a suite of runnable attacks, a triage pipeline, and a replay bundle — that the risk register (mod-103) cites, that CI runs on release candidates, and that an auditor can inspect. This chapter pins the shape of that artifact, using the OWASP GenAI Red-Team Guide as the reference frame and the harm model from mod-102 as the input contract.

## Motivation

Two failure modes an auditor immediately spots when they open a red-team folder:

- **The event-shaped programme.** "We did a red-team in Q2." A vendor came in, produced a slide deck, the deck was archived, no artifact from it lives in the risk-engineering repo, and nothing runs on the next release candidate. The auditor asks *"how do you know that finding is still fixed?"* — no answer.
- **The screenshot-shaped programme.** A folder of chat transcripts and screenshots, one per interesting jailbreak somebody tried. No target definition, no coverage claim, no reproduction path, no threshold. The auditor asks *"which risk register row does this cover, and would it re-fire if I ran it now?"* — no answer.

The engineering way out is to model the red-team programme itself: enumerate its fields, run it on a cadence, keep its findings tied back to the harm model, and version its payloads out-of-repo so the repo stays safe to hand to a new hire. That is what this chapter pins.

## The five artifacts a shippable programme carries

A red-team programme for one target — one system, one release — is a set of five artifacts. If any of them is missing the programme is a demo, not a production risk-engineering asset.

| Artifact | What it holds | Where it lives |
|---|---|---|
| **Target profile** | The system under test — model(s), prompts, tools, retrieval sources, integration surface, deployment tier | `red-team/targets/<target-id>.yaml` |
| **Coverage matrix** | Techniques × capability surfaces × harm-scenario rows the programme claims to cover | `red-team/coverage/<target-id>.md` |
| **Attack suite** | The runnable attacks — probes, orchestrators, seeds, scorers — pinned at a commit hash | `red-team/suites/<suite-id>/` |
| **Replay bundle** | Every triaged finding as a reproducible payload + expected-behaviour spec + provenance | out-of-repo evidence store, referenced by hash |
| **Triage log** | Per-finding disposition (true positive / false positive / duplicate / accepted / mitigated), owner, register row, regression fixture | `red-team/triage/<target-id>/<run-id>.md` |

The five artifacts compose. The target profile pins *what*. The coverage matrix pins *how much*. The attack suite pins *the runnable form*. The replay bundle pins *what was found*. The triage log pins *what was done about it*. A programme without all five has a hole an auditor will land on.

## The target profile

The target profile is a one-page-per-system contract that the programme is scoped to. It names:

- **System identifier and version** — the system under test at a specific release, model, and prompt commit.
- **In-scope surfaces** — text generation, retrieval, tool calling, memory / state, multimodal input, output rendering, human-oversight surface, admin surface. Same axis list as the capability-surface table in mod-102 chapter 01.
- **Out-of-scope surfaces** — the surfaces you explicitly do not test in this run (e.g., billing API integration handled by a separate SecOps engagement). Out-of-scope is *named*, not implied.
- **Personas modelled** — which of the four mod-102 personas (adversarial user, insider, upstream data supplier, downstream integrator) the suite exercises.
- **Deployment tier** — the enterprise deployment tier borrowed from the frontier-lab RSP-shape work in mod-102 chapter 04. Tier drives suite depth and cadence.
- **Model + prompt + tool inventory** — model IDs, prompt commit, tool schemas at the version under test. This is what makes replay possible six months later.
- **Data provenance for test inputs** — where the seed prompts come from, whether harmful strings are stored in-repo (never) or in the out-of-repo payload store (yes; see the harmful-payload hosting section below).
- **Success criteria** — the pass rule for release. Almost always framed as attack-success-rate ceilings on named categories, not a single pass/fail.

The target profile is what makes the programme *this-system-specific*. A generic programme is a compliance decoration; a target-profile-anchored programme is engineering.

## The coverage matrix

The coverage matrix is a two-dimensional table: **attack technique classes** down the rows, **capability surfaces** across the columns. Each cell is one of:

- `covered:<suite-id>` — a runnable attack in the suite exercises this technique against this surface.
- `n/a` — technique does not apply to this surface for this system.
- `deferred:<reason>` — the technique applies but is out-of-scope for this run; a follow-up ticket exists.
- `escalated:L40` — the technique is frontier-agent depth and is handed off to `agentic-safety-engineer` at level 40 (see chapter 06).

A working technique-class list at this level of maturity — the same list the OWASP GenAI Red-Team Guide expects programmes to walk against — covers:

| Technique class | Chapter |
|---|---|
| Direct prompt injection | ch 05 |
| Indirect prompt injection (retrieval / tool output) | ch 04 |
| Encoding / obfuscation attacks | ch 05 |
| Multi-turn refusal-bypass | ch 05 |
| Role-play / persona-hijack (DAN-family) | ch 05 |
| GCG-style adversarial suffixes | ch 05 |
| Many-shot in-context jailbreaking | ch 05 |
| Unintended tool invocation | ch 04 |
| Data exfiltration through retrieval | ch 04 |
| Unauthorised action chains | ch 04 |
| Sensitive-information disclosure (system prompt, credentials, KB secrets) | chs 02, 05 |
| Refusal robustness on policy-negative asks | chs 05, 06 |

Cells left blank are audit findings. Cells marked `deferred` need a linked follow-up. The coverage matrix is the artifact you show a reviewer when they ask *"what does this programme claim to cover?"*.

## The attack suite

The attack suite is the runnable code. In practice at this level of maturity it is a composition of four kinds of authoring:

- **Scanner-shaped** — NVIDIA garak (mod-104 ch 02), providing a first automated pass across a library of probes and detectors. Scanners run per commit and produce the baseline noise floor.
- **Orchestrator-shaped** — Microsoft PyRIT (mod-104 ch 03), providing scripted attack authoring with prompt converters, targets, orchestrators, and scorers. Orchestrators encode the multi-turn and converter-chained attacks that scanners do not express.
- **Environment-shaped** — AgentDojo, InjecAgent (mod-104 ch 04), providing benchmark environments for indirect-prompt-injection and tool-abuse evaluation on agentic systems.
- **Harness-shaped** — Inspect (UK AISI) (mod-104 ch 06), providing the eval-authoring framework that hosts your policy-derived internal suite and your regression fixtures on the same substrate.

The suite is pinned at a commit hash. Every run records the suite hash, the target hash, and the model / prompt / tool inventory. Without those three hashes, "we ran the red-team" is not reproducible.

## The replay bundle

Every finding — every payload that produced a policy-negative response, an unintended tool call, or a refusal-bypass — is captured as a *replay bundle*:

- **Payload** — the input string(s) or file(s), often base64-wrapped and marked as `WARNING: harmful content, do not render`. Payload is stored in the out-of-repo payload store (see below), not in git.
- **Provenance** — where the payload came from (a suite probe, a specific PyRIT orchestrator run, a hand-crafted variant, a published academic attack).
- **Target snapshot** — the target profile hash under which it fired.
- **Observed behaviour** — the model / agent's actual response, with any tool calls, retrieved chunks, and downstream side effects logged.
- **Expected behaviour** — the refusal, filter block, tool-call refusal, or bounded response that the policy required.
- **Severity and triage disposition** — set by the triage step below.

The replay bundle is what a regression fixture (mod-111) is built from. Every confirmed jailbreak becomes a permanent CI check that the fix stays fixed.

## The triage log

Every scanner or orchestrator run produces a list of *candidate findings*. Triage is the step that turns candidates into engineering work items.

A triage row for one finding carries:

- **Candidate id** — stable, e.g. `RT-CS-2026Q3-0142`.
- **Suite + run hash** — the run that produced it.
- **Replay bundle hash** — the payload it points at.
- **Disposition** — true positive / false positive / duplicate / accepted / mitigated.
- **Severity** — from the harm model's impact axis (mod-103).
- **Register row** — the mod-103 register row this finding attaches to. New categories may drive new rows.
- **Regression fixture id** — the mod-111 fixture built from the finding, once the fix is in.
- **Owner** — the engineer accountable for the fix or the accept decision.

A triage log without dispositions is a shame log. A triage log with dispositions but no register-row references is a disconnected artifact. Both patterns show up in first-pass programmes; both are audit findings.

## Harmful-payload hosting — out of the repo

A load-bearing operational discipline: **harmful payloads do not live in the source repository**. Reasons the discipline is non-negotiable:

- Cloning the repo should never expose a new engineer to CSAM-adjacent, self-harm-inducing, or CBRN-uplift payloads.
- Public mirroring (open-source projects, forks, backups) should never surface these strings.
- Search and grep across the codebase should not surface them either — many payloads are designed to be persuasive against humans, not just models.
- The strings themselves may violate content policies of the code-hosting provider.

The practical shape:

| Where | What goes here | Access |
|---|---|---|
| In-repo | Suite code, converters, probe definitions, coverage matrix, triage log, target profile, hashes | Anyone with repo access |
| Out-of-repo payload store (private object store or dedicated vault) | Raw payload strings, replay bundles, transcripts of successful attacks | Named allow-list; access-logged |
| Signed manifest in-repo | Payload hashes and their store URIs | Anyone with repo access; the manifest links code to payloads without exposing them |

The out-of-repo store is the *canonical* home for harmful strings. When the suite runs, it pulls payloads from the store using its hash, exercises them against the target, and never writes them back into the repo. The Inspect harness (ch 06) is one implementation of this pattern; garak's probe corpus is another (probes ship the payloads bundled into the library, not into your project repo).

Chapter 05 revisits the discipline in the specific context of GCG suffixes and many-shot payloads, where the operational risk of leaking the string is highest.

## Cadence

A red-team programme runs on more than one cadence:

| Cadence | Trigger | Suite subset |
|---|---|---|
| **Per commit / per PR** | Model, prompt, tool, or retriever change | garak smoke suite (ch 02); regression fixtures (mod-111) |
| **Per release candidate** | Release-gate check | Full scanner suite + PyRIT orchestrator suite + AgentDojo / InjecAgent subset relevant to tools in use |
| **Quarterly deep run** | Scheduled | Full suite + policy-derived internal suite + GCG re-tuning (ch 05) against the current model |
| **Event-triggered** | New attack class published, model provider swap, capability-surface expansion | Targeted suite additions + full re-run of affected coverage cells |

Cadence maps to register-row review triggers in mod-103 chapter 06. A red-team programme that only runs quarterly cannot support the per-release evidence artifact that mod-101 chapter 01 asks for on MEASURE 2.7.

## The threshold you release against

A pass rule for the release gate does not read as "no findings." It reads as an attack-success-rate ceiling on each named technique class, sized against the risk register's residual-appetite bands.

An example shape (numbers are illustrative and system-specific):

- Direct prompt injection on policy-negative asks: ASR ≤ target set in the register for this tier.
- Indirect prompt injection through retrieval: ASR ≤ target, plus zero unintended `escalate_ticket` calls in the AgentDojo variant.
- Refusal-bypass on the internal policy-derived suite: ASR ≤ target.
- Regression fixtures (mod-111): 100% pass. Any regression fixture failing is an automatic block.

The threshold is written down, versioned with the rubric, and referenced from both the target profile and the register row it defends. If the threshold moves, the register-row sign-off refreshes (mod-103 ch 06).

## Programme ownership and boundaries

At level 25 you *own* the red-team programme for the target — the artifacts, the suite composition, the triage pipeline, the replay-bundle hygiene, and the threshold contract. You do not own:

- **App-side safety measurement.** That is `ai-eval-engineer` at level 30. They consume your suite outputs and integrate them into the app's release-gate metrics. Chapter 06 draws the boundary.
- **Frontier-agent depth.** Deep multi-agent planning attacks, sub-agent spawning attacks, and capability-elicitation studies against frontier models with novel tool topologies are `agentic-safety-engineer` at level 40. Chapter 06 also draws that boundary.
- **Classical application security.** Network exploitation, authentication bypass, RCE, or supply-chain compromise of the underlying stack route to SecOps. AI red-team covers AI-specific attack surfaces.

The coverage matrix's `escalated:L40` cells are the visible form of the second boundary. The out-of-scope surface list in the target profile is the visible form of the third.

## Interfaces with adjacent modules

- **Mod-102 (harm modelling)** — every row of the coverage matrix ties to a harm-model scenario row. If a red-team finding has no matching harm-model row, either the harm model needs an update or the finding is out-of-scope for this programme.
- **Mod-103 (risk register)** — every triaged true-positive updates a register row's likelihood anchor and, once mitigated, a residual-controls credit; every accepted finding is an appetite decision routed via mod-103 chapter 06.
- **Mod-108 (guardrails)** — mitigations for red-team findings are guardrail configurations; the red-team suite is the *effectiveness measurement* for those guardrails.
- **Mod-109 (monitoring)** — patterns from confirmed findings become production signals; the red-team is where the monitor's detector logic is first evaluated.
- **Mod-111 (incident response)** — every confirmed finding produces a regression fixture; every incident post-mortem produces a red-team suite addition. The back-feed loop is the how-we-learn artifact.

## Summary

- A red-team programme is five artifacts: target profile, coverage matrix, attack suite, replay bundle, triage log. Any missing artifact is an audit finding.
- The target profile pins *this system, this version, these surfaces, this tier*. Generic programmes are compliance decoration.
- The coverage matrix is techniques × capability surfaces; blank cells are audit findings and `escalated:L40` cells are the visible frontier-agent boundary.
- The attack suite composes scanner-shaped (garak), orchestrator-shaped (PyRIT), environment-shaped (AgentDojo, InjecAgent), and harness-shaped (Inspect) authoring.
- Harmful payloads live in an out-of-repo store; the repo holds hashes and a signed manifest.
- Cadence is per-commit smoke, per-release full, quarterly deep, and event-triggered — mapped to register-row review triggers.
- The threshold reads as attack-success-rate ceilings per technique class, sized against the register, not "no findings."
- Boundaries: `ai-eval-engineer` (L30) consumes; `agentic-safety-engineer` (L40) takes frontier-agent depth; SecOps takes classical AppSec.
- Exercise 01 turns this shape into a filled charter for the running system.
