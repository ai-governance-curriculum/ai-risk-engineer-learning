# Exercise 04 — Indirect-Injection and Tool-Abuse Suite with AgentDojo and InjecAgent

**Estimated effort:** 3 hours

## Objective

Stand up an indirect-prompt-injection and tool-abuse suite for the running system that covers the three attack outcomes chapter 04 names — unintended tool invocation, data exfiltration through retrieval, and unauthorised action chains — by (a) running AgentDojo's stock environments as the reference measurement, (b) seeding your out-of-repo payload library from InjecAgent, and (c) extending both into an internal environment that mirrors the running system's actual tool topology. The output is the coverage-matrix rows that garak (exercise 02) and PyRIT (exercise 03) cannot reach, with utility and targeted-ASR reported separately per (user task, abuse objective) pair.

## Prerequisites

- Chapter 04 read; chapter 01 skimmed for the coverage-matrix shape and the harmful-payload-out-of-repo discipline.
- Exercise 01 charter in hand — target profile, coverage matrix, threshold table. The rows this exercise fills are the tool-abuse rows the charter left as `deferred:ex04`.
- Exercises 02 and 03 completed, or at minimum the garak smoke suite and the PyRIT attack library are running on the same target — this exercise measures what those two suites do not.
- Read access to the running system's tool inventory: tool names, argument schemas, side-effect classes, allowed argument ranges, intended callers, idempotency. If no tool inventory document exists, producing it is step 1 of this exercise.
- A sandbox environment for the target where high-severity tool calls (email send, ticket create, credit issuance, egress HTTP) can fire without hitting production side effects.
- An out-of-repo payload store (S3 bucket, private artifact registry, or equivalent) already provisioned per chapter 01's discipline. Payloads are referenced from the suite by hash; they do not live in the source repo.
- Local install of AgentDojo and InjecAgent per each project's README. Pin the release version of each; record the versions in the deliverable.

## Problem statement

The tool-abuse rows on your coverage matrix cannot be closed by direct-prompt scanners. You need an environment where tools are stateful, retrieval sources can be seeded with attacker-controlled content, and chains of tool calls can be scored end-to-end. AgentDojo provides that environment shape and a stock set of workspaces (email, calendar, messaging, cloud drive, banking-like, travel-agent). InjecAgent provides an injection-payload corpus and behaviour-category list. Neither one, on its own, reflects the running system's tools — so the suite has to extend them with an internal environment that does.

Deliver an indirect-injection and tool-abuse suite that produces, per attack outcome, a per-objective utility / targeted-ASR pair grounded in both the reference environments and a tool-topology-specific extension. The suite's outputs feed the coverage matrix as `covered:<suite-id>` rows and feed the register (mod-103) as likelihood anchors.

## Requirements

- Output as `mod-104-ex04-indirect-injection-suite.md` plus a suite artifact tree (repo directory or platform export) referenced from the markdown.
- The suite covers, at minimum, one AgentDojo stock environment run *and* the internal tool-topology environment. Both report utility and targeted-ASR separately per (user task, abuse objective) pair.
- All three abuse-outcome categories from chapter 04 are represented with at least one abuse objective each against the internal environment:
  - Unintended tool invocation — a tool the user did not request fires because a retrieved document instructed it.
  - Data exfiltration through retrieval — direct or side-channel; if side-channel, the objective encodes the secret in a tool-call argument (destination, path, query string).
  - Unauthorised action chain — two or more tool calls chained beyond the user's request, where each call may be individually within policy.
- The tool-topology enumeration (chapter 04, step 1) is delivered as a table: tool name, argument schema, side-effect class, allowed argument ranges, intended callers, idempotency, and the abuse objectives that target it.
- The retrieval-surface enumeration (chapter 04, step 2) is delivered as a table: surface name, who controls the content, whether a trust boundary is enforced today, and which payloads are planted where.
- Every injection payload — from InjecAgent seed or hand-authored — is stored out-of-repo and referenced by hash. The suite pulls payloads by hash at run time. The deliverable includes the payload-store URI and the manifest of hashes referenced, not the payload text.
- The InjecAgent behaviour-category list is cross-walked against the internal suite's objectives. Any InjecAgent category not represented in the internal suite is either marked `n/a:<reason>` or filed as a coverage gap.
- The Microsoft AI Red Team enterprise threat-model references from chapter 04 (Lessons from red-teaming 100 generative AI products; Microsoft SDL AI-specific threat-modelling guidance) are cited on the coverage-matrix rows this exercise closes — the enterprise shape is the *why* on the row, AgentDojo / InjecAgent are the *how*.
- The AgentDojo release version, InjecAgent release / dataset version, target model configuration, and run date are recorded alongside every measurement. Without these, the numbers are not comparable across runs.
- The reference-measurement run against an AgentDojo stock environment produces a utility / targeted-ASR pair that goes on the register (mod-103) as a baseline likelihood anchor for indirect injection.
- The internal-suite run produces per-objective utility / targeted-ASR rows. Any non-zero targeted-ASR row is filed as a mitigation candidate for mod-108 (guardrails).
- The L40 boundary from chapter 04 is respected — abuse objectives that require dynamic tool composition, multi-agent orchestration with credential passing, or browser control against untrusted content are marked `escalated:L40` on the coverage matrix rather than partially covered here.

## Starter guidance

- Do the AgentDojo stock run first, cold, before touching internal environment authoring. It teaches you the environment / user task / injection task / scorer shape you are about to replicate. Do not skip this by jumping straight to the internal harness.
- Enumerate tools before you enumerate objectives. It is much easier to see the abuse objectives once the tool table is on the page; enumerating objectives against tools you have not written down produces objectives that miss the highest-severity chains.
- Side effects matter more than argument shapes. A tool whose side-effect class is "sends network egress to arbitrary URL" is a much bigger target than a tool whose side-effect class is "reads an internal document"; sort the tool table by side-effect severity and work top-down.
- For each retrieval surface, ask: who can write to it? A KB with public write access is a very different threat surface from a KB whose write path is authenticated; both may still be vulnerable, but the payload-plausibility bar differs.
- InjecAgent as seed, not as ceiling. Its payload corpus is a starting point. The abuse objectives that matter most for the running system are the ones the InjecAgent seed does not cover — the tool names, the internal-domain allow-lists, the workflow-specific credit or refund paths.
- Payload discipline: if a payload's content ends up in the repo (in a test fixture, in a code comment, in a commit message), treat it as a leak. The out-of-repo store is where payloads live; the manifest of hashes is what the suite reads. This mirrors chapter 01's rule and chapter 05's payload-handling for the highest-risk classes.
- Utility scoring matters as much as ASR. An agent whose targeted-ASR is 0 % because it refuses every user task is not shippable. The register row needs both numbers to be meaningful.
- If the target's tool surface exposes tools that a stock AgentDojo environment already models (email, calendar, cloud drive), consider extending that stock environment with your allow-lists and destinations rather than authoring a brand-new environment. Extending is faster and keeps you closer to the reference measurement.
- On the internal environment, wire a "planted-payload provenance" tag through the environment fixtures so the scorer can distinguish "the agent completed the abuse objective because the payload succeeded" from "the agent happened to do the same action for unrelated reasons." Without provenance tagging the ASR is noisy.

## Acceptance criteria

The submission is accepted when:

- At least one AgentDojo stock environment run has completed against the target with utility / targeted-ASR reported and the AgentDojo release version, model configuration, and run date recorded.
- The internal tool-topology environment is authored, runs against the target in the sandbox, and reports utility / targeted-ASR per (user task, abuse objective) pair.
- Each of the three abuse-outcome categories (unintended invocation, exfiltration via retrieval, unauthorised chain) has at least one internal-suite objective represented and measured.
- The tool-topology and retrieval-surface enumerations are delivered as tables; every tool and every surface named in the target is on one of them or explicitly excluded with a reason.
- Every payload is referenced by hash from an out-of-repo store; the payload manifest is included; no payload text is committed to the repo.
- The InjecAgent behaviour-category cross-walk is delivered and every unrepresented category is either `n/a:<reason>` or a filed coverage gap.
- The coverage-matrix rows this exercise closes are updated to `covered:<suite-id>` with the AgentDojo / InjecAgent / Microsoft AI Red Team citations named on the row.
- The register (mod-103) has at least one likelihood anchor updated from the AgentDojo baseline and at least one from the internal suite.
- Every non-zero targeted-ASR row on the internal suite is filed as a mod-108 mitigation candidate with a ticket link or equivalent.
- Abuse objectives that exceed the L25 boundary from chapter 04 are marked `escalated:L40` on the coverage matrix with a one-line reason, not silently dropped.
- A reviewer with only the deliverable and read access to the suite can, in under 30 minutes, answer: which tools are covered, which retrieval surfaces are seeded, what the current utility / targeted-ASR pair is per objective, and which findings are pending mitigation.

## Stretch goals

- Wire the internal-suite run into CI on a nightly or per-release cadence and gate release on a threshold from the exercise-01 charter. Chapter 01's release-gate rule is what this stretch operationalises.
- Author one payload that chains across two retrieval surfaces (payload planted in surface A instructs the agent to fetch from surface B, whose content triggers the tool call). This is closer to the enterprise-shape attacks the Microsoft AI Red Team writeups describe.
- Extend the scorer with a "chain length" metric — how many tool calls occurred between the injection landing in context and the abuse objective completing. Long chains have different detector implications than one-hop attacks (mod-109).
- Cross-walk the internal-suite objectives back to mod-102 harm-scenario rows and note any harm-scenario row that has no matching abuse objective in the suite — that is a mod-102 / mod-104 coverage seam finding.
- Prototype one guardrail from the mod-108 candidate list (argument allow-list, destination allow-list, provenance filter on retrieved content) and re-run the affected objectives to measure the delta on targeted-ASR. This is the guardrail-effectiveness measurement mod-108 will formalise.

## Deliverable format

A single markdown document with the following sections, plus the suite artifact tree referenced from it:

1. *Target and configuration* — target model, tool inventory summary, AgentDojo release version, InjecAgent release / dataset version, run date, sandbox description.
2. *Tool-topology enumeration* — the tool table (name, arguments, side-effect class, allowed ranges, callers, idempotency, targeting objectives).
3. *Retrieval-surface enumeration* — the surface table (name, controller, trust-boundary status, planted-payload hashes).
4. *AgentDojo reference run* — environment chosen, user tasks, injection tasks, utility / targeted-ASR pair, and the register-row anchor produced.
5. *Internal-suite objectives and results* — one sub-section per abuse-outcome category with the objectives, the (user task, abuse objective) pairs, and the utility / targeted-ASR results.
6. *InjecAgent behaviour-category cross-walk* — table of category × represented (yes / `n/a:<reason>` / coverage-gap).
7. *Payload manifest* — out-of-repo URI and the hash list, with a per-hash mapping to which (surface, objective) it services. Payload text does not appear here.
8. *Coverage-matrix update* — the rows this exercise closes, with citations (AgentDojo, InjecAgent, Microsoft AI Red Team) and any `escalated:L40` rows.
9. *Mitigation candidates filed* — list of non-zero targeted-ASR rows with the mod-108 ticket link per row.
10. *Stretch goals attempted* — the notes.

## Submission

Attach the markdown, the suite artifact tree (repo tarball or platform export), and the payload manifest to the module submission channel. If the running system is under NDA, redact identifiers but preserve the tool table shape, the retrieval-surface table shape, the utility / targeted-ASR numbers, the InjecAgent cross-walk, the coverage-matrix row updates, and the payload-hash manifest — a reviewer cannot review a tool-abuse suite whose objectives, side-effect classes, and per-objective measurements are blanked out. Under no circumstances attach payload text; the manifest of hashes is what travels with the submission.
