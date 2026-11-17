# Lab 02 — Cause-Bucket Attribution Drill

**Duration:** 40 minutes · **Format:** guided practice, markdown authoring

## Objective

Internalise the chapter-05 four-bucket attribution model — **operator / model / data / integration** — and the "what one change ends this scenario?" test. You will triage eight pre-written one-line harm scenarios against a fixed toy system, picking one primary bucket per scenario (splitting into two rows with a shared `id_group` when the chapter-05 discipline requires it), writing the one-line cheapest-intervention reasoning, and naming the role that owns the mitigation. The lab is a drill for the attribution reflex that exercises 02–05 and the whole mitigation-contract pipeline assume you already have.

## Prerequisites

- Chapter 05 read. Chapter 01 (for cause_bucket as a required field) and chapter 06 (for L25 / L40 escalation shape) are useful but not required.
- Lab 01 is a helpful warm-up because it trains the chain reasoning that feeds cleanly into "what one change ends this scenario?" but is not strictly required.
- A terminal for creating a markdown file. No external access required.

## The fixed example system

Use the same `PolicyPal` toy from lab 01 (and from mod-101 labs). Short recap:

> **`PolicyPal`** — an internal HR-policy Q&A assistant: read-only RAG over the company's HR handbook, hosted LLM behind it, responses text-only in the employee intranet, no decisions automated, no PII inputs, no employee-record access.

This lab treats PolicyPal as deployed, under modest traffic, with a release pipeline, a release-pinned red-team suite, a trace store, and a standing on-call rotation.

## Classification taxonomy (chapter 05, reproduced for reference)

- **Operator-caused** — would not occur if the operator (your organisation) had configured, monitored, or governed the system differently. Owner: operator-side roles (risk engineering, ops, security, governance).
- **Model-caused** — the trained model's *behaviour* is the proximate cause — learned behaviour, calibration, coverage. Owner: ML side (internal / vendor), fine-tuning team, L30 eval-engineering peer; L25 authors the risk contract and consumes evidence.
- **Data-caused** — training, fine-tuning, retrieval, feedback, or eval data is the proximate cause. Owner: data-side roles — data engineering, data governance, vendor management for supplied datasets, retrieval-pipeline owner.
- **Integration-caused** — composition / consumption / embedding in a workflow is the proximate cause. Owner: application team, downstream product, platform-team-owning tool topology, UX / design for oversight surfaces.

Attribution rule: pick the bucket where **removing the cause would end the harm at least cost.** If two interventions are both required, use *two rows with a shared `id_group`*.

## The fixed scenario set

Below are eight one-line scenarios. Treat them as *post-exercise-01* harm-model rows that have not yet been cause-bucket-tagged. Every scenario is set against PolicyPal.

1. `HR-PP-OP-A` — During a capacity crunch, ops silently relaxed trace retention from 30 days to 48 hours; a user complaint comes in two weeks later and the originating session has already aged out.
2. `HR-PP-OP-B` — A prompt engineer edited the system prompt to drop a refusal line because it was "over-refusing," without opening a review PR; the production version now answers requests that previously refused.
3. `HR-PP-MD-A` — PolicyPal confidently hallucinates a parental-leave policy that does not exist; the employee quotes it in a Slack thread; the HR team has to issue a correction.
4. `HR-PP-DT-A` — An HR editor pastes a handbook update that contains a hidden instruction copied from a public help thread; the chunk is indexed and is later retrieved in a legitimate session, causing an attacker-aligned answer.
5. `HR-PP-DT-B` — A fine-tuning batch ordered from a labelling vendor systematically mis-tags parental-leave answers as "declines"; after fine-tune, PolicyPal over-refuses parental-leave questions.
6. `HR-PP-IN-A` — The intranet team adds an auto-forward integration that pipes PolicyPal answers into a Slack channel shared between the employee and their manager, stripping the shipped advisory caveat; a manager treats the stripped answer as authoritative HR direction.
7. `HR-PP-IN-B` — PolicyPal output is rendered as HTML in the intranet console; the rendering path does not sanitise, so an answer that quotes a malformed handbook snippet executes an embedded `onerror` handler in the support agent's browser.
8. `HR-PP-COMP` — An adversarial employee crafts a jailbreak preamble that the production model complies with *and* the system prompt that was edited in `HR-PP-OP-B` is the enabler (the dropped refusal would have blocked the jailbreak).

## Steps

### Step 1 — Create the triage file

```bash
mkdir -p mod102-lab02 && cd mod102-lab02
touch cause-bucket-triage.md
```

Paste this skeleton:

```markdown
# PolicyPal — cause-bucket attribution triage

Chapter reference: mod-102 ch 05.

| id | Scenario (one line) | Primary bucket | "What one change ends this scenario?" (≤ one sentence) | Mitigation owner (role) | Needs id_group split? |
|---|---|---|---|---|---|
| HR-PP-OP-A | ... | ... | ... | ... | ... |
| HR-PP-OP-B | ... | ... | ... | ... | ... |
| HR-PP-MD-A | ... | ... | ... | ... | ... |
| HR-PP-DT-A | ... | ... | ... | ... | ... |
| HR-PP-DT-B | ... | ... | ... | ... | ... |
| HR-PP-IN-A | ... | ... | ... | ... | ... |
| HR-PP-IN-B | ... | ... | ... | ... | ... |
| HR-PP-COMP | ... | ... | ... | ... | ... |
```

### Step 2 — Attribute each scenario

For each row, in order:

1. Re-read the scenario and apply the chapter-05 test: *what one change would make this scenario obsolete?* If the answer is a policy change, bucket is *operator*; a model behaviour change, *model*; a data-quality change, *data*; a system-boundary / downstream-behaviour change, *integration*.
2. Write the "what one change" sentence in the cheapest-intervention column. The sentence is as important as the bucket label — it is what a reviewer will read to agree with your attribution.
3. Name the mitigation owner by *role* (not a person) from the chapter-05 owner mapping. For PolicyPal, the plausible owners are: *risk engineering (you)*, *platform ops / release pipeline*, *application team (intranet)*, *HR content owners*, *data governance*, *ML / model vendor*, *UX / design*.
4. For any scenario where two interventions are genuinely required, mark the row as needing an `id_group` split and add a sibling row below it. Chapter 05 is explicit: "Do not collapse them; you will lose a mitigation."

Work through all eight. Resist two temptations:

- *Collapsing every scenario into "integration."* Integration is the easiest bucket to over-apply because most harms show up at an integration point. The test is not "where did the harm show up" — it is "where does the cheapest intervention live."
- *Attributing to "model" because the output was wrong.* The hallucination row (`HR-PP-MD-A`) looks like a model-caused row, and in isolation it is; but if you also had an oversight-design row where human-in-the-loop would catch it cheaper, chapter 05 says attribute to *integration* on the oversight-design axis and let model quality be the defence-in-depth layer. Make the call deliberately.

### Step 3 — Suggested triage (compare after you have written yours)

Fill your own first. Then compare to the suggested answers below — they are *suggested*, not canonical; the chapter-05 citation and the one-line cheapest-intervention reasoning are what matters, not matching the bucket label exactly.

| id | Primary bucket | "What one change ends this scenario?" | Mitigation owner (role) | id_group split? |
|---|---|---|---|---|
| HR-PP-OP-A | Operator | Protected retention policy + enforcement check in the release pipeline that alerts if retention drops below the register-pinned value. | Risk engineering (you) + platform ops; chapter 05 operator examples. | No. |
| HR-PP-OP-B | Operator | Protected system-prompt file under change-control; prompt edits land only via a review PR gated on the red-team regression. | Risk engineering (you) + release pipeline owner. | No. |
| HR-PP-MD-A | Model | Groundedness-eval improvement + refusal-on-low-confidence; register the behaviour contract the model must satisfy at release. | ML side (vendor or internal fine-tune team) + L30 eval-engineering peer; you own the risk contract. | *Candidate split* — if human-in-the-loop presentation (an "advisory" flag the UI surfaces) would end the harm cheaper, split into a model row and an integration row. |
| HR-PP-DT-A | Integration | Enforce a retrieval trust boundary: provenance filter + authentication / allow-list on which handbook pages feed the index; see chapter-05 worked example. | Application / intranet team (implement) + you (spec); data provenance is defence-in-depth. | No (but note: data-caused is tempting; the test picks the cheapest intervention — integration). |
| HR-PP-DT-B | Data | Labelling-vendor contract with provenance + a labelling-quality sample audit before the fine-tune batch is accepted. | Data governance + vendor management; you own the data-quality risk contract. | No. |
| HR-PP-IN-A | Integration | Downstream integrators cannot strip caveats; enforce caveat preservation at the output-forwarding API or refuse the integration. | Application / intranet team + platform-team owning the integration API. | No. |
| HR-PP-IN-B | Integration | Output sanitisation at the rendering boundary; no HTML execution in the console. | Application / intranet team (rendering layer) — this is also a classical web-security concern and appropriately routes to app security. | No. |
| HR-PP-COMP | *Two rows, shared `id_group`.* | Row 1 (operator): protected system-prompt + review-gated edits; Row 2 (model): refusal-robustness against the jailbreak preamble as a release-gating red-team fixture. | Row 1: risk engineering + release pipeline; Row 2: ML side + L30 eval peer. | **Yes** — chapter 05's compound-scenario rule: two interventions are both required. |

### Step 4 — Write mitigation-contract blocks for two of the eight

Pick any two rows and write out the full mitigation-contract block in the chapter-05 format, boring on purpose:

```markdown
### HR-PP-<id> — mitigation contract

- cause_bucket: <operator / model / data / integration>
- mitigation_owner: <role>
- deliverable: <concrete artifact — guardrail config / monitor spec / dataset provenance record / tool ACL / oversight-surface redesign>
- deadline: <absolute date — target set at register-write time (mod-103)>
- evidence_manifest_ref: evidence/harm/HR-PP-<id>/<release>/manifest.json
```

Keep each block to five lines. If the deliverable field runs long, you have probably combined two mitigations that want two rows.

### Step 5 — Role-boundary / residual footnote

At the end of the file, write a *three-to-five-line* footnote covering:

- Any row whose mitigation owner is *outside* the operator's control (`HR-PP-DT-B` plausibly routes to an external labelling vendor; `HR-PP-MD-A` plausibly routes to a hosted-model vendor). Chapter 05: attribute correctly, add a *residual owner* internally, record the third-party dependency for mod-110 (MRM + supply-chain).
- Any row that *could* be read as escalating to `agentic-safety-engineer` (L40). PolicyPal is a modest RAG assistant; chapter 06 says the L40 escalations are *frontier-model capability triggers*, *multi-step autonomous-agent scenarios at depth*, and *frontier red-team methodology*. None of the eight rows here should escalate; write that explicitly so the reviewer sees you applied the test rather than skipped it.

### Step 6 — Final checkpoint

Verify:

- Every row has a primary bucket, a one-sentence cheapest-intervention reasoning, a role-named mitigation owner, and an explicit `id_group`-split flag (even if "No").
- `HR-PP-COMP` was split into two rows sharing an `id_group`, with two distinct interventions. If you attributed it to a single bucket, re-read chapter 05's compound-scenario rule and split.
- The two mitigation-contract blocks are five lines each, no longer.
- The role-boundary footnote names any third-party-dependent rows and declares (not implies) that no row escalates to L40.

## Expected output

`mod102-lab02/cause-bucket-triage.md` — an eight-row table (plus the `HR-PP-COMP` sibling row, so nine total), two full mitigation-contract blocks, and a three-to-five-line footnote.

## Checkpoint questions (self-check)

1. On `HR-PP-DT-A`, why does chapter 05 attribute the scenario to *integration* rather than *data*, even though the harm originates in the corpus? (Because content scrubbing of an unbounded corpus is an infinite job; enforcing a retrieval trust boundary is the cheapest intervention that ends the scenario. Data provenance remains valuable as defence-in-depth, not as the primary attribution.)
2. On `HR-PP-MD-A`, if you split it into a model row and an integration (oversight-design) row, what did you just do? (You applied the `id_group` discipline on an advisory-system harm where both the model behaviour and the oversight surface are plausible interventions. Chapter 05 says to split when both are needed — the risk is that the review loop reduces either to the other and no mitigation ships.)
3. Looking at your eight rows, which bucket is your model *under-populated* in? (If you have zero data-caused rows you may be generalising PolicyPal as "all integration"; if you have zero model-caused rows you may be missing hallucination / over-refusal classes. Note which bucket is thinnest and file it as a coverage-gap candidate for the exercise-01 reverse sweep.)
4. Which of the eight rows would, if PolicyPal moved into SR 11-7 scope (mod-110), immediately grow a new reviewer? (`HR-PP-DT-B` — data-quality on a vendor-supplied labelling batch would attract independent validation and vendor-management review under SR 11-7. Record this as a *conditional* finding; it does not fire until PolicyPal moves scope.)

## Why this lab matters

The mitigation-contract format in chapter 05 is boring on purpose — "boring contracts get honoured; ornate contracts get argued over." But the *attribution* step is where most harm-model work goes wrong: a mis-attributed row sits in the wrong owner's queue and the fix stalls for months. Training the attribution reflex on eight fixed rows with a known-good reference answer is the cheapest way to get it to feel routine before exercise 02 and exercise 04 ask you to attribute dozens of rows against your own running system.
