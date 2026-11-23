# Lab 03 — Build-vs-Buy Matrix, Two Summary Numbers, Signed Decision Record, and Measurement Contract Drill

**Duration:** 45 minutes · **Format:** guided practice, markdown authoring on fabricated vendor profiles and fabricated in-house measurements

## Objective

Internalise chapter 06's **build-vs-buy artefacts** — the twelve-axis matrix where every cell is a number or a defended bounded rating, the **two summary numbers** per option (total residual-risk delta and total three-year TCO), the **signed decision record**, and the **app-side measurement contract** handed to `ai-eval-engineer` (L30) — by filling the four artefacts end-to-end on fabricated vendor profiles and fabricated in-house measurements for the `HelpDesk` running system from labs 01 and 02. The lab drills the artefact shape chapter 06 pins so that when exercise 06 scores six real vendors against your real running system, the engineering judgement goes into vendor-evidence collection, TCO honesty, and compose-seam design — not into inventing the matrix format.

The chapter-06 anti-patterns "vendor marketing as coverage story", "build with hidden TCO", and "silent duplication of eval methodology" are three of the four first-pass defects the shape forecloses; "compose without owning the seam" is the fourth and is drilled explicitly in the decision-record step.

## Prerequisites

- Chapter 06 read — specifically "The six vendors", "The build-vs-buy matrix", "Two summary numbers per option", "The decision — build, buy, or compose", "Handoff to `ai-eval-engineer` (peer, level 30)", and "Anti-patterns".
- Labs 01 and 02 completed (or at least read) — the `HelpDesk` system brief, the slot-to-mechanism assignment, and the fabricated in-house measurement numbers from lab 02 are the reference. The lab is self-contained if you need to run it standalone.
- A terminal where you can create a working directory and author a markdown file.
- No external access required; the vendor profiles and the in-house measurement anchors are fabricated below.

## The fixed example system (shared with labs 01 and 02)

Use this system throughout the lab — do *not* substitute your own.

> **`HelpDesk`** — a customer-support RAG assistant for a mid-size B2B SaaS company. In-house guardrail stack (lab 01 mechanism assignment): NeMo GR + Llama Guard 3 at input; NeMo GR + ShieldGemma 9B at retrieval; custom FastAPI middleware for tool-invocation policy; Guardrails AI + Llama Guard 3 + ShieldGemma 9B + Presidio at output; NeMo GR refusal template at refusal shaping.
>
> Current in-house measurement anchor numbers (abbreviated from lab 02):
>
> - Recall on `prompt injection — direct`: **0.91** (input slot).
> - Recall on `prompt injection — indirect`: **0.71** (retrieval slot) + tool allow-list catches 0.99 → composition residual inside appetite.
> - Recall on `PII — email`: **0.92**.
> - Recall on `PII — internal ticket ID`: **0.56** (open work).
> - FP on `internal ticket ID` legitimate slice: **10.9%** (open work).
> - FP on `advice-OOS` legitimate slice: **10.6%** (open work).
> - Output p95 (serial): **486 ms**; parallel: **238 ms** (slot over budget on serial; parallel sign-off queued in lab 02 decision #3).
> - Monthly cost: **$15,200** ($9,400 self-hosted GPU for Llama Guard + ShieldGemma; $1,600 CPU + NER; $4,200 operational; $0 vendor licence).
> - Data-residency posture: strict — customer content must not leave the organisation's VPC.

### Fabricated vendor profiles — authoritative for this lab, do not revise

Use the five fabricated vendor snapshots below. This lab only scores **five** of the chapter-06 six (Lakera, Arthur, Robust Intelligence, Calypso, HiddenLayer); Protect AI is dropped from the fabricated scope to keep the matrix bounded. The chapter-06 exercise scores all six against real vendor documentation; the lab's reduced scope is a scaffolding choice, not a recommendation. All numbers below are **fabricated for the lab** — do not treat them as real vendor claims or real evaluations.

**Lakera Guard (fabricated snapshot)**

- Deployment: SaaS (primary); self-hosted option is listed but not available on their standard contract.
- Data-residency: SaaS egress — customer content leaves HelpDesk's VPC. Red-flag against HelpDesk's strict posture.
- Vendor-reported recall on prompt-injection: ≈ 0.94 on their hosted harness (vendor marketing; not measured on `mod-104/adversarial-v0.9.2`).
- Vendor-reported latency: p95 ≈ 90 ms for the hosted endpoint (vendor marketing).
- Price: ≈ $0.004/call published list price (fabricated).
- Threshold / policy control: adjustable; custom rules via a policy builder.
- Evidence trail: dashboard with per-call verdict + category; export is CSV; version-pinning of the classifier is not advertised.

**Arthur AI (fabricated snapshot)**

- Deployment: SaaS and VPC (VPC via enterprise contract).
- Data-residency: VPC option meets HelpDesk's posture.
- Vendor-reported recall on prompt-injection: ≈ 0.89 on their published evaluation.
- Vendor-reported latency: p95 ≈ 140 ms in VPC.
- Price: subscription-shaped ($ per request block + platform fee); fabricated TCO at HelpDesk volume ≈ $22k/month.
- Threshold / policy control: policy-driven; supports custom rules; integrates with their monitoring platform (overlap with mod-109).
- Evidence trail: platform audit log; integrates with GRC-for-AI platforms including Monitaur; version-pinning via platform versioning.

**Robust Intelligence (fabricated snapshot; "now Cisco AI Defense")**

- Deployment: SaaS + self-hosted appliances (post-Cisco acquisition productisation; treat as evolving).
- Data-residency: self-hosted meets posture; SaaS does not.
- Vendor-reported recall on prompt-injection: ≈ 0.91 on their published evaluation.
- Vendor-reported latency: p95 ≈ 110 ms in self-hosted appliance.
- Price: enterprise-only; fabricated TCO at HelpDesk volume ≈ $38k/month inclusive of appliance.
- Threshold / policy control: broad — covers pre-deployment validation + runtime; strong on adversarial-ML testing overlap with mod-105.
- Evidence trail: enterprise-grade; integrates with SIEM; corporate risk after acquisition flagged as medium.

**Calypso AI (fabricated snapshot)**

- Deployment: VPC gateway + SaaS variant.
- Data-residency: VPC gateway meets posture.
- Vendor-reported recall on prompt-injection: ≈ 0.87.
- Vendor-reported latency: p95 ≈ 160 ms.
- Price: ≈ $26k/month at HelpDesk volume (fabricated).
- Threshold / policy control: policy builder with DLP focus.
- Evidence trail: platform logs; SOC2 attestations referenced in their marketing.

**HiddenLayer (fabricated snapshot)**

- Deployment: hybrid (agents + SaaS control plane).
- Data-residency: control-plane leaks metadata to SaaS; content stays local; HelpDesk needs to confirm the metadata-egress scope against its data-residency policy.
- Vendor-reported recall on prompt-injection: ≈ 0.82; historic strength is classical adversarial-ML detection (mod-105 overlap), not LLM-runtime.
- Vendor-reported latency: p95 ≈ 100 ms.
- Price: ≈ $19k/month (fabricated).
- Threshold / policy control: policy builder; strong on model-integrity and model-provenance scanning side.
- Evidence trail: integrates with MLOps security tooling; strong supply-chain side (mod-110 overlap).

### Fabricated measurement anchor — one real-vendor trial row

HelpDesk's team procured a two-week trial for **Lakera Guard** on a non-sensitive synthetic corpus (not production data; the data-residency constraint prevented a production trial). Trial run against `mod-104/adversarial-v0.9.2` on **prompt injection — direct** only:

- Recall: **0.89** (slightly below Lakera's vendor-reported 0.94).
- FP on the matching legitimate slice: **4.2%**.
- p95 latency in the trial harness: **132 ms** (above the vendor-reported 90 ms; the gap is the network round-trip from HelpDesk's VPC to Lakera's SaaS endpoint).

This is one measured cell out of the matrix's 60 — the rest are vendor-reported with the flag.

## Steps

### Step 1 — Create your working directory and the four-artefact scaffold

```bash
mkdir -p mod108-lab03 && cd mod108-lab03
touch helpdesk-matrix.md helpdesk-summary-numbers.md helpdesk-decision.md helpdesk-measurement-contract.md
```

These four files are the chapter-06 §"Requirements" artefacts you produce.

### Step 2 — Fill `helpdesk-matrix.md` (the twelve-axis matrix)

Paste this scaffold and fill the cells using the fabricated vendor profiles and the lab-02 in-house numbers. Every cell is either a **number** (recall %, FP %, ms, $/call), a **vendor-reported** flag, or a **defended qualitative rating** with a one-sentence note. No naked "high/medium/low" — chapter 06 §"The build-vs-buy matrix" is explicit.

```markdown
# HelpDesk — build-vs-buy matrix (lab 03)

System: HelpDesk (see lab 01 / 02 brief)
Chapter references: mod-108 ch 06 (vendor scope; matrix; summary numbers; decision; handoff contract)

Scope: five vendors profiled (Lakera Guard, Arthur AI, Robust Intelligence / Cisco AI Defense, Calypso AI, HiddenLayer) + the in-house stack. Protect AI dropped from this lab's scope (the chapter-06 exercise scores all six).

| Axis | In-house stack | Lakera Guard | Arthur AI | Robust Intelligence | Calypso AI | HiddenLayer |
|---|---|---|---|---|---|---|
| 1. Coverage on our adversarial set (recall on mod-104 injection-direct) | 0.91 (measured, run-helpdesk-2026-03-11-01) | 0.89 (measured, 2-week trial, non-prod corpus); 0.94 vendor-reported | 0.89 (vendor-reported; no trial) | 0.91 (vendor-reported) | 0.87 (vendor-reported) | 0.82 (vendor-reported; LLM-runtime is secondary strength) |
| 2. FP on our legitimate set | 3.1% (input Llama Guard, measured) | 4.2% (measured on injection-direct slice, non-prod) | vendor-reported "comparable"; no number | vendor-reported "low"; no number | vendor-reported; no number | vendor-reported; no number |
| 3. Latency added per slot (p95 in request path, ms) | input 118; retrieval 194; output serial 486 / parallel 238 | 132 (measured, injection-direct; SaaS round-trip adds ~40 ms over vendor-reported) | 140 (vendor-reported VPC) | 110 (vendor-reported appliance) | 160 (vendor-reported) | 100 (vendor-reported hybrid) |
| 4. Cost per call and per month (fully loaded) | $15.2k/month at 1.2M req/month | ≈ $4.8k/month at HelpDesk volume (list price × req count) | ≈ $22k/month (platform fee + per-req) | ≈ $38k/month (enterprise + appliance) | ≈ $26k/month | ≈ $19k/month |
| 5. Deployment shape / data residency | Self-hosted in VPC; **meets strict posture** | SaaS only on standard contract; **does not meet posture** | VPC on enterprise contract; **meets posture** | Self-hosted appliance; **meets posture** | VPC gateway; **meets posture** | Hybrid; metadata egress to SaaS control plane; **requires residency-policy confirmation** |
| 6. Portability / lock-in | Full ownership; portability high (NeMo GR + Guardrails AI are open source) | SDK + API; portability medium — classifier and taxonomy are vendor-specific | SDK + API; portability low — platform integration deepens over time | SDK + API; portability medium — appliance is replaceable but policy library is vendor-specific | SDK + API; portability medium | SDK + agent; portability medium |
| 7. Evidence trail | Full — NeMo GR trace + Guardrails AI validation report + middleware log; chapter-05 measurement rows are first-class artefacts | Dashboard + CSV export; version-pinning not advertised — **gap for ISO/IEC 42001 reproducibility** | Platform audit log; version-pinning via platform versioning; integrates with Monitaur / GRC-for-AI | Enterprise-grade; SIEM integration; version-pinning advertised | Platform logs; SOC2 attestations marketed | Agent logs + MLOps integrations; strong on supply-chain evidence (mod-110 overlap) |
| 8. Threshold / policy control | Full — per-policy thresholds, custom ShieldGemma policies, custom Presidio recognisers | Custom rules via policy builder; limited visibility into the classifier threshold surface | Policy-driven; custom rules; thresholds adjustable on the platform | Broad — covers pre-deployment validation + runtime | Policy builder with DLP focus | Policy builder; stronger on model-integrity than LLM-content |
| 9. Update cadence and predictability | Internally controlled — we pin versions, we decide rollout | Updates cadence vendor-controlled; pin-at-version is not advertised — **surprise-update risk** | Platform versioning advertised; roll-forward controllable | Appliance versioning controllable | Vendor-controlled cadence; details not public | Vendor-controlled cadence |
| 10. Support model | In-house on-call (fractional engineer); no external SLA | Vendor SLA on API uptime; LLM-safety-incident response tiered by contract | Enterprise SLA; platform-team support | Enterprise SLA post-acquisition; roadmap continuity to confirm | Enterprise SLA | Enterprise SLA; strong on security-side incidents |
| 11. Ecosystem fit (GRC-for-AI platform integration; mod-109 monitoring overlap) | Direct — our chapter-05 rows feed the register ourselves | Limited GRC integration today | Strong — Arthur Shield + Arthur Monitoring overlap mod-108 + mod-109; integrates with Monitaur | Enterprise security stack; strong SIEM fit; mod-109 overlap partial | SOC2 integrations advertised; GRC fit partial | Strong MLOps and supply-chain integrations (mod-110); weaker GRC-for-AI |
| 12. Long-term viability | In-house; subject to internal team changes; portability covers exit | Growing standalone vendor; funding runway published | Mid-stage vendor; recent product renames (Shield branding); watch | Post-Cisco acquisition; productisation evolving — medium risk flagged | Mid-stage vendor | Growing vendor with security-side focus |
```

Reading check: twelve rows; six columns; every cell is one of (number, measured / vendor-reported flag, defended qualitative rating with a note). No cell is blank; no cell is "TBD"; no cell is "High" without a sentence after it. The "data residency" row bolds the disqualification state so the next step can read the matrix cleanly.

### Step 3 — Fill `helpdesk-summary-numbers.md` (the two summary numbers per option)

Chapter 06 §"Two summary numbers per option" defines the two: **total residual-risk delta** (change in the register's aggregated residual score when this option is credited) and **total three-year TCO** (all cost lines summed over three years, including exit cost). Fabricate per-option numbers that are consistent with the matrix cells.

```markdown
# HelpDesk — build-vs-buy summary numbers (lab 03)

Chapter reference: mod-108 ch 06 §"Two summary numbers per option"

## Residual-risk delta (vs. the current in-house stack as baseline)

| Option | Residual-risk delta | One-sentence rationale |
|---|---|---|
| In-house stack (baseline) | 0 | Current measurement (lab 02) is the baseline aggregate |
| Lakera Guard | **Disqualified** on residency (axis 5) | Content egress to SaaS violates the HelpDesk residency posture — no trade to compute |
| Arthur AI | −0.5pp | Marginal recall improvement on injection-indirect if deployed at retrieval (vendor-reported); offset by platform operational complexity |
| Robust Intelligence | −0.3pp | Comparable recall, strong adversarial-ML evaluation layer; small register-row improvement on LLM03 / LLM04 cross-track (mod-110) |
| Calypso AI | +0.2pp | Slightly weaker vendor-reported recall; FP on legitimate set not measured against mod-104 — residual slightly worse until measured |
| HiddenLayer | +0.4pp | LLM-runtime is secondary strength; register improvement shows on supply-chain / mod-110 side rather than mod-108 slots |

Caveat: these are **not** measured deltas. The trial for Lakera is a single-category measurement and the other vendors were not trialled; the deltas above are **projections** against the vendor-reported recall numbers with the "vendor-reported" uncertainty margin explicit in the matrix. Chapter 06 §"Handoff" names the L30 eval engineer as the owner of statistical calibration; the signed decision record is annotated accordingly.

## Total three-year TCO

| Option | Year-1 ($k) | Year-2 ($k) | Year-3 ($k) | Exit cost ($k) | 3-year total ($k) |
|---|---|---|---|---|---|
| In-house stack | 182 (= $15.2k × 12) + on-call amortised | 191 (3% drift / growth) | 201 | 0 (we own it) | **574** |
| Lakera Guard | n/a — disqualified | | | | **n/a** |
| Arthur AI | 264 (= $22k × 12) + 40 integration | 277 | 291 | 60 exit (re-integrate in-house + retraining) | **932** |
| Robust Intelligence | 456 (= $38k × 12) + 80 appliance on-board | 479 | 503 | 100 exit | **1,618** |
| Calypso AI | 312 (= $26k × 12) + 40 integration | 328 | 344 | 70 exit | **1,094** |
| HiddenLayer | 228 (= $19k × 12) + 30 integration | 239 | 251 | 50 exit | **798** |

Line composition:
- Year-1 vendor TCO: list price × 12 + integration cost + on-boarding.
- Year-2 and year-3 compounded with 5% vendor-price assumption + 3% volume growth.
- Exit cost: integration removal + data-pipeline refactor + retraining or re-on-boarding a replacement.
- In-house exit cost = 0 because the IP stays with us.
- In-house TCO **includes** on-call rotation (fractional engineer × loaded rate), drift monitoring compute, model-retraining GPU spend. Chapter 06 §"Motivation" case 2 ("build with hidden TCO") is the anti-pattern this calculation is designed to foreclose.

## Scatter

Plotting residual-risk delta (x-axis) against three-year TCO (y-axis), the in-house stack is at **(0, 574)**, the vendor options cluster at **(−0.5 to +0.4, 798 to 1,618)**. Lakera is excluded (disqualified). The scatter shape makes the decision shape visible: no vendor clearly dominates the in-house baseline on both axes.
```

Reading check: both tables fully populated; Lakera is explicitly marked **Disqualified** at the residual-delta step (not dropped silently); the TCO line composition names the chapter-06 §"Motivation" anti-pattern that the in-house TCO calculation forecloses. The caveat about projections-not-measurements is not hidden.

### Step 4 — Fill `helpdesk-decision.md` (signed decision record)

Chapter 06 §"The decision — build, buy, or compose" names the three shapes and §"Anti-patterns" the "compose without owning the seam" discipline. Fabricate a plausible decision given the fabricated numbers.

```markdown
# HelpDesk — build-vs-buy signed decision record (lab 03)

Chapter reference: mod-108 ch 06 §"The decision — build, buy, or compose"; §"Two summary numbers per option"; §"Anti-patterns"

## Decision

**Build** — retain the in-house stack for the current product cycle. Compose is considered and rejected for this cycle; revisit at the next adversarial-set version bump (`mod-104/adversarial-v0.9.3`) or on any of the trigger events below.

## Driving axes

- **Axis 5 — Deployment shape / data residency.** HelpDesk's strict residency posture disqualifies Lakera on SaaS and requires VPC / self-hosted shapes for the other vendors. The cost difference between in-house and the residency-qualified vendors (Arthur, Robust Intelligence, Calypso, HiddenLayer) is material.
- **Axis 4 — Cost.** In-house three-year TCO is **$574k**; the cheapest residency-qualified vendor option (HiddenLayer) is **$798k**, a **~$224k gap over three years** for a vendor whose LLM-runtime strength is secondary to its mod-110 overlap. Arthur's TCO is **$932k** for a projected **−0.5pp residual improvement** — the improvement is both small and vendor-reported, not measured on our set.
- **Axis 1 and 2 — Coverage and FP.** The vendor-reported recall numbers cluster inside 0.07 of the in-house measured recall on injection-direct; none of the vendor numbers is measured on mod-104 for HelpDesk's set. The epistemic weight of the matrix's coverage row is lower for vendors than for in-house; this gap matters for a build-vs-buy decision whose residual-risk delta is near zero.
- **Axis 6 — Portability.** Full ownership today; moving to any vendor reduces portability and increases exit cost.

## Axes that were closest to changing the decision

- **Axis 11 — Ecosystem fit (Arthur AI).** Arthur's integration with Monitaur (our GRC-for-AI platform per mod-103) is strong and the Shield + monitoring combination overlaps mod-108 + mod-109. If HelpDesk's monitoring buildout in mod-109 turns out to carry non-trivial custom-engineering cost, Arthur's compose-at-retrieval-and-output option becomes a serious revisit candidate.
- **Axis 10 — Support model.** The in-house stack's "no external SLA" row is the one a risk council will ask about in an incident. If HelpDesk's on-call rotation loads beyond 5% of a senior engineer's time, the vendor-SLA option gains weight.

## Summary numbers supporting the decision

- Residual-risk delta vs. in-house: no residency-qualified vendor offers more than **−0.5pp** improvement, and the delta is projected not measured.
- Three-year TCO: in-house **$574k** vs. cheapest residency-qualified vendor **$798k**; TCO gap **~$224k**.

## Trigger events for revisit

- `mod-104/adversarial-v0.9.3` is released with new injection-indirect or obfuscated-encoding fixtures (the lab-02 FN categories). Re-measure in-house; re-score vendors; the decision revisits.
- HelpDesk's request volume crosses **3M requests / month** (vendor per-call pricing no longer scales linearly; the TCO gap narrows or inverts).
- A residency-posture change (e.g., a specific customer segment approved for SaaS) unlocks Lakera's cell for that segment — compose becomes feasible.
- Material change in vendor corporate status (Robust Intelligence post-acquisition productisation stabilises or regresses; any vendor is acquired or stops shipping updates).
- In-house on-call load for the stack crosses the 5%-of-a-senior-engineer threshold.

## Compose scenario considered and rejected for this cycle

The compose option evaluated was: **in-house for input / tool-invocation / Presidio + Arthur AI at retrieval and output**. Chapter 06 §"Anti-patterns" "compose without owning the seam" is the discipline:

- **Seam 1 — retrieval filter verdict handoff to the tool-invocation middleware.** If Arthur flags a chunk as "injection-likely" and drops it, the tool-invocation middleware's allow-list is the only defence left on HM-HD-INJ-01 — who owns the FP tuning if Arthur's FP rate is higher than ShieldGemma's today? Owning this seam means owning **both** sides of the FP / FN tradeoff at that handoff, including trace correlation (Arthur's trace format joined to our middleware's trace format via `request_id`).
- **Seam 2 — output validator interleaving.** Arthur Shield at the output alongside Guardrails AI + Presidio requires deciding ordering (Arthur before Presidio? after? both parallel?) and merging verdicts into one output decision. Chapter 06 names "compose without owning the seam" as the specific failure mode.
- The compose option is **viable** but not owned at this lab's timeline; the decision record names the seams and queues them as a formal design task for the next cycle, so a future compose decision is not blocked by a missing contract.

## Signatories

- Risk engineer: _(exercise-only placeholder)_
- Accountable executive: _(exercise-only placeholder)_
- Date of signature: _(placeholder; filed as evidence against the mod-103 register)_

The record is authored in the shape it would be signed in production. Exercise-only signatures do not commit a real procurement decision; the shape is what matters.
```

Reading check: the decision is explicit (one of build / buy / compose); the driving axes are **named by number** and quoted from the matrix, not vague; the trigger events are five concrete events a monitor could catch; the compose scenario is **considered and rejected with a stated reason**, not silently omitted. The signed-decision-record shape is the one chapter 06 §"The decision" pins.

### Step 5 — Fill `helpdesk-measurement-contract.md` (handoff to `ai-eval-engineer`)

Chapter 06 §"Handoff to `ai-eval-engineer` (peer, level 30)" defines the fields. The contract is a self-contained document — the L30 eval engineer consumes without reconstructing your slot set from the code.

```markdown
# HelpDesk — measurement contract to `ai-eval-engineer` (L30)

Chapter references: mod-108 ch 05 (measurement contract fields); ch 06 §"Handoff to `ai-eval-engineer` (peer, level 30)"
Cadence: re-issued on every `mod-104/adversarial-*` set version bump and every major control change (e.g., ShieldGemma threshold change > ±0.1; classifier swap; Presidio recogniser addition).

## 1. Slot definitions (from lab 01 / exercise 01)

| Slot id | Responsibility | Named failure mode | Mechanism |
|---|---|---|---|
| S1 — input filter | Reject harmful / injection-shaped user turns | Bypassed by novel / obfuscated attack | NeMo GR input rail + Llama Guard 3 |
| S2 — retrieval filter | Reject injection / disallowed / PII chunks pre-assembly | Trusts upstream text; misses obfuscated payloads | NeMo GR retrieval rail + ShieldGemma 9B + chunk-provenance rule |
| S3 — tool-invocation policy | Enforce allow-list / schema / quota | Allow-list drift | Custom FastAPI middleware |
| S4 — output validator | Enforce policy + PII scrub on response | Schema-only validation; streaming bypass | Guardrails AI + Llama Guard 3 + ShieldGemma 9B + Presidio per field |
| S5 — refusal shaping | Non-leaky refusal template per category | Refusal restates disallowed content | NeMo GR output rail refusal templates |

## 2. Control instances (from lab 01 mechanism table; chapters 02 / 03 / 04)

| Control instance id | Slot | Pinned version | Threshold / taxonomy scope |
|---|---|---|---|
| CI-LG3-S1 | S1 | Llama Guard 3, checkpoint `<ckpt>` | Taxonomy scope `{S1–S13}` |
| CI-SG9B-S2-inj | S2 | ShieldGemma 9B, checkpoint `<ckpt>` | Custom policy "injection-likely" @ 0.6 |
| CI-SG9B-S2-pii | S2 | ShieldGemma 9B, checkpoint `<ckpt>` | Built-in "sensitive-info" @ 0.5 |
| CI-TOOLPOL-S3 | S3 | Middleware git commit `<sha>` | Deterministic — allow-list `<destination-registry-version>` |
| CI-LG3-S4 | S4 | Llama Guard 3, checkpoint `<ckpt>` | Taxonomy scope `{S1–S13}` |
| CI-SG9B-S4-advice | S4 | ShieldGemma 9B, checkpoint `<ckpt>` | Custom policy "advice-outside-scope" @ 0.55 |
| CI-PRESIDIO-S4-email | S4 | Presidio `<ver>` + spaCy `en_core_web_lg <ver>` | Built-in `EMAIL_ADDRESS` recogniser @ 0.5 |
| CI-PRESIDIO-S4-tkt | S4 | Presidio `<ver>` + custom `PatternRecognizer` `<commit>` | Custom `TKT` recogniser @ 0.65 |

## 3. Set versions

- Adversarial: `mod-104/adversarial-v0.9.2` (hash `<hash>` in run manifest).
- Legitimate: `mod-104/legit-v0.9.2` (hash `<hash>` in run manifest).
- Cadence: a new row of this table is appended when mod-104 bumps either set version.

## 4. Measurement rows

Pointer: `risk/mod-108/helpdesk/measurement/measurements.csv` (lab 02 fabricated an excerpt). All 17 chapter-05 contract fields (`slot`, `control_instance`, `category`, `TP`, `FP`, `TN`, `FN`, `precision`, `recall`, `mean_cost_per_call`, `p50_latency_ms`, `p95_latency_ms`, `n_adversarial`, `n_legitimate`, `set_version`, `control_version`, `run_id`) are populated. Run id of current measurement: `run-helpdesk-2026-03-11-01`.

## 5. Cost currency and time units

- Currency: USD, matched to the mod-103 register's financial-impact bands.
- Latency unit: milliseconds, measured in the FastAPI middleware (wall-clock in the app, per chapter 05 §"Latency accounting").
- Reporting roll-up: per-slot per-day, per-surface per-month.

## 6. Register linkage

Each measurement row updates one or more mod-103 register rows:

| Measurement row category | Register row | Updates |
|---|---|---|
| prompt injection — direct (S1) | REG-HD-INJ-01 | Residual, and control-defeated "input filter down" |
| prompt injection — indirect (S2) | REG-HD-INJ-01 | Residual, and control-defeated "retrieval filter down" |
| destination allow-list violation (S3) | REG-HD-INJ-01 | Control-defeated "tool-invocation policy down" |
| PII — email (S4) | REG-HD-PII-02 | Residual |
| PII — internal ticket ID (S4) | REG-HD-PII-02 | Residual (currently below appetite — open work) |
| disallowed content — advice-OOS (S4) | REG-HD-OOS-03 | Residual |
| system-prompt leak (S4) | REG-HD-SYS-04 | Residual |

## 7. Explicit list of what this contract does NOT supply

Chapter 06 §"Handoff" is explicit that the following are the L30 eval engineer's contribution and must not be duplicated by the L25 risk engineer:

- **Bootstrap confidence intervals** on the recall / precision numbers. The L30 eval engineer computes these on the same run manifest.
- **Cohen's kappa** on judge-vs-human agreement where any row uses judge-based scoring.
- **Benchmark methodology-of-methodology** — the comparative framework across benchmarks (for the categories where HelpDesk's measurement maps to a public benchmark).
- **Distribution-shift robustness** quantification — how recall degrades under a stated shift in the adversarial-set shape.

The L25 risk engineer **uses** the L30 numbers (e.g., "recall 0.91 with 95% CI [0.88, 0.93]") in the register's evidence trail. "Silent duplication of eval methodology" is the named chapter-06 anti-pattern this section forecloses.

## 8. Contact and cadence

- Contract re-issued on every `mod-104/adversarial-*` set version bump.
- Contract re-issued on any control-instance version change (classifier checkpoint swap; threshold change beyond ±0.1 band; Presidio recogniser addition / major modification).
- Run manifests shared alongside the measurement CSV; both tracks reference the same manifest hash.
- Point of contact: HelpDesk risk engineer (this document's author) ↔ `ai-eval-engineer` at L30 (named counterpart).
```

Reading check: eight sections; the slot table carries the same slot ids the measurement CSV uses (`S1`–`S5`); the control-instance table pins versions explicitly; section 7 is the "what we do not supply" block that chapter 06 §"Handoff" pins; the cadence section names *when* the contract re-issues (not an abstract "regularly").

### Step 6 — Final checkpoint

Re-read the four artefacts. Confirm:

- **Matrix (`helpdesk-matrix.md`).** Twelve rows (axes 1–12); six columns; every cell is a number, a measured / vendor-reported flag, or a defended qualitative rating with a note. No "high/medium/low" alone. Disqualifying cells (Lakera residency) are explicit, not implied.
- **Summary numbers (`helpdesk-summary-numbers.md`).** Both summary numbers computed per option; the disqualified option is marked, not silently dropped; the in-house TCO explicitly includes on-call, drift monitoring, retraining (the "build with hidden TCO" anti-pattern foreclosed); projections-not-measurements caveat stated.
- **Decision (`helpdesk-decision.md`).** One of build / buy / compose; driving axes named by number; summary numbers cited; trigger events for revisit listed (not "we'll revisit later"); compose scenario considered with the seam-ownership failure named explicitly. Signatories block present even if placeholder for the lab.
- **Measurement contract (`helpdesk-measurement-contract.md`).** Slot ids match across lab 01, the matrix, the measurement CSV, and the register linkage; section 7 names what the L25 engineer does *not* supply so the L30 engineer's work is not duplicated; cadence is defined.

## Expected output

Four markdown documents under `mod108-lab03/`:

- `helpdesk-matrix.md` — the 12 × 6 matrix with defended cells.
- `helpdesk-summary-numbers.md` — residual-risk delta and three-year TCO per option; disqualifications explicit.
- `helpdesk-decision.md` — signed decision record naming the chosen option, driving axes, summary numbers, trigger events, and the considered-and-rejected compose scenario with seam-ownership notes.
- `helpdesk-measurement-contract.md` — self-contained handoff to `ai-eval-engineer` (L30) with slot ids, control instances with pinned versions, set versions, register linkage, and the explicit "what we don't supply" block.

Together these are the four artefacts chapter 06 §"Requirements" items 2–5 produce for the exercise; the lab's vendor profiles, numbers, and decision are fabricated, but the shape is the one exercise 06 asks you to produce on real vendor evidence and real measurements.

## Checkpoint questions (self-check)

1. A reviewer reads the matrix and says "most cells are vendor-reported; the only measured cell on a vendor is Lakera's single trial. Doesn't that make the matrix mostly fiction?" Per chapter 06, how is "vendor-reported" the right shape when a trial is not available, and what does the lab's single-trial row contribute to the matrix as a whole? (*Chapter 06 §"The build-vs-buy matrix" is explicit: cells are "numbers where measured, vendor-reported numbers where only the vendor's claim is available (flagged explicitly), or bounded ratings with a defended one-sentence note." The matrix is not fiction when "vendor-reported" is marked and the epistemic weight is lower; it is fiction only when a vendor's marketing number is passed off as measurement (the chapter-06 "vendor marketing as coverage story" anti-pattern). The single-trial row (Lakera on injection-direct at recall 0.89, below the vendor-reported 0.94) is the anchor point — it quantifies the vendor-reported-vs-measured gap for one cell, and gives every other vendor-reported cell a scale. The decision record is honest about the caveat in Section 3 and names the revisit trigger (new adversarial-set version → re-measure).*)

2. The decision record names an in-house TCO of **$574k** and the cheapest residency-qualified vendor at **$798k** — a $224k gap — and chooses build. A risk-council reviewer asks "why not take the TCO win and go build?" Per chapter 06 §"Two summary numbers", is TCO the sole decision driver, and what else does the record cite? (*Chapter 06 §"Two summary numbers per option" is explicit: "the chosen option is **not always the lowest TCO or the best residual**; it is the one whose combination the risk council can sign off on." The decision record cites four driving axes (residency, cost, coverage-and-FP epistemic weight, portability) and names two axes that were closest to changing the decision (ecosystem fit; support model). TCO is one axis among twelve; the record makes the whole weighing legible so that if a trigger event (volume crossing 3M req/month; on-call load crossing 5%; residency-posture change) moves the trade, the revisit is quick.*)

3. The compose scenario in the decision record names two seams (retrieval verdict handoff; output interleaving ordering) and files them as formal design tasks. Per chapter 06 §"Anti-patterns", why is "compose scenario considered and rejected" the right shape in a build decision rather than silently omitting it? (*Chapter 06 §"Anti-patterns" "compose without owning the seam" is the named failure mode. The seams exist whether the decision is compose or build — if build is revisited and compose becomes viable at a trigger event, the seams must still be owned. Naming them in the current decision record (even as rejected) means the seam design is **not** blocked by a missing contract when a future cycle considers compose. Silently omitting the compose scenario would hide the seam-ownership work from the risk council, which is exactly the anti-pattern the chapter names.*)

4. The measurement contract's Section 7 lists four things the L25 engineer explicitly does not supply. A new team lead asks "why not just compute bootstrap CIs ourselves — we have the data?" Per chapter 06 §"Handoff" and chapter 05 §"Cross-references", what is the specific anti-pattern this foreclosure addresses, and what is the escalation path when the L25 numbers and the L30 numbers disagree? (*Chapter 06 §"Handoff" and §"Anti-patterns" "silent duplication of eval methodology" is explicit: if the L25 risk engineer computes CIs manually because they weren't sure the L30 eval track was doing it, there are now two numbers with different CI methodologies, and the register row cannot cite both. The foreclosure is a coordination contract: L25 produces the measurement rows and the register linkage; L30 produces the statistical calibration; both reference the same run manifest. Chapter 06 §"Handoff — Cadence" names the disagreement resolution: "disagreements become a joint investigation" between the two tracks, with the shared run manifest as the common ground. The L25 register row cites the L30 CI in its evidence trail rather than duplicating the methodology.*)

## Why this lab matters

Exercise 06 asks you to profile six real vendors, score them against your measurement rows on twelve axes, collapse the matrix into two summary numbers, sign a decision record, and hand a measurement contract to `ai-eval-engineer` (L30). If the shape is new at exercise-06 authoring time, the budget blows on inventing the matrix format and figuring out what a signed decision record looks like, instead of on honest TCO computation, defensible vendor scoring, and seam-ownership design. This lab drills the four-artefact shape on fabricated vendors and fabricated numbers so that exercise 06's time goes into current-state vendor documentation (the "The six vendors" block is "a map, not a spec sheet" per chapter 06), trial measurement where feasible, and the compose-seam contract where a compose decision is on the table. The four artefacts are also the last ones the module produces; they close the build-vs-buy question, hand the measurement contract sideways to the eval track, and leave the running system with an auditable control story the risk council can sign off on.
