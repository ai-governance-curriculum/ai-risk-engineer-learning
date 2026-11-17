# Quiz 01 — Harm Modelling and Risk Taxonomy Engineering

**Format:** 11 multiple-choice questions · **Open-book.** Use the chapters as reference.

Each question tests a *concept* from the module, not phrasing. The answer key at the bottom cites the chapter the answer comes from.

---

## 1. Chapter 01 names the four misuse personas that make up *floor coverage* for a level-25 harm model. Which of the following is that floor set?

- A. Adversarial user · benign user · insider · regulator.
- B. Adversarial user · insider · upstream data supplier · downstream integrator.
- C. Script kiddie · insider · model vendor · auditor.
- D. Red-teamer · insider · vendor · end user.

## 2. A colleague proposes that the harm model and the risk register are the same artifact, because both list risks. What does chapter 01 say about the distinction?

- A. They are the same artifact; the "risk register" name is legacy.
- B. The harm model is the risk register plus evidence references.
- C. The harm model enumerates scenarios and evidence; the register (mod-103) quantifies likelihood, severity, and residual risk. They feed each other but are distinct files.
- D. The harm model is a draft of the risk register until it is signed.

## 3. Chapter 02 says a defensible harm-model row cites at least one source from two categories. Which pairing is correct?

- A. AIID *or* OECD.AI AIM — *plus* MIT AI Risk Repository.
- B. OWASP LLM Top 10 — *plus* MITRE ATLAS.
- C. Any one incident corpus; a single source is sufficient.
- D. AIR-Bench 2024 — *plus* NIST AI 600-1.

## 4. Your harm-model row describes a scenario your system is genuinely novel for — no AIID or OECD.AI incident matches. What does chapter 02 say to do?

- A. Drop the row; unevidenced scenarios do not ship.
- B. Cite the strongest literature match (MIT AIR row, published paper, OWASP category), mark the row's `confidence` as *literature-only*, and set a *watch* trigger so the row is upgraded when a matching incident lands.
- C. Invent a plausible incident ID and note "pending publication."
- D. Cite only OECD.AI Principles and call the row evidenced.

## 5. Chapter 03 requires every shippable harm-model row to carry tags from four overlays. Which set is correct?

- A. NIST AI 100-1 · ISO/IEC 42001 Annex A · AVID · MITRE ATT&CK.
- B. NIST AI 600-1 (GenAI Profile) risk # · ISO/IEC 23894 risk sources · AIRO ontology classes · AIR-Bench 2024 policy-derived category.
- C. OWASP LLM Top 10 · MITRE ATLAS · AIID taxonomy · OECD Principles.
- D. NIST AI RMF function · ISO/IEC 42005 clause · EU AI Act article · SR 11-7 pillar.

## 6. Chapter 03 explains why AIRO ontology tags matter in a way that none of the other three overlays replicate. What is AIRO's specific value?

- A. It is the only overlay that maps to the EU AI Act article numbers.
- B. It assigns severity scores to each harm so the register does not have to.
- C. It is a *machine-readable* target — a W3C-compatible ontology that turns the Markdown table into a data structure a broader governance / risk-graph platform can query across systems.
- D. It replaces the need for incident-corpus evidence because the ontology is authoritative.

## 7. Chapter 04 says the frontier-lab documents (Anthropic RSP, OpenAI Preparedness, DeepMind FSF) are *shape templates* for enterprise deployment tiering. What is the reader supposed to lift, and what are they supposed to leave?

- A. Lift the specific capability thresholds (CBRN uplift, autonomous replication); leave the governance ceremony.
- B. Lift the *shape* — pre-committed tiers, pre-triggered safeguards, named decision authority, named evaluation cadence — and leave the frontier-specific thresholds, the non-public evaluation harnesses, and the developer's external commitments.
- C. Lift the full policy text verbatim and rename "Anthropic" to the enterprise.
- D. Lift the evaluation suites; leave the threshold language.

## 8. When reading the three frontier risk-tier frameworks side by side, chapter 04 gives six comparison questions. Which of these is *not* one of the six?

- A. What capability categories does it track?
- B. What safeguards attach at each threshold?
- C. What is the frontier lab's annual revenue from the model family?
- D. How is the framework itself updated?

## 9. Chapter 05 says attribution to a `cause_bucket` should follow one specific test when a scenario touches multiple buckets. Which test?

- A. Pick the bucket that is cheapest to staff.
- B. Pick the bucket owned by the loudest reviewer.
- C. Attribute to the bucket where *removing the cause would end the harm at least cost* — the intervention that ends the scenario cheapest — and record that reasoning in the row.
- D. Always attribute to the operator bucket; it keeps the fix in-house.

## 10. A harm scenario is caused by a third-party model vendor's behaviour and only the vendor can ship the fix. Chapter 05 says the attribution should be:

- A. Re-attributed to the operator bucket so the fix stays in-house.
- B. Attributed to the correct (model / data / etc.) bucket regardless; add an internal *residual owner* responsible for carrying the residual until the vendor ships the fix or you route around them, and record the third-party dependency for mod-110.
- C. Dropped from the harm model because external fixes are out of scope.
- D. Attributed to "integration" because the third party is downstream.

## 11. Per chapter 06, which of the following is a scenario that the level-25 harm modeller *names* in the harm model but *escalates* to `agentic-safety-engineer` (L40) rather than resolving?

- A. An insider who rewrites the system prompt without review.
- B. An indirect prompt-injection scenario via retrieved KB content.
- C. A multi-step autonomous-agent scenario that requires dangerous-capability evaluation of agent traces at depth (goal-directed behaviour outside the instruction envelope, not just "the agent called the wrong tool").
- D. A fairness scenario on a customer-support classifier.

---

## Answer key

1. **B.** Chapter 01, "Misuse personas — the four you must cover": adversarial user, insider, upstream data supplier, downstream integrator. "If any of the four is absent from your harm model, an auditor will ask why."
2. **C.** Chapter 01, "What a good harm model is *not*": "The harm model enumerates scenarios and evidence; the register (mod-103) quantifies likelihood, severity, and residual risk. The two artifacts feed each other but are not the same file."
3. **A.** Chapter 02, "How the three combine": "A defensible harm-model row cites at least one of AIID / OECD.AI *plus* MIT AI Risk Repository." Literature-only rows are legitimate but must be marked as such.
4. **B.** Chapter 02, "What to do when no incident matches": cite the closest AIID / OECD.AI near-analog if any, cite the strongest MIT AIR / literature row as the *pattern* citation, mark `confidence: literature-only`, and set a *watch* trigger so the row is upgraded when a matching incident lands.
5. **B.** Chapter 03, "The four overlays this module requires": NIST AI RMF Generative AI Profile (AI 600-1) risks, ISO/IEC 23894 risk sources, AIRO ontology, AIR-Bench 2024 policy-derived categories.
6. **C.** Chapter 03, "Overlay 3 — AIRO ontology": AIRO is "the machine-readable target for your row tags … what turns your harm model from a Markdown table into a data structure a broader governance platform can query, aggregate across systems, and reason about at portfolio scale."
7. **B.** Chapter 04, motivation and "Lifting the shape to your enterprise deployment tiering": lift pre-committed tiers / pre-triggered safeguards / decision authority / evaluation cadence; the "Where you *don't* copy from the frontier docs" section explicitly rules out frontier capability thresholds, non-public evaluation harnesses, and the developer's external commitments.
8. **C.** Chapter 04, "The comparative read — six questions to ask each document": categories tracked, threshold definition, attached safeguards, decision authority, deployment consequence, and framework-update mechanism. Revenue is not one of the six.
9. **C.** Chapter 05, "Picking a bucket cleanly": "Attribute to the one where *removing the cause would remove the harm at least cost.*" And: "Write that reasoning into the row alongside the bucket tag — the reviewer will read it."
10. **B.** Chapter 05, "When the mitigation owner is *outside* the organisation": attribute to the correct bucket regardless; add a residual owner on your side; record the third-party dependency for mod-110 (MRM + supply-chain) to pick up.
11. **C.** Chapter 06, "Downstream: what you hand off to `agentic-safety-engineer` (L40)", item 2 "Multi-step autonomous agent scenarios": dangerous-capability evaluation of agent traces at depth escalates; the L25 harm model *names* the risk and cites the escalation. The other options are in-scope L25 work (operator attribution, integration-caused injection, fairness evaluation).
