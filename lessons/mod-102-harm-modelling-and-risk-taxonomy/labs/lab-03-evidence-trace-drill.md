# Lab 03 — Evidence-Trace Drill (AIID + OECD.AI + MIT AIR + Literature)

**Duration:** 45 minutes · **Format:** guided practice, markdown authoring with light browsing

## Objective

Walk the chapter-02 evidencing workflow end-to-end on *one* fixed harm-model row — producing the full `evidence_refs` cell plus a `confidence` tag plus a *watch* trigger if applicable. The lab trains the per-row evidencing step that exercise 02 will ask you to run against an entire persona × capability-surface matrix. You will:

- search AIID,
- search OECD.AI AI Incidents Monitor,
- open MIT AI Risk Repository and pick the right domain × sub-domain cell,
- add literature citations where relevant,
- mark analogy strength honestly, and
- decide the row's `confidence` with the citations it actually carries.

## Prerequisites

- Chapter 02 read. Lab 01 is a useful warm-up because it makes the scenario shape concrete.
- A terminal where you can create a working directory and author markdown files.
- Open these three tabs in a browser:
  - AI Incident Database — <https://incidentdatabase.ai/>
  - OECD.AI AI Incidents Monitor — <https://oecd.ai/en/incidents>
  - MIT AI Risk Repository — <https://airisk.mit.edu/>
- Optional reference tabs: OWASP Top 10 for LLM Applications; MITRE ATLAS; AVID.

Expect to open roughly 6–10 incident / repository pages during the lab. Keep a scratch list of the IDs and URLs you consult, including the ones you *rejected* — rejected candidates are useful evidence that you ran the search, not shortcuts.

## The fixed scenario

Use this exact row — do *not* substitute your own. The scenario is lifted from lab 01's `HR-PP-RTV-01` and carried forward so the evidencing step can be checked against a known shape.

> **`HR-PP-RTV-01` — indirect prompt injection via HR handbook content, PolicyPal.**
> Persona: upstream data supplier — an HR editor pastes handbook content from a public help thread that carries a hidden instruction; the paragraph is indexed into PolicyPal's RAG store.
> Capability surface: retrieval → context assembly.
> Immediate harm: in a legitimate employee's later session, the retrieved chunk causes PolicyPal to execute the attacker's instruction, producing an answer outside the policy envelope (e.g., leaking prior-turn context, or complying with attacker intent in place of the user's).
> Secondary harm: employee trust drops; HR content owners are implicated in a security event they did not author; intranet team pulls the affected handbook section.
> Cascading / systemic: as in lab 01.

## Steps

### Step 1 — Create the evidence-trace file

```bash
mkdir -p mod102-lab03 && cd mod102-lab03
touch evidence-trace-HR-PP-RTV-01.md
```

Paste this skeleton:

```markdown
# HR-PP-RTV-01 — evidence trace (lab 03)

System: PolicyPal (internal HR-policy Q&A RAG assistant; see lab-01 brief)
Chapter reference: mod-102 ch 02 (incident corpora as evidence)
Evidence collected against AIID / OECD.AI / MIT AIR as of: <YYYY-MM-DD>

## AIID citations
- <AIID Incident N — one-line summary — analogy strength: direct / near-analog / weak>
- <AIID Incident N — ... >

## OECD.AI AI Incidents Monitor citations
- <record URL — one-line summary — tagged principle(s) — analogy strength>
- <record URL — ... >

## MIT AI Risk Repository citations
- Release version: <v>
- Domain / sub-domain: <>
- Rows cited:
  - <row identifier — one-line summary — analogy strength>

## Literature / adjacent citations
- Greshake et al., indirect prompt injection (paper + arXiv ID)
- OWASP LLM Top 10 — LLM01 (Prompt Injection), LLM06 (Sensitive Information Disclosure)
- MITRE ATLAS — specific technique IDs if applicable
- AVID — specific vulnerability IDs if applicable
- <other papers / preprints>

## Rejected candidates (searched, not cited)
- <ID / URL — one-line reason for rejection>

## Confidence decision
- Value: reported-incident / near-analog / literature-only
- Reasoning: <one-to-two sentences>

## Watch trigger (if literature-only)
- Trigger: <what condition upgrades the row to reported-incident>
- Watched by: <role>
- Review cadence: <per chapter 02 cadence section>
```

Fill the date at the top before you start searching. That marker is required; chapter 02 is explicit: "a dated cadence marker at the top so future re-checks can diff against a known corpus state."

### Step 2 — Search AIID

- Open <https://incidentdatabase.ai/> and use the discovery app. Try these search phrases in order, noting which return hits that are even remotely in scope:
  - `prompt injection`
  - `indirect prompt injection`
  - `RAG` / `retrieval augmented`
  - `chatbot` combined with `support` or `assistant` or `HR`
  - `LLM data exfiltration`
- For each incident that returns, read the one-paragraph narrative and at least one linked report. Decide:
  - *Direct match* — same mechanism (indirect prompt injection via retrieved / ingested content) on a similar deployment class (internal assistant, support chatbot, RAG).
  - *Near-analog* — similar system, similar mechanism, different target population or capability surface. Chapter 02: "Auditors are fine with disciplined analogy; they object to unmarked analogy dressed up as evidence."
  - *Weak / reject* — mechanism or deployment class does not transfer.
- Cite **all** direct and near-analog matches by their stable `Incident N` identifier, each with a one-line summary and an analogy-strength tag. List at least one rejected candidate — this is the evidence that you ran the search, not shortcut it.

If AIID returns zero direct matches (plausible depending on the corpus state when you read this), cite the strongest near-analogs and move on; the `confidence` decision in Step 7 will absorb the lack of direct matches.

### Step 3 — Search OECD.AI AI Incidents Monitor

- Open <https://oecd.ai/en/incidents>.
- Filter by:
  - OECD AI Principle — "Robustness, security and safety" (the principle most likely tagged on prompt-injection incidents).
  - System type / deployment — conversational / assistant where the filter supports it.
- As with AIID, note direct matches and near-analogs with analogy strength.
- Cite each record by its URL (the records do not have standalone IDs as stable as AIID's incident numbers — URL is what you have).
- If the AIM has no matching records, write "no matching AIM record after search filters applied on <date>" and keep going. Chapter 02 is explicit that an empty category must be marked, not silently dropped.

### Step 4 — Open MIT AI Risk Repository

- Open <https://airisk.mit.edu/>. Note the release version at the top of your file.
- Navigate the domain taxonomy to the top-level domain most likely to match. For this scenario, candidates include *Privacy & Security* and *AI System Safety & Failures*. The sub-domain cells containing prompt-injection and retrieval-pipeline risks should both be opened.
- Download the current release's spreadsheet if you have not already. The spreadsheet carries row identifiers and source-framework citations per row — those are what you cite.
- Cite at least one domain row identifier that describes the scenario. If you find rows that describe adjacent but distinct scenarios (e.g., prompt injection broadly, or sensitive-information-disclosure broadly), cite them *and* mark their relationship to your row.
- If the specific sub-domain cell has zero matching rows, do not reach for a vague parent row — write "no MIT AIR row matches at the sub-domain cell level on <release>; closest adjacent rows are <list>" and move on.

<!-- needs-research: verify current MIT AIR release version and the exact domain / sub-domain labels at authoring time; counts and labels change between releases. -->

### Step 5 — Add literature and adjacent-corpus citations

- Cite the canonical indirect-prompt-injection paper (Greshake et al.). Use the published title + arXiv identifier. Do not invent a title you do not have.
- Cite **OWASP LLM Top 10**: LLM01 (Prompt Injection) and, if your scenario's immediate harm includes leaking prior-turn context, LLM06 (Sensitive Information Disclosure).
- Cite **MITRE ATLAS** techniques if any apply (poisoning / prompt-injection related). If no technique cleanly applies, write "no ATLAS technique cleanly applies" and move on.
- Cite **AVID** if you find a matching vulnerability record. If AVID is not reachable at authoring time (<!-- needs-research: confirm AVID availability -->), note that and move on.

For each citation, add a one-line note on *why it supports the row* rather than pasting a long quotation. Chapter 02 is explicit: do not paste incident narratives verbatim behind paywalls or containing personal data.

### Step 6 — Reverse-sweep (chapter 02's audit-survival step)

While you are in the AIID / MIT AIR tabs, skim *adjacent* rows / incidents that would apply to PolicyPal but are not represented by `HR-PP-RTV-01`. Candidates:

- Jailbreak of a customer-support assistant (an adversarial-user harm; would live in a different lab-01 row).
- Downstream integrator stripping an advisory caveat (an integration harm; a different lab-01 row).
- Memorisation / sensitive-information exposure via extraction prompts.

For each, record a one-liner under a *coverage-gap candidates* section at the bottom of the file:

```markdown
## Coverage-gap candidates (surfaced during the reverse sweep)
- Proposed row-ID: HR-PP-INS-02 — one-line scenario description — citation that surfaced it (AIID / MIT AIR row).
- ...
```

Do not silently promote these to real rows; chapter 02 says they *wait for review* (exercise 02 is where they graduate).

### Step 7 — Decide `confidence` and set a watch trigger if needed

Apply the chapter-02 rules:

- `reported-incident` — at least one direct AIID / OECD.AI match.
- `near-analog` — matches exist but differ in system class, mechanism specifics, or affected population.
- `literature-only` — only MIT AIR / academic / OWASP / ATLAS / AVID citations.

Pick the value your citations actually support. Resist upgrading to `reported-incident` because you *want* to move on; chapter 02: "A watched literature-only row is defensible; a dressed-up literature-only row is not."

If the row ends up `literature-only`, set a *watch trigger*. The trigger block is:

```markdown
## Watch trigger
- Trigger: a new AIID or OECD.AI incident lands that describes indirect prompt injection via a retrieval corpus on an internal / customer-facing assistant.
- Watched by: risk engineering (you); monthly review cadence per chapter 02.
- Upgrade behaviour: when a matching incident lands, the row is upgraded from literature-only to reported-incident (or near-analog, depending on the match), and the harm-model changelog records the upgrade.
```

If the row is `near-analog`, consider whether a watch trigger is still useful (a matching direct incident would upgrade the row to `reported-incident`); it often is.

### Step 8 — Final checkpoint

Verify:

- The dated corpus marker is present at the top of the file.
- AIID, OECD.AI, and MIT AIR each have at least one entry — a cited match *or* an explicit "none found on <date>" line.
- Every citation has a one-line summary and an analogy-strength tag.
- The rejected-candidates section has at least one entry — proof that you ran the search rather than cherry-picked.
- The literature section cites Greshake et al. (paper + arXiv) and OWASP LLM Top 10 by its standard category identifier.
- The reverse-sweep section lists at least one coverage-gap candidate with a proposed row-ID.
- `confidence` is set to a value the citations support, with one-to-two sentences of reasoning.
- If `literature-only` or `near-analog`, a watch trigger block is present.

## Expected output

`mod102-lab03/evidence-trace-HR-PP-RTV-01.md` — a single markdown file, roughly one page, with:

- a dated cadence marker,
- AIID / OECD.AI / MIT AIR citations with analogy strength,
- a literature section,
- a rejected-candidates list,
- a coverage-gap-candidates list from the reverse sweep,
- a `confidence` decision with reasoning,
- a watch-trigger block if applicable.

## Checkpoint questions (self-check)

1. If your AIID search returned zero direct matches and only two weak near-analogs, is it legitimate to leave the row's `confidence` as `reported-incident` because "prompt injection is well-known"? (No. Chapter 02 ties the confidence value to the *citations the row actually carries*. Being well-known is not evidence; cited incidents are. Mark `near-analog` or `literature-only` depending on the strength of the OECD.AI and MIT AIR citations.)
2. The MIT AIR domain row you cited describes *prompt injection broadly* but not *indirect prompt injection via RAG*. What should the row-cell note reflect? (That the MIT AIR citation covers the general class; your row is a more specific sub-class. The literature-strength tag should acknowledge the specificity gap. This is exactly the "disciplined analogy" chapter 02 requires.)
3. The reverse-sweep turned up two new candidate rows — downstream-integrator caveat stripping and memorisation / extraction. Why do you *not* add them to the harm model directly in this lab? (Because chapter 02 reserves that promotion for exercise 02's deliberate review. Coverage-gap candidates sit in a separate section until a reviewer agrees they are in scope; silent additions are audit findings.)
4. In exercise 04 this row will be tagged with NIST AI 600-1 risks. Which two risk numbers from mod-101 chapter 02's list would you expect to tag? (Risk 9 — Information Security as primary; Risk 7 — Human-AI Configuration as secondary, because the retrieval-content trust boundary is an oversight-design choice. The chapter-03 worked example on `HR-CS-INJ-01` says exactly this.)

## Why this lab matters

Evidence collection is the step that most often decays in a shipping harm-modelling program — because it feels like "extra research" rather than core engineering. The chapter-02 workflow is deliberately short, but it has to run *on every row*. Training the workflow on one row with a known-good shape is the way to get it to feel like 30 minutes of routine instead of a day of ad-hoc web search. Exercise 02 then multiplies this drill across your running system without re-learning the pattern each time.
