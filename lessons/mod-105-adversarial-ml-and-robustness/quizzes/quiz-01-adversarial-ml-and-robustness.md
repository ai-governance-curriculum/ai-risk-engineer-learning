# Quiz 01 — Adversarial ML and Robustness Engineering

**Format:** 12 multiple-choice questions · **Open-book.** Use the chapters as reference.

Each question tests a *concept* from the module, not phrasing. The answer key at the bottom cites the chapter the answer comes from.

---

## 1. A team's risk register tags an attack as "adversarial example" in one row and "input manipulation attack" in another, and a third row calls the same technique a "robustness evasion." A new auditor reads the register and cannot tell whether the three rows are the same scenario in three vocabularies or three different scenarios. Per chapter 01, what did the team get wrong, and what is the fix?

- A. Nothing — different vocabularies are normal; the auditor should read the technique descriptions.
- B. They should have collapsed all three terms into whichever name the newest paper uses.
- C. They never pinned one taxonomy as the *authoritative* name and routed the other vocabularies through it as cross-tags. Chapter 01 pins NIST AI 100-2 e2025 as the taxonomy spine; MITRE ATLAS and OWASP ML Top 10 are overlays (operational and AppSec), *not* replacements. The fix is one NIST-category column plus explicit ATLAS and OWASP columns per row — not three competing names for the primary category.
- D. They should move to MITRE ATLAS as the authoritative taxonomy because it has case studies.

## 2. A scenario-row on the cross-tag sheet reads: "FGSM, ε=8/255 L∞ on the customer-support classifier." A reviewer asks for the threat model. The author answers "white-box." Per chapter 01, what is missing?

- A. Nothing — "white-box" is a complete threat model.
- B. The three-axis rubric requires all three axes: **knowledge** (white / grey / black-box), **capability** (passive / active-querier / training-contributor / supply-chain), and **goal** (targeted / untargeted / availability / extraction). "White-box" alone names only knowledge; the capability axis and the goal axis are unstated, so the success metric is undefined and the attack is not reviewable.
- C. The reviewer needs the perturbation-norm and the ε; those are on the row already.
- D. The threat model must specify the compute budget in GPU-hours.

## 3. A junior engineer argues that because NIST AI 100-2 category and OWASP ML Top 10 entry "say roughly the same thing" for most attacks, the cross-tag sheet can collapse to a single "category/owasp" column to save width. Per chapter 01, what specifically breaks?

- A. Nothing — the two columns are redundant and should be collapsed.
- B. NIST and OWASP answer *different review questions*. NIST is for taxonomy defensibility in front of a regulator or auditor ("is this a recognised attack name?"). OWASP ML Top 10 is for AppSec routing ("which security team owns the mitigation, and whose SDLC does it live in?"). Collapsing them loses the second question's answer; a row with a NIST tag only cannot be routed to an owning team, and a row with an OWASP tag only is weaker in an audit.
- C. The columns differ only in casing; collapsing is fine if you preserve the longer name.
- D. OWASP ML Top 10 is an older list that should be dropped in favour of NIST.

## 4. A team reports "our classifier survives FGSM at ε=8/255" and claims robustness. Per chapter 02, what does the chapter say the claim does and does not establish?

- A. It establishes robustness against first-order white-box attacks at the stated ε.
- B. It establishes that the model is not catastrophically broken at a single point by a single-step attack, and nothing more. FGSM is a floor attack — surviving FGSM only suggests the gradients are usable; it may also indicate *gradient masking* rather than true robustness. The chapter's panel discipline requires PGD at the same ε (with stated steps, step size, and restarts) as the reference first-order attack, plus C&W as the defence-defeat check. A model that "survives FGSM" but has never been tested with PGD and C&W is not a model with an evaluated robustness claim.
- C. It establishes robustness for all ε up to 8/255.
- D. It establishes that the model is adversarially trained.

## 5. A team ships a vision classifier with adversarial training and reports "PGD attack success rate dropped from 78% to 34% — ~44 points of robustness gained." Per chapter 02's panel discipline (and the Carlini-et-al. guidance it cites), what is the specific risk in that headline?

- A. There is no risk; the delta number is what the register wants.
- B. The headline is a perturbation-budget *point*. The chapter requires the primary artifact to be the **perturbation-budget curve** — success rate as a function of ε across the sweep — not a single ε. If the defence only moves the number at the training ε and the curve is flat (or worse) at larger ε, a single-point headline hides the shape. The second risk is that the reported delta is only meaningful if the defence was evaluated under an **adaptive attack** constructed against it; non-adaptive defence evaluations have a track record of being overturned.
- C. The delta is too large and is probably a logging bug.
- D. The team should report attack success rate at ε=0 as well; it will be close to the clean-accuracy complement.

## 6. In the text domain, a team configures TextFooler with no semantic-similarity constraint and reports an attack success rate of 97%. Per chapter 02, what is the specific problem and how is it fixed?

- A. Nothing — higher success rates are better; the attack is strong.
- B. With no semantic-similarity constraint, the "adversarial" text is free to change meaning entirely, which trivially changes the classification — the team has shown that different inputs get different outputs, not that an *adversarial* example evades the classifier. The constraint set (e.g., Universal Sentence Encoder cosine ≥ 0.84) is part of the threat model; a text-attack success rate without the constraint reported alongside is uninterpretable. The fix is to report the constraint set, sweep the constraint severity (at least two or three settings), and keep the strictest setting as the primary reported number.
- C. The problem is that TextFooler is a weaker recipe than BAE; the team should switch.
- D. The issue is that constraint-set is a search hyperparameter and should always be left unset.

## 7. A team's eval tested *label-flipping* poisoning across a poison-rate sweep and produced a defence-delta curve for activation clustering. The team claims this validates the clean-training-pipeline control. Per chapter 03, what does the chapter say is still missing?

- A. Nothing — a label-flipping sweep with a defence-delta is a complete poisoning eval.
- B. Label-flipping is the easy case because the labels disagree with ground truth, so label-consistency checks catch it. Chapter 03 names three *other* attack families that bypass label-consistency checks: targeted data poisoning (dirty-label), backdoor / BadNets triggers (correctly-labelled triggered examples), and clean-label poisoning (Poison Frogs / Bullseye, correctly-labelled examples optimised to shift the decision boundary). Backdoor and clean-label attacks also preserve clean accuracy, so held-out metrics cannot detect them. A "clean pipeline" claim that only tested label-flipping is a one-family claim, not a poisoning claim.
- C. The team should have used spectral-signature detection instead of activation clustering.
- D. The eval should have included the OWASP LLM Top 10 cross-tag.

## 8. A team baselines activation clustering against a BadNets backdoor at a 5% poison rate and reports "detection rate 92% — defence ships." Per chapter 03's measurement rules, what are the specific numbers still missing before this is shippable, and why does chapter 03 call them load-bearing?

- A. Nothing — detection rate at a given poison rate is the shippable number.
- B. The chapter requires four more numbers per defence per attack: **clean FPR** (of clean examples in the same class, what fraction were wrongly flagged), **attack-success-rate with the defence enabled** (does the poison still work after quarantine), **clean-accuracy delta** (does the defence degrade legitimate training), and **latency / cost delta**. A detector with a 5% clean FPR silently quarantines 5% of legitimate training data every release and degrades the model; without the FPR number the defence can be worse than no defence. The clean-accuracy and latency deltas determine whether the defence is *operable* in production, not just technically effective.
- C. The team should also report the attacker's compute budget; the four other numbers are unnecessary.
- D. Detection rate at 5% poison is below 95%, so the defence should be rejected outright.

## 9. A team runs an evasion attack panel, a poisoning-detector baseline, and a membership-inference attack with LiRA-style calibration at 0.1% FPR. Per chapter 04, which of these is in scope for mod-105 at this depth, and which belongs to another module?

- A. All three belong to mod-105 as described.
- B. All three belong to mod-107.
- C. Evasion and poisoning-detector baselining are mod-105; LiRA-calibrated advantage curves at low FPR are mod-107's depth. Chapter 04 is explicit that mod-105 runs enough MIA to establish *that* the model is materially vulnerable and reports baseline advantage; the LiRA-shadow-model-per-record calibration and the formal (ε, δ) DP accounting live in mod-107 (privacy-risk engineering). If the harm model rates MIA as material, mod-105 flags the handoff and mod-107 does the depth work.
- D. All three belong to the `security-learning` / `ai-infra-security` peer at level 35.

## 10. A platform team proposes defending the inference API against extraction and MIA using "per-caller query rate limit + aggregate query-pattern anomaly monitor + top-k-only confidence redaction." Per chapter 04, what does the shared-defence-set table say about which role specifies these, which role implements them, and what the measurement discipline looks like in the robustness-eval report?

- A. The platform team specifies and implements; mod-105's eval is downstream.
- B. Chapter 04 pairs each lever with an owner split: per-caller rate limit and the aggregate-anomaly metric are *specified* by this module (level 25 names the rate value and the metric formula) and *implemented / enforced* by the `security-learning` / `ai-infra-security` peer at level 35 at the API gateway and SIEM. Output redaction is specified by this module and implemented in the serving path. Every lever in the eval report carries a **defence-delta** against the targeted attack — baseline extraction-agreement or MIA advantage vs. agreement / advantage with the lever enabled — and a utility / cost side-effect number. A lever without a measured delta and a measured utility hit is not a shippable control.
- C. Mod-105 both specifies and implements all three levers because they are ML-specific.
- D. The levers are redundant with DP-training and should not both ship.

## 11. A reviewer opens a robustness-eval report and finds (a) methodology, (b) attack success rates with 95% CIs, (c) transferability, and (d) defence deltas. The reviewer flags the report as not-shippable. Per chapter 05, what sections are missing, and why do the missing ones specifically block the hand-off to the risk register?

- A. Nothing is missing — the four sections listed cover a shippable report.
- B. Three sections are still missing: **perturbation-budget curves** (the primary artifact — attack success vs. ε, or vs. poison rate, or vs. query budget, per category — because a single-point number is a slide not a defence), **residual-risk update** (the register-hand-off section that moves scenarios from *estimated* residual to *quantified* residual and populates the control-defeated column with real numbers, each paired with a monitor spec and an incident-response-threshold entry), and **boundary and handoffs** (which parts of the control stack hand to `ai-infra-security` at level 35, which to mod-107 for formal privacy accounting, which to mod-108 for guardrail deployment, which to mod-109 for monitors, which to mod-111 for runbooks). Without the residual-risk section, the register cannot be updated from the report; without boundary and handoffs, the report is a black-box "we did the eval" claim rather than an interface document.
- C. Only the incident-response section is missing.
- D. The report should drop transferability and add a DP-epsilon table instead.

## 12. A new co-owned control is being written into the running system's handoff contract: a STRIP-style runtime backdoor detector in the serving path. Per chapter 06, which of the following is the correct split of responsibilities, and what does the chapter say about the common "double-owned" and "orphan-owned" failure modes?

- A. Mod-105 owns the detector end-to-end — specification, implementation, operation, and evidence.
- B. The ML platform team owns it end-to-end because serving-path code is theirs.
- C. The contract splits by role: this module (level 25) specifies the detector choice and the operating point (detection rate vs. clean FPR from the chapter-03 baseline); the ML platform team implements the detector in the serving path; the ML platform on-call operates it; the residual-risk owner in this module revisits when the clean-FPR drifts. The chapter calls out two boundary failure modes — **double-owned controls** (two roles both believe they own the rate limit, neither's on-call pages when it fails) and **orphan-owned controls** (neither role owns signed-model-artifact provenance, and a supply-chain attack later succeeds through the unsigned-artifact path). The contract table (control / specifier / implementer / operator / evidence) prevents both; a control with a specifier but no named implementer, or an implementer with no operator, is the contract failure the chapter names.
- D. The `security-learning` / `ai-infra-security` peer owns it end-to-end because it ships in production.

---

## Answer key

1. **C.** Chapter 01, "Motivation" and the opening framing on pinning one taxonomy: NIST AI 100-2 e2025 is the taxonomy spine because it is a standards-body document, partitions the attack surface by attacker objective, and is maintained for the current revision; ATLAS and OWASP ML Top 10 are overlays with different framings that answer different review questions and are *not* replacements. The cross-tag sheet carries explicit columns for each, not a merged primary category.
2. **B.** Chapter 01, "The threat model rubric AI 100-2 uses": every technique is described against three axes — knowledge, capability, and goal — and the chapter is explicit that *"'PGD on a customer-support classifier' is not enough; '*white-box, active-querier, untargeted* PGD with L∞ perturbation budget ε=8/255' is a threat model that can be reviewed."* Naming only knowledge leaves capability and goal unstated; the success metric is then undefined.
3. **B.** Chapter 01, "OWASP ML Top 10 as the application-security overlay" and the closing rule *"Do not collapse NIST and OWASP into a single label per scenario. The two labels answer different review questions — NIST for the taxonomy defensibility, OWASP for the AppSec routing."* Losing the OWASP column loses the owning-team routing; losing the NIST column weakens audit defensibility.
4. **B.** Chapter 02, "FGSM — Fast Gradient Sign Method" and "The panel discipline": FGSM is the floor attack; its role is to detect gradient-masking regressions and set a cheap regression sentinel, not to license a robustness claim. PGD at a stated ε / steps / step size / restarts is the minimum bar for a meaningful first-order claim, and C&W is the defence-defeat check. A "we survived FGSM" claim without PGD and C&W is not a claim about robustness.
5. **B.** Chapter 02, "Threat models and perturbation budgets" ("the perturbation-budget curve is the primary output") and "The three classifier-evasion attacks that anchor the panel" (the adaptive-attack rule and the Carlini-et-al. guidance): a single ε is a point on a curve; the curve is the artifact. Defence claims are tested against adaptive attacks constructed to defeat the specific defence.
6. **B.** Chapter 02, "TextAttack as the text-domain runner" and the "constraint sanity checks" discipline: a text attack's constraint set (e.g., USE cosine ≥ 0.84) is part of the threat model; a success rate at no constraint proves only that changing words changes outputs. Reporting the constraint and sweeping its severity is the fix; the strictest setting is the primary reported number.
7. **B.** Chapter 03, "The poisoning-attack taxonomy": four attack families — label flipping, targeted (dirty-label), backdoor / BadNets, clean-label — have different detection surfaces. Backdoor and clean-label attacks preserve clean accuracy; held-out metrics cannot detect them. A clean-pipeline claim that only tested label-flipping is a one-family claim.
8. **B.** Chapter 03, "Measuring a defence — the required numbers": the four numbers beyond detection rate are **clean FPR**, **attack-success with defence enabled**, **clean-accuracy delta**, and **latency / cost delta**. The chapter calls out that "a backdoor detector with a 5% clean FPR that flags 5% of legitimate training data as poison is worse than no defence at all." A detection-rate-only report is a half-baseline.
9. **C.** Chapter 04, "Membership inference (MIA)" and "The privacy handoff to mod-107 — what stays here vs. what goes there": this module baselines MIA advantage and runs DP-SGD as a defence lever to measure the delta; mod-107 owns LiRA-calibrated advantage curves at low FPR, formal (ε, δ) accounting, and the DP-epsilon target decision. Chapter 04 is explicit that if the harm model rates MIA as material, mark the mod-107 handoff in the register.
10. **B.** Chapter 04, "The shared defence set — query-rate, DP-training, output perturbation" (the lever / attacks-addressed / cost / owner table) and chapter 06's handoff-contract shape: this module *specifies* query-rate values and anomaly-metric formulas; level 35 *enforces* at the API gateway and wires the SIEM metric. Every lever ships with a measured delta against the attack it addresses and a measured utility / cost side-effect; the register's residual-risk cells read from those deltas.
11. **B.** Chapter 05, "Sections of the report" (the seven-section list: methodology, attack success rates, transferability, perturbation-budget curves, defence deltas, residual-risk update, boundary and handoffs): the three missing sections are the curves (the primary artifact), the register hand-off (which the report *produces* and the register consumes), and the boundary / handoffs that pin the ownership of each co-owned control. Without the residual-risk section, the register cannot be updated from the report; without boundary and handoffs, the report is a black-box claim rather than an interface document.
12. **C.** Chapter 06, "What is co-owned (and how)" (STRIP-style detector row: mod-105 specifies, ML platform implements and operates, residual-risk owner revisits on FPR drift) and "Motivation" (double-owned vs. orphan-owned failure modes). The handoff contract table — control / specifier / implementer / operator / evidence artifact — is what prevents both failure modes; a row with a missing column is the contract defect the chapter names.
