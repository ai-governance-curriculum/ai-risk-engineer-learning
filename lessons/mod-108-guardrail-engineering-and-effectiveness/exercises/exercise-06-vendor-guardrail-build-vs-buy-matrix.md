# Exercise 06 — Vendor Guardrail Build-vs-Buy Matrix and App-Side Measurement Contract

**Estimated effort:** 3 hours

## Objective

Evaluate the six third-party guardrail vendors from chapter 06 — **Lakera Guard**, **Arthur AI**, **Robust Intelligence**, **Calypso AI**, **HiddenLayer**, **Protect AI** — against the running system's exercise-01 slot set and exercise-05 measurement rows. Produce the **build-vs-buy matrix** with the twelve chapter-06 axes; collapse it into the **two summary numbers** (residual-risk delta, three-year TCO) per option; author the **signed decision record** (build, buy, or compose); and write the **app-side measurement contract** you hand to `ai-eval-engineer` (peer, level 30) so the eval track consumes your slot definitions without a translation project.

## Prerequisites

- Exercises 01–05 completed. The in-house stack's measurement rows are the reference against which vendors are compared.
- Chapter 06 read in full. The vendor scope block, the matrix, the two summary numbers, the three decision shapes, and the handoff contract are all reference shapes.
- Read access to each vendor's own current product documentation (chapter 06 §"The six vendors" is a map, not a spec sheet; vendor product lines evolve — cite current pages, not a cached read).
- Optional but strongly recommended: a **trial or sandbox** for at least one vendor for hands-on latency and coverage measurement, so at least one row of the matrix is numbers rather than paper. If your organisation cannot procure a trial in the exercise window, that is a real constraint — score that row as "vendor-reported" and be explicit about the limitation.
- The mod-103 register file so the residual-risk delta line of the summary numbers can be computed against the same appetite bands used in exercise 05.
- The mod-103 GRC-for-AI-platform notes (Credo AI / Holistic AI / ModelOp / Monitaur) for the "ecosystem fit" row.
- Contact with `ai-eval-engineer` (L30) before authoring the measurement contract — the contract is *for* them; you author, they consume; a five-minute conversation up front prevents a translation project later.

## Problem statement

Score the six vendors and the in-house stack on the chapter-06 matrix; produce the two summary numbers per option; sign a decision record; write the measurement contract to `ai-eval-engineer`.

### The six vendors

Chapter 06 §"The six vendors" is the map. For each vendor:

- **Lakera Guard** — real-time content-and-injection classifier product; SaaS and (in some deployments) self-hosted.
- **Arthur AI** — model-monitoring and guardrails platform, includes Arthur Shield for runtime guardrails.
- **Robust Intelligence** — AI security and validation platform; acquired by Cisco in 2024; confirm current productisation.
- **Calypso AI** — LLM security platform focused on prompt-injection detection, DLP, content filtering.
- **HiddenLayer** — AI security platform; historic strength on adversarial-ML detection; more recent LLM runtime protection.
- **Protect AI** — broad AI security portfolio (`ModelScan`, `Guardian`, `Recon`, `LayerX`, `Radar`); some open source.

None of these evaluations is a recommendation. Each vendor's coverage story is *its own*; the exercise is to measure each against your adversarial set and score against the same axes.

### The matrix

Chapter 06 §"The build-vs-buy matrix" defines the axes. One row per axis; one column per option (in-house stack, six vendors). Twelve axes:

1. **Coverage on our adversarial set** — recall on the mod-104 adversarial set per category.
2. **False-positive rate on our legitimate set.**
3. **Latency added per slot** — p50 and p95 in the request path.
4. **Cost per call and per month** — fully loaded.
5. **Deployment shape** — SaaS / VPC / on-prem; data-residency.
6. **Portability / lock-in.**
7. **Evidence trail.**
8. **Threshold / policy control.**
9. **Update cadence and predictability.**
10. **Support model.**
11. **Ecosystem fit.**
12. **Long-term viability.**

Every cell is a **number** (recall %, FP %, ms, $/call) or a **bounded qualitative rating with a defended one-sentence note** (never a naked "high/medium/low"). Chapter 06 §"The build-vs-buy matrix" is explicit: naked ratings turn the matrix into vibes.

### The two summary numbers

Chapter 06 §"Two summary numbers per option":

- **Total residual-risk delta.** The change in the mod-103 register's aggregated residual score when this option is credited as the control set. Positive = worse residual; negative = better.
- **Total three-year TCO.** All cost lines summed over three years, including exit cost (integration removal, data extraction, retraining, on-boarding replacement).

Plot both on a small scatter. The chosen option is not always the lowest TCO or the best residual; it is the combination the risk council can sign off on.

### The decision

Chapter 06 §"The decision — build, buy, or compose" defines the three shapes. Pick one for the running system:

- **Build.** In-house stack.
- **Buy.** Vendor stack.
- **Compose.** In-house for application-specific slots; vendor for commodity slots.

File the decision as a **signed decision record** naming the axes that drove it, the summary numbers that support it, and the trigger events that will cause it to be revisited (vendor pricing change, adversarial set version change, in-house on-call load change, corporate change of a vendor).

### The measurement contract to `ai-eval-engineer`

Chapter 06 §"Handoff to `ai-eval-engineer` (peer, level 30)" defines the contract fields. Author the contract as a self-contained document the L30 eval engineer can consume without needing to reconstruct your slot set from the code:

- Slot definitions (from exercise 01).
- Control instances (from exercises 02–04).
- Adversarial set version and legitimate set version (from exercise 05).
- Measurement rows (from exercise 05).
- Cost currency and time units.
- Register linkage (which mod-103 rows each measurement row updates).
- Explicit list of what you do **not** supply — bootstrap CIs, judge-vs-human agreement, benchmark methodology — that is the L30's contribution; do not duplicate.

Also name the cadence: contract re-issued on every adversarial-set version bump and every major control change.

## Requirements

Produce all seven items:

1. **Vendor profiles** — `risk/mod-108/<system-id>/vendors/<vendor>.md` — one file per vendor. Each file: the vendor's currently published product name(s), the deployment options today, the documented category taxonomy, the pricing model (per-call, per-throughput, per-seat, per-endpoint), the published evaluation methodology, and the source URL(s) with the access date. Chapter 06 §"The six vendors" is the map; the file is your own current-state snapshot.
2. **Matrix** — `risk/mod-108/<system-id>/vendors/matrix.md`. The chapter-06 axes as rows; the in-house stack and each vendor as columns. Cells are numbers where measured, "vendor-reported" numbers where only the vendor's claim is available (flagged explicitly), or bounded ratings with a defended one-sentence note. Chapter 06 §"The build-vs-buy matrix" is the discipline.
3. **Two summary numbers** — `risk/mod-108/<system-id>/vendors/summary-numbers.md`. Per option: total residual-risk delta (referenced against the mod-103 register aggregate) and total three-year TCO (with the line breakdown). Scatter plot or a small table equivalent.
4. **Signed decision record** — `risk/mod-108/<system-id>/vendors/decision.md`. Names the chosen option (build / buy / compose), the axes that drove it, the two summary numbers that support it, the trigger events for revisit, and the signatories (the risk engineer's name, the accountable executive's name — even if the exercise-only decision is not shipped to production, the record is authored in the shape it would be signed).
5. **Measurement contract** — `risk/mod-108/<system-id>/vendors/measurement-contract.md`. The chapter-06 §"Handoff" fields, populated for your system. Includes the slot definitions, the control instances with pinned versions, the mod-104 set versions, the exercise-05 measurement rows (pointer + inlined excerpt), the cost currency and time units, the register linkage, and the explicit list of what you do not supply.
6. **Cross-track handshake note** — `risk/mod-108/<system-id>/vendors/handshakes.md`. Names the peer / upstream / downstream tracks the decision affects: `ai-eval-engineer` (L30, receives the measurement contract), `senior-ai-governance-architect` (L50, receives the signed decision record if residual or TCO crosses program-level bands), `ai-privacy-engineer` (peer, coordinates on PII-vendor selection), `procurement / legal` (contract negotiation), `security architecture` (deployment shape / data-residency review). One paragraph per track: what they receive and when.
7. **Anti-pattern probe report** — `risk/mod-108/<system-id>/vendors/anti-patterns.md`. Four sections, one per chapter-06 anti-pattern:
   - **Vendor marketing as coverage story.** Confirmed each vendor's coverage cell is either your measurement on the mod-104 set (best), a vendor-reported number flagged as such, or a documented gap. No cell is a paraphrase of a vendor slide.
   - **Build with hidden TCO.** Confirmed the in-house stack's TCO includes on-call, drift monitoring, model retraining, GPU spend, integration maintenance. If any line is omitted, called out.
   - **Compose without owning the seam.** If the decision is compose, confirmed the contract at the seam (who owns FP tuning where the vendor and in-house controls meet, who owns latency when both fire) is written.
   - **Silent duplication of eval methodology.** Confirmed the measurement contract explicitly names what the risk engineer does not supply (bootstrap CIs, judge-vs-human agreement) so the L30 eval engineer's work is not duplicated.

Cross-cutting rules:

- **Cite vendor sources with access dates** (chapter 06 §"The six vendors"). Vendor product pages are volatile; a citation without an access date is a cached read that will be wrong by the next quarter.
- **Score every option on the same axes with the same evidence source.** The in-house stack's coverage number comes from exercise 05; the vendor coverage number should come from a trial run against the same mod-104 set if at all possible. If a vendor cannot be measured, that limitation is a matrix cell, not a footnote.
- **Numbers or defended ratings; no naked "high/medium/low"** (chapter 06 §"The build-vs-buy matrix"). The auditor will ask "why medium?" and a naked rating collapses.
- **The decision is not always the cheapest or the best residual** (chapter 06 §"Two summary numbers"). The decision record names the trade explicitly so the reader knows the axes that drove it.
- **The measurement contract is what your peer at L30 needs** (chapter 06 §"Handoff"). Author it for that reader, not for your own notes.

## Starter guidance

- **Start from the exercise-01 slot set, not from vendor feature lists.** A vendor may sell twelve features; the ones that matter to your running system are the ones that address a slot you have. Score against your slots, not against their feature grid.
- **Get one vendor into a trial if at all possible.** Even one measured row (Lakera Guard trial on your mod-104 set for one adversarial category) turns the matrix from "all paper" into "at least one anchor point." The other vendors' cells then have a scale to be measured against.
- **The TCO calculation is the part shops most often get wrong.** In-house is not free — chapter 06 §"Motivation" case 2 is the anti-pattern. Amortise the on-call rotation (fraction of an engineer's time × their loaded rate), the drift-monitoring compute, the model-retraining GPU spend, and the maintenance cost of every LangChain / LangGraph API break. Vendor is not just list price — include integration cost, data-egress cost, on-boarding time, exit cost (integration removal, retraining a replacement).
- **The vendor-reported cell is honest when marked.** Do not pretend a vendor's marketing recall number was measured on your set. Cite the source, mark the cell as vendor-reported, and note the epistemic weight is lower.
- **The compose seam is where most compose decisions fail** (chapter 06 §"Anti-patterns"). If you pick compose, walk one end-to-end request path in writing and name every hand-off between vendor and in-house code; the contract lives at those hand-offs.
- **The measurement contract to L30 is not a debrief.** It is a machine-readable-ish handoff: slot IDs, control instance IDs, set version identifiers, register row identifiers. Write it in a way the L30 engineer can consume without a meeting.
- **Long-term viability is one row, not the whole matrix.** M&A activity, funding-round runway, and geographic-concentration risk are inputs; do not let one bad viability rating dominate the decision if the coverage / cost / latency numbers are strong. The risk council weighs.

## Acceptance criteria

The submission is accepted when:

- Six vendor profiles exist, each citing current product name(s), deployment options, taxonomy, pricing model, evaluation methodology, and source URL with access date.
- The matrix has all twelve axes as rows and all seven options as columns; every cell is populated with a number, a vendor-reported flag, or a defended qualitative rating.
- The two summary numbers per option are computed and plotted or tabulated together.
- The signed decision record exists in the chapter-06 shape (chosen option, driving axes, summary numbers, trigger events, signatories).
- The measurement contract to `ai-eval-engineer` is a self-contained document with every chapter-06 §"Handoff" field populated, including the explicit list of what is not supplied.
- The cross-track handshake note names each affected track and what they receive.
- The anti-pattern probe report covers all four chapter-06 anti-patterns and either resolves each or files a follow-up.
- A one-paragraph interpretive note names which axis drove the decision and which axis (if any) was closest to changing it.

## Stretch goals

- **A real vendor trial for one category.** Procure a trial or sandbox for one vendor, run the mod-104 adversarial and legitimate sets for one category through it, populate one full row of the matrix with measured numbers, and compare to the in-house stack's number on the same category. This turns one column of the matrix into evidence.
- **A per-slot vendor breakdown.** Some vendors are strong on the input slot and weak on the output slot; the matrix hides that. Author a supplementary per-slot table that scores each vendor per slot on coverage / FP / latency / cost so a "compose" decision can be built cleanly.
- **A three-year TCO sensitivity analysis.** Vary the request volume, the on-call load, and the vendor list-price assumptions ±30% and observe how the summary number moves. Chapter 06 §"The decision — build, buy, or compose" is stronger when the decision survives sensitivity variation.
- **A supply-chain due-diligence appendix.** Chapter 06 §"Cross-references" flags mod-110 for supply-chain concerns. Draft the vendor supply-chain due-diligence questions for the top two vendors (data-lineage of training data for their classifiers, incident-disclosure policy, sub-processor list, key-management shape). Do not conduct the full mod-110 review here; produce the handoff.
- **A GRC-platform integration column.** For each vendor, name whether it integrates with your organisation's chosen GRC-for-AI platform (Credo AI / Holistic AI / ModelOp / Monitaur — mod-103 has the survey). This is the "ecosystem fit" row made concrete.
- **A revisit trigger runbook.** Chapter 06 §"The decision" mentions trigger events. Write the runbook for one of them (e.g., "vendor announces material product change") — who is notified, what evidence is regathered, how quickly the decision is re-scored.

## Deliverable format

A single markdown document `mod-108-ex06-vendors.md` that indexes the artefacts. The index document contains:

1. *System and scope* — one paragraph naming the running system, the slot set from exercise 01, and the six vendors evaluated.
2. *Vendor profile summaries* — one paragraph per vendor (short) with pointer to the full profile file.
3. *Matrix* — inline the matrix in a compact form or table pointer.
4. *Two summary numbers* — the scatter plot / table.
5. *Signed decision record* — the block from `decision.md`.
6. *Measurement contract* — the block from `measurement-contract.md`, ready to hand to L30.
7. *Cross-track handshake note* — the block from `handshakes.md`.
8. *Interpretive paragraph* — the axis that drove the decision, the axis that was closest to changing it, the trigger event most likely to force a revisit.
9. *Anti-pattern probe report* — the four chapter-06 sections.
10. *Open items* — any vendor that could not be evaluated with confidence, any axis that needs new evidence before the decision can be signed for real, any escalation to L50 (`senior-ai-governance-architect`) if residual or TCO crosses program-level bands.

## Submission

Attach the index markdown plus the seven artefacts (six vendor profiles + matrix + summary numbers + decision + measurement contract + handshakes + anti-patterns). Send the measurement contract separately to `ai-eval-engineer` (L30). File the signed decision record with the risk register (mod-103) so the register's control column carries a pointer to the decision that credits the current stack. If the running-system decision would trigger a real procurement, coordinate with procurement and legal before treating the exercise decision as production commitment; the exercise produces the artefact, not the purchase order.
