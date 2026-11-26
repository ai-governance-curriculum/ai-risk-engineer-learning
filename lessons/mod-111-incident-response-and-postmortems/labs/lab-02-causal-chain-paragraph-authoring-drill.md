# Lab 02 — Causal-Chain Paragraph Authoring Drill (Cause vs Symptom Ranking + Two-Sentence Convergence)

**Duration:** 45 minutes · **Format:** guided practice, markdown authoring on a fixed fabricated LLM application

## Objective

Internalise the chapter-02 **step-4 explanatory-power ranking** (ordering observed deltas by how well each predicts the traces, with at least one delta labelled as a *symptom* of another delta higher up) and the chapter-02 **step-6 converged causal-chain paragraph** (one paragraph of at most two sentences, naming the causal delta, the mechanism, the observed effect, and the guardrail interaction) by authoring both artefacts for a pre-computed six-surface delta record on a fixed fabricated running system (`HelpDesk`). The lab drills the atomic paragraph shape chapter 02 pins — the paragraph is what feeds post-mortem section 4 (chapter 03), what the Article 73 package quotes as "preliminary root cause" (chapter 05), and what the regulator extract lifts verbatim — so that when **exercise 02** runs the full six-step workflow on your real incident, the engineering budget goes into pinning the window, snapshotting the six surfaces, and arguing the replay-confirmation, not into re-authoring the paragraph shape under deadline.

The chapter-02 anti-patterns the paragraph shape forecloses: the **list-of-correlated-changes without a story** (three deltas moved; all three get CAP items; none is named as causal); the **cause-and-symptom confusion** (a symptom delta is treated as an independent cause and funds a CAP that fixes the wrong surface); the **four-sentence causal chain** (the paragraph did not converge; the RCA is still open); and the **unpinned-artifact citation** ("the current model" instead of a version identifier — chapter 02 §"Anti-patterns").

## Prerequisites

- Chapter 02 read — specifically "The evidence surfaces you inspect" (the six surfaces and their owners); "Step 3 — Compute the deltas" (the YAML shape with a line per surface); "Step 4 — Order the deltas by explanatory power" (cause vs symptom; "naming the causal delta explicitly is what stops the RCA from producing a list of correlated changes without a story"); "Step 5 — Confirm by trace replay" (replay turns plausible into confirmed); "Step 6 — Name the causal chain" (two-sentence paragraph with the four elements — delta, mechanism, effect, guardrail); and "Anti-patterns" (the one-trace RCA; the unpinned artifact).
- Chapter 03 skimmed at section 4 (Causal chain) — the paragraph lifts straight in as the section opener.
- Chapter 05 skimmed at the Article 73 field-map row "Preliminary root cause" — the paragraph is what the extract quotes.
- A terminal where you can create a working directory and author a markdown file.
- No external access or compute required; the six-surface delta record, the trace IDs, and the paired-good trace are pre-computed and fixed below.

## The fixed example system

Use this system throughout the lab — do *not* substitute your own. Carrying a fixed scenario keeps the lab bounded to the paragraph-authoring skill. The system matches the `HelpDesk` carried through the mod-109 and mod-110 labs.

> **`HelpDesk`** — a customer-support RAG assistant for a mid-size B2B SaaS company (same brief as lab 01). In-house fine-tune `helpdesk-finetune` v1.3.0 on the vendor-hosted base `FoundationCo-LLM-3.5` v3.5.2. Two production tool calls: `escalate_ticket(destination, summary)` and `lookup_account(account_id)`.

### The incident the lab runs on

> **Incident:** `INC-2026-11-21-hd-qual-01`. Opened at 2026-11-21 08:40Z. A mod-109 monitor fire triggered the ticket: `evidently-quality-daily` reports a 9-point drop on the composite (faithfulness + coverage + safe-refusal) quality metric for 2026-11-21 against the rolling 7-day baseline. Sampled traces on the incident window show the assistant consistently refusing legitimate billing-lookup queries with a generic safe-refusal message, breaking the `escalate_ticket` tool call it should have emitted. Triage from the equivalent of lab 01: class *model regression (safety-adjacent — safe-refusal threshold)*; SEV2 (quality regression that violates the release envelope on a production slice; no named-party harm yet); routing Product IR + Governance IR; Security IR No. Immediate containment: traffic shadowed to `helpdesk-finetune` v1.3.0 on the `safe-refusal` monitor's trigger at 09:12Z.

### Pre-computed window and trace IDs (do not change)

- `first_known_bad`: 2026-11-21T06:02:00Z — trace `tr-hd-5c3a`, user prompt `"Can you look up the status of invoice INV-2187?"` → model emitted a safe-refusal string with no `lookup_account` tool call. Hash `sha256:ff02...1d`.
- `last_known_good`: 2026-11-20T23:48:00Z — trace `tr-hd-5b9e`, user prompt `"What is the status of invoice INV-2104?"` → model emitted a correct `lookup_account` tool call followed by a grounded answer. Hash `sha256:aa71...9c`.
- Paired-good trace for layered diff: `tr-hd-5b9e` (above) is the paired good trace; user prompt is near-identical shape, differing only in the invoice identifier.

### Pre-computed six-surface delta record (do not change)

Chapter 02 §"Step 3 — Compute the deltas" shape. All snapshots are hash-pinned; all owners are named per chapter 02 §"The evidence surfaces you inspect."

```yaml
incident_id: INC-2026-11-21-hd-qual-01
window:
  first_known_bad: 2026-11-21T06:02:00Z  # trace tr-hd-5c3a
  last_known_good: 2026-11-20T23:48:00Z  # trace tr-hd-5b9e

deltas:
  trace_store:
    - observation: safe-refusal pattern on invoice-lookup prompts after 2026-11-21T04:00Z
      trace_ids_bad: [tr-hd-5c3a, tr-hd-5c4b, tr-hd-5c5c, tr-hd-5c6d]
      trace_ids_good: [tr-hd-5b9e]
      owner: ai-eval-engineer-L35 (trace-store custodian)
  model_version:
    changed: false
    manifest_before: helpdesk-finetune v1.3.0 on FoundationCo-LLM-3.5 v3.5.2 r-2026-11-10 adapter:none
    manifest_after:  helpdesk-finetune v1.3.0 on FoundationCo-LLM-3.5 v3.5.2 r-2026-11-10 adapter:none
    manifest_hash_before: sha256:3a1f...08
    manifest_hash_after:  sha256:3a1f...08
    owner: ml-platform
  prompt_retriever_decoding_config:
    system_prompt:   no diff (hash sha256:91cc...e3 before and after)
    tool_descriptors: diff — the `escalate_ticket` tool description was edited at 2026-11-21T03:47Z from "Use when the caller's issue cannot be resolved by lookup" to "Use only when the caller explicitly requests human escalation OR the caller's issue is a clear safety concern"
    tool_descriptor_hash_before: sha256:4e01...ab
    tool_descriptor_hash_after:  sha256:7f12...cd
    retriever:
      index_version: no diff (idx-2026-11-14 before and after)
      embedding_model: no diff
      top_k: no diff
      filters: no diff
    decoding:
      temperature: no diff (0.0)
      top_p: no diff
      max_tokens: no diff (512)
      stop: no diff
      seed: no diff
    owner: application-platform
  guardrail_effectiveness_delta:
    suite_version_before: mod-108 suite v2026-11-14
    suite_version_after:  mod-108 suite v2026-11-14   # same suite — guardrails did not re-run
    safe_refusal_classifier_asr: 0.02 → 0.02 (no change)
    injection_classifier_asr:   0.04 → 0.04 (no change)
    note: no guardrail-side delta on this incident; the suite did not re-run and the per-attack-family numbers are flat
    owner: mod-108 owner (risk engineer / peer)
  drift:
    input_distribution: no shift (cosine distance 0.011 on top-20 features vs baseline, inside +/-0.02 band)
    retrieval_distribution: no shift (content-fingerprint on top-k chunks flat)
    output_distribution: shift — safe-refusal-string frequency moved from 0.03 to 0.41 on 2026-11-21 (dashboard snapshot sha256:ba2e...44)
    tool_call_distribution: shift — `escalate_ticket` tool-call count dropped from baseline 48/hr to 2/hr; `lookup_account` tool-call count dropped from baseline 112/hr to 9/hr
    owner: mod-109 owner
  upstream_supply_chain:
    model_provider_changelog: no change in window
    safety_classifier_provider_changelog: no change in window
    kb_corpus_refresh: no change in window (last refresh 2026-11-18)
    fine_tune_data_revision: no change in window
    tool_descriptor_edit_log:
      - 2026-11-21T03:47:00Z — commit abc1234 on application-platform repo; author: platform-eng; PR title "small tuning to escalate_ticket description"; merged on normal review path
    owner: mod-110 owner / application-platform
```

### Pre-computed layered trace diff (do not change)

Chapter 02 §"Trace-diff as an evidence discipline" shape. The paired-good `tr-hd-5b9e` vs. the first-known-bad `tr-hd-5c3a`:

| Layer | `tr-hd-5c3a` (bad) | `tr-hd-5b9e` (good) | Diff |
|---|---|---|---|
| User prompt | "Can you look up the status of invoice INV-2187?" | "What is the status of invoice INV-2104?" | Near-identical shape; only the invoice id and a trivial surface-form phrasing differ |
| System prompt | hash sha256:91cc...e3 | hash sha256:91cc...e3 | Same |
| Tool descriptors | hash sha256:7f12...cd (post-edit) | hash sha256:4e01...ab (pre-edit) | **Diff here** — `escalate_ticket` description edited at 03:47Z narrows the tool to "explicitly requests human escalation OR clear safety concern" |
| Retrieval context | 3 chunks from `idx-2026-11-14`, scores 0.84 / 0.79 / 0.71 | 3 chunks from `idx-2026-11-14`, scores 0.86 / 0.80 / 0.72 | Equivalent retrieval |
| Intermediate model output | Internal planner trace: "User is asking about invoice status. Checking available tools. No tool matches — `lookup_account` requires an account id not an invoice id; `escalate_ticket` only applicable on explicit human-escalation request or safety concern. Emitting safe refusal." | Internal planner trace: "User is asking about invoice status. Calling `lookup_account(invoice=INV-2104)` via the invoice-to-account resolver." | **Diff here** — the planner's tool-selection step diverges; the bad trace reasons from the narrowed tool descriptor and concludes no tool is applicable |
| Tool calls | None | `lookup_account(...)` | **Diff here** — direct consequence of planner divergence |
| Final output | Generic safe refusal: "I'm sorry, I'm not able to help with that request." | Grounded answer with invoice status | **Diff here** — direct consequence of missing tool call |
| Guardrail verdicts | `safe-refusal-classifier`: pass (output is a safe refusal; this is the classifier's "nominal" case, not an attack); `injection-classifier`: pass | Same both guardrails pass | Same — no guardrail-side delta |

The trace diff localises the divergence to the tool-selection step inside the planner — upstream from retrieval (equivalent), upstream from decoding (same config), downstream from the system prompt (same), downstream from the tool descriptors (differ).

### Pre-computed replay record (do not change)

Chapter 02 §"Step 5 — Confirm by trace replay" shape:

```yaml
replay:
  injected_input: "Can you look up the status of invoice INV-2187?"  # the bad-trace user prompt verbatim
  replayed_against:
    config_bundle: pre-edit tool descriptors (hash sha256:4e01...ab) with everything else held at incident-time state (model manifest, retriever idx-2026-11-14, decoding config)
  result:
    - trace_id: tr-replay-7a21
      planner_output: "Calling lookup_account(invoice=INV-2187) via the invoice-to-account resolver."
      tool_calls: [lookup_account(...)]
      final_output: grounded answer with invoice status
      guardrail_verdicts: all pass
      outcome: exploit does NOT reproduce under the pre-edit tool descriptors — the planner selects lookup_account correctly
  signatories:
    - risk-engineer-L25 (lab drill — <name>)
    - ai-eval-engineer-L35 (lab drill — <name>)
```

## Steps

### Step 1 — Create your working directory and the scaffold

```bash
mkdir -p mod111-lab02 && cd mod111-lab02
touch rca-step-4-6.md
```

Open `rca-step-4-6.md` and paste this scaffold. You will fill the sections in order.

```markdown
# INC-2026-11-21-hd-qual-01 — RCA step-4 ranking and step-6 causal chain (lab 02)

System: HelpDesk (see lab brief)
Chapter references: mod-111 ch 02 §"Step 4 — Order the deltas by explanatory power"; §"Step 5 — Confirm by trace replay"; §"Step 6 — Name the causal chain"; §"Trace-diff as an evidence discipline"; §"Anti-patterns"
Window: first_known_bad 2026-11-21T06:02:00Z (tr-hd-5c3a); last_known_good 2026-11-20T23:48:00Z (tr-hd-5b9e)
Paired good trace: tr-hd-5b9e
Replay trace: tr-replay-7a21

## 1. Observed deltas (summary)

(one-line enumeration of deltas lifted from the pre-computed delta YAML; no analysis)

## 2. Explanatory-power ranking

(numbered list — one rank per observed delta; one-sentence justification per rank citing evidence; at least one delta labelled as a symptom of another)

## 3. Replay confirmation

(one short paragraph quoting the replay record outcome and confirming the top-ranked delta as causal)

## 4. Converged causal-chain paragraph

(ONE paragraph, AT MOST two sentences — the delta, the mechanism, the observed effect, the guardrail interaction; version-pinned citations; this paragraph feeds post-mortem section 4 verbatim and the Article 73 extract's "preliminary root cause" field verbatim)

## 5. Contributing factors (non-causal)

(bullet list — contributing factors are NOT causes; naming them separately is what stops the CAP from over-crediting any one delta with the entire outcome; chapter 03 §"4. Causal chain" cross-reference)

## 6. Anti-pattern self-check

(one line per chapter-02 anti-pattern — RCA as code review; one-trace RCA; unpinned artifact; SOC hand-off by chat — quoting the specific line in this file that demonstrates it was avoided)
```

### Step 2 — Fill Section 1 (Observed deltas, summary only)

Chapter 02 §"Step 3 — Compute the deltas" produced the delta YAML already. Section 1 is a one-line enumeration lifted straight from it — no analysis. Enumerating the deltas separately from the ranking makes it hard to silently drop one.

```markdown
## 1. Observed deltas (summary)

- **Trace store** — safe-refusal pattern on invoice-lookup prompts after 2026-11-21T04:00Z; 4 bad traces vs. 1 paired-good trace.
- **Model version** — no change in the window (manifest hash sha256:3a1f...08 before and after).
- **Prompt / retriever / decoding config** — **tool-descriptor edit at 2026-11-21T03:47Z** narrowing the `escalate_ticket` description (hash sha256:4e01...ab → sha256:7f12...cd); retriever index unchanged; system prompt unchanged; decoding config unchanged.
- **Guardrail-effectiveness delta** — no delta; suite v2026-11-14 did not re-run in the window; per-attack-family ASRs flat.
- **Drift** — **output-distribution shift** (safe-refusal-string frequency 0.03 → 0.41) and **tool-call-distribution shift** (`escalate_ticket` 48/hr → 2/hr; `lookup_account` 112/hr → 9/hr); input-distribution and retrieval-distribution flat.
- **Upstream / supply-chain** — no model-provider / safety-classifier / corpus-refresh changes; tool-descriptor edit log shows one commit `abc1234` at 03:47Z on the application-platform repo.
```

Reading check: six surfaces, one line each; "no change" is a first-class observation (chapter 02 §"Step 2" — "the negative result is a first-class fact in the RCA"); every observed delta carries its version pin or timestamp.

### Step 3 — Fill Section 2 (Explanatory-power ranking)

Chapter 02 §"Step 4 — Order the deltas by explanatory power" is the engineering step that converts correlated deltas into a causal story. **At least one delta has to be labelled as a symptom of another delta higher up** — "the KB corpus refresh at 05:47Z best explains the injected-chunk observation; the guardrail-effectiveness delta is a *symptom* of the same underlying content shift, not an independent cause" (chapter 02 §"Step 4"). The same shape applies here: the tool-descriptor edit is the causal delta; the output-distribution and tool-call-distribution drifts are *symptoms* of it.

```markdown
## 2. Explanatory-power ranking

1. **Tool-descriptor edit on `escalate_ticket` at 2026-11-21T03:47Z (commit `abc1234`)** — the only prompt-side configuration change in the window. The narrowed description ("Use only when the caller explicitly requests human escalation OR the caller's issue is a clear safety concern") removes the planner's precedent for invoking `escalate_ticket` on nominal billing-lookup paths; when the planner now has no applicable tool, it defaults to a safe refusal. The paired-trace layered diff (lab-brief §"Pre-computed layered trace diff") shows the divergence localised to the planner's tool-selection step, downstream from the tool descriptors and upstream from retrieval / decoding. Causal.
2. **Output-distribution drift (safe-refusal-string frequency 0.03 → 0.41 on 2026-11-21)** — a *symptom* of the top-ranked delta: when the planner defaults to safe refusal on nominal paths, the output distribution shifts to safe-refusal-shaped strings. Not an independent cause. If treated as independent, a CAP that tunes the safe-refusal-classifier threshold would fund the wrong fix.
3. **Tool-call-distribution drift (`escalate_ticket` 48/hr → 2/hr; `lookup_account` 112/hr → 9/hr)** — also a *symptom* of rank 1: the narrowed tool descriptor is what suppressed `escalate_ticket`, and the planner's refusal-default is what suppressed `lookup_account` on the invoice-lookup path (the planner never got to `lookup_account` because it concluded no tool was applicable). Not an independent cause.
4. **Trace-store observation (safe-refusal pattern on invoice-lookup prompts after 2026-11-21T04:00Z)** — the signal, not a cause. The 17-minute lag from the tool-descriptor commit at 03:47Z to the first observed bad trace at ~04:00Z is consistent with deployment propagation.
5. **Model version — no change** — ruled out as a cause; the manifest hash sha256:3a1f...08 is identical before and after.
6. **Guardrail-effectiveness delta — none** — ruled out as a cause; the suite did not re-run in the window and per-family ASRs are flat. The guardrail-side is not implicated.
7. **Retriever / embedding / decoding config — no change** — ruled out as a cause.
8. **Input-distribution / retrieval-distribution — no shift** — ruled out as a cause; the user-population shape and the retrieved-content shape are stable.
```

Reading check: ranks 2 and 3 are explicitly labelled as symptoms of rank 1 with the mechanism stated; ranks 5-8 are "ruled out" with the specific hash / version evidence that rules them out; the "no change" negative results are on the record; no delta is unaccounted for.

### Step 4 — Fill Section 3 (Replay confirmation)

Chapter 02 §"Step 5 — Confirm by trace replay" is the step that turns *plausible* into *confirmed*. The replay record is pre-computed in the lab brief; Section 3 quotes its outcome and ties it to rank 1.

```markdown
## 3. Replay confirmation

Replay `tr-replay-7a21` — the bad-trace user prompt (`"Can you look up the status of invoice INV-2187?"`) replayed against the pre-edit tool descriptors (hash `sha256:4e01...ab`) with every other surface held at incident-time state (model manifest, retriever `idx-2026-11-14`, decoding config, guardrail configuration). The replay planner emits `lookup_account(invoice=INV-2187)`, the tool call succeeds, and the final output is a grounded answer with invoice status. The exploit does **not** reproduce under the pre-edit descriptors. The replay therefore confirms that the tool-descriptor edit at 2026-11-21T03:47Z is the causal delta and that the output / tool-call drifts in ranks 2 and 3 are consequential symptoms rather than independent causes. Signed off by risk-engineer-L25 and ai-eval-engineer-L35 (lab drill).
```

Reading check: the replay names the pre-edit hash it replayed against; it holds every other surface at incident-time state (confirming the test isolates the tool-descriptor surface); the outcome is a specific statement ("exploit does not reproduce") tied to a specific replay trace ID; the sign-off cites both roles per chapter 02 §"Coordination with `ai-eval-engineer`."

### Step 5 — Fill Section 4 (Converged causal-chain paragraph)

Chapter 02 §"Step 6 — Name the causal chain" pins the paragraph shape: **one paragraph, at most two sentences, naming the delta, the mechanism, the observed effect, and the guardrail interaction**. Longer than two sentences means the RCA has not converged. The paragraph feeds post-mortem section 4 (chapter 03) and the Article 73 "preliminary root cause" field (chapter 05) verbatim.

```markdown
## 4. Converged causal-chain paragraph

On 2026-11-21 at 03:47Z a tool-descriptor edit on `escalate_ticket` (commit `abc1234`; hash `sha256:4e01...ab` → `sha256:7f12...cd`) narrowed the tool's applicability to "explicit human-escalation request OR clear safety concern," which caused the planner on nominal invoice-lookup paths to conclude no tool applied and default to a safe refusal — suppressing the correct `lookup_account` call on 4 observed bad traces starting at 2026-11-21T06:02:00Z. The guardrail chain did not fire because the safe-refusal output is the classifier's nominal case and not an attack pattern (safe-refusal-classifier ASR 0.02 before and after on suite v2026-11-14, no delta), so no guardrail-side signal surfaced the regression.
```

Reading check: exactly two sentences; the first names the delta (version-pinned commit SHA, timestamp, hashes), the mechanism (narrowed applicability → planner's tool-selection default), and the observed effect (safe refusal, suppressed tool call, 4 bad traces); the second names the guardrail interaction (why the chain did not fire); no "the model got it wrong" phrasing; no unpinned "the current version" references; the paragraph is self-contained and can be quoted verbatim into post-mortem section 4 and into the Article 73 extract's "preliminary root cause" field without edits.

### Step 6 — Fill Section 5 (Contributing factors, non-causal)

Chapter 03 §"4. Causal chain" is explicit: "Contributing factors are not causes; separating the two prevents the post-mortem from over-crediting any one delta with the entire outcome." For this incident the contributing factors sit on the *process* that let the change ship without a safety-eval check, not on the *mechanism* of the failure.

```markdown
## 5. Contributing factors (non-causal)

- The tool-descriptor edit was merged on the **normal review path** (standard PR with a minor-tuning title); there is no safety-eval gate required for tool-descriptor edits in the current release pipeline, so the change bypassed any pre-deploy check that would have caught the planner's refusal-default shift.
- The **mod-109 output-distribution monitor** alerted after the fact (the safe-refusal-string frequency crossed its band) but no monitor fires *on tool-descriptor edits themselves* — a change-side monitor would have flagged the edit at 03:47Z, 15-17 minutes ahead of the first bad trace.
- The **guardrail-side is not implicated** — the safe-refusal-classifier ASR did not move; this is an availability-of-correct-behaviour regression rather than a safety-classifier failure. Any CAP item that "improves the safe-refusal classifier" would fix the wrong surface (over-crediting rank 2 as cause); any CAP that "retrains the model" would fix the wrong surface (over-crediting the model-version row despite no diff).
```

Reading check: each contributing factor sits on process, monitoring, or scope — not on the mechanism; each names the specific CAP mis-direction it would cause if elevated to a cause; the rank-2 symptom is explicitly called out as a CAP mis-direction target.

### Step 7 — Fill Section 6 (Anti-pattern self-check)

Chapter 02 §"Anti-patterns" lists four that the paragraph shape and the ranking discipline defeat.

```markdown
## 6. Anti-pattern self-check

| Anti-pattern (ch 02 §"Anti-patterns") | Quote from this file that demonstrates it was avoided |
|---|---|
| RCA as a code review | Section 4 names the delta, the mechanism, and the observed effect in two sentences; no aesthetic argument about what the model should have said. |
| The one-trace RCA | Section 1 cites 4 bad traces (`tr-hd-5c3a`, `tr-hd-5c4b`, `tr-hd-5c5c`, `tr-hd-5c6d`) and 1 paired-good trace (`tr-hd-5b9e`); Section 3 cites 1 replay trace (`tr-replay-7a21`). |
| The unpinned artifact | Every surface in Section 2 carries a version pin or hash (`sha256:3a1f...08`, `sha256:4e01...ab` → `sha256:7f12...cd`, `idx-2026-11-14`, suite `v2026-11-14`, commit `abc1234`). Section 4 cites the two tool-descriptor hashes and the suite version. |
| The SOC hand-off by chat | Not triggered on this incident (routing was Product IR + Governance IR only; Security IR = No; see lab-01-equivalent triage row). If Security IR had been engaged, Section 3's replay confirmation would be filed alongside a SOC hand-off record as a repo artefact, not as a Slack thread. |
```

Reading check: each anti-pattern row quotes a specific line in the file; no row is justified by "we did not do that" without evidence.

### Step 8 — Final checkpoint

Re-read the six sections. Confirm:

- **Section 1 (observed deltas)** has one line per surface; "no change" observations are recorded as first-class facts with version pins.
- **Section 2 (ranking)** labels at least one delta as a symptom of another delta higher up; every "ruled out" row carries the specific evidence that rules it out.
- **Section 3 (replay confirmation)** names the replay trace, the surface it isolated, the specific pre-edit hash it replayed against, and the outcome tied to rank 1.
- **Section 4 (causal chain)** is **at most two sentences** and names the four elements — delta, mechanism, effect, guardrail interaction; all citations are version-pinned; the paragraph is lift-ready for post-mortem section 4 and the Article 73 extract.
- **Section 5 (contributing factors)** separates process / monitoring / scope factors from the mechanism; each factor names the CAP mis-direction it would cause if elevated.
- **Section 6 (anti-pattern self-check)** quotes a specific line per anti-pattern.
- Re-read Section 4 with a timer: reading it aloud should take no more than 25 seconds. If it takes longer, it is too long — converge further.

## Expected output

`mod111-lab02/rca-step-4-6.md` — one populated six-section document; one to two pages. Section 2 (ranking) and Section 4 (causal-chain paragraph) are the load-bearing artefacts; Section 4 is the one every downstream consumer (post-mortem section 4, Article 73 extract, SR 11-7 memo, council packet) lifts verbatim.

## Checkpoint questions (self-check)

1. The output-distribution drift and the tool-call-distribution drift are ranked as **symptoms** of the tool-descriptor edit rather than as independent causes. Why does the chapter-02 §"Step 4" discipline require this labelling, and what specific CAP mis-direction does it prevent? (*Chapter 02 §"Step 4 — Order the deltas by explanatory power" says "the guardrail-effectiveness delta is a symptom of the same underlying content shift, not an independent cause. Naming the causal delta explicitly is what stops the RCA from producing a list of correlated changes without a story." Applied to this incident: if the output-distribution drift were treated as an independent cause, a CAP would fund "tune the safe-refusal-classifier threshold," which does not touch the tool-descriptor edit that actually caused the regression. If the tool-call-distribution drift were treated as independent, a CAP would fund "add a tool-call-count floor alert," which detects the symptom but does not prevent the next tool-descriptor edit from recurring the incident. The cause-vs-symptom labelling is what directs the CAP to the actual mechanism — a safety-eval gate on tool-descriptor edits in the release pipeline.*)

2. The causal-chain paragraph is **two sentences**. Chapter 02 §"Step 6" says "if the paragraph is more than two sentences, the RCA has not converged." What does "converged" mean here, and why is the two-sentence budget load-bearing rather than cosmetic? (*Chapter 02 §"Step 6 — Name the causal chain" pins the paragraph as the artefact that feeds post-mortem section 4 (chapter 03) and the Article 73 extract's "preliminary root cause" field (chapter 05) verbatim. "Converged" means every element — delta, mechanism, effect, guardrail interaction — is named with evidence, and no element is deferred to prose. A longer paragraph signals that the author is still reasoning in-document rather than citing a resolved analysis. The two-sentence budget forces the author to decide which delta is causal (not "one of these three things could be responsible"), which mechanism is operative (not "possibly the planner misinterpreted the descriptor"), and which guardrail interaction is relevant (not "the guardrail may or may not have fired"). A regulator reading the Article 73 extract's root-cause field needs a resolved answer, not a narrative — the two-sentence budget enforces that resolution.*)

3. Section 5 (contributing factors) calls out that "any CAP that 'retrains the model' would fix the wrong surface." Why is this call-out worth making explicit, when the ranking in Section 2 already rules out the model-version row as a cause? (*Chapter 02 §"Motivation" names "model-blame default" as one of the three failure modes: the investigator locates the bad output, notes the model produced it, concludes "we need a new model," and the register acquires a "model quality" row that no tractable change will address. Even after Section 2 rules out the model-version surface, a post-mortem reader under time pressure may skim to the CAP and propose a model retrain if the contributing-factors section does not explicitly ward off that failure mode. Chapter 03 §"4. Causal chain" is explicit that contributing factors are not causes, and the CAP mis-direction call-outs are the engineering artefact that enforces the separation on the reader. Over-crediting a contributing factor with the outcome produces a CAP that fixes the wrong thing — chapter 03's canonical anti-pattern — and this incident's shape (no model change, no guardrail change, but a prompt-side config change) is exactly the shape where the model-blame default is tempting.*)

4. The replay in Section 3 held every other surface at incident-time state and varied only the tool-descriptor hash. Chapter 02 §"Trace-diff as an evidence discipline" says "pair traces on inputs that differ only where you're investigating." How does the replay satisfy that discipline, and what would be the failure mode if the replay had varied two surfaces at once? (*Chapter 02 §"Trace-diff as an evidence discipline" pins two rules: "pair traces on inputs that differ only where you're investigating" and "diff at every layer, not just the output." The replay in Section 3 varies exactly one surface — the tool-descriptor hash (pre-edit vs. post-edit) — and holds the model manifest, retriever index, decoding config, guardrail configuration, and user prompt constant. The replay's outcome (the planner now selects `lookup_account` correctly, the exploit does not reproduce) is therefore attributable to the tool-descriptor surface alone and not to a confound. If the replay had also varied the retriever index or the decoding config, the outcome would be ambiguous — the "exploit does not reproduce" result could be explained by either the tool-descriptor rollback or the confounded surface, and the causal chain would not be confirmed. Varying one surface at a time is what turns a replay from a smoke test into causal evidence.*)

## Why this lab matters

Exercise 02 asks you to run the full six-step "what changed since the incident began" workflow on your own real incident — pin the window, snapshot the six surfaces, compute the deltas, order by explanatory power, replay-confirm, and converge on the paragraph — with seven deliverables including the SOC hand-off and the eval-engineer co-signed chain-of-custody. If the paragraph shape and the ranking discipline are new at exercise-02 authoring time, the budget blows on re-authoring the two-sentence paragraph under deadline, arguing in-document about whether a given delta is cause or symptom, and re-deriving what the replay record has to prove. This lab drills the two atomic artefacts — the ranking and the paragraph — on a fixed fabricated system with pre-computed deltas so exercise-02 time goes into the real engineering judgement: pinning the window across the drift-shaped vs. moment-shaped first-known-good boundary, snapshotting the six surfaces reliably on your real trace store, and negotiating the SOC hand-off without reproducing SOC's work. The paragraph shape it drills is also what chapter 03 (post-mortem section 4), chapter 05 (Article 73 "preliminary root cause," SR 11-7 "root cause," FDA MDR "manufacturer narrative"), and chapter 06 (council-packet incident-summary row) all lift. Getting the paragraph shape right here cascades through the module.
