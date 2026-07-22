# Third-Party Vendors, Build vs. Buy, and the App-Side Measurement Contract to `ai-eval-engineer`

Chapters 01–05 shipped and measured a stack built from open tooling. That is not the only option. A cluster of vendors sells guardrails as products — some as hosted APIs, some as platforms, some as enterprise stacks with UI and evidence trails. This chapter evaluates the six vendors most commonly seen on RFPs — **Lakera Guard**, **Arthur AI**, **Robust Intelligence**, **Calypso AI**, **HiddenLayer**, **Protect AI** — teaches the build-vs-buy matrix that produces a defensible decision, and defines the app-side measurement contract you hand to `ai-eval-engineer` (peer, level 30) so the eval track can consume your slot definitions without a translation project.

## Motivation

Two failure modes an experienced reviewer catches:

- **"We bought it, so we're covered."** A vendor guardrail is procured; the vendor's marketing coverage claims are read as the coverage story; nobody measures what the vendor's product actually catches on the mod-104 adversarial set for *this* application; the residual number credits the vendor and is unsafe to defend.
- **"We built it, so it's cheaper."** An in-house stack ships and is credited as free because "we already have the team." Nobody amortises the on-call, the drift monitoring, the model retraining, the classifier-fine-tune GPU spend, or the integration maintenance across LangChain/LangGraph API changes. Two years later, the TCO is above the vendor price the shop rejected.

The build-vs-buy decision has to be made against a matrix where both options are priced in the same currency, measured on the same adversarial set, and evidenced with the same trail.

## The six vendors

Vendor descriptions here are anchored to what each vendor publishes as its own scope. Vendor product lines evolve; treat this section as a map, not a spec sheet, and cite the vendor's own current documentation for the exact feature list at RFP time.

- **Lakera Guard.** Real-time content-and-injection classifier product; ships as a hosted API and (in some deployments) as a self-hosted option. Marketed for prompt-injection detection, unsafe-content filtering, and PII detection at input and output slots. Publishes classifier updates on a cadence and shares evaluation methodology in blog and technical write-ups on their site. <!-- needs-research: pin the current deployment options (SaaS, VPC, on-prem) and the current classifier taxonomy at RFP time. -->
- **Arthur AI.** Model-monitoring and guardrails platform; includes a runtime guardrail product ("Arthur Shield") alongside model performance / drift / bias monitoring. Vendor positioning: platform that spans mod-108 (guardrails) and mod-109 (monitoring). <!-- needs-research: confirm current product naming; Arthur's runtime-guardrail offering has been branded and re-branded. -->
- **Robust Intelligence.** AI security and validation platform; ships runtime protection and validation testing. Vendor positioning: enterprise AI security stack that covers pre-deployment testing and runtime protection. Acquired / product-changed context: <!-- needs-research: confirm current corporate status (Robust Intelligence was acquired by Cisco in 2024) and how the product is currently offered. -->
- **Calypso AI.** LLM security platform focused on prompt-injection detection, DLP, and content filtering for enterprise LLM adoption. Vendor positioning: enterprise gateway that sits between users and the LLM.
- **HiddenLayer.** AI security platform with focus on model integrity, adversarial-ML detection, and (more recently) LLM-runtime protection. Historic strength is the classical adversarial-ML detection side (mod-105 overlap).
- **Protect AI.** AI security platform with components covering model scanning (`ModelScan`, `Guardian`), LLM security (`Recon` red-team, `LayerX` runtime), and MLOps supply-chain hygiene. Vendor positioning: broad AI security portfolio; some components are open source. <!-- needs-research: pin current productisation and component naming; Protect AI's portfolio changes; confirm which parts remain open source (e.g., ModelScan is on GitHub as of authoring). -->

None of these evaluations is a recommendation. The point is that each vendor's coverage story is *its own*, and each must be measured against the same adversarial set the in-house stack is measured on, and evaluated on the same axes.

## The build-vs-buy matrix

The matrix rows are the axes on which build and buy are compared. One row per axis; one column per option (in-house stack, and each vendor under evaluation).

| Axis | What is measured | How to score |
|---|---|---|
| **Coverage on our adversarial set** | Recall on the mod-104 adversarial set, per category | Numbers from chapter 05 measurement; compare vendor to in-house at same threshold |
| **False-positive rate on our legitimate set** | FP rate on the mod-104 legitimate set | Same source; compare at the same threshold |
| **Latency added per slot** | p50 and p95 added latency in request path | Measure both options in the same middleware harness |
| **Cost per call and per month** | Compute + operational + licence, amortised | Vendor list price + integration cost + on-call; in-house at fully loaded rate including drift monitoring and retraining |
| **Deployment shape** | SaaS / VPC / on-prem; data residency; who runs the model | Explicit for each; if vendor is SaaS-only and data-residency requires no-egress, vendor is disqualified |
| **Portability / lock-in** | How much application code changes if the vendor is swapped; how easy is exit | Score high if vendor is behind a middleware interface you own; low if vendor SDK is embedded across the app |
| **Evidence trail** | Auditable trace of every decision; version pinning of classifier; changelog | Score based on whether the evidence needed for ISO/IEC 42001 and EU AI Act filings comes out of the box or has to be constructed |
| **Threshold / policy control** | Can you tune thresholds; can you author custom policies; can you pin classifier version | Score based on the vendor API surface |
| **Update cadence and predictability** | How often the classifier updates; whether updates can be pinned; whether behaviour changes are announced | Score based on the vendor's release policy |
| **Support model** | Named SLAs; incident-response terms; access to vendor engineering during an incident | From vendor contract |
| **Ecosystem fit** | Integration with your existing MLOps, GRC-for-AI platform (Credo AI / Holistic AI / ModelOp / Monitaur — mod-103), monitoring stack (mod-109), incident-response tooling (mod-111) | Score integrations that exist today, not ones on the roadmap |
| **Long-term viability** | Corporate stability; recent M&A activity; funding runway if applicable | Public information; annotate M&A carefully |

Each cell is either a *number* (recall %, FP %, ms, $/call) or a *bounded qualitative rating* with a defended note (never a naked "high/medium/low"). Naked ratings turn the matrix into vibes.

### Two summary numbers per option

The matrix collapses into two summary numbers per option:

- **Total residual-risk delta.** The change in the register's aggregated residual score when this option is credited as the control set. Positive = worse residual; negative = better.
- **Total three-year TCO.** All cost lines summed over three years, including exit cost (integration removal, data extraction).

Plot both. The chosen option is not always the lowest TCO or the best residual; it is the one whose combination the risk council can sign off on.

## The decision — build, buy, or compose

Three shapes of decision the matrix produces:

- **Build.** In-house stack; residual and cost both defensible; the org has the engineering capacity to maintain the drift-monitoring and re-training pipeline. Most defensible when the guardrail is deeply application-specific (bespoke policy, custom identifiers, custom tool-invocation logic).
- **Buy.** Vendor stack; residual within appetite and TCO acceptable; the vendor's coverage of the standard categories exceeds what the in-house team can build and maintain. Most defensible when the categories are commodity (standard toxicity, PII, prompt injection) and the vendor's investment in the classifier exceeds anything an internal team could match.
- **Compose.** In-house stack for the application-specific slots (tool-invocation policy, custom PII recognisers, bespoke output shape); vendor for the commodity slots (input classification, output moderation). Most common at scale; also the most complex to maintain because you have two evidence trails and two update cadences to manage. The residual and TCO gains have to be worth the operational complexity.

The decision is filed with the matrix as a signed decision record. When the vendor pricing changes, when the adversarial set version changes, or when the in-house team's on-call load changes materially, the decision is revisited.

## Handoff to `ai-eval-engineer` (peer, level 30)

The peer eval track (`ai-eval-engineer`, level 30) runs the model-side evaluation programme — statistical calibration, judge-vs-human calibration, benchmark harnesses, uncertainty quantification. The guardrail effectiveness measurement produced in chapter 05 is *input* to their programme. The handoff contract:

**Contract fields.**

| Field | What the risk engineer supplies | What the eval engineer does with it |
|---|---|---|
| Slot definitions | The five-slot list with named responsibilities and named failure modes (chapter 01) | Aligns their eval harness so per-slot numbers can be produced |
| Control instances | Named control per slot, with pinned version, threshold, mechanism (chapters 02–04) | Runs the control instance in their harness against the same adversarial set |
| Adversarial set version | mod-104 hosted version identifier | Consumes; does not fork |
| Legitimate set version | mod-104 hosted version identifier | Consumes; does not fork |
| Measurement rows | Chapter 05 measurement rows on the current run | Compares against their independent run; disagreements become a joint investigation |
| Cost currency and time units | Same currency as the risk register; latency in ms in request path | Uses in their comparative reports |
| Register linkage | Which register rows each measurement row updates | Uses to keep their statistical calibration and the register's residual numbers aligned |

**What the risk engineer explicitly does *not* supply.**

- Bootstrap confidence intervals on the recall numbers — the eval track computes these with the statistical rigour their level requires.
- Cohen's kappa on judge-vs-human agreement for judge-based scoring — same.
- The full benchmark methodology-of-methodology — same.

The risk engineer *uses* the numbers the eval track produces (e.g., "recall 91.2% with 95% CI [89.4%, 92.8%]") in the register. The methodology is linked, not duplicated.

**Cadence.** Handoff runs on every measurement re-run (typically per adversarial-set version bump and per major control change). Both tracks reference the same run manifest.

## Anti-patterns

Four anti-patterns specific to build-vs-buy and the eval handoff:

- **Vendor marketing as coverage story.** The RFP is decided from the vendor's coverage slides, not from measurement on your adversarial set. The residual you credit is fiction.
- **Build with hidden TCO.** In-house stack scored as "free" because the team is a fixed cost. Everything has a cost; amortise it.
- **Compose without owning the seam.** You pick vendor A for input and in-house for output, but the seam has no contract — who owns FP tuning at the seam? Who owns latency when both fire? Write the contract or don't compose.
- **Silent duplication of eval methodology.** The risk engineer computes bootstrap CIs manually because they weren't sure the eval track was doing it. Now there are two numbers with different CI methodologies. Coordinate up front.

## Cross-references

- **Chapter 05** — supplies the measurement rows that fill the matrix's coverage / FP / cost / latency cells.
- **Mod-103** — supplies the register into which the decision writes back (which residual number changes if we go build vs. buy vs. compose).
- **Mod-103 ch on GRC-for-AI platforms** (Credo AI / Holistic AI / ModelOp / Monitaur) — supplies the "ecosystem fit" row for each vendor.
- **Mod-104** — supplies the adversarial and legitimate sets on which every option is measured.
- **Mod-109** — vendors that also sell monitoring (Arthur AI, HiddenLayer, Protect AI variants) are scored on that surface at mod-109's chapter on tooling; keep the two evaluations coordinated.
- **Mod-110** — supply-chain and MRM considerations feed the "long-term viability" row and the "supply-chain compromise" side of vendor scoring.
- **Peer `ai-eval-engineer` (L30)** — receives the measurement contract; owns the statistical calibration.
- **Escalation to `senior-ai-governance-architect` (L50)** — the build-vs-buy decision is filed as a signed decision record; escalation happens if the residual or TCO exceeds bands defined at program level.

## Summary

- Six vendors dominate current guardrail RFPs: Lakera Guard, Arthur AI, Robust Intelligence, Calypso AI, HiddenLayer, Protect AI. Read each vendor's own current documentation before scoring.
- Build-vs-buy scores each option on twelve axes: coverage, FP, latency, cost, deployment shape, portability, evidence trail, threshold control, update cadence, support, ecosystem fit, viability.
- Cells are numbers or defended qualitative ratings, not naked colours. Collapse into two summary numbers per option: residual-risk delta and three-year TCO.
- Decision shapes: build, buy, compose. Filed as a signed decision record; revisited on trigger events.
- The measurement contract to `ai-eval-engineer` names slots, controls, set versions, register linkage; the eval track produces the statistical calibration; the risk engineer does not duplicate it.
- Anti-patterns: vendor-marketing coverage, hidden-TCO build, unowned seam in compose, silent duplication of eval methodology.
- Exercise 06 produces the matrix and the signed decision record for your running system.
