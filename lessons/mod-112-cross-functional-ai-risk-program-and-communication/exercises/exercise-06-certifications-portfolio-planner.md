# Exercise 06 — Certifications Portfolio Planner

**Estimated effort:** 1 hour

## Objective

Draft a defensible **three-year certification and career plan** for the reader's own employer profile and target career trajectory. The plan is a hiring-signal decision, not a badge-collection exercise: two to four credentials chosen against a named target employer type, sequenced across three years, with cross-training and public-contribution moves on the same timeline. The output is the artefact chapter 06's decision table exists to produce.

## Prerequisites

- Chapter 06 read closely — the five credential families, the which-one-when decision table, the career hand-offs to L35 / L40 / L50, and the sideways cross-training options.
- The certification-body landing pages skimmed for the current curriculum outline (they evolve per cycle; the resource list in `resources.md` has the entry points):
  - IAPP AIGP, CIPP/US, CIPP/E — <https://iapp.org/certify/>
  - ForHumanity Independent AI Auditor and specialisations — <https://forhumanity.center/>
  - BABL AI Algorithm Bias Auditor — <https://babl.ai/>
  - ISO/IEC 42001 Lead Implementer / Lead Auditor at PECB, BSI, TÜV, DNV — <https://pecb.com/en/education-and-certification-for-individuals/iso-42001>
- A short read of two or three job postings that match your target employer profile — you are choosing credentials against a *specific hiring signal*, and the postings tell you what that signal is.

## Problem statement

Draft a plan across three sections: (i) target employer profile and current baseline, (ii) three-year credential and public-contribution sequence, (iii) career hand-off shape at the end of year three. The plan is *your* plan; the exercise is not to produce a generic template.

## Requirements

Produce a single markdown document `mod-112-ex06-certifications-plan.md` with these sections:

### 1. Target employer profile

- **Employer type**: pick one from chapter 06's decision table rows (enterprise financial services, US-headquartered enterprise, audit firm, consulting firm, NYC LL 144-covered employer, and so on) *or* describe your own if it does not fit those rows. Name the specific organisations or organisational types (redact specific employer names if in a public repo).
- **Role level target**: level 25 (holding), or a next-up move (L35, L40, L50). Choose one; a plan that hedges across all four is not a plan.
- **Geography**: primary jurisdiction (US, EU, UK, other). Some credentials are geographically weighted (CIPP/US vs. CIPP/E; national accreditation bodies for ISO/IEC 42001).
- **Sector obligations in scope**: SR 11-7 for financial services; FDA GMLP + PCCP for medical devices; NYC LL 144; Colorado AI Act; EU AI Act high-risk. Choose the ones that apply.

### 2. Current baseline

- Credentials you already hold (with issue date and next-renewal date).
- Modules of this track completed or in progress (mods 101-112).
- Public contributions of hiring-signal shape: publications, talks, engagement with a public evaluation programme (MLCommons, AISI, open-source red-team), open-source AI-safety project maintenance.
- Non-credential experience that is load-bearing on your CV: on-call rotation for an AI system, incident-response ownership, red-team engagement leadership, model-card publication authorship.

### 3. Three-year sequence

For each of years 1, 2, and 3 (name the calendar years — 2026, 2027, 2028 given today is 2026-08-02), populate:

- **Primary credential to pursue** (one per year; two only if genuinely necessary). Name the credential, the training provider, the target exam date, the cost band (in your currency), and the renewal cadence.
- **Public-contribution move** (one per year). Options: submit to an MLCommons AI Safety benchmark; contribute to an open-source red-team programme; publish a blog post cited by a standards body or an AI-safety institute; present at FAccT, RSA, DEF CON AI Village, or an equivalent recognised venue; contribute to an AISI evaluation programme where possible.
- **Cross-training move** (one per year). Options: sideways into `ai-eval-engineer` (L30, AI Engineering family) for eval-gated CI/CD depth; sideways into `model-evaluation-engineer` (L30, ML Engineering family) for statistical methodology depth; deeper reading and practice in a specific module you are weaker on.
- **Rationale**: 2-3 lines. Why this credential this year against your target employer profile. Why this public contribution. Why this cross-training.

### 4. Career hand-off shape

- **Target next-up role** at the end of year 3: `ai-evaluation-engineer` (L35) / `agentic-safety-engineer` (L40) / `senior-ai-governance-architect` (L50) / holding at L25 with sideways strengthening.
- **The three concrete cross-training items** chapter 06 lists for the chosen next-up role, mapped to which year of the plan delivers each item. For example, for the L35 move: (a) deep familiarity with the crosswalk (delivered by exercise 05 + one real audit cycle in year 2); (b) ownership of a full audit cycle end-to-end (delivered by year 3 internal audit engagement); (c) experience packaging evidence for a real regulator engagement (delivered by year 3 sector-regulator response, if available).
- **Gap analysis**: what the plan does *not* deliver and what the next plan cycle would cover.

### 5. Risk register on the plan itself

Two or three lines on what could invalidate the plan — a change of employer type; a jurisdictional move; a credential body's programme evolving significantly; a new load-bearing credential appearing (per chapter 06's `needs-research` markers, the field evolves). The point is not to hedge; it is to name the reasons you would deliberately re-plan.

## Starter guidance

- Pick the employer profile *first*, then the credentials. Chapter 06's decision table is organised employer-first for a reason: the same credential is load-bearing in one employer type and marginal in another.
- Do not pick more than two credentials in a single year unless you have a genuine reason (e.g., you are between roles and can invest full time). Certification study time is real; chapter 06's five-year plan template invests in one significant credential per year for exactly this reason.
- On the ForHumanity credential, note which *specialisation* you would target — the Independent AI Auditor base credential is the entry point, but the specialisations (bias audit, EU AI Act audit) are what a specific employer type filters on.
- On ISO/IEC 42001 LI vs. LA: LI (Lead Implementer) is the credential for a risk engineer who will *build* an AIMS inside the enterprise. LA (Lead Auditor) is the credential for a risk engineer who will *audit* an AIMS (internal audit function or external audit firm). Both, in sequence, is the L50 shape. Neither is the shape of an engineer whose target is L40 frontier work.
- On CIPP/US vs. CIPP/E: choose by jurisdictional load, not by prestige. If your primary customer base is EU, CIPP/E; if US, CIPP/US. Both is unusual and only makes sense for cross-Atlantic advisory roles.
- On public contributions: authorship of one hiring-signal artefact (a benchmark contribution, an AI-village talk, a blog post the FMF cites) generally outweighs three of lower profile. Chapter 06's "publications and speaking" note is where this becomes concrete.
- On cross-training: sideways moves *strengthen* the L25 role — they do not replace credential investment. A year with a credential *and* a cross-training move is normal; a year with only a cross-training move is a slow year that a hiring manager notices.
- If you cannot commit to the plan financially (credentials cost real money), note the constraint honestly and pick the highest-leverage single credential per year rather than the full portfolio. AIGP alone is a defensible year-1 investment for most target profiles.

## Acceptance criteria

The plan is accepted when:

- The target employer profile is specific: employer type, role-level target, geography, sector obligations in scope — no hedging.
- The current baseline enumerates credentials held (with dates), track progress, and any public contributions.
- The three-year sequence names one primary credential per year, one public-contribution move, one cross-training move, and 2-3 lines of rationale per year.
- The career hand-off section names the target next-up role and maps the three cross-training items to specific years.
- The gap analysis names what the plan does not deliver.
- The plan risk register names 2-3 conditions that would trigger re-planning.
- A hiring manager reading the plan would agree the credentials chosen match the employer type, and that the sequence is realistic (not "AIGP + ISO 42001 LI + LA + AIGP renewal all in year 1").
- A reader who is not the author can, from the plan alone, describe the reader's target career trajectory in two sentences.

## Stretch goals

- Do the **budget line** for the plan: sum the credential costs, training-course fees, exam fees, and renewal-credit fees for the three years. Add estimated time investment (weeks of study per credential). This is the artefact you take to a manager conversation about learning budget and time.
- Draft a **fallback plan**: what to substitute if your employer changes, or if a target credential is deprecated or restructured mid-plan. Chapter 06 flags the volatility of certification programmes — a plan without a fallback assumes stability the market does not have.
- Do the **read-across** against your existing CV: for each credential in the plan, what CV bullet does the credential enable that you cannot write today. If the answer is "none," the credential may not be load-bearing for your case. If the answer is "the enterprise-hire filter I keep failing at," the credential is well-chosen.
- Draft a **peer-review request**: send the plan to one person in your target employer type (an existing L25 who works there; an ex-colleague; a hiring manager you know) and ask for one paragraph of feedback. The feedback is the acceptance test the exercise cannot run for you.
- If your target is the L40 move, do the **non-credential alternative**: chapter 06 flags that L40 depth is largely non-credentialled. Draft an alternative version of year 2 and year 3 that swaps a credential for a public research contribution or a public red-team engagement. This is the more likely shape of a genuine L40 trajectory.

## Deliverable format

A single markdown document with the five sections above, in order. Two to four pages is the working target.

## Submission

The plan is a personal artefact; unless your programme runs a shared career-planning cycle, the submission is to yourself and to a peer or manager you trust for feedback. If you are working through the track as part of a formal cohort, submit to the cohort review channel; the plan is the artefact that closes the track and opens the next phase.
