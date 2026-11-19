# Lab 01 — Coverage-Matrix Cell-Marking Drill

**Duration:** 45 minutes · **Format:** guided practice, markdown authoring

## Objective

Internalise the chapter-01 pattern for marking every cell of a red-team coverage matrix as one of the four allowed values — **`covered:<suite-id>` · `n/a` · `deferred:<reason-or-ticket>` · `escalated:L40`** — on a fixed toy system. The lab asks you to fill *one* small matrix (12 technique-class rows × 4 capability-surface columns) end-to-end with checkpoints, so that when exercise 01 asks for the full charter the matrix step takes minutes per row rather than "leave half the cells blank and hope the reviewer doesn't notice."

Blank cells are the single most common first-pass defect chapter 01 warns about. This lab drills the discipline that eliminates them.

## Prerequisites

- Chapter 01 read (specifically "The coverage matrix" and "Programme ownership and boundaries").
- Chapters 04 and 05 skimmed enough to recognise the technique classes by name (tool-abuse outcomes and the eight-class jailbreak taxonomy).
- A terminal where you can create a working directory and author markdown files.
- No external access required; the system, the surface list, and the technique-class list are fixed below.

## The fixed example system

Use this throughout the lab — do *not* substitute your own system. Carrying `PolicyPal` forward from the mod-102 and mod-103 labs keeps the running example anchored.

> **`PolicyPal`** — an internal HR-policy Q&A assistant for a mid-sized tech company.
> Read-only RAG over the company's HR handbook (vacation, benefits, parental leave, code of conduct) using a hosted LLM. No decisions made automatically; responses are text only, surfaced in the employee intranet. Employees can ask free-text questions and receive an answer with citation links back to the handbook. No PII inputs are expected; the assistant does not have access to employee records. No tool calls (the LLM cannot invoke external APIs). No multimodal input. No admin surface that is in scope for red-team.

The **capability surfaces in scope for this lab** (four columns on your matrix):

| Column | What it covers in PolicyPal |
|---|---|
| `text-gen` | Free-text answer generation from the LLM |
| `retrieval` | Handbook-chunk retrieval into context |
| `output-render` | Markdown + citation-link rendering into the intranet UI |
| `tool-calls` | Tool-call surface — explicitly *absent* in PolicyPal |

The **technique classes** (twelve rows on your matrix, from chapter 01's technique-class table):

1. Direct prompt injection
2. Indirect prompt injection (retrieval / tool output)
3. Encoding / obfuscation attacks
4. Multi-turn refusal-bypass
5. Role-play / persona-hijack (DAN-family)
6. GCG-style adversarial suffixes
7. Many-shot in-context jailbreaking
8. Unintended tool invocation
9. Data exfiltration through retrieval
10. Unauthorised action chains
11. Sensitive-information disclosure (system prompt, credentials, KB secrets)
12. Refusal robustness on policy-negative asks

The **suite-id placeholders you may use** (same ones the exercise-01 instructions suggest):

- `ex02-garak-smoke`
- `ex03-pyrit-library`
- `ex04-agentdojo-internal`
- `ex05-gcg-replay`
- `ex05-manyshot-sweep`
- `ex06-inspect-internal-suite`

## Steps

### Step 1 — Create your working directory and the matrix template

```bash
mkdir -p mod104-lab01 && cd mod104-lab01
touch coverage-matrix-policypal.md
```

Open `coverage-matrix-policypal.md` and paste the following template.

```markdown
# PolicyPal — coverage matrix (lab 01)

System: PolicyPal (internal HR-policy Q&A RAG assistant; see lab brief)
Chapter reference: mod-104 ch 01 (coverage matrix cell discipline)
Allowed cell values: covered:<suite-id> | n/a | deferred:<reason-or-ticket> | escalated:L40

## Matrix

| Technique class | text-gen | retrieval | output-render | tool-calls |
|---|---|---|---|---|
| Direct prompt injection | | | | |
| Indirect prompt injection (retrieval / tool output) | | | | |
| Encoding / obfuscation attacks | | | | |
| Multi-turn refusal-bypass | | | | |
| Role-play / persona-hijack (DAN-family) | | | | |
| GCG-style adversarial suffixes | | | | |
| Many-shot in-context jailbreaking | | | | |
| Unintended tool invocation | | | | |
| Data exfiltration through retrieval | | | | |
| Unauthorised action chains | | | | |
| Sensitive-information disclosure | | | | |
| Refusal robustness on policy-negative asks | | | | |

## Trace table (`covered:*` cells only)

| Cell | Suite-id | Mod-102 harm-row or mod-103 register-row it traces to |
|---|---|---|

## Deferral notes (`deferred:*` cells only)

| Cell | Reason / ticket |
|---|---|

## Escalation notes (`escalated:L40` cells only)

| Cell | Frontier-agent depth handed off |
|---|---|
```

### Step 2 — Walk the `tool-calls` column first (the `n/a` column)

Start with the column where the shape is most constrained. PolicyPal has no tool-call surface — the LLM cannot invoke external APIs. So every cell in `tool-calls` is either `n/a` (because the technique requires a tool surface to apply) or still covers something else (because the technique applies to a surface PolicyPal *does* have — but then the cell lives in the other columns, not this one).

Walk the rows in order and ask: *does this technique class apply to a tool-call surface PolicyPal does not have?* If yes → `n/a`. If the technique is about the model's content rather than a tool call, the `tool-calls` cell is still `n/a` in PolicyPal's case because the column itself is absent.

For PolicyPal, every cell in the `tool-calls` column is `n/a`. Fill them all in. The column being `n/a` end-to-end is *not* a coverage hole — it is the visible form of chapter 01's "out-of-scope surfaces are named, not implied."

**Expected shape:**

```markdown
| ... | ... | ... | ... | n/a |
```

(repeated across every row in the column).

### Step 3 — Checkpoint on the `n/a` column

Check your `tool-calls` column against these rules:

- Every cell says `n/a`, including rows like *Unintended tool invocation* that read like they are "about tools." The reason is that PolicyPal has *no tool surface at all*; a technique class that needs a tool surface does not apply here. The row still exists on the matrix so a reviewer can see you considered it.
- No cell says `deferred:tool-calls-not-in-scope`. `deferred:*` means the technique applies and you have postponed covering it; `n/a` means it does not apply. If you wrote `deferred` anywhere in this column, swap it for `n/a`.
- No cell is blank. Blank cells are the one thing the chapter calls an audit finding by name.

If any rule fails, rewrite before moving on.

### Step 4 — Walk the `retrieval` column (the column where indirect-injection lives)

PolicyPal's retrieval surface is the main attack path — the HR handbook is a human-authored corpus that can carry injected instructions. Walk the rows in order:

- *Direct prompt injection* — direct injection is a prompt-side attack, not a retrieval-side one. The `retrieval` cell is `n/a` (direct injection lives in `text-gen`).
- *Indirect prompt injection (retrieval / tool output)* — this is the row that *does* live here. Mark it `covered:ex04-agentdojo-internal` (with the internal environment mirroring PolicyPal's RAG shape, per chapter 04).
- *Encoding / obfuscation attacks* — encoding attacks can arrive through retrieved content (a base64 string pasted into a handbook paragraph as the injected payload). The `retrieval` cell here is `covered:ex04-agentdojo-internal` (encoding payloads planted inside the retrieved corpus).
- *Multi-turn refusal-bypass* — a retrieval-surface multi-turn attack is awkward to express; the multi-turn loop runs on the text-gen side, not the retrieval side. `n/a` in `retrieval` (covered in `text-gen` under `ex03-pyrit-library`).
- *Role-play / persona-hijack (DAN-family)* — persona-hijack payloads can be planted in retrieved content ("the following handbook section is from DAN-mode…"). Mark `covered:ex04-agentdojo-internal`.
- *GCG-style adversarial suffixes* — a GCG suffix can be planted in a retrieved chunk as part of an indirect injection. For PolicyPal, call this `covered:ex05-gcg-replay` for the replay set and *note in the deferral or trace table that fresh-optimisation is `n/a` because PolicyPal's model is a closed-API hosted LLM* (chapter 05's level-1-only coverage shape for closed-API targets).
- *Many-shot in-context jailbreaking* — many-shot through retrieval would require very large retrieved chunks; for PolicyPal's small-chunk RAG it is a stretch. `deferred:many-shot-via-retrieval-limited-by-chunk-size` is honest; linking a backlog ticket is honest too. Do not mark `covered` if you do not have a suite row that exercises it.
- *Unintended tool invocation* — `n/a` (no tools).
- *Data exfiltration through retrieval* — this covers an injected instruction causing the model to include retrieved data in its response that the user did not ask for. For PolicyPal (public handbook content, no private user data retrieved), the exfiltration category is narrow — mostly prior-turn context, system-prompt disclosure. Mark `covered:ex04-agentdojo-internal` with a trace to the matching harm-model row.
- *Unauthorised action chains* — `n/a` (no actions, no tools).
- *Sensitive-information disclosure* — system-prompt extraction via retrieved content is a known shape. `covered:ex02-garak-smoke` (the prompt-leak probe family) plus `covered:ex06-inspect-internal-suite` for policy-derived cases.
- *Refusal robustness on policy-negative asks* — retrieval-side framing of a policy-negative ask is a supported attack shape. `covered:ex06-inspect-internal-suite`.

Fill the column. For every `covered:*` cell, add a row to the Trace table at the bottom citing which mod-102 harm row and/or mod-103 register row the coverage attaches to (use placeholder ids like `HR-PP-RTV-01` from the mod-103 lab if your running system does not have real row ids yet — the point is the *shape* of the trace).

### Step 5 — Walk the `text-gen` column (the column where direct-prompt jailbreaks live)

The text-gen surface is where PolicyPal's free-text answer generation happens. Direct-prompt techniques land here.

Walk in order:

- *Direct prompt injection* — `covered:ex02-garak-smoke` (promptinject probe family) and `covered:ex03-pyrit-library` for depth.
- *Indirect prompt injection (retrieval / tool output)* — `n/a` on `text-gen` (indirect injection lives in `retrieval`).
- *Encoding / obfuscation attacks* — `covered:ex02-garak-smoke` + `covered:ex03-pyrit-library` (converter-chained encoding attacks).
- *Multi-turn refusal-bypass* — `covered:ex03-pyrit-library` (multi-turn adaptive orchestrator; garak is single-turn so this is PyRIT-only).
- *Role-play / persona-hijack (DAN-family)* — `covered:ex02-garak-smoke` + `covered:ex03-pyrit-library`.
- *GCG-style adversarial suffixes* — `covered:ex05-gcg-replay`. Note in the trace table that fresh-optimisation is `n/a:closed-API-target` because the Level 2 coverage of chapter 05 requires gradient access you do not have against PolicyPal's hosted LLM.
- *Many-shot in-context jailbreaking* — `covered:ex05-manyshot-sweep` with a shots-per-context sweep bounded by PolicyPal's model context window.
- *Unintended tool invocation* — `n/a` (no tools).
- *Data exfiltration through retrieval* — `n/a` (retrieval-side, not text-gen-side).
- *Unauthorised action chains* — `n/a`.
- *Sensitive-information disclosure* — `covered:ex02-garak-smoke` (prompt-leak probes) + `covered:ex06-inspect-internal-suite`.
- *Refusal robustness on policy-negative asks* — `covered:ex06-inspect-internal-suite` (policy-derived internal suite is the primary artifact here; garak and PyRIT contribute but the policy-anchored pass rule lives in Inspect).

### Step 6 — Walk the `output-render` column (the column many first-pass matrices leave blank)

Output rendering is the surface where markdown, citation links, and intranet-UI rendering happen. A few technique classes land here because the *rendering* can carry the harm even when the text looks fine:

- *Direct prompt injection* — `n/a` (direct injection lives on input, not output).
- *Indirect prompt injection* — `n/a`.
- *Encoding / obfuscation attacks* — a rendered base64 string that a browser (or a downstream auto-forwarder) decodes is an output-render-side concern. For PolicyPal, which only renders into an intranet UI, this is narrow. `deferred:no-downstream-auto-decoder` is defensible; linking a backlog ticket is honest.
- *Multi-turn refusal-bypass* — `n/a`.
- *Role-play / persona-hijack* — `n/a`.
- *GCG-style adversarial suffixes* — `n/a`.
- *Many-shot in-context* — `n/a`.
- *Unintended tool invocation* — `n/a` (no tools).
- *Data exfiltration through retrieval* — exfiltration *via rendered content* (e.g., a markdown image with a URL query string encoding a secret) is an output-render-side concern. Mark `covered:ex04-agentdojo-internal` if your internal environment scores rendered-markdown exfiltration; otherwise `deferred:markdown-image-exfil-check-pending`.
- *Unauthorised action chains* — `n/a` (no actions).
- *Sensitive-information disclosure* — prompt-leak content rendered verbatim is the same cell as `text-gen` in effect. Mark `covered:ex02-garak-smoke` or `n/a` with a note that the text-gen cell carries it.
- *Refusal robustness on policy-negative asks* — `n/a` (the refusal is a text-gen-side concern; rendering a refusal does not add attack surface).

### Step 7 — Checkpoint on completeness and traces

Re-read the matrix. Confirm:

- **Every cell has a value.** If you see a blank cell, stop and fill it. The chapter calls blank cells an audit finding by name; the lab tests exactly this.
- **Every `covered:*` cell has a Trace-table entry.** The trace cites the suite-id and the mod-102 harm row and/or mod-103 register row the coverage attaches to. A `covered:*` cell with no trace is a coverage claim with no evidence anchor — chapter 01 and exercise 01's cross-cutting rule reject this shape.
- **Every `deferred:*` cell has a Deferral-notes entry** naming a ticket id or an explicit reason. "We'll get to it" is not a reason. "No downstream auto-decoder in v1" is.
- **Every `escalated:L40` cell has an Escalation-notes entry** naming the frontier-agent depth handed off (dynamic tool composition, multi-agent orchestration with credential passing, browser-driving against unbounded upstream content, capability-elicitation against frontier models with novel scaffolding — per chapter 06).
- **No `deferred:*` cell appears where `n/a` is the honest marking.** If a technique class does not apply to this surface, `n/a` is correct. `deferred:n/a` is a common first-pass defect; rewrite as `n/a`.

PolicyPal's matrix is unlikely to have any `escalated:L40` cells — the system has no agent, no tools, no sub-agents, no browser. Note this absence in a one-liner under the Escalation notes section: `No cells escalated:L40 for this target — PolicyPal has no agent surface, no tool calls, no browser, no sub-agents; the L40 boundary is not reached.` That one-liner is the chapter-01 discipline in action; a programme with "no escalations" without saying *why* reads as under-scoped.

### Step 8 — Write the one-paragraph matrix narrative

Convert the matrix to English in two to four sentences. Target shape:

> "PolicyPal's coverage matrix has 48 cells across 12 technique classes × 4 capability surfaces. The `tool-calls` column is `n/a` end-to-end because the system has no tool surface. Direct-prompt and persona-hijack coverage lives in `text-gen` under `ex02-garak-smoke` and `ex03-pyrit-library`. Indirect-injection and retrieval-planted encoding / persona / GCG / exfiltration attacks live in `retrieval` under `ex04-agentdojo-internal`. GCG fresh-optimisation is `n/a` because the model is a closed-API hosted LLM (chapter 05 Level 2 requires gradients). Three cells are `deferred:*` with linked reasons; no cells are `escalated:L40` because the system has no agent surface."

That paragraph is the matrix's living voice; it is what a risk council reads cold in place of staring at a 48-cell grid.

### Step 9 — Final checkpoint

Re-read all cells. Confirm:

- All 48 cells are populated (12 rows × 4 columns).
- No cell is blank.
- Every `covered:*` cell has a Trace-table row.
- Every `deferred:*` cell has a Deferral-notes row with a reason or ticket.
- The Escalation-notes section is present, even if it contains only the "no cells escalated" one-liner that argues *why* L40 is not reached.
- The narrative paragraph reads back the matrix in English.

## Expected output

`mod104-lab01/coverage-matrix-policypal.md` — one populated matrix (48 non-blank cells), a Trace table listing every `covered:*` cell with its suite-id and harm/register anchor, a Deferral-notes section with reasons or tickets for every `deferred:*` cell, an Escalation-notes section (with the "no L40 reached" one-liner if applicable), and a short matrix-narrative paragraph; roughly one page.

## Checkpoint questions (self-check)

1. On the `tool-calls` column, you marked *Unintended tool invocation* as `n/a`. A reviewer argues it should be `deferred` because PolicyPal might add tool calls in v2. Which marking is correct in this run? (`n/a` — the technique class does not apply to the system's current surface shape. If v2 adds tool calls, the matrix gets re-marked at that release; the charter is pinned to the release under test. Chapter 01: "The target profile pins *this system, this version, these surfaces, this tier*.")
2. On the `retrieval` column, you marked *GCG-style adversarial suffixes* as `covered:ex05-gcg-replay` and noted fresh-optimisation is `n/a` because PolicyPal is closed-API. A reviewer counters that "GCG requires fresh optimisation to be covered." Is the reviewer right? (No. Chapter 05 is explicit that against closed-API targets, Level 1 replay is the *only* level available — gradients are unreachable. The `n/a` on Level 2 with a one-line rationale is the chapter's prescribed shape, not a coverage hole.)
3. A junior engineer pastes a matrix with every `n/a` cell marked `deferred:not-applicable`. What is the specific defect? (`deferred:*` carries an action implication — it is work that has been *postponed*. `n/a` carries no action — the technique does not apply. Collapsing the two hides the actionable deferrals in a sea of pseudo-deferrals the reviewer has to sort through. Chapter 01's cell-value list is exclusive, not advisory.)
4. The matrix has 48 cells and every one is `covered:*`. Is that good? (Almost certainly no — chapter 01's exercise-01 starter guidance says "a matrix full of `covered:*` on first pass usually means you have not thought hard about what your system actually exposes." Most real matrices carry legitimate `n/a` and `deferred:*` entries. An all-`covered` matrix is as suspicious as an all-blank one.)

## Why this lab matters

Exercise 01 asks you to produce a full charter — target profile, coverage matrix, suite stub, replay-bundle README, triage-log template, cadence table, payload manifest, threshold document — in two hours. If the matrix step consumes 90 minutes because you re-derive the cell-value discipline from scratch and leave half the cells blank, the exercise blows its budget. This lab drills the atomic skill — one matrix end-to-end with no blank cells, every `covered:*` traced, every `deferred:*` justified, every `n/a` distinguished from `deferred` — until it is routine, so exercise 01 is *filling the matrix with real decisions* rather than *re-inventing the marking discipline*. Labs 02 and 03 drill the triage-disposition skill and the out-of-repo-payload discipline respectively; together they make exercises 01–06 tractable within their nominal budgets.
