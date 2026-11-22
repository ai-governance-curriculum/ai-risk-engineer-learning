# Lab 03 — Privacy-Risk Report GDPR Mapping and Boundary-Notes Authoring Drill

**Duration:** 45 minutes · **Format:** guided practice, markdown authoring on the fabricated running system from labs 01 and 02

## Objective

Internalise chapter 06's **privacy-risk report** shape by authoring two load-bearing sections end-to-end on the lab-01 / lab-02 running system (`CareTriage`): **Section 7 (GDPR mapping table, Article 22 in particular)** and **Section 10 (open items and boundary notes)**. The lab drills the two shapes where the report either over-reaches (engineer giving a legal opinion) or under-reaches (calling something audit-grade when it needs level-30 methodological calibration) — and where the "GDPR-compliant" anti-pattern is foreclosed. The target output is the shape the risk council, the DPO, and (indirectly) counsel read from the same document with two entry points.

The two anti-patterns chapter 06 names — "the system is GDPR-compliant" as a summary sentence, and "TBD legal" open items without a specific fact-pattern — are the shapes this lab's output forecloses.

## Prerequisites

- Chapter 06 read — specifically "The report shape" (ten-section layout), "Section 7 — GDPR mapping table (shape)" (Article 22 in particular), "The residual-risk claim the report is entitled to make" (attack strength / time / threat model as the three bounds), "Boundary 1 — what is out of scope to legal counsel", and "Boundary 2 — what escalates to `model-evaluation-engineer` (level 30)".
- Chapter 01 skimmed — the four-attack taxonomy and the "Mapping to GDPR concepts" table.
- Labs 01 and 02 completed — the MIA slice-report and the DP-SGD trade-off / operating-point memo are the Section 3 and Section 4 content this lab's Section 7 and Section 10 anchor to. If you have not done labs 01 and 02, do them first; the boundary-notes drill cannot land without specific register numbers to escalate.
- A terminal where you can create a working directory and author a markdown file. No external access, no compute — the numbers are inherited from labs 01 / 02 verbatim.

## The fixed example system (carried over from labs 01 and 02)

**`CareTriage`** — the gradient-boosted tabular classifier for care-coordination follow-up prioritisation at a regional care-coordination organisation. Article-9 cohort in the training data; API returns top-1 label plus two-class probability vector; DP-SGD operating point from lab 02 is Variant A at ε = 8, δ = 10⁻⁵; the lab-01 LiRA residual is on the register row `REG-CT-MIA-01`.

### Additional scenario context — authoritative for this lab, do not revise

Chapter 06's Section 7 is where GDPR Article 22 (automated individual decision-making) attaches or does not; the relevant fact pattern is whether the system's output has legal or similarly significant effect for the subject *without meaningful human intervention*. For this lab:

- The `CareTriage` output is read by a care-coordination nurse who prioritises outreach from the queue; the nurse makes the outreach decision, but in practice the queue order is followed verbatim roughly 94 % of the time.
- The 94 %-follow-rate is from an internal process-mining study of the nurse's workflow across the past six months.
- The downstream consequence of being high in the queue is "earlier outreach, more coordination contacts over the quarter"; the consequence of being low in the queue is a risk of being deferred past the quarter cutoff.
- A legal opinion has not been commissioned on whether this constitutes a decision with legal or similarly significant effect for the purposes of Article 22.

Additional scenario stubs (fabricated — for authoring the Section 7 table):

| Register row id (carried / new) | Scenario |
|---|---|
| `REG-CT-MIA-01` | MIA on `CareTriage`; Article-9 cohort implicated (labs 01, 02) |
| `REG-CT-AI-01` | Attribute inference of the Article-9 cohort membership from quasi-identifiers + model output |
| `REG-CT-DEC-01` | The queue order from `CareTriage` plausibly constitutes an automated decision with similarly significant effect under Art. 22 — unresolved; routed to legal |

Open items known going in (fabricated — these are what Section 10 escalates):

1. The legal question of whether the 94 %-follow-rate crosses the "meaningful human intervention" threshold under GDPR Article 22 — not resolved internally; routed to DPO + counsel.
2. The question of whether a separate lawful basis under Article 6 applies to the training-data reuse from the Article-9 cohort — the original collection was for care-delivery purposes; the training use is a secondary purpose. Routed to DPO + counsel.
3. 95 % confidence intervals on the LiRA TPR @ fixed FPR numbers from lab 01 — point estimates reported; CI estimation is level-30 methodological work. Routed to `model-evaluation-engineer`.
4. Statistical-significance framing of the Article-9-cohort-vs-overall LiRA delta — the delta is 0.049 vs 0.028 (lab 02 Variant A) but a significance test on the per-slice sample sizes is not in the level-25 report. Routed to `model-evaluation-engineer`.
5. Distribution-shift robustness of the LiRA attack against the DP-SGD-trained variants when the shadow data is drawn from a distribution further from `CareTriage`'s training distribution. Routed to `model-evaluation-engineer`.

## Steps

### Step 1 — Create your working directory and the scaffold

```bash
mkdir -p mod107-lab03 && cd mod107-lab03
touch caretriage-privacy-report-sections-7-and-10.md
```

Open `caretriage-privacy-report-sections-7-and-10.md` and paste this scaffold. You will fill the two sections in order.

```markdown
# CareTriage — privacy-risk report (lab 03 slice: Sections 7 and 10)

System: CareTriage (see lab 01 brief)
Chapter references: mod-107 ch 06 (ten-section report; GDPR Section 7; residual-risk claim bounds; boundaries to counsel and L30); ch 01 (GDPR concept table)
Lab-01 cross-ref: `caretriage-mia-slice-report.md` — Section 3 of the full report
Lab-02 cross-ref: `caretriage-dp-sgd-tradeoff.md`, `caretriage-dp-sgd-defence-memo.md`, `caretriage-dp-sgd-checklist.md` — Section 4 of the full report
Report version: v0.1 (pre-review)

(Sections 1–6 and 8–9 are out of scope for this lab; only Sections 7 and 10 are authored here. Sections 7 and 10 depend on lab-01 and lab-02 content being pinned, which it is.)

## Section 7 — GDPR mapping

### 7.1 Mapping table

(one row per register scenario; GDPR concept invoked; Article; notes)

### 7.2 Article 22 — the automated-individual-decision-making anchor

(does Art. 22 apply? engineering read — not a legal conclusion; if it may apply, record the human-in-the-loop configuration and the safeguards from Art. 22(3))

### 7.3 Article 35 — DPIA note

(does a DPIA apply? engineering evidence named; DPIA authorship is DPO / L15, not this report)

### 7.4 Residual-risk claim — the three bounds

(the chapter-06 "the report's residual-risk claim is bounded on three axes" paragraph — attack strength, time, threat model; the exact engineering sentence and the "not GDPR-compliant" anti-pattern note)

## Section 10 — Open items and boundary notes

### 10.1 Items routing to legal counsel

(each item: fact pattern; technical evidence attached; escalation destination — one of DPO + counsel)

### 10.2 Items routing to `model-evaluation-engineer` (level 30)

(each item: the level-25 position; the methodological calibration needed; routing destination)

### 10.3 Items routing to the risk council

(each item: the decision the council owns; the engineering evidence attached)

### 10.4 Cross-references

(pointers to lab 01, lab 02, and to mod-101 / mod-102 / mod-103 / mod-109 artefacts)
```

### Step 2 — Fill Section 7.1 (GDPR mapping table)

Chapter 01's "Mapping to GDPR concepts" and chapter 06's "Section 7 — GDPR mapping table (shape)" together pin the shape: one row per register scenario, GDPR concept invoked, Article, notes. Paste the table.

```markdown
| Register row | Attack / scenario | GDPR concept invoked | Article | Notes |
|---|---|---|---|---|
| `REG-CT-MIA-01` | MIA on `CareTriage`; adversary decides whether a specific patient record was in the training set | Personal data; **special-category personal data** when the training-set identity encodes Article 9 health data, which the Article-9 cohort does | Art. 4(1); Art. 9 | Membership in the Article-9 cohort is itself Art. 9 data by chapter 01's framing; lab 01 LiRA TPR @ 1 % FPR = 0.16 on this slice is the inherent-column evidence |
| `REG-CT-AI-01` | Attribute inference of Article-9 cohort membership from quasi-identifiers + model output | Personal data (the inferred attribute is personal data about the subject); **special-category personal data** on the Article-9 inference specifically; **profiling** if the inference feeds a decision | Art. 4(1); Art. 4(4); Art. 9 | Attribute inference against an Article-9 attribute is itself Art. 9 processing; note this is the row exercise 02 populates with the inherent number, lab 03 inherits only the GDPR tag |
| `REG-CT-DEC-01` | The queue order from `CareTriage` plausibly constitutes an automated decision with similarly significant effect for the subject | **Automated individual decision-making, including profiling**; subject has rights under Article 22 if the engineering read applies (see §7.2) | Art. 22 | 94 %-follow-rate from the care nurse's workflow is the fact pattern that pushes this scenario into Art. 22 territory; the legal conclusion is counsel's, not this report's |
| (any scenario on this system with material privacy risk to subjects) | — | Controller obligation to run a **Data Protection Impact Assessment** | Art. 35 | See §7.3 — DPIA authorship is DPO + `ai-governance-analyst` (L15); this report is a material input, not the DPIA itself |
| (any credited control from lab 02) | — | **Data protection by design and by default**; **security of processing** | Art. 25; Art. 32 | DP-SGD training + top-1-label-only API are evidence for Art. 25 / 32; the mapping is not one-to-one, but these are the two articles most of the lab-02 credited controls contribute to |
```

Reading check: five rows; each with register-row id or scenario class, GDPR concept, Article number, and a specific note tying to lab-01 / lab-02 evidence. If any row cites an Article without a specific fact pattern, the mapping is decorative rather than load-bearing.

### Step 3 — Fill Section 7.2 (Article 22 anchor)

Chapter 06 is explicit that Article 22 is where the privacy report intersects most directly with the wider governance stack, and that the engineer records whether Article 22 applies *as an engineering read*, not as a legal conclusion. Write the paragraph.

```markdown
**Engineering read of Article 22 applicability.** The `CareTriage` output is read by a human
care nurse who makes the outreach decision in nominal terms; however, the internal
process-mining study of the nurse's workflow across the past six months shows the queue
order is followed verbatim ~94 % of the time. Whether that cross-over rate crosses the
"meaningful human intervention" threshold under Art. 22 is a legal question this report
does not resolve — it is specifically routed to DPO + counsel in §10.1. For the engineering
record:

- *If Art. 22 applies (as counsel may determine)*, the subject has rights under Art. 22(3)
  to obtain human intervention, express a view, and contest the decision. The engineering
  surfaces that support those rights are: (i) the nurse's queue-ordering UI, which is where
  human intervention is nominally expressed — a review of this UI against the Art. 22(3)
  "meaningful" bar is a product-level action owned by the care-coordination product team
  and the DPO; (ii) a subject-facing mechanism to contest the follow-up prioritisation does
  not currently exist on the patient portal — a product action item is routed in §10.3;
  (iii) transparency about the decision logic — the model's feature list is documented;
  the specific per-patient reasoning is not surfaced to the subject today. Each surface is
  either an engineering artefact (UI, API, workflow) or a policy artefact; the legal
  interpretation of "meaningful intervention" sits on counsel.
- *If Art. 22 does not apply (counsel may also determine this based on the human-nurse
  layer)*, the Art. 22(3) safeguards do not formally attach, but the subject-facing contest
  mechanism in (ii) above is still a product-quality improvement; it is not blocked by the
  Art. 22 determination.

This paragraph is the **engineering evidence** counsel needs to form the legal opinion; it
is not the legal opinion itself.
```

Reading check: the paragraph (a) names the fact pattern (94 %-follow-rate), (b) names the two possible legal outcomes without choosing between them, (c) catalogues the engineering surfaces that correspond to Art. 22(3) safeguards, and (d) explicitly marks that the legal conclusion is counsel's. A paragraph that reads "Art. 22 applies" or "Art. 22 does not apply" without the escalation is the exact chapter-06 over-reach anti-pattern.

### Step 4 — Fill Section 7.3 (Article 35 — DPIA note)

Short paragraph — chapter 06 pins DPIA authorship as DPO / L15, not this report.

```markdown
A Data Protection Impact Assessment (Art. 35) is a controller obligation when processing is
likely to result in high risk to the rights and freedoms of natural persons. The combination
of (a) special-category data under Art. 9 in the training set, (b) potentially-in-scope
automated decision-making under Art. 22 (§7.2), and (c) the measured residual MIA risk on
the Article-9 cohort (LiRA TPR @ 1 % FPR = 0.049 after lab-02 Variant A) is, in the
engineering read, the fact pattern that typically triggers a DPIA — but the trigger
decision is a governance decision owned by the DPO. This report is material to the DPIA (it
is one of the documents the DPIA cites); the DPIA itself is authored by the DPO and by
`ai-governance-analyst` (L15, mod-101 chapter 03; ISO/IEC 42005). An open item to confirm
the DPIA trigger decision with the DPO is routed in §10.1.
```

Reading check: the paragraph names the fact pattern, names the owner of the DPIA trigger decision (DPO; not this report), and routes the open item. A paragraph that says "we have done the DPIA" or "a DPIA is not required" is either over-reach or under-reach.

### Step 5 — Fill Section 7.4 (Residual-risk claim — the three bounds)

Chapter 06 pins the residual-risk claim as bounded on **attack strength**, **time**, and **threat model** and names "the system is GDPR-compliant" as the anti-pattern summary sentence. Write the paragraph.

```markdown
The residual-risk claim this report is entitled to make, carrying forward lab-01 and lab-02:

> Under the threat model in lab 01 Section 1 (black-box API; probability-vector exposure
> downgraded to top-1-label-only; population-shaped shadow data) and the attack
> configuration in lab 01 Section 2 (LiRA online, N = 128 shadow models, scaled-true-class-logit
> statistic), and under the DP-SGD mechanism in lab 02 Section 1 (Opacus RDP accountant;
> ε = 8, δ = 10⁻⁵; σ = 0.7; C = 1.0; Poisson sampling; record-level neighbouring;
> `ModuleValidator.fix` applied; no warm-start), as of release R / training-data snapshot T,
> the residual LiRA TPR @ 1 % FPR on `CareTriage` is **0.028 overall, 0.049 on the Article-9
> cohort slice, and 0.090 on the outlier-decile slice**.

This claim is bounded on:

- **Attack strength** — a stronger MIA attack, a larger shadow-model count, or a shadow
  dataset drawn closer to the training distribution would raise the number.
- **Time** — as of release R and training-data snapshot T; the claim is valid for the
  current release cadence only.
- **Threat model** — the black-box labels-only API is the current exposure; an API upgrade
  to full probability vectors or to logits would raise the number.

The report **does not** conclude "the system is GDPR-compliant." That is a legal
conclusion, not an engineering claim, and it is specifically foreclosed by chapter 06. What
this report concludes is: the technical controls listed in lab 02 close the empirical risks
measured in lab 01 to the residual levels above; the mapping to GDPR concepts is in §7.1
and §7.2; the compliance conclusion is counsel's.
```

Reading check: the claim is parameter-conditioned on attack strength, threat model, and time; it cites specific lab-01 and lab-02 configurations; it names the three bounds explicitly; and it explicitly forecloses the "GDPR-compliant" sentence. A claim that reads "the system is privacy-safe" has stripped the three bounds.

### Step 6 — Fill Section 10.1 (Items routing to legal counsel)

Each item in Section 10 has a **fact pattern**, **concrete technical evidence attached**, and an **escalation destination** — chapter 06 is explicit that nothing is left as an unspecified "TBD legal." Paste the three counsel items.

```markdown
Each item below is routed to the DPO + external counsel. The engineering evidence is
attached; the legal opinion is counsel's.

1. **Whether the queue order from `CareTriage` constitutes an automated decision with legal
   or similarly significant effect under GDPR Article 22.**
   - Fact pattern: the care nurse nominally makes the outreach decision, but the internal
     process-mining study of the past six months shows the queue order is followed verbatim
     ~94 % of the time. The downstream consequence of queue position is earlier vs. later
     outreach, with deferral past the quarter cutoff as the tail-risk outcome. The
     "meaningful human intervention" threshold is contested in this fact pattern.
   - Evidence attached: the process-mining study (link in §10.4); the register row
     `REG-CT-DEC-01`; §7.2 engineering surfaces catalogue.
   - Escalation: DPO + external counsel.
2. **Whether a separate lawful basis under GDPR Article 6 applies to the secondary use of
   Article-9-cohort training data for `CareTriage`'s model training.**
   - Fact pattern: the original collection of the Article-9-cohort records was for direct
     care-delivery purposes under a prior lawful basis. The training use is a secondary
     purpose. The compatibility of the secondary purpose with the original lawful basis is
     a legal determination.
   - Evidence attached: the dataset-provenance manifest (lab-02 reproducibility manifest);
     the retention and consent record held by the organisation's data-governance team.
   - Escalation: DPO + external counsel.
3. **Whether the DPIA trigger threshold (Art. 35) is met and, if so, which existing document
   set (this report + the mod-103 register + the mod-102 harm model) is the DPIA's
   material input.**
   - Fact pattern: Art. 9 data in training; potentially in-scope Art. 22 processing; the
     residual MIA risk on the Article-9 cohort is measured but not known to be acceptable
     to the controller without a formal risk-appetite statement.
   - Evidence attached: §7.3; the residual-risk claim in §7.4; the register row
     `REG-CT-MIA-01` with inherent / residual / control-defeated cells.
   - Escalation: DPO.
```

Reading check: three items, each with fact pattern, technical evidence attached, escalation destination. No "TBD legal" entries; no items without a fact pattern. If any item reads "ask legal about privacy," it has violated the chapter-06 §10 discipline.

### Step 7 — Fill Section 10.2 (Items routing to `model-evaluation-engineer` level 30)

Chapter 06 "Boundary 2" lists the level-30 methodological calibration items. Three fabricated items were pinned in the lab brief; paste them with the level-25 position and the routing.

```markdown
Each item below is routed to `model-evaluation-engineer` at level 30 for the methodological
calibration pass. The level-25 position is stated for each so the peer reviewer sees what the
report currently claims and where the methodological delta applies.

1. **95 % confidence intervals on the LiRA TPR @ fixed FPR numbers from lab 01 / lab 02.**
   - Level-25 position: point estimates reported — LiRA TPR @ 1 % FPR = 0.028 overall / 0.049
     Article-9 / 0.090 outlier-decile at the chosen operating point (lab 02 Variant A). No
     CI is attached in this report.
   - L30 calibration needed: 95 % CIs on the ROC and on TPR-at-fixed-FPR under the current
     shadow-model count (N = 128) and candidate-set size (20 000). Chapter 06 "Boundary 2"
     is the escalation reference.
   - Routing: `model-evaluation-engineer` at level 30.
2. **Statistical-significance framing of the Article-9-cohort-vs-overall LiRA delta.**
   - Level-25 position: Variant A LiRA TPR @ 1 % FPR is 0.049 on the Article-9 cohort vs
     0.028 overall — a delta of +0.021 at this attack configuration. The delta is reported;
     no significance test on the per-slice sample sizes (n = 2 200 cohort, 20 000 overall)
     is attached.
   - L30 calibration needed: a significance-framed read of the delta — is 0.049 vs 0.028
     statistically distinguishable given the slice sample size, and if so with what power?
   - Routing: `model-evaluation-engineer` at level 30.
3. **Distribution-shift robustness of the LiRA attack against the DP-SGD-trained variants
   when the shadow data is drawn from a distribution further from `CareTriage`'s training
   distribution.**
   - Level-25 position: the lab-01 shadow data is population-shaped but non-identical to
     the target's training distribution; the "distribution gap is small but non-zero" note
     is in the threat-model header. The eval reports the strongest attack run under that
     configuration; no distribution-shift sensitivity curve is attached.
   - L30 calibration needed: characterise how the LiRA residual numbers move as the shadow
     data's distributional distance from the target's training distribution grows; the
     register row's residual-risk signal needs an envelope, not just a point.
   - Routing: `model-evaluation-engineer` at level 30.
```

Reading check: three items, each with the level-25 position (what the report currently says), the L30 calibration needed (what the peer would add), and the routing. No vague "needs more work" entries.

### Step 8 — Fill Section 10.3 (Items routing to the risk council) and Section 10.4 (Cross-references)

```markdown
### 10.3 Items routing to the risk council

1. **Residual-risk appetite on Article-9-flagged register rows.** The current working
   appetite is slice TPR @ 1 % FPR ≤ 0.10, which Variant A clears (Article-9 cohort at
   0.049). Formal ratification of this appetite is a risk-council decision; the engineering
   evidence attached is the lab-02 defence memo plus the full trade-off table.
2. **Subject-facing contest mechanism on the patient portal.** §7.2 notes the contest
   mechanism does not currently exist; adding it is a product-quality action independent of
   the Art. 22 determination. The risk council owns the prioritisation of the product-side
   work.
3. **Return top-1 label only on the production API — formal adoption of the API-shape
   control.** Lab-01 §6 and lab-02 §5 both credit this control's residual-risk delta (LiRA
   TPR @ 1 % FPR = 0.082 with probability vector exposed vs 0.019 top-1-label-only on the
   non-DP baseline; comparable deltas on the DP variants). Formal adoption as a
   release-blocking control is a risk-council + product-owner decision.

### 10.4 Cross-references

- `caretriage-mia-slice-report.md` (lab 01) — Section 3 attack evaluation summary for the
  full privacy-risk report
- `caretriage-dp-sgd-tradeoff.md`, `caretriage-dp-sgd-defence-memo.md`,
  `caretriage-dp-sgd-checklist.md` (lab 02) — Section 4 defence summary
- `REG-CT-MIA-01`, `REG-CT-AI-01`, `REG-CT-DEC-01` on the mod-103 register — Section 5
  residual-risk claim per scenario
- mod-102 harm-model row `HM-CT-MIA-01` — Section 2 scope anchor
- mod-101 framework mapping — Section 8 NIST Privacy Framework function mapping (not
  authored in this lab; chapter 06 §8 is the shape)
- mod-109 monitoring hooks — Section 9 (not authored in this lab; chapter 06 §9 is the
  shape)
- Internal process-mining study of care-nurse workflow (referenced in §10.1 item 1) —
  <!-- needs-research: pin the study's artefact ID at authoring time -->
```

Reading check: Section 10.3 has three risk-council items each with a specific decision and the evidence attached; Section 10.4 points to each of the upstream and downstream artefacts the report composes with. The `<!-- needs-research: ... -->` marker is the chapter-06-sanctioned way to flag a pointer that has to be pinned at authoring time.

### Step 9 — Final checkpoint

Re-read Section 7 and Section 10. Confirm:

- **Section 7.1 has one row per register scenario, with GDPR concept, Article number, and lab-01 / lab-02 evidence note.** Not a free-form paragraph; not Articles without fact patterns.
- **Section 7.2 names the fact pattern for Article 22 (94 %-follow-rate), catalogues the engineering surfaces for Art. 22(3) safeguards, and marks the legal conclusion as counsel's.** Not a conclusion either way.
- **Section 7.3 names the DPIA-trigger fact pattern and names DPO / L15 as the owner.** Not "we have done the DPIA."
- **Section 7.4 states the residual-risk claim parameter-conditioned on attack strength, time, and threat model, and explicitly forecloses the "GDPR-compliant" sentence.** The three bounds are each named.
- **Section 10.1 has fact pattern + evidence attached + escalation destination for each counsel item.** No "TBD legal" entries.
- **Section 10.2 has level-25 position + L30 calibration needed + routing for each L30 item.** No "needs more work" entries.
- **Section 10.3 names risk-council decisions with evidence pointers.** Not generic "need to decide" items.
- **Section 10.4 points to the lab-01 and lab-02 outputs plus the register rows.** The pointers compose with the rest of the ten-section report.

## Expected output

`mod107-lab03/caretriage-privacy-report-sections-7-and-10.md` — a two-section report slice; one-and-a-half to two pages. Section 7 has a five-row GDPR mapping table, an Article 22 engineering-read paragraph, a DPIA note, and a parameter-conditioned residual-risk claim that forecloses the "GDPR-compliant" summary. Section 10 has three counsel items, three L30 items, and three risk-council items, each with fact pattern / evidence / routing.

## Checkpoint questions (self-check)

1. A product manager reads §7.2 and asks for a shorter version: "just write that Art. 22 applies and we need a contest mechanism." Per chapter 06, what is wrong with that request and what is the shippable rebuttal? (*Chapter 06 is explicit that the engineer records whether Art. 22 applies **as an engineering read**, not as a legal conclusion — the sentence "Art. 22 applies" is counsel's to make, not the engineer's. The §7.2 paragraph's job is to catalogue the fact pattern (94 %-follow-rate), the two possible legal outcomes, and the engineering surfaces that correspond to Art. 22(3) safeguards; the legal determination is escalated in §10.1. The rebuttal shape is: "the engineering read is that the fact pattern plausibly engages Art. 22; the legal conclusion is counsel's — see §10.1 item 1; the engineering surfaces that support the Art. 22(3) safeguards if Art. 22 applies are catalogued in §7.2." A shorter version that writes the legal conclusion is the over-reach anti-pattern chapter 06 names.*)
2. A reviewer suggests merging §10.1 (counsel items) and §10.2 (L30 items) into a single "open items" list "because an open item is an open item." Per chapter 06, what is the chapter's framing of the two boundaries and why does merging them break the report? (*Chapter 06 Boundaries 1 and 2 are **distinct** — items routing to counsel are legal-interpretation questions (Art. 22 applicability, Art. 6 lawful basis, DPIA authorship) where the engineer attaches evidence and counsel forms the opinion; items routing to L30 are methodological-calibration questions (CIs on ROC, statistical-significance framing, distribution-shift robustness) where the engineer has a point estimate and the peer role adds the error envelope. The two boundaries have different owners, different kinds of deliverable, and different follow-up cycles. Merging them loses that routing — a counsel item mislabelled as an "open item" sits without a destination; a L30 item mislabelled as a counsel item generates confusion about whether the engineer should have done more work before filing. Chapter 06 is explicit that each item has a specific escalation destination and nothing is left as an unspecified "TBD."*)
3. A DPO asks why §7.4 says "the compliance conclusion is counsel's" rather than committing to a conclusion; the DPO argues that the engineering evidence is strong and the controls are in place. Per chapter 06, what is the engineering position and what is the shippable sentence? (*Chapter 06 "The residual-risk claim the report is entitled to make" names "the system is GDPR-compliant" as an anti-pattern summary sentence specifically because it is a **legal conclusion**, not an engineering claim. The strength of the engineering evidence is not what makes a sentence a legal conclusion; its content does. The engineering position is: strong evidence makes the eventual legal opinion defensible, but the opinion itself is counsel's to form. The shippable sentence (also from §7.4) is: "the technical controls listed in §4 close the empirical risks measured in §3 to the residual levels in §5; the mapping to GDPR concepts is in §7; the compliance conclusion is counsel's." A sentence that collapses that structure into "we are compliant" has moved outside the engineer's scope, regardless of the evidence strength.*)
4. A level-30 reviewer looks at §10.2 and asks why the residual-risk claim in §7.4 cites point estimates rather than intervals. Per chapter 06, what is the level-25 position on this and what is the chapter's framing of why this is not a weakness? (*Chapter 06 "Boundary 2" is explicit that confidence intervals on ROC and TPR-at-fixed-FPR, statistical-significance framing of slice-level deltas, and distribution-shift robustness of the eval itself are level-30 work — not level-25. The level-25 report cites the point estimate at the run's shadow-model count and names the escalation; the level-30 review adds the interval. This is **not a weakness** of the level-25 report — chapter 06 explicitly says "this is not a weakness of the report — it is the shape of a shippable level-25 artefact, and it is how the peer track and this track compose without overlap." The shippable sentence for a reviewer asking the question is: "point estimate at the run's configuration is in §7.4; interval estimation is escalated to `model-evaluation-engineer` at level 30 — see §10.2 item 1."*)

## Why this lab matters

Exercises 01–05 produce the register rows, the attack evaluations, the defence memos, and the Presidio pipeline — but the final privacy-risk report (chapter 06) is where those artefacts compose into one document that reads top-down as an argument in numbers and bottom-up as an evidence trail. Section 7 (the GDPR mapping) and Section 10 (open items and boundaries) are the two sections where the engineer most often either over-reaches (writing a legal conclusion) or under-reaches (calling something audit-grade when it needs L30 calibration). This lab drills the atomic shape of those two sections on the lab-01 / lab-02 fabricated system so that when the real report is authored at the end of the module, the shape is routine and the time goes into pinning the specific fact patterns, evidence pointers, and escalation destinations — not into inventing the boundary-notes format. Section 7's "GDPR-compliant" anti-pattern foreclosure and Section 10's "fact-pattern + evidence + routing" discipline are the two shippable shapes chapter 06 names as non-negotiable.
