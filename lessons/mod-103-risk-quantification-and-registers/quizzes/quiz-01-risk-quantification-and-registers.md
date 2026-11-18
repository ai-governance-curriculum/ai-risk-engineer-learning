# Quiz 01 — Risk Quantification, Scoring, and Enterprise AI Risk Registers

**Format:** 12 multiple-choice questions · **Open-book.** Use the chapters as reference.

Each question tests a *concept* from the module, not phrasing. The answer key at the bottom cites the chapter the answer comes from.

---

## 1. Chapter 01 says every shippable register row carries three states. A colleague proposes collapsing residual and control-defeated into one "degraded residual" column to save space. What does chapter 01 say is wrong with that?

- A. Nothing — the two states carry the same information and the collapse is lossless.
- B. The three states share a scoring rubric and must be scored on separate rubrics, so collapsing them would break the rubric citation.
- C. Residual assumes *all* credited controls working; control-defeated scores *one* control failing at a time, worst-case per control. Collapsing them hides the per-control blast radius — which is what sizes the monitoring plan and incident-response thresholds.
- D. ISO/IEC 27005 forbids the collapse by name.

## 2. Chapter 01 names "inherent" as the row's risk *before any risk treatment is applied*. A junior engineer scores inherent as "what the raw model would do with no system around it." What is wrong with that read?

- A. Nothing — raw-model behaviour is exactly what inherent captures.
- B. Inherent should be *system-with-no-risk-treatment*, not *raw-model-with-nothing*. User reach, data sensitivity, and integration surface stay as they are; what is stripped is the risk treatment (filters, validators, monitors, oversight, rate limits).
- C. Inherent should instead be scored as the worst historical incident in AIID for this scenario class.
- D. Inherent should credit the LLM provider's built-in safety training as a control.

## 3. Chapter 01 names two deltas that "carry the audit argument." Which pair is correct?

- A. Inherent − control-defeated and residual − appetite.
- B. Inherent − residual (ROI of the credited controls) and control-defeated − residual (blast radius of one control failing).
- C. Residual − appetite and control-defeated − appetite.
- D. Inherent − appetite and inherent − residual.

## 4. Chapter 02 argues AI risk needs an *exposure* axis separated from likelihood. What specifically is the argument for splitting them?

- A. Exposure is a legal requirement in the EU AI Act; likelihood is not.
- B. Operational controls (rate limits, canary populations, tier restrictions) cap exposure rather than likelihood or impact; aggregation across systems needs exposure explicitly; sector overlays (SR 11-7 materiality, EU AI Act "significant harm", FDA patient population) are exposure-shaped questions. Hiding exposure inside likelihood makes the lever invisible.
- C. Classical IT risk registers already use three axes; keeping exposure separate aligns with their shape.
- D. Likelihood cannot be measured, so exposure is the only numeric axis available.

## 5. A row's likelihood is scored "Medium" with no observation window named. Per chapter 02, what is the defect?

- A. "Medium" is a non-standard level name; it must be "Possible."
- B. Likelihood needs an observation window (annualised, per-release, per-session) picked at the register level and applied consistently, plus a cited prior (incident-corpus base rate, red-team ASR, guardrail effectiveness number, in-system observation). A no-window, no-anchor level is a guess.
- C. Likelihood should never be scored on an ordinal scale; only anchored numeric is defensible.
- D. Likelihood should be collapsed into impact; a separate axis is unnecessary.

## 6. Chapter 03 says reuse ISO/IEC 27005 as the *parent* frame. What role does ISO/IEC 23894 then play in the same process?

- A. 23894 replaces the 27005 process when AI systems are in scope.
- B. 23894 adds parallel *steps* ("AI risk identification," "AI risk evaluation") that run alongside the 27005 steps.
- C. Both 27005 and 23894 specialise ISO 31000, so they share the process shape. 23894 supplies the AI-specific *content* (AI risk sources, consequences, lifecycle stages, control characteristics) that a pure 27005 process would under-cover. The process steps line up 1:1; only the content changes.
- D. 23894 is an audit-side framework only; it has no operational content.

## 7. A team spins up a GRC-for-AI tool and lets the AI register live there, disconnected from the enterprise 27005 security-side register. Per chapter 03, which specific failure mode does that produce?

- A. None — a separate AI register is the recommended pattern.
- B. The risk council reads two heat maps and cannot state the combined enterprise risk; auditors see the same supplier (e.g., LLM API vendor) scored differently on each register; appetite conversations happen twice, differently; incident-response coordination across both registers is manual. This is the "parallel AI risk register" anti-pattern.
- C. The AI register is automatically marked non-certifiable.
- D. ISO/IEC 42001 certification is forfeited.

## 8. Chapter 04 pairs ISO/IEC 25059 and NIST AI RMF MEASURE. What is the correct description of their roles on a register row?

- A. They are interchangeable — pick whichever the auditor prefers.
- B. ISO/IEC 25059 answers "which AI quality characteristic is at stake?" NIST AI RMF MEASURE answers "which measurement outcome closes it?" A row with only one tag is coverage-orphaned; a row needs *at least one of each* so the register says both what is being scored and what evidence pipeline is closing it.
- C. 25059 is a management-system standard; MEASURE is a quality model. They do not compose.
- D. 25059 replaces MEASURE for ISO-aligned organisations; MEASURE applies only to NIST-aligned ones.

## 9. Chapter 05 calls "sum-of-averages" a portfolio-reporting anti-pattern. Specifically, why is averaging residuals across a system's rows misleading?

- A. Averaging is correct; the "sum-of-averages fallacy" is only about summing after averaging.
- B. Averaging requires a numeric rubric, and most registers use ordinal rubrics.
- C. A system with fifty medium rows and one severe row averages to "medium" — the severe row is invisible. The tail is where decisions live; portfolio numbers should be max, count, and top-N, never mean.
- D. Averaging inflates portfolio numbers; using max avoids the inflation.

## 10. Chapter 05 names six standard reports the risk council reads. Which of the following is *not* one of the six?

- A. Portfolio residual heat map.
- B. Above-appetite exception list.
- C. Control-defeated priority list.
- D. Monthly LLM-vendor pricing comparison.

## 11. Chapter 06 says the register carries three distinct ownership contracts per row. Which trio is correct?

- A. Row author · product manager · legal.
- B. Row author · system owner · control owner(s). None should silently collapse at any real scale; when they do overlap (small deployment), the overlap is *recorded* rather than hidden.
- C. Row author · security partner · regulator.
- D. System owner · risk council · auditor.

## 12. Chapter 07 pins the peer boundary with `model-evaluation-engineer` (L30). A vendor scorecard reports "97% accuracy on our safety benchmark" with no CI, no sample size, and no judge-vs-human calibration. Per chapter 07, what is the level-25 risk engineer's correct move?

- A. File the number as a residual-likelihood anchor; vendor-supplied numbers are authoritative.
- B. Compute an in-house bootstrap CI and judge-vs-human κ to fill the gaps, then file.
- C. Refuse to credit the number as a calibration anchor on its own. File a calibration request at the L25↔L30 seam; in the meantime, either consume the vendor number under an explicit internal review (naming the reviewer) with pessimistic-choice conventions (CI-upper-bound for adverse likelihood, CI-lower-bound for guardrail credit) or drop the row into the uncalibrated-rows exclusion list.
- D. Credit the number only if the vendor is on the enterprise's approved-supplier list.

---

## Answer key

1. **C.** Chapter 01, "The three states" and "Control-defeated — the 'one control at a time' score": residual assumes all credited controls working; control-defeated scores one named control's failure, worst-case per control. Collapsing them hides the per-control blast radius the monitoring plan (mod-109) and incident-response thresholds (mod-111) must be sized against.
2. **B.** Chapter 01, "Inherent — the 'no controls' score": "Inherent is not 'raw model behaviour.' It is the risk to the *system in operation with no risk-treatment controls*. So user reach, data sensitivity, and integration surface are still what they are — you do not pretend the system does not exist. What you strip is the *risk treatment*." The "raw-model-only inherent" shape is also called out explicitly as an anti-pattern.
3. **B.** Chapter 01, "The two deltas that matter": inherent − residual argues the ROI of the credited controls; control-defeated − residual argues the blast radius of a single control failing. Both are what portfolio reports (chapter 05) roll up, not just the residual number.
4. **B.** Chapter 02, "Exposure — the scale reached when the scenario occurs" lists the three reasons: operational controls act on exposure, aggregation needs it explicit, sector overlays are exposure-shaped questions.
5. **B.** Chapter 02, "Likelihood — the probability of occurrence": "No window" and "No prior" are the two mistakes called out explicitly, with the fix being an annualised (or equivalent) window and a cited anchor (incident-corpus base rate, red-team ASR, guardrail effectiveness, in-system observation).
6. **C.** Chapter 03, "What ISO/IEC 23894 adds without breaking the shape": both specialise ISO 31000, so the process steps line up 1:1; 23894 adds AI-specific risk sources, consequences, lifecycle stages, and control characteristics as *content* within the 27005 process steps.
7. **B.** Chapter 03, "An anti-pattern: the parallel AI risk register" enumerates these consequences by name — two heat maps, duplicated suppliers with different residuals, duplicated appetite conversations, and manual cross-register coordination during incident response.
8. **B.** Chapter 04, "ISO/IEC 25059 as the quality dimensions" and "Composing MEASURE with 25059": 25059 says *what quality characteristic is at stake*; MEASURE sub-categories say *which measurement outcome closes it*. "A row with only a 25059 tag reads as 'we care about robustness for this scenario' but doesn't say what evidence closes it. A row with only a MEASURE tag reads as 'we ran evidence for MEASURE-2.7' but doesn't say which quality characteristic is being asserted. Both together read defensibly."
9. **C.** Chapter 05, "Motivation" (sum-of-averages fallacies) and "Anti-patterns" (Averaging): "A portfolio 'average residual' is meaningless; the tail is where decisions live. Portfolio numbers are max, count, and top-N, never mean."
10. **D.** Chapter 05, "Aggregation reports the risk council actually reads": the six are portfolio residual heat map, above-appetite exception list, control-defeated priority list, shared-component report, coverage-grid completeness report, and regulatory-jurisdiction exception list. A vendor pricing comparison is not one of them.
11. **B.** Chapter 06, "Ownership contract": three ownership contracts per row — row author, system owner, control owner(s). "None of the three should be the same person on a system operating at any real scale. If two overlap (a small deployment where the risk engineer is also the system owner), *record that fact*; do not silently collapse."
12. **C.** Chapter 07, "What the risk engineer *uses*", the "point-estimate laundering" anti-pattern, "What happens when the L30 role does not exist", and "The consumption contract": refuse uncalibrated numbers; file a calibration request; in the interim either consume vendor calibration under a named internal review with pessimistic-choice conventions (CI-upper-bound for adverse likelihood, CI-lower-bound for credited guardrail effect) or drop the row from aggregation. Running an in-house bootstrap (option B) is explicitly called out as the "in-house bootstrap by a non-statistician" anti-pattern.
