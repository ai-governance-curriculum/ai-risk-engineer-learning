# Lab 03 — Ladder Triage Drill

**Duration:** 40 minutes · **Format:** guided practice, markdown authoring

## Objective

Given a short inbox of ten incoming requests, classify each per the chapter-06 deferral contract: **owned** (you, L25), **consumed** from below (L15 analyst / L20 ML engineer), **handed off sideways** (peer eval / MLOps / security at L25-L35), **escalated upward** (L35 release-assurance / L40 agentic-safety / L50 architect / L60 head), or **out of scope** (legal, SOC). You will name the correct receiver per request and — the point of the drill — identify which requests you should be *refusing* even if nobody else is picking them up.

## Prerequisites

- Chapter 06 read.
- Chapter 05 read (because two of the ten items are SR 11-7 flavoured).
- A terminal for creating a markdown file.
- No external access required — all content comes from the chapters.

## The fixed inbox

Below is your Monday-morning inbox of ten requests. Assume they all arrived in the same week. The organisation is a mid-sized enterprise with analysts at L15, ML engineers at L20, an MLOps peer at L25, an application-layer eval engineer at L30, and a governance architect at L50. **It does not currently staff a level-35 role.** This staffing gap is important — it is the shape in which chapter 06's anti-pattern "substituting for release-assurance" typically surfaces.

1. "Please add the newly shipped internal RAG summariser to the AI system inventory in our GRC platform. Five rows, ten minutes."
2. "We need you to be the independent validator and sign the release of the credit-decisioning model shipping next Tuesday."
3. "Legal asks: does the EU AI Act Article 10 paragraph-5 safeguard apply to our use of protected attributes for bias evaluation? Can you tell them yes or no?"
4. "The red-team suite found two confirmed indirect prompt-injection vectors. We need the regression-fixtures authored and the guardrail patched before the next release."
5. "Our SOC is investigating a credential-theft incident on the vector DB's ops VM. They want an incident commander — can you take it?"
6. "The frontier-model research group at a sister org wants us to design the dangerous-capability evaluation methodology for their next pre-deployment gate."
7. "The groundedness judge implementation in our eval harness is throwing on JSON-mode outputs. Can you fix the judge code before Friday?"
8. "We need to update the org-wide control library to reflect a new ISO/IEC 42001 Annex A control interpretation the auditor flagged last cycle."
9. "The MLOps team is rolling out a new canary-deployment hook. They asked for the input / threshold contract for the risk-gated release step."
10. "A model-provider vendor wants a Statement of Applicability showing which Annex A controls apply so they can map their artefacts in."

## Classification taxonomy (chapter 06, reproduced for reference)

- **Owned (L25)** — risk-engineering craft at team scope: harm model, risk register, red-team programs, adversarial-ML, fairness / bias, privacy, guardrails, monitoring wiring, supply chain, incident RCA, risk program at team scope.
- **Consumed from L15 analyst** — intake, inventory, jurisdictional tracking, first-draft impact-assessment authoring, framework-crosswalk legwork.
- **Consumed from L20 ML engineer** — classical ML toolchain, independent train/test discipline.
- **Handed off sideways to L30 `ai-eval-engineer`** — app-layer eval plumbing, traces, judges, RAG eval, eval-gated CI/CD; *you consume the metric, you do not own the judge code*.
- **Handed off sideways to L30 `model-evaluation-engineer`** — benchmark methodology, statistical methodology, judge-vs-human calibration.
- **Handed off sideways to L25 `ai-infra-mlops`** — registry, deployment gates, canary infrastructure, feature-store integration (you specify thresholds; they implement the pipeline).
- **Handed off sideways to L35 `security-learning` / `ai-infra-security`** — MLSecOps at platform scale, judge supply-chain, eval-set exfiltration prevention.
- **Escalated to L35 `ai-evaluation-engineer`** — release-assurance sign-off, regulator-facing audit-trail methodology, evidence-packaging for external audit.
- **Escalated to L40 `agentic-safety-engineer`** — frontier-agent red-team methodology, dangerous-capability evaluation at frontier scale.
- **Escalated to L50 `senior-ai-governance-architect`** — control-library architecture, policy taxonomy, cross-jurisdiction reconciliation, SoA authorship.
- **Escalated to L60 `head-of-ai-governance`** — board reporting, regulator engagement, staffing / budget.
- **Out of scope** — legal opinion (to counsel) and SOC-side DFIR (to security operations).

## Steps

### Step 1 — Create the triage file

```bash
mkdir -p mod101-lab03 && cd mod101-lab03
touch inbox-triage.md
```

Paste this skeleton:

```markdown
# Monday inbox — ladder triage

For each request: classification, receiver, trigger / reason, what I do for the requester today.

| # | Request (one line) | Classification | Receiver (role, level) | Trigger / reason | What I do for the requester today |
|---|---|---|---|---|---|
| 1 | ... | ... | ... | ... | ... |
| ... | ... | ... | ... | ... | ... |
```

### Step 2 — Triage each item

Work through items 1-10 in order. For each, decide:

- **Classification** — one of the six buckets (owned / consumed / handed-off / escalated / out-of-scope).
- **Receiver** — the specific role on the ladder. Not "the governance team." Name a level.
- **Trigger / reason** — a one-sentence chapter-06 citation ("intake is level-15 work per the deferral contract").
- **What I do today** — the single-action response you send to the requester. Options: *accept and schedule*, *route to <receiver> and cc me*, *decline and escalate to <receiver>*, *refuse and name the role gap*, *send to legal / SOC*.

Resist two temptations:

- **Scope creep** — "the L35 role is vacant, so I'll sign the release just this once." Chapter 06's anti-pattern catalogue names this as "substituting for release-assurance." The correct behaviour is to refuse and name the staffing gap.
- **Over-escalating** — not every hard thing escalates; MEASURE and MANAGE work is *your* scope. If you escalate harm-model authoring upward because "it feels architectural," you are hollowing your own role.

### Step 3 — Suggested triage (compare after you have written yours)

Fill your own first. Then compare to the suggested answers below — they are *suggested*, not canonical; the chapter-06 citation is what matters, not matching the receiver exactly.

1. *"Add five inventory rows to the GRC platform."* → **Consumed** from L15 analyst. Anti-pattern catalogue: "Doing analyst legwork. If you are typing inventory rows into a GRC platform, escalate the ask." *Action:* route to L15 analyst; if none staffed, file as a role-boundary risk and send the request to whichever role is covering intake, cc your lead.
2. *"Be independent validator and sign the credit-decisioning release."* → **Escalated** to L35 `ai-evaluation-engineer`. Chapter 06: "Release-assurance sign-off" is L35. Chapter 05: SR 11-7 independence requirement — the credit-decisioning model is almost certainly MRM-scope, and validation independence from development is a *structural* rule. *Action:* refuse; name the L35 staffing gap in writing, escalate to whoever owns governance staffing (L50 / L60).
3. *"Legal asks: does Article 10(5) apply — yes or no?"* → **Out of scope** (legal opinion). Chapter 06: "Any sentence starting 'we can/can't do X because $LAW says…' is a legal-counsel sentence. Bring the facts, not the ruling." *Action:* send the *technical facts* (we process attributes A, B, C for bias evaluation at step X, access control D is in place) to legal; let counsel issue the ruling.
4. *"Author regression fixtures and patch guardrail for the two confirmed injection vectors."* → **Owned** (L25). This is mod-104 / mod-108 / mod-111 work by chapter and the role's twelve-module scope. *Action:* accept; schedule into the release.
5. *"Take incident command for the SOC credential-theft investigation."* → **Out of scope** (SOC-side DFIR). Chapter 06: "SOC-side incident handling (network compromise, credential theft, endpoint response) is not yours." *Action:* decline incident-command role; offer AI-specific signal if the IOCs touch the vector DB's model-serving surface; let SecOps run the SOC.
6. *"Design dangerous-capability eval methodology for a sister org's frontier model."* → **Escalated** to L40 `agentic-safety-engineer`. Chapter 06: "Frontier-agent red-team methodology, dangerous-capability evaluations … Frontier depth escalates." *Action:* decline the methodology-authoring ask; connect the sister org with the L40 role; you may contribute enterprise-shape input if asked.
7. *"Fix the groundedness-judge code before Friday."* → **Handed off sideways** to L30 `ai-eval-engineer`. Chapter 06: judge implementations are app-layer peer-eval work; you *consume the metric*, you do not maintain the judge. *Action:* file the bug to the peer; if urgent and the peer is unavailable, escalate the staffing gap.
8. *"Update the org-wide control library for a new ISO Annex A interpretation."* → **Escalated** to L50 `senior-ai-governance-architect`. Chapter 03: SoA and control-library authorship are governance-architect scope. Chapter 06: "You contribute in mod-112; you do not design the org-wide control library." *Action:* escalate; offer the implementation-evidence impact for the systems you own.
9. *"Give MLOps the input / threshold contract for the risk-gated release step."* → **Owned deliverable + handed off for implementation**. Chapter 06: "Peer / `ai-infra-mlops` — you author the *risk-gated release contract*; they implement the pipeline." *Action:* accept — specify the contract (inputs, thresholds, failure behaviour); MLOps implements.
10. *"Produce a Statement of Applicability for a vendor so they can map their artefacts in."* → **Escalated** to L50 `senior-ai-governance-architect`. Chapter 03 and chapter 06 both: SoA authorship is explicitly *not* a level-25 deliverable. *Action:* refuse the authorship ask; route to L50; offer to supply implementation-evidence pointers for any already-applicable controls once the SoA exists.

### Step 4 — Add a role-boundary-risks footnote

At the end of the file, write a *five-line* role-boundary risks section drawn from how the triage went. Required content:

- The organisation does not staff an `ai-evaluation-engineer` (L35). Items 2, 8 (partially), and 10 surfaced this gap. Release-assurance is silently landing on someone — name who, or name the gap explicitly if nobody.
- At least one *fallback* per upstream consumer — what do you do if the L15 analyst in item 1 is unavailable? Chapter 06 says: route the ask upward, do not silently absorb.
- One line on how often this contract should be reviewed. Chapter 06 doesn't prescribe a cadence; the exercise-05 stretch goal suggests quarterly — use that.

### Step 5 — Final checkpoint

Verify:

- Every row has a classification, a *level-numbered* receiver, and a chapter-06 (or chapter 03 / 05) citation for the reason.
- No row says "I'll do it just this once." Scope creep is explicitly out.
- The out-of-scope rows (items 3 and 5) route to the actual receivers (legal counsel, security operations), with the correct *shape of response* (facts not rulings for legal; AI-specific signal only for SecOps).
- The footnote names at least one role-boundary risk and one fallback.

## Expected output

`mod101-lab03/inbox-triage.md` — the ten-row table plus the footnote, approximately one page.

## Checkpoint questions (self-check)

1. On item 2, if the organisation has no L35 role and the release is Tuesday, what does *good* look like? (Refusing the sign-off ask in writing, escalating the staffing gap, and offering to supply your signed evidence packet so the eventual L35 — or the AI risk council acting in that capacity on a one-time, documented exception basis — has what they need to sign. The *gap is visible*; it is not absorbed.)
2. On item 5, what signal *would* be in scope for you to hand to the SOC? (A jailbreak that successfully exfiltrated system-prompt content, a memorised training-set exposure, a poisoned model artefact at registry ingestion — mod-110 / mod-111 work. Those are *AI-specific* signals the SOC does not have primary visibility into.)
3. On item 9, what are the three required fields of the contract you hand to MLOps? (Inputs: which evidence files / metric values arrive at the gate. Thresholds: what pass / block means, cited to the risk register. Failure behaviour: rollback / canary / escalate — the behaviour on a block, not just the block itself.)
4. Which of the ten items would have been the easiest to silently absorb at level 25 and is therefore the most dangerous to accept? (Item 2 — release-assurance sign-off. It *looks* like you doing your job; it is in fact the anti-pattern chapter 06 names first.)

## Why this lab matters

Most governance programs do not fail at authoring; they fail at *routing*. A clean deferral contract (exercise 05) is a design document; this lab trains the *runtime* behaviour — the inbox decision. If you can get through ten items in forty minutes with every row chapter-cited, the running deferral contract starts enforcing itself.
