# Lab 01 — Slot × Attack-Class Coverage Table Authoring Drill

**Duration:** 45 minutes · **Format:** guided practice, markdown authoring on a fixed fabricated LLM application

## Objective

Internalise chapter 01's **five-slot architecture** and the **slot × attack-class coverage table** — slot specification with named responsibility and named failure mode, coverage rows that pin OWASP LLM Top 10 / MITRE ATLAS / Google SAIF cells, the composition walk-through that defends defence in depth against single defeats, and the empty-cells-enumerated / addressed-not-claimed disciplines — by filling the shape end-to-end for a fixed example system (`HelpDesk`). The lab drills the artefact shape chapter 01 pins so that when exercise 01 produces the coverage table for your real running system, the engineering judgement goes into mechanism choice and defeat-sequence analysis, not into inventing the artefact format.

The two first-pass defects chapter 01 names — the "one guardrail" architecture and the "many guardrails, no coverage" architecture — are the anti-patterns the shape forecloses.

## Prerequisites

- Chapter 01 read — specifically "The slot set", "Defence in depth — the composition rule", "The silent single-point-of-failure anti-pattern", "Slot × attack-class coverage table", and "Worked example — customer-support RAG assistant".
- Chapter 02 skimmed at the "When rule-based fits and when classifier-based fits" and "A worked composition" tables so the mechanism-per-slot choices make sense; chapters 03 and 04 skimmed at their summary tables so classifier and Presidio candidates are recognisable.
- A terminal where you can create a working directory and author a markdown file.
- No external access or compute required; the system description and the harm-model stubs are fixed below.

## The fixed example system

Use this system throughout the lab — do *not* substitute your own. Carrying a fixed scenario keeps the lab bounded to the coverage-table authoring skill.

> **`HelpDesk`** — a customer-support RAG assistant for a mid-size B2B SaaS company. Serves English-language customers of a billing-and-provisioning product. Assembles context from a knowledge-base store (product docs + internal runbooks) and from the authenticated caller's own recent ticket history. Backed by an instruction-tuned LLM behind a FastAPI service wrapping a LangGraph agent. Two production tool calls: `escalate_ticket(destination, summary)` which hands a thread to a human support queue, and `lookup_account(account_id)` which reads the caller's own account record from an internal service. Returns a structured JSON response with fields `answer` (free text), `citations` (list of KB chunk URLs), and optional `escalate_ticket` tool-call block.
>
> Deployment: FastAPI + LangGraph + LangChain retriever over an internal pgvector store; streaming completion endpoint is enabled for the caller-facing UI.
> Downstream consumer: the caller (via the chat UI) and, on escalation, a human support agent reading the thread.
> Population: authenticated paying customers of the SaaS product; a small internal-admin surface shares the same service but a different caller identity.

### Harm-model and register row stubs (fixed — do not add more)

| Row id | Scope line |
|---|---|
| `HM-HD-INJ-01` | Indirect prompt injection through a retrieved KB article that induces `escalate_ticket` to an attacker-controlled destination (loss of ticket content; potential exfiltration of caller PII inside `summary`). |
| `HM-HD-PII-02` | The assistant quotes customer PII (email, phone, account number) from a retrieved chunk back to the caller who is *not* entitled to see that field (per-field entitlement failure). |
| `HM-HD-OOS-03` | The assistant answers a specialised-advice question that is out of scope (financial / legal advice about the customer's business) in a calibrated-sounding way and the caller relies on it. |
| `HM-HD-SYS-04` | The assistant discloses parts of its system prompt or internal policy to a caller who crafted a leak-shaped user turn. |
| `HM-HD-ABU-05` | A caller sends disallowed-content user turns (slurs, harassment directed at the assistant) and the assistant either engages or logs the content unredacted into the KB feedback pipeline. |

Register rows (fixed stubs — the exercise-01 artefact populates these):

| Register row id | Scope line |
|---|---|
| `REG-HD-INJ-01` | Inherent and residual risk on `HM-HD-INJ-01` — defence in depth across retrieval filter, tool-invocation policy, and output validator. |
| `REG-HD-PII-02` | Inherent and residual risk on `HM-HD-PII-02` — defence in depth across ingest-side Presidio, retrieval-side Presidio, and output-side Presidio. |
| `REG-HD-OOS-03` | Inherent and residual risk on `HM-HD-OOS-03` — defence in depth across input scoping and output policy classifier plus refusal-shaping template. |
| `REG-HD-SYS-04` | Inherent and residual risk on `HM-HD-SYS-04` — refusal-shaping template plus output validator scrub. |
| `REG-HD-ABU-05` | Inherent and residual risk on `HM-HD-ABU-05` — input classifier plus logging-side Presidio scrub. |

### Framework versions (fixed — pin these verbatim at the top of the coverage table)

Copy these as given. Chapter 01's "pin the version" discipline requires these to live in the artefact header.

- **OWASP LLM Top 10** — v2025 revision (LLM01–LLM10 codes per the 2025 reorder). <!-- needs-research: confirm the exact revision identifier shipped at reader's authoring time; the v2025 codes are the authoritative set for this lab. -->
- **MITRE ATLAS matrix** — current public matrix at authoring time. <!-- needs-research: pin the exact matrix version identifier from the ATLAS site. -->
- **Google SAIF** — the published six-pillar set. <!-- needs-research: cite the current SAIF pillar list from the Google SAIF site. -->

### Fabricated mechanism candidates (authoritative for this lab, do not revise)

Use these mechanism choices per slot so the coverage table and the composition walk-through are reproducible. Chapter-02 §"A worked composition" is the pattern.

| Slot | Mechanism candidate |
|---|---|
| Input filter | NeMo GR input rail: signature denylist + Llama Guard 3 action (self-hosted, taxonomy scope `{S1–S13}`) |
| Retrieval filter | NeMo GR retrieval rail: chunk-provenance check (KB source must be on the trusted-source allow-list) + ShieldGemma 9B action with custom policy "injection-likely" and built-in "unsafe" / "sensitive-info" |
| Tool-invocation policy | Custom FastAPI middleware wrapping the LangGraph tool nodes: destination allow-list for `escalate_ticket`, argument schema for both tools, per-caller quota |
| Output validator | Guardrails AI guard on the response JSON: per-field validators (`answer` — Llama Guard 3 + ShieldGemma 9B + Presidio PII; `citations` — URL-allow-list; `escalate_ticket` — argument schema), both classifiers must pass on `answer` |
| Safe-completion / refusal shaping | NeMo GR output rail: templated refusal per named category (no classifier) |

## Steps

### Step 1 — Create your working directory and the report scaffold

```bash
mkdir -p mod108-lab01 && cd mod108-lab01
touch helpdesk-coverage.md
```

Open `helpdesk-coverage.md` and paste this scaffold. You will fill the seven sections in order.

```markdown
# HelpDesk — slot × attack-class coverage artefact (lab 01)

System: HelpDesk (customer-support RAG assistant; see lab brief)
Chapter references: mod-108 ch 01 (slot set; defence in depth; silent SPOF anti-patterns; coverage table); ch 02 (rule / flow layer); ch 03 (classifier layer); ch 04 (Presidio PII layer)
Harm-model rows cited: HM-HD-INJ-01, HM-HD-PII-02, HM-HD-OOS-03, HM-HD-SYS-04, HM-HD-ABU-05
Register rows cited: REG-HD-INJ-01, REG-HD-PII-02, REG-HD-OOS-03, REG-HD-SYS-04, REG-HD-ABU-05
Framework versions pinned: OWASP LLM Top 10 v2025; MITRE ATLAS current matrix; Google SAIF six-pillar set

## 1. Framework version header

(OWASP LLM Top 10 version; MITRE ATLAS matrix version; Google SAIF pillar set)

## 2. Slot specification

(one block per slot: named responsibility, named failure mode, chosen mechanism candidate)

## 3. Coverage table — slot × OWASP LLM Top 10 / MITRE ATLAS / Google SAIF

(rows = slots; columns = the three frameworks; cells are cell codes or "—")

## 4. Scenario-slot matrix

(rows = HM-HD-* scenarios; columns = the five slots; each cell is Y / partial / N; empty rows flagged)

## 5. Composition walk-through — scenarios A, B, C

(scenario A: at least two slots address, walk the defeat sequence; scenario B: only one slot addresses, propose a second mechanism or file open work; scenario C: name the silent-SPOF anti-pattern most plausible for HelpDesk)

## 6. Register handoff note

(for each HM-HD-* scenario, name which REG-HD-* row's residual / control-defeated columns will update after chapter-05 measurement)

## 7. Open items

(what escalates to mod-112; what the chapter-05 run must measure to credit residual)
```

### Step 2 — Fill Section 1 (Framework version header)

Pin the versions verbatim. Chapter 01 §"Slot × attack-class coverage table" is explicit that empty cells get marked "—" and that the framework versions live in the header.

```markdown
| Framework | Pinned version |
|---|---|
| OWASP LLM Top 10 | v2025 revision (LLM01–LLM10 codes per the 2025 reorder) |
| MITRE ATLAS matrix | Current public matrix at authoring time |
| Google SAIF | Published six-pillar set |
```

Reading check: three rows; each row's version is named and unambiguous. If a reader cannot tell which OWASP numbering you cite, the coverage table's codes are not comparable.

### Step 3 — Fill Section 2 (Slot specification)

One block per slot. Each block has three lines: named responsibility, named failure mode (from chapter 01 §"The slot set" or application-specific if deeper), chosen mechanism candidate (from the lab's fabricated candidates table).

```markdown
### Slot 1 — Input filter
- Responsibility: Reject or transform obviously harmful, injection-shaped, or policy-violating user turns before they reach the LLM.
- Named failure mode: Bypassed by novel or obfuscated attack patterns; drifts as the attack surface changes.
- Mechanism: NeMo GR input rail: signature denylist + Llama Guard 3 action (self-hosted; taxonomy scope `{S1–S13}`).

### Slot 2 — Retrieval filter
- Responsibility: Reject or transform retrieved chunks that carry injection payloads, disallowed content, or PII the caller must not see.
- Named failure mode: Trusts unauthenticated upstream text; misses injection payloads that survive chunking.
- Mechanism: NeMo GR retrieval rail: chunk-provenance check (trusted-source allow-list) + ShieldGemma 9B with custom policy "injection-likely" and built-in "unsafe" / "sensitive-info".

### Slot 3 — Tool-invocation policy
- Responsibility: Enforce allow-list, argument schema, per-tool quota on every tool call (`escalate_ticket`, `lookup_account`).
- Named failure mode: Allow-list drift; argument-schema bypass; destination validation weakened under a "known enterprise customer" exception.
- Mechanism: Custom FastAPI middleware wrapping the LangGraph tool nodes (deterministic; chapter 02 §"When rule-based fits" — tool-call policy is deterministic by definition).

### Slot 4 — Output validator
- Responsibility: Enforce JSON schema and policy on the response body; catch what the input and retrieval filters missed; strip PII fields the caller must not see.
- Named failure mode: Schema-only validation with no policy layer; validation off in the streaming path.
- Mechanism: Guardrails AI guard: `answer` field validated by Llama Guard 3 + ShieldGemma 9B (both must pass) + Presidio PII; `citations` URL allow-list; `escalate_ticket` argument schema.

### Slot 5 — Safe-completion / refusal shaping
- Responsibility: On refusal or partial answer, produce a calibrated, non-leaky refusal that does not itself become an attack surface.
- Named failure mode: Refusal string leaks internal policy or restates the disallowed content.
- Mechanism: NeMo GR output rail with per-category refusal templates (no classifier).
```

Reading check: five blocks, each with three lines; each failure mode is specific to the slot, not generic; each mechanism names a concrete tool, not "classifier" or "regex" in the abstract.

### Step 4 — Fill Section 3 (Coverage table)

Rows are the five slots; columns are the three frameworks. Cells carry the entry code (e.g., `LLM01`) or `—` where the slot does not address that framework's category. Chapter 01 §"Slot × attack-class coverage table" is the pattern.

```markdown
| Slot | OWASP LLM Top 10 (v2025) | MITRE ATLAS tactic | Google SAIF pillar |
|---|---|---|---|
| Input filter | LLM01 Prompt Injection (direct); LLM05 Improper Output Handling (upstream-partial: catching obviously-malicious inputs that would otherwise shape bad output); LLM09 Misinformation (input side) | Initial Access; ML Model Access; ML Attack Staging | Extend strong security foundations to the AI ecosystem; Extend detection and response |
| Retrieval filter | LLM01 Prompt Injection (indirect); LLM06 Sensitive Information Disclosure (via retrieved PII); LLM08 Vector and Embedding Weaknesses | ML Supply Chain Compromise; Initial Access via a compromised data source | Automate defences to keep pace; Adapt controls to adjust mitigations and create faster feedback loops |
| Tool-invocation policy | LLM06 Sensitive Information Disclosure (tool-layer exfiltration via `escalate_ticket` destination); LLM07 System Prompt Leakage (agent path — partial, where tool-call shape would carry prompt fragments); LLM10 Unbounded Consumption (per-tool quota partial) | Execution; Exfiltration; Impact | Contextualise AI system risks in surrounding business processes; Extend strong security foundations |
| Output validator | LLM01 (post-hoc check); LLM05 Improper Output Handling; LLM06 Sensitive Information Disclosure (per-field PII scrub); LLM09 Misinformation | Exfiltration; Impact | Automate defences to keep pace; Harmonise platform-level controls |
| Safe-completion / refusal shaping | LLM07 System Prompt Leakage (don't restate policy in the refusal); LLM09 Misinformation (calibrated refusal instead of confident wrong answer) | Impact (reduce blast radius on refusal); Defence Evasion (deny leaking policy to the attacker) | Harmonise platform-level controls; Contextualise AI system risks |
```

Reading check: every cell is either a specific framework code / tactic / pillar name, or `—`. No cell reads "various" or "multiple." Partial-coverage cells name the sub-scenario the slot addresses (e.g., "LLM05 Improper Output Handling (upstream-partial: catching obviously-malicious inputs that would otherwise shape bad output)"). The LLM02 (Sensitive Information Disclosure retiring code), LLM03, and LLM04 codes do not appear because they are not addressed by any of the five slots in this design — that is a cell *considered*, and section 7 (Open items) will name them explicitly.

### Step 5 — Fill Section 4 (Scenario-slot matrix)

Rows are the five harm-model scenarios; columns are the five slots. Each cell is `Y` (slot addresses the scenario), `partial` (addresses a sub-scenario with a note), or `N` (does not). Chapter 01 §"Slot × attack-class coverage table" "empty cells are enumerated, not hidden" — every cell is filled.

```markdown
| Scenario | Input filter | Retrieval filter | Tool-invocation policy | Output validator | Refusal shaping |
|---|---|---|---|---|---|
| HM-HD-INJ-01 (indirect injection → tool exfil) | partial (catches direct injection phrasing if the chunk has one) | Y (provenance + ShieldGemma "injection-likely") | Y (destination allow-list) | partial (classifier may flag the response) | N |
| HM-HD-PII-02 (quoted PII back to non-entitled caller) | N | Y (per-caller entitlement redacts PII before assembly) | N | Y (per-field Presidio scrub) | N |
| HM-HD-OOS-03 (out-of-scope specialised advice) | partial (Llama Guard may flag the ask) | N | N | Y (classifier catches advice-shaped output) | Y (refusal template routes to canned decline) |
| HM-HD-SYS-04 (system-prompt / policy leak) | partial (input classifier catches obvious leak-shaped user turns) | N | N | Y (output validator scrubs prompt fragments) | Y (refusal template does not restate policy) |
| HM-HD-ABU-05 (disallowed-content user turns) | Y (Llama Guard rejects) | N | N | Y (classifier on the engagement, if generated) | Y (templated refusal, not engagement) |
```

Reading check: every cell is `Y`, `partial`, or `N` — no blanks. Any row that is entirely `N` would be a scenario with zero coverage (none exists in this matrix; if one did, section 5 scenario B would be where it goes). Any cell marked `partial` in a load-bearing slot is a design signal that the composition rests on the other `Y` slots.

### Step 6 — Fill Section 5 (Composition walk-through)

Three scenarios A, B, C. Each walks the chapter-01 §"Defence in depth" composition test.

```markdown
### Scenario A — HM-HD-INJ-01 (indirect prompt injection via retrieved KB article → `escalate_ticket` to attacker-controlled destination)

Addressing slots (from the matrix): retrieval filter (Y), tool-invocation policy (Y), output validator (partial).

Defeat sequence per chapter 01 §"Worked example":

- **Retrieval filter defeated.** A novel injection payload the ShieldGemma "injection-likely" policy does not catch *and* whose source is on the trusted allow-list (e.g., a KB article edited by an internal author who did not realise the embedded content was adversarial). The tool-invocation policy's destination allow-list still rejects an off-registry destination → residual moves from Medium to High but **not** to inherent. The attacker can induce an escalation to an *allowed* destination, which is a reduced-impact outcome but still an incident.
- **Tool-invocation policy defeated** (on top of retrieval filter defeated). A change-management gap during a debug session disables the destination allow-list. The injection payload now redirects `escalate_ticket` to an arbitrary destination. Residual moves to Very High / near-inherent. The output validator does not see the tool call's side effect because the call has already fired by the time the response is validated.
- **Only tool-invocation policy defeated** (retrieval filter still catching novel injection shapes). The injection payload is stopped at retrieval; the tool allow-list being disabled does not create an exposure on *this* scenario (it might on another). Residual is unchanged for HM-HD-INJ-01.

The composition is defence in depth: two slots use **different mechanisms** (chunk-provenance rule + classifier at retrieval; deterministic allow-list + schema at the tool policy); each has independent evidence trail (NeMo GR trace per retrieval event; middleware log per tool call). The output-validator cell is marked `partial` because the classifier on the response text is a probabilistic additional catch, not a load-bearing control for this scenario.

### Scenario B — HM-HD-PII-02 (quoted customer PII to a non-entitled caller) — only "Y" slots are retrieval filter and output validator

Both addressing slots are Presidio-based (the retrieval-side pass and the output-side pass). Chapter 01 §"Defence in depth — the composition rule" is explicit that the two slots must use **different mechanisms**. Two Presidio passes with similar recogniser configurations are not strictly independent — they catch different runtime manifestations (chunk vs. final response) but share one library's recogniser engine. If the recogniser engine mis-tunes, both slots mis-tune together.

Propose a second mechanism to complete defence in depth: add an **ingest-side Presidio pass** at the pre-slot (the KB ingest pipeline, chapter 04 §"The three integration paths") and treat "chunks without the ingest-side pass having run" as a provenance-failure the retrieval filter rejects. Ingest-side runs offline with the heavy transformer-based NER recognisers; retrieval and output use the fast regex+context recognisers. Mechanisms now differ: offline NER (ingest) + fast regex (retrieval and output) + per-caller entitlement rule — three slots, three mechanisms, independent evidence trails.

File this as a REG-HD-PII-02 residual-update candidate for chapter-05 measurement; the ingest-side layer lands in exercise 04.

### Scenario C — silent-SPOF anti-pattern most plausible for HelpDesk

Chapter 01 §"The silent single-point-of-failure anti-pattern" names four. Score each for HelpDesk:

- **Shared model, shared bypass.** Plausible but mitigated — the output-slot composition is Llama Guard **plus** ShieldGemma, not two instances of the same model.
- **Fail-open on error.** Highly plausible — HelpDesk's classifier calls run in a LangGraph node; if the classifier HTTP call times out, the default behaviour of a naive middleware is to return the raw model response. The fix is explicit in chapter 02 §"Anti-patterns" ("actions fail closed on error, not open"); the NeMo GR action layer must raise on error and the middleware must translate that to a refusal.
- **Streaming bypass.** Highly plausible — HelpDesk ships a streaming completion endpoint. Chapter 01 §"The silent single-point-of-failure anti-pattern" case 3 names this: "the output validator runs on the completed response, but the streaming path forwards tokens as they arrive. If the caller consumes the streaming path, the validator never runs on the delivered bytes." The fix is to ship the output guard in a way that validates incrementally on the streamed tokens or buffers to a size that makes validation cheap enough per chunk; either way, the design must name this.
- **Debug bypass.** Plausible — the admin surface shares the service; an `X-Debug: true` header or an internal-caller identity could silently disable slots. The fix is that any bypass is logged, rate-limited, and network-gated.

The most plausible silent SPOF for HelpDesk is **streaming bypass** (the streaming endpoint is a product feature; the output validator is specified on the non-streaming path). File this as a design finding; the exercise-02 custom-middleware work must address it before HelpDesk ships to the streaming UI.
```

Reading check: scenario A walks three defeat sequences and names the mechanism-difference that qualifies as defence in depth; scenario B proposes a specific second mechanism (not "add another classifier"); scenario C names *one* specific anti-pattern as most plausible with a reason, not a generic survey.

### Step 7 — Fill Section 6 (Register handoff note) and Section 7 (Open items)

```markdown
## 6. Register handoff note

Each row below names the REG-HD-* register row that will update after chapter-05 measurement (exercise 05). **No residual credit yet** — the register is updated only when the chapter-05 measurement produces the TP / FP / TN / FN row.

| Scenario | Register row to update | What the chapter-05 run must measure to credit residual |
|---|---|---|
| HM-HD-INJ-01 | REG-HD-INJ-01 | Recall on "prompt injection — indirect" at the retrieval filter *and* allow-list-violation rate at the tool-invocation policy on the mod-104 adversarial set; legitimate-set FP on retrieval filter to show benign retrieved chunks pass |
| HM-HD-PII-02 | REG-HD-PII-02 | Per-recogniser precision / recall at the retrieval and output Presidio passes on a labelled slice; ingest-side pre-slot once it lands in exercise 04 |
| HM-HD-OOS-03 | REG-HD-OOS-03 | ShieldGemma custom-policy recall for "specialised advice outside scope"; refusal-template trigger rate on the legitimate slice |
| HM-HD-SYS-04 | REG-HD-SYS-04 | Output-slot classifier recall on the prompt-leak adversarial category; refusal-template check that does not restate policy |
| REG-HD-ABU-05 | REG-HD-ABU-05 | Input-slot Llama Guard recall on the disallowed-content category; logging-side Presidio scrub coverage |

## 7. Open items

- **LLM02 (Sensitive Information Disclosure retired code) / LLM03 (Supply Chain) / LLM04 (Data and Model Poisoning).** Not addressed in the current slot design. LLM03 and LLM04 are substantially owned by mod-110 (supply chain and MRM); file as cross-track handoff, not as HelpDesk-local open work.
- **Scenario B second mechanism (ingest-side Presidio).** Candidate for REG-HD-PII-02 residual update; implementation is exercise 04; measurement is exercise 05.
- **Streaming bypass design fix.** The output validator's streaming behaviour is the chapter-01 silent-SPOF case 3 risk; the exercise-02 custom-middleware work must specify incremental validation or buffered validation before the streaming endpoint ships.
- **Partial-coverage cells on the matrix.** Three `partial` cells (HM-HD-INJ-01 at input filter; HM-HD-OOS-03 at input filter; HM-HD-SYS-04 at input filter) — each is a sub-scenario the slot addresses, not a load-bearing control. The chapter-05 measurement will quantify whether the partial contribution is material.
- **Escalations.** No slot is absent and no scenario has zero coverage; nothing escalates to mod-112 from this artefact. If a `partial` cell measures below treatment appetite at chapter-05 time, escalation is reconsidered then.
```

### Step 8 — Final checkpoint

Re-read the seven sections. Confirm:

- **Framework versions pinned in the header** — OWASP LLM Top 10, MITRE ATLAS, Google SAIF. Not "latest" or "current"; a named revision.
- **Five slot blocks, each with three lines** — responsibility, failure mode, mechanism candidate. Mechanism is concrete (tool + action), not abstract ("classifier").
- **Coverage table has rows for every slot and cells populated or `—`.** No "various" or blank cells.
- **Scenario-slot matrix uses `Y` / `partial` / `N`** — every cell filled; no rows entirely `N` without a composition-walk-through follow-up.
- **Composition walk-through has three scenarios A, B, C** — scenario A walks the defeat sequence; scenario B proposes a specific second mechanism; scenario C names one anti-pattern as most plausible with a reason.
- **Register handoff note names register rows and what chapter-05 must measure** — not pre-credited residual numbers.
- **Open items enumerate non-addressed OWASP codes, candidate second mechanisms, design fixes, and partial-coverage cells** — the artefact is not pretending to cover what it does not.

## Expected output

`mod108-lab01/helpdesk-coverage.md` — one populated seven-section document; one and a half to two pages. The slot specification in Section 2 and the coverage table in Section 3 are the load-bearing artefacts; the composition walk-through in Section 5 is the one an auditor will read first.

## Checkpoint questions (self-check)

1. A reviewer reads Section 3's coverage table and asks "the tool-invocation policy cell has three OWASP codes — LLM06, LLM07, LLM10 — but no classifier row. How is this defence in depth?" What is the chapter-01 answer? (*Chapter 01 §"The slot set" and chapter 02 §"When rule-based fits" both pin that tool-invocation policy is deterministic — allow-list + argument schema + quota. The defence-in-depth composition for HM-HD-INJ-01 is the retrieval filter (classifier + provenance rule) **plus** the tool-invocation policy (allow-list + schema), two slots with different mechanisms; the tool slot is not meant to be classifier-based and adding a classifier here would be the chapter-02 "classifier where rule fits" anti-pattern. The two slots addressing HM-HD-INJ-01 use different mechanisms — that is the defence-in-depth test.*)

2. A reviewer flags the HM-HD-PII-02 row in the scenario-slot matrix as "two `Y` cells, both Presidio-based" and asks whether this counts as defence in depth. How does Section 5 scenario B answer, and what does the register-handoff note (Section 6) capture as the forward-looking commitment? (*Scenario B names that two Presidio passes sharing the same recogniser engine are not strictly independent — they share a mechanism, which is exactly the chapter-01 composition rule the reviewer is invoking. The proposed second mechanism is the **ingest-side Presidio pass** at a different latency profile (offline NER vs. in-request regex) plus per-caller entitlement as a deterministic rule — three slots, three mechanisms, independent evidence trails. The register handoff note commits REG-HD-PII-02 to a chapter-05 measurement that includes the ingest-side layer once exercise 04 ships it; the residual column is not pre-credited.*)

3. A product manager reads the artefact and asks "which slot is the single most important for HelpDesk — if I had to invest in strengthening one, which?" Per chapter 01, how do you answer *responsibly*? (*Chapter 01 §"Defence in depth" is explicit that **no single slot may be the only thing standing between the attacker and the harm**. Answering "slot X is the most important" implies a ranking that defence-in-depth is designed to prevent; the responsible answer names **which slots are addressing which scenarios** (Section 4's matrix) and **which scenario would move most if a specific slot were defeated** (Section 5's composition walk-through — for HelpDesk, HM-HD-INJ-01 moves most if the tool-invocation policy is disabled, because the retrieval filter catches novel injection shapes less reliably than an allow-list enforces a destination registry). The right investment question is "which scenario has the thinnest defence-in-depth margin?" not "which slot is most important?" — the former is answerable from the artefact; the latter is a category error.*)

4. A reviewer asks why the artefact does not claim measured coverage on any cell. Per chapter 01, what is the answer, and what does the exercise 05 run contribute? (*Chapter 01 §"Slot × attack-class coverage table" is explicit that **cells are addressed, not claimed** — a cell means the slot has an implemented mechanism that measurably catches instances on the bounded adversarial set. This artefact is the **design**, not the measurement. The chapter-05 run (exercise 05) produces the TP / FP / TN / FN / cost / latency rows on the mod-104 adversarial and legitimate sets that upgrade cells from "addressed" to "measured." The register handoff note (Section 6) names what each run must measure; the open-items list (Section 7) names any cell the measurement might turn out not to credit.*)

## Why this lab matters

Exercise 01 asks you to produce six artefacts — architecture diagram, slot specification, coverage table, composition walk-through, scenario-slot matrix, register handoff note — for your real running system. If the shape is new at exercise-01 authoring time, the budget blows on inventing the format instead of on mechanism choice, harm-scenario mapping, and defeat-sequence analysis. This lab drills the shape on a fixed fabricated system so that exercise 01's time goes into the engineering judgement: which mechanism in which slot for your system, which harm-model rows have thin defence-in-depth margins, and which silent-SPOF anti-pattern is most plausible for your deployment shape. The artefact shape it drills is also the one exercise 02 wires rules and flows into, exercise 03 stacks classifiers on top of, exercise 04 extends with Presidio, exercise 05 measures the cells of, and exercise 06 scores each vendor against. Getting the shape right here cascades.
