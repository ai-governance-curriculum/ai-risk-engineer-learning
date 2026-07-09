# Causal Attribution: Operator, Model, Data, and Integration Buckets

Chapter 01 required every harm-model row to carry a `cause_bucket`. This chapter defines the four buckets, tells you how to pick the right one, and explains why the choice determines *who owns the mitigation*. Mis-attribution is the single most common reason a harm goes unmitigated — because it sat in the wrong queue and nobody claimed it.

## Motivation

An AI harm never lands "because of the AI." It lands because someone configured the operator surface a certain way, because the trained model has a certain behaviour, because the data flowing through the system carries a certain property, or because the system was integrated into an environment where its outputs get consumed a certain way. Those four proximate causes route to four different owners inside your organisation. If the harm-model row doesn't name which one is at play, the mitigation ticket has no natural home and the fix stalls.

Attribution also matters for *audit narrative*. When a regulator asks "why did this happen," an unclear attribution reads as "we don't know." A clean attribution reads as "the operator failed to enforce a policy X, we identified it as an operator-caused risk in row Y, and we shipped mitigation Z." Same underlying incident, different narrative posture.

## The four buckets

### Operator-caused

Harm scenarios that would not occur if the operator (your organisation, the deploying team) had configured, monitored, or governed the system differently.

Examples:
- The system prompt was rewritten without review and disabled a refusal.
- A canary release was promoted to full deployment without waiting for the scheduled monitor window.
- The abuse-report queue was routed to a mailbox nobody read.
- A model-version pin was bumped without regenerating the regression fixtures.
- Retention policy was silently relaxed by ops during a capacity crunch.

Mitigation owner: the operator side — depending on the harm, this is the risk-engineering role you occupy, the operations team, the security team, or governance. The mitigation is usually a policy plus enforcement (e.g., protected system-prompt file with change-control; canary-window enforcement in the release pipeline; monitor spec that alerts on unread abuse-queue accumulation).

### Model-caused

Harm scenarios where the trained model's *behaviour* is the proximate cause — the model produces harmful output because of what it learned, its calibration, or its coverage.

Examples:
- Hallucinated procedure that a human-in-the-loop accepted because it was plausibly worded.
- A classifier's decision boundary systematically disadvantages a protected group.
- The model refuses harmless requests in a way that pushes users to bypass the guardrail.
- The model's tool-use planner produces valid-looking calls that misuse a downstream API.
- Memorised training-data exposure through a specific prompt pattern.

Mitigation owner: the ML side — the model developer (internal or external), the fine-tuning team, the eval-engineering peer (level 30) who owns methodology. Your role authors the *risk contract* — what the model behaviour must satisfy for release — and consumes their evidence. For frontier-model consumers, some model-caused harms are only mitigable at the provider (chapter 04) and your role is to document the residual and route it to product / legal.

### Data-caused

Harm scenarios where the data (training data, fine-tuning data, retrieval corpus, feedback / RLHF preference data, evaluation set, or online drift) is the proximate cause.

Examples:
- Retrieval corpus contains an author's indirect-prompt-injection strings.
- Training data over-represents one demographic and under-represents another.
- Feedback loop trains on model-produced content and homogenises the model.
- An eval set was contaminated by inclusion in a pre-training corpus.
- A synthetic-data generator's distribution drifts over time.
- A vendor-supplied labelling batch has systematic bias.

Mitigation owner: the data-side owners — data engineering, data governance, the vendor management function (for supplied datasets), the retrieval-pipeline owner. Your role authors the *data-quality risk contract* — provenance requirements, contamination checks, drift monitoring — and consumes their evidence. For upstream data suppliers, some data-caused harms are only mitigable at the supplier and residual routing is again the play.

### Integration-caused

Harm scenarios where the *integration* — how the model is composed into a larger system, how its outputs are consumed by downstream components, or how it is embedded in a workflow — is the proximate cause.

Examples:
- Advisory recommendation is auto-approved by a downstream system that treats the output as authoritative.
- Model output is concatenated into another model's prompt without sanitisation.
- Human-oversight surface is designed so the operator rubber-stamps every recommendation.
- A tool the model can call has an over-broad permission set.
- Model output is rendered as HTML into a console and the rendering path allows script execution.
- A partner product surfaces your model's suggestions as "expert advice" while stripping the caveats.

Mitigation owner: the integration side — the application team, the downstream product team, the platform team that owns tool topology, the UX / design team that owns the oversight surface. Your role authors the *integration risk contract* — trust boundaries, output handling, oversight-design principles — and consumes their engineering.

## Picking a bucket cleanly

Some scenarios touch multiple buckets. Attribute to the one where *removing the cause would remove the harm at least cost.*

- If the injected KB article is the direct source of the exfiltration, and the tool-call permission is the enabler — attribute to **integration** (the retrieval trust boundary), not to **data** (an infinite content-scrubbing job). Data provenance is a defence-in-depth layer, not the primary attribution.
- If the hallucinated procedure lands harmfully because the human agent has no way to verify it — attribute to **integration** (oversight design), even though the model is the proximate producer of the wrong content. Model quality is a defence-in-depth layer, not the primary attribution.
- If a fine-tuning batch was silently swapped and the model now over-refuses — attribute to **operator** (change-control failure), not to **model** (the resulting model is a symptom).

Attribution follows *the intervention that ends the scenario cheapest.* Write that reasoning into the row alongside the bucket tag — the reviewer will read it.

For scenarios where two interventions are genuinely both required, use *two rows* with the same `id_group`. Do not collapse them; you will lose a mitigation.

## The mitigation-contract format

Once a row is attributed to a bucket, the *mitigation contract* is a short block naming:

| Field | Content |
|---|---|
| `cause_bucket` | Operator / model / data / integration |
| `mitigation_owner` | Named role (not a person — the role) that owns the fix |
| `deliverable` | The concrete artifact the owner ships (guardrail config, monitor spec, dataset provenance record, tool ACL, oversight-surface redesign) |
| `deadline` | The date by which the deliverable must exist for the row to move out of *open* |
| `evidence_manifest_ref` | Where the evidence lives once shipped |

The contract format is boring on purpose. Boring contracts get honoured; ornate contracts get argued over.

## Worked example — attributing HR-CS-INJ-01

Continuing the customer-support RAG assistant's indirect-prompt-injection row:

| Field | Value |
|---|---|
| `cause_bucket` | Integration (retrieval trust boundary + tool-call allow-list gap) |
| `mitigation_owner` | Application team (implementation) + you (spec / eval) |
| `deliverable` | (1) provenance filter that strips known injection markers; (2) tool-call allow-list preventing arbitrary `escalate_ticket` recipients; (3) monitor for `escalate_ticket` calls with unusual recipient domains |
| `deadline` | Target date set at register-write time (mod-103) |
| `evidence_manifest_ref` | `evidence/harm/HR-CS-INJ-01/<release>/manifest.json` |

The row is now routable. The application team gets an issue with the deliverable and deadline; you get a review of the mitigation once it lands.

## When the mitigation owner is *outside* the organisation

Sometimes the fix genuinely lives with a third party — a model vendor, a data supplier, a downstream integrator you do not control. In that case:

- Attribute to the correct bucket regardless (do not distort attribution to keep the fix in-house).
- Add a *residual owner* — an internal role responsible for *carrying* the residual risk until the third party ships the fix, or until you route around them.
- Route to `senior-ai-governance-architect` (level 50) if the residual is broad enough to affect multiple systems.
- Record the third-party dependency and the anticipated close date; mod-110 (MRM + supply-chain) will pick up the ongoing accounting.

Residual routing is not defeat. It is honest attribution plus honest accounting.

## Attribution is a checkable claim

A useful test: for every row, ask *what one change would make this row obsolete?* If the answer is a policy change, the row is operator-caused. If the answer is a model behaviour change, it is model-caused. If it is a data-quality change, it is data-caused. If it is a system-boundary or downstream-behaviour change, it is integration-caused. If the answer is "many things across many owners," the row is under-specified — split it.

That test is what a reviewer applies. Apply it to yourself first.

## Why this matters for the register (mod-103)

The register (mod-103) uses `cause_bucket` and `mitigation_owner` to sort risks by owner and by kind. Ownership dashboards read this attribution directly. Incident retrospectives (mod-111) cite the attribution when explaining *why the mitigation was where it was*. Getting the attribution right at the harm-model stage saves the register from being a fog of unassigned rows.

## Summary

- Every harm-model row carries one primary `cause_bucket`: operator, model, data, or integration.
- Attribution routes ownership: operator to your side; model to ML / eval-engineering; data to data governance; integration to application / integration teams.
- Pick the bucket where *removing the cause ends the harm at least cost*; use two rows with a shared `id_group` when two interventions are both required.
- The mitigation contract is a small block: bucket, owner, deliverable, deadline, evidence path. Boring on purpose.
- When the mitigation lives with a third party, attribute correctly and add a *residual owner* on your side; mod-110 accounts for the residual.
- Applying the "what one change ends this scenario?" test to each row catches under-specified rows before a reviewer does.
