# Root-Cause Analysis by Trace Diff, Version Diff, and Guardrail-Effectiveness Delta

Once triage (chapter 01) has assigned a class, a severity, and a routing, the technical root-cause analysis begins. On a classical distributed system the RCA reads logs, metrics, and configuration diffs since the last known-good state. AI systems layer additional evidence surfaces: the trace store (prompts, retrievals, tool calls, model outputs, guardrail verdicts), model version diffs, prompt / retriever / decoding config diffs, and guardrail-effectiveness deltas. This chapter defines the "what changed since the incident began" workflow that reads those surfaces and the coordination interface with `ai-eval-engineer` (app-side chain) and security (SOC-side chain).

## Motivation

Three failure modes an inexperienced AI RCA falls into:

- **Log-only forensics.** The investigator reads request / response logs but not the trace store. They see a bad output but not the prompt that produced it, not the retrieval context, not the intermediate tool calls, not the guardrail verdict. RCA concludes "the model got it wrong" with no localised cause.
- **Model-blame default.** The investigator locates the bad output, notes that the model produced it, and concludes "we need a new model." No inspection of prompt / retriever / decoding config or the guardrail chain. The register acquires a "model quality" row that will not be addressed by any tractable change.
- **Silent-change hunt.** The investigator asks "what changed" and looks only at the deploy log. They miss the prompt-template edit that shipped through a config-only channel, the retriever-index refresh that changed KB contents overnight, the guardrail-model provider's silent update, and the decoding temperature bump that shipped with a "small tuning" PR. Nothing on the deploy log looks unusual and the RCA stalls.

The workflow below is designed to defeat all three modes. It reads every changed surface and it pins evidence at each step so the RCA can be re-run by a different investigator and produce the same conclusion.

## The evidence surfaces you inspect

Six surfaces cover the AI-specific RCA. Each has an owner in the day-to-day system and each has a matching evidence artifact.

| Surface | What lives here | Owner | Evidence artifact |
|---|---|---|---|
| **Trace store** | Prompts (system, tool, user), retrieval context, tool-call arguments and results, intermediate model outputs, final output, guardrail verdicts, latency and cost, trace ID linking to the request | Application platform / `ai-eval-engineer` | Trace ID(s) attached to the incident record, hash-pinned |
| **Model version diff** | The model identifier(s) in use at incident time vs. the last known-good time — base model, fine-tune revision, adapter / LoRA revision, safety-classifier version | ML platform | Model manifest before/after, hash-pinned |
| **Prompt / retriever / decoding config diff** | System prompt, tool descriptors, retriever config (index version, embedding model, top-k, filters), decoding config (temperature, top-p, max tokens, stop sequences) | Application platform | Config manifest before/after, hash-pinned |
| **Guardrail-effectiveness delta** | Attack-success-rate, false-negative rate, false-positive rate on the guardrail suite (mod-108) at incident time vs. the last measured baseline | Risk-engineering / mod-108 owner | Guardrail-effectiveness report, versioned |
| **Drift signals** | Input-distribution, retrieval-distribution, output-distribution, tool-call-distribution drift metrics (mod-109) leading into the incident window | Monitoring / mod-109 owner | Drift dashboard snapshot at incident window |
| **Upstream / supply-chain change log** | Model-provider changelog, safety-classifier vendor changelog, retriever-index rebuild log, RAG corpus refresh log, fine-tune data revision log (mod-110) | MRM / supply-chain owner (mod-110) | Supplier-change record with pinned version identifiers |

Every RCA cites at least the trace store and one of the "what changed" surfaces. An RCA that names no changed surface has not found a cause — it has found a symptom.

## The "what changed since the incident began" workflow

The workflow below runs in the order that keeps blast radius shrinking as quickly as possible. Step 1 is done before RCA "starts" in the classical sense because containment can begin after step 3 without a full RCA.

### Step 1 — Pin the incident window

Establish two timestamps: the *first known bad* trace and the *last known good* trace. The window between them is the RCA scope; nothing outside the window is under investigation. Both timestamps are recorded on the incident record with the trace IDs.

Pinning the window is not always easy. For drift-driven incidents the "last known good" is a gradient rather than a moment — take the last trace whose relevant metric was inside its band. For jailbreak-class incidents where the exploit was tried before it succeeded, the "first known bad" is the first trace where the exploit's success can be confirmed, not the first attempt.

### Step 2 — Snapshot each surface at both timestamps

For each of the six surfaces above, pull the state at `first_known_bad` and at `last_known_good`. This is a diff-collection step, not a diff-analysis step; keep it disciplined. Every snapshot gets a hash so the RCA can be re-run.

### Step 3 — Compute the deltas

Diff each pair of snapshots. Record the diffs in a single RCA scratch file. A working shape:

```yaml
incident_id: INC-2026-08-14-cs-rag-inj-01
window:
  first_known_bad: 2026-08-14T09:12:04Z  # trace tr-8a3b...
  last_known_good: 2026-08-13T22:07:19Z  # trace tr-7c2f...

deltas:
  trace_store:
    - observation: injected KB chunk appears in retrieval context after 2026-08-14T06:00Z
      trace_ids: [tr-8a3b..., tr-8a4c..., tr-8a5d...]
  model_version:
    changed: false
  prompt_retriever_decoding_config:
    prompt: no diff
    tool_descriptors: no diff
    retriever:
      index_version: idx-2026-08-14 (rebuild at 05:47Z)  # vs idx-2026-08-07
      embedding_model: no diff
      top_k: no diff
      filters: no diff
    decoding: no diff
  guardrail_effectiveness_delta:
    asr_before: 0.03
    asr_after:  0.19          # measured on suite 2026-08-14
    baseline_floor: 0.05      # exceeded → guardrail regression
  drift:
    input_distribution: no shift
    retrieval_distribution: content-fingerprint drift on top-k chunks (+27% novel n-grams)
  upstream_supply_chain:
    - kb_corpus_refresh: 2026-08-14T05:47Z (mod-110 supplier record ref)
```

### Step 4 — Order the deltas by explanatory power

Not every delta explains the incident. Order the observed deltas by how well each *predicts* the observed traces. In the sketch above, three deltas moved: retriever index rebuild, guardrail-effectiveness ASR, retrieval-distribution shift. The KB corpus refresh at 05:47Z best explains the injected-chunk observation; the guardrail-effectiveness delta is a *symptom* of the same underlying content shift, not an independent cause. Naming the causal delta explicitly is what stops the RCA from producing a list of correlated changes without a story.

### Step 5 — Confirm by trace replay

Take the injected KB chunk, replay it against the pre-refresh trace path, and confirm the pre-refresh path does not produce the exploit. Replay is what turns *plausible* into *confirmed*. The replay traces are attached to the RCA record.

### Step 6 — Name the causal chain

Write the causal chain as one paragraph before writing the post-mortem. If the paragraph is more than two sentences, the RCA has not converged. A converged causal chain reads:

> On 2026-08-14 at 05:47Z the KB corpus refresh ingested a public help-forum thread containing indirect-prompt-injection strings. Retrieval on the RAG path selected that chunk on user sessions that included the affected topic, and the assistant executed the injected `send_email` tool call, exfiltrating a customer email address before the guardrail rejected the pattern (ASR on the refreshed suite: 0.19, above the 0.05 floor).

That paragraph is what feeds the post-mortem's "root cause" field (chapter 03).

## Trace-diff as an evidence discipline

Trace diffing — comparing a bad trace to a paired good trace — is the AI equivalent of the classical `diff` on config files. Two disciplines matter:

- **Pair traces on inputs that differ only where you're investigating.** If you're isolating whether the retriever change is causal, pair traces where the user prompt is (nearly) identical and only the retrieval output differs. If you cannot construct such a pair from live traffic, generate the pair via replay.
- **Diff at every layer, not just the output.** A trace diff that only shows "the outputs are different" is not a trace diff; that is a symptom diff. Show that the prompts are (or are not) different, the retrieval contexts differ *here*, the intermediate model outputs diverge *here*, the tool calls change *here*, and the final outputs differ *here* as a consequence.

Both disciplines make the RCA reproducible and both make the eventual regression fixture (chapter 04) buildable from the evidence in the RCA record.

## Guardrail-effectiveness delta as RCA evidence (mod-108 cross-reference)

The guardrail-effectiveness suite (mod-108) produces an attack-success-rate (ASR) and per-attack-family metrics for each guardrail on the shipped model. In an RCA, the delta between the pre-incident and post-incident run of that suite is a first-class piece of evidence:

- If ASR moved *up* through the incident window with no model / guardrail change, the shift is behavioural drift or content drift — the guardrail is doing less against the same test set.
- If ASR moved *up* only against a specific attack family (say, indirect prompt injection), that family became more effective — worth checking whether the exploit fits.
- If ASR is unchanged but *observed* attacks in the trace store are succeeding, the incident's attack pattern is not represented in the guardrail suite — a coverage gap, and a fixture to add (chapter 04).

The RCA cites the specific suite version and the pre/post ASR numbers. That citation is part of the post-mortem and part of the risk-register update.

## Drift signals as RCA evidence (mod-109 cross-reference)

Monitors from mod-109 produce time-series for input distribution, retrieval distribution, output distribution, and tool-call distribution. In an RCA:

- **Input distribution drift** points at a user-population shift (new geography, new use case, an adversary campaign).
- **Retrieval distribution drift** points at a corpus refresh or an index-version change — the exact surface investigated in step 3 above.
- **Output distribution drift** points at a model / prompt / decoding change or at a downstream population shift.
- **Tool-call distribution drift** points at a change in what the model is being asked to *do* — new topics, new user flows, or an exploit reshaping the call mix.

An RCA that touches at least one drift signal explicitly is a more defensible RCA than one that reads only trace samples. The signal-to-cause narrative is the reason monitoring exists.

## Coordination with `ai-eval-engineer` (app-side evidence chain)

The app-side evidence chain is owned by `ai-eval-engineer` at level 35. In an incident, that role:

- Owns the trace store, the eval-suite runs, and the release-manifest tooling — the surfaces of steps 2, 3, and 5 above depend on artifacts they maintain.
- Runs the shadow-eval / replay comparisons that confirm or refute a proposed causal chain.
- Certifies the "chain-of-custody" for the trace snapshots — trace IDs, hashes, and timestamps, so the evidence survives regulator inspection.

The risk-engineer role's job in that coordination is to specify what the RCA needs, not to do the eval-engineering work. Concretely: you name the trace IDs, the surfaces, the timestamps, and the comparison you need; the eval engineer provides pinned snapshots and the replay results. Both roles sign off on the RCA record.

## Coordination with security (SOC-side evidence chain)

The SOC-side evidence chain is owned by the security IR track. When an incident is routed to security IR (chapter 01), the classical evidence sources apply:

- Network logs — model-provider outbound calls, tool-integration outbound calls, retrieval-source fetch logs.
- Host / container logs — the process that made the call, the container image, the file-integrity monitor state.
- IAM logs — which credential invoked which tool, which service account signed which request.
- Model-provider audit logs — provider-side records of the prompts and completions, if the provider offers them.

The risk-engineer role's job at the SOC boundary is to *hand off cleanly*, not to reproduce SOC's work. Concretely: you name the trace IDs (SOC pivots from there), the tool-call arguments that concern you, the outbound endpoints that shouldn't have been called, and the timestamps. SOC returns whether an intrusion is confirmed and where the compromised credentials live.

The interface artifact is a short evidence hand-off record filed on the incident ticket; it names the sources, the timestamps, and the expected callback. Without that record the SOC-side chain and the app-side chain drift and the post-mortem cannot reconcile them.

## Anti-patterns

- **RCA as a code review.** The investigator reads the model output and argues about whether the model should have said what it said. Fix: the RCA is a diff exercise, not an aesthetic exercise.
- **The one-trace RCA.** A single bad trace is used to build the causal chain. Fix: at least one paired good trace is required; multiple bad traces are required to distinguish a single incident from a pattern.
- **The unpinned artifact.** The RCA cites "the current model" or "the latest config." Fix: cite version identifiers and hashes; the RCA has to be re-runnable next quarter.
- **The SOC hand-off by chat.** Evidence hand-off is a Slack thread that no one archives. Fix: the hand-off record is a ticket comment or a repo artifact.

## Cross-references

- **Trace store and eval infrastructure** — owned by `ai-eval-engineer` (L35); RCA depends on their artifacts and their sign-off on the chain-of-custody.
- **Guardrail-effectiveness suite** — mod-108 owns the suite; RCA reads the delta.
- **Monitoring signals** — mod-109 owns the drift and safety signals; RCA reads the time-series through the incident window.
- **Supply-chain change log** — mod-110 owns the supplier / dataset / model-version records that populate the upstream-change surface.
- **Post-mortem** — chapter 03 of this module consumes the causal-chain paragraph and the delta record produced here.

## Summary

- AI-specific RCA reads six surfaces: trace store, model version, prompt / retriever / decoding config, guardrail-effectiveness delta, drift signals, and upstream / supply-chain change log.
- The "what changed" workflow pins an incident window, snapshots each surface at both ends, computes the deltas, orders them by explanatory power, confirms by trace replay, and writes a one-paragraph causal chain.
- Trace diffing is a discipline: pair traces on near-identical inputs and diff at every layer, not just the output.
- Guardrail-effectiveness deltas and drift signals are first-class RCA evidence; the RCA cites versions and numbers, not prose.
- The app-side evidence chain is co-owned with `ai-eval-engineer` (L35); the SOC-side chain is co-owned with security IR. Both hand-offs are explicit records, not chat threads.
- Exercise-02 puts the workflow through a mock incident with the six surfaces synthesised for practice.
