# Quiz 01 — AI Risk Frameworks and the Risk-Engineer Position

**Format:** 10 multiple-choice questions · **Open-book.** Use the chapters as reference.

Each question tests a *concept* from the module, not phrasing. Answer-key at the bottom cites the chapter the answer comes from.

---

## 1. Which NIST AI RMF 1.0 function does the AI Risk Engineer (level 25) *own*, as opposed to feeding, co-owning, or escalating?

- A. GOVERN
- B. MAP
- C. MEASURE
- D. MANAGE

## 2. How does chapter 01 describe the relationship between AI 100-1 and the NIST AI RMF Playbook for a level-25 engineer's day-to-day work?

- A. AI 100-1 is the day-to-day reference; the Playbook is a historical artifact.
- B. They are equivalent and interchangeable.
- C. AI 100-1 is the treaty; the Playbook is the RFC-style enumeration of sub-categories and suggested actions, and is the day-to-day reference.
- D. The Playbook is deprecated; only AI 100-1 is cited in audits.

## 3. Chapter 01 says the engineering read of an RMF sub-category is five sentences. Which list correctly enumerates all five?

- A. Policy / owner / stakeholder / sign-off / cadence.
- B. Says / means / test / pass / evidence.
- C. Says / owner / cost / risk / threshold.
- D. Definition / rationale / cadence / artifact / sign-off.

## 4. The NIST GenAI Profile (AI 600-1) enumerates twelve generative-AI risks. According to chapter 02, how should the risk engineer use that list?

- A. Pick the subset that is obviously relevant; skip the rest to save cycles.
- B. Treat it as a complete harm model — no further taxonomies required.
- C. Treat it as a *starting checklist*, produce a twelve-row applicability sheet where every risk is explicitly ruled in-scope or out-of-scope with a reason, and re-visit the sheet when scope changes.
- D. Treat it as a ranking of the twelve most important risks across all systems.

## 5. Which of the ISO/IEC AI-family standards in chapter 03 is the *auditable / certifiable* management-system standard whose Annex A lists reference controls?

- A. ISO/IEC 23894
- B. ISO/IEC 42005
- C. ISO/IEC 42001
- D. ISO/IEC 25059

## 6. A governance analyst asks a level-25 risk engineer to author the Statement of Applicability (SoA) for the organisation's ISO/IEC 42001 AIMS. What is the correct response per chapter 03 and chapter 06?

- A. Author it — SoA authorship is a core level-25 deliverable.
- B. Decline and escalate: SoA authorship is owned by senior architecture / governance leadership (level 50+). The level-25 engineer *supplies implementation evidence* for the Annex A controls marked applicable, but does not draft the SoA.
- C. Author it jointly with the ML engineer (level 20).
- D. Author it only if certification is not pursued.

## 7. A single engineering artifact — say, a signed red-team report produced under mod-104 — files against how many of the frameworks in this module, per the table in chapter 03?

- A. Exactly one (NIST AI RMF only).
- B. Exactly two (NIST and ISO).
- C. None — a separate ISO evidence set must be built.
- D. Several at once — e.g. NIST RMF MEASURE 2.7, ISO/IEC 42001 Annex A operational control, ISO/IEC 23894 analysis/treatment evidence, ISO/IEC 42005 robustness section, ISO/IEC 25059 robustness characteristic, and EU AI Act Article 15 cybersecurity report.

## 8. Chapter 04 identifies which EU AI Act article as "where *all* your evaluation-engineering work lands" — the article that consumes accuracy declarations, robustness reports, and cybersecurity reports?

- A. Article 9
- B. Article 10
- C. Article 14
- D. Article 15

## 9. Chapter 04 notes that Article 72 of the EU AI Act is functionally the same requirement as which pair of NIST AI RMF sub-category families?

- A. GOVERN 1.x and GOVERN 5.x
- B. MAP 1.x and MAP 5.x
- C. MEASURE 3.x and MANAGE 4.x
- D. GOVERN 3.x and MANAGE 2.x

## 10. Chapter 05 names one element of the SR 11-7 shape that is *not* just a relabeling of the NIST / ISO shape and genuinely changes how release-assurance is done. Which?

- A. The name of the risk register.
- B. The use of the term "model" instead of "AI system".
- C. The requirement that validation be *independent* of development.
- D. The list of reference controls.

## 11. Chapter 05 describes the FDA Predetermined Change Control Plan (PCCP) as requiring which three elements for an ML-enabled medical-device submission?

- A. A Statement of Applicability, a Model Card, and a Datasheet for Datasets.
- B. A Description of Modifications, a Modification Protocol, and an Impact Assessment for each type of change.
- C. A SR 11-7 validation report, a GMLP crosswalk, and a Colorado AI Act notice.
- D. An EU AI Act Article 61 registration record, an Article 72 post-market report, and an Article 55 systemic-risk evaluation.

## 12. Per chapter 06, which pair is *permanently out-of-scope* for the level-25 AI Risk Engineer role and should be escalated the moment it appears?

- A. Red-team methodology and fairness evaluation.
- B. Risk-register authorship and post-market monitoring wiring.
- C. Legal opinion and SOC-side DFIR / network-incident handling.
- D. Harm-model authoring and guardrail-effectiveness measurement.

---

## Answer key

1. **C — MEASURE.** Chapter 01's function table lists MEASURE as "You *own* this — every sub-category is an evaluation, a threshold, or a monitor." GOVERN is fed, MAP and MANAGE are co-owned.
2. **C.** Chapter 01: "AI 100-1 is the treaty; the Playbook is the RFC. Treat the Playbook — not AI 100-1 itself — as your day-to-day reference."
3. **B — Says / means / test / pass / evidence.** Chapter 01, "What 'reading it as an engineer' looks like in practice": one sentence each for what the sub-category *says*, what it *means* for this system, how we *test* it, what we treat as *pass*, and what we file as *evidence*.
4. **C.** Chapter 02: AI 600-1 is a "checklist your harm model must clear, not the harm model itself"; every system requires a twelve-row applicability sheet; "out" rulings must state a reason and get re-visited when scope changes — silent re-rulings are audit findings.
5. **C — ISO/IEC 42001.** Chapter 03's four-standard table: 42001 (2023) is "Management-system requirements for AI (the auditable, certifiable AIMS)" with Annex A reference controls; 23894 is *guidance*, 42005 is impact-assessment guidance, 25059 is a quality model.
6. **B.** Chapter 03: "You do **not** author the Statement of Applicability (that is a senior-architect / governance-leadership job)." Chapter 06's deferral contract also places SoA authorship at level 50 (`senior-ai-governance-architect`).
7. **D.** Chapter 03's artifact-to-standard table shows the red-team report filing as 42001 Annex A operational control evidence, 23894 analysis + treatment evidence, 42005 impact-assessment robustness section, and 25059 *robustness* characteristic evidence — plus Article 15 cybersecurity-report evidence per chapter 04.
8. **D — Article 15.** Chapter 04: "Article 15 is where *all* your evaluation-engineering work lands" — accuracy declaration, robustness report, cybersecurity report, and traceability table.
9. **C — MEASURE 3.x and MANAGE 4.x.** Chapter 04: "Article 72 and NIST AI RMF MEASURE 3.x + MANAGE 4.x are functionally the same requirement viewed from two governance systems."
10. **C — independence of validation from development.** Chapter 05: "**Independence** is the one thing that is *not* just a label swap. In an SR 11-7 MRM program, the validator cannot report to the developer."
11. **B.** Chapter 05's PCCP section: the PCCP guidance requires a Description of Modifications, a Modification Protocol, and an Impact Assessment for each type of change.
12. **C.** Chapter 06, "What is *out of scope*": the two permanently out-of-scope categories are **legal opinion** (goes to counsel) and **SOC-side DFIR** (goes to security operations). The engineer feeds AI-specific signal in but does not run the SOC or issue legal rulings.
