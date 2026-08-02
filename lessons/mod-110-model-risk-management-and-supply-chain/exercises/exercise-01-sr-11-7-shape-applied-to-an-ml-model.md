# Exercise 01 — SR 11-7 Shape Applied to an ML/AI Model

**Estimated effort:** 3 hours

## Objective

Take the **primary running system** you have carried since mod-101/mod-102 and assemble a single **MRM validation packet** in the shape SR 11-7 / OCC 2011-12 demands. The packet is a folder of files, one per MRM heading; every heading is filled with an engineering artefact you already produced in mods 102-109, or is called out as a gap. The deliverable is the packet a Federal Reserve or OCC examiner would open first if your system were in scope of a US financial-institution examination tomorrow.

## Prerequisites

- Chapter 01 read in full. The four-pillar model, the packet table of contents, and the "packaging failure vs engineering failure" distinction are the reference shapes.
- Chapter 02 skimmed — the third-party components section of the packet will point at the SR 23-4 file exercise 02 produces; you do not need to fill it in this exercise, but you must reserve the slot.
- Access to the mod-102 harm model, the mod-103 register (three-state — inherent, residual, control-defeated), the mod-104 red-team suite output, the mod-105 adversarial evaluation, the mod-106 fairness evaluation, the mod-107 privacy evaluation, the mod-108 guardrail catalogue, and the mod-109 monitor spec for the primary running system.
- A pointer to whoever plays the *independent validator* role in your organisation — an L35 evaluation engineer, an internal audit contact, or (if no such role exists) a named peer who will act as the validator for the purpose of this exercise. Independence of validation from development is one of the four pillars and it must be visible on the packet.
- If you are not at a US financial institution, pick a system whose *decisions* would matter to an MRM shop if they were — a credit-tier-adjacent classifier, a customer-service RAG assistant that touches account information, a fraud-triage prioritiser. The shape transfers even when the regulator does not sit above you.

## Problem statement

Assemble the MRM validation packet for the primary running system. The packet is a folder with a fixed table of contents (chapter 01 §"MRM-shaped validation documentation for an ML/AI model"). Every heading is one of:

- **Filled** — points at a concrete engineering artefact with an owner and a revision hash.
- **Deferred** — a placeholder file naming the artefact that will fill the slot, who owns it, and the target date.
- **Not applicable** — a written justification for why this heading does not apply to this system (e.g., "no adaptive re-training in scope; change-management heading captured by the platform's normal release process, not a PCCP").

Silent blanks are not acceptable. Auditors read the empty cells first.

### The packet layout

```
mrm-packet/<system-id>/<packet-version>/
  README.md                              # index + pillar-coverage summary
  01-model-description-and-purpose.md
  02-tiering-and-risk-rating.md          # cites mod-103 register rows
  03-data.md                             # cites mod-110 chapter 04 Data Cards
  04-methodology-and-conceptual-soundness.md
  05-assumptions-and-limitations.md
  06-development-testing/
    red-team-report.md                   # mod-104
    adversarial-eval-report.md           # mod-105
    fairness-eval-report.md              # mod-106
    privacy-eval-report.md               # mod-107
  07-independent-validation-memo.md      # signed by the validator
  08-implementation-testing.md           # mod-108 guardrail integration + canary
  09-ongoing-monitoring-plan.md          # mod-109 monitor spec
  10-change-management.md
  11-third-party-components/
    README.md                            # points at exercise-02 output per vendor
  12-governance-sign-off.md
  attestations/
    packet.manifest.json                 # hash + owner per file in the packet
    packet.signature.sig                 # cosign signature per chapter 05 (optional but preferred)
```

### The four-pillar coverage rubric

The packet's `README.md` opens with a four-row table — one row per pillar — that resolves each pillar to the specific file(s) in the packet that evidence it, and to the specific engineering practice that makes the evidence credible.

| Pillar | File(s) in this packet | Practice that makes it credible |
|---|---|---|
| Independence of validation from development | | |
| Conceptual soundness | | |
| Ongoing monitoring (incl. outcome analysis / back-testing) | | |
| Effective challenge | | |

A pillar row that cites zero files or that cannot name the practice is a pillar the packet does not evidence. The validator's memo (file 07) will say so.

## Requirements

Produce all six items:

1. **The packet folder** — every file in the layout above either exists with real content, is a deferred placeholder that names the owner and target date, or is a not-applicable justification file. No blanks.
2. **The four-pillar coverage table** — inside `README.md`, filled per the rubric above. Every pillar cites at least one file. Pillars that cannot cite a file are flagged as gaps and appear in the gap list (item 5).
3. **The heading-to-artefact map** — a table inside `README.md` mirroring chapter 01 §"MRM-shaped validation documentation for an ML/AI model", with the file path and the mod-101-through-mod-109 artefact each heading consumes. Empty cells are named as gaps.
4. **The independent-validation memo** (`07-independent-validation-memo.md`) — authored by someone other than the model developer, one page maximum, structured as: scope, evidence reviewed, pillar-by-pillar assessment (pass / conditional / fail with reason), material findings, and validator's disposition (approve / conditional / reject). Even a dry-run memo where the validator is a peer must be present in this exercise.
5. **The gap list** — a table at the end of `README.md`: MRM heading × missing artefact × owner × target date. Every gap corresponds to a follow-up item that will land in the mod-103 register as an operational risk on the model.
6. **The change-management stub** (`10-change-management.md`) — a one-page description of how a change to this model will be routed. If the system is (or could be) a SaMD-eligible device, the stub points at the PCCP that exercise 03 will draft. Otherwise the stub points at the FI's model-change policy (or the placeholder policy your team follows in its absence). Silent "we deploy on Fridays" is not a change-management artefact.

## Starter guidance

- **Fill the easy headings first.** *Model description and purpose*, *data*, *development testing*, *ongoing monitoring plan* — you already produced these in mods 101, 102, 104-109. The exercise is largely re-pointing existing artefacts, not writing new evidence.
- **Independence is a practice, not a document.** The independence pillar is not evidenced by having a document titled "Independence Attestation" — it is evidenced by *who owns the evaluation suite definitions* and *who signs the validator memo*. If the same person writes the tests, sets the thresholds, runs them, and signs the memo, the pillar fails no matter how the memo is worded.
- **Conceptual soundness is where mod-102 pays off.** Every design choice in the methodology file traces to a harm-model row it targets. If the file describes the *technique* (transformer with LoRA on 8x A100s) without describing the *harm-model rows the design is meant to resist*, the reviewer cannot judge conceptual soundness.
- **Ongoing monitoring is not just "we have dashboards."** The `09-ongoing-monitoring-plan.md` file must be the mod-109 monitor spec: metric name, threshold, cadence, alert routing, action-on-fire, evidence-pointer. A screenshot of Grafana is not a monitoring plan; a signed spec is.
- **Effective challenge is evidenced by disagreement in the record.** Include the mod-104 red-team fixtures that were *not* written by the model team, the mod-103 register rows that were *not* volunteered by the developer, and the mod-111 post-mortem "what did we miss" sections owned by someone other than the developer. If every artefact was written by the developer, effective challenge is missing.
- **Deferred is a legitimate status; silent is not.** A heading whose artefact does not exist yet gets a placeholder file that says *what* is missing, *who* owns it, and *when* it is expected. The validator will accept a deferred packet with a credible timeline; they will not accept a blank.
- **The packet is versioned.** `<packet-version>` is a semantic version. Every material change to any file in the packet triggers a version bump and a change-log entry. Do not treat the packet as a shared drive.
- **Cite mod-103 register rows by ID.** The tiering-and-risk-rating file (`02-tiering-and-risk-rating.md`) must reference the exact register row IDs, not a paraphrase. Traceability from packet to register is a first-order MRM expectation.

## Acceptance criteria

The submission is accepted when:

- Every file in the layout above is present as content, deferred placeholder, or not-applicable justification. No silent blanks.
- The four-pillar coverage table in `README.md` cites at least one file per pillar; unfilled pillar cells appear in the gap list.
- The heading-to-artefact map cites the mod-101-through-mod-109 artefact each heading consumes; empty cells are named as gaps with an owner.
- The independent-validation memo is signed by someone other than the model developer and gives a pillar-by-pillar assessment plus a disposition.
- The gap list captures every empty or deferred cell with an owner and a target date; each gap has a corresponding mod-103 register row (either existing or newly added in this exercise).
- The change-management stub names the routing (PCCP-eligible path or the FI's model-change policy) and does not consist solely of a release cadence.
- A reader with no prior context can open `README.md` and, in under five minutes, answer: which pillars are evidenced, which are gapped, which register rows carry the risk of the gaps, and who owns closing each gap.

## Stretch goals

- **Packet-linter script** — write a small script (`mrm-packet/lint.py` or equivalent) that walks the packet folder, checks the file layout against the required table of contents, reads `README.md`'s four-pillar table and heading-map, and fails CI if any pillar has zero files, any heading has no artefact and no gap entry, or any gap has no owner or target date. Running this in CI blocks silent regressions.
- **Cross-track dry-run review** — hand the packet to a peer L25 (from `ai-infra-mlops` or `ai-eval-engineer`) and collect a written review. Note where they read the packet in a different order than you expected, which files they could not find, and which pillar assessments they disagreed with. Their disagreement *is* effective challenge.
- **Register-side back-link** — in the mod-103 register file, add an `mrm_packet_pointer` field per row that cites the packet version and file the row is credited in. This is the two-way lookup that turns "the register" and "the packet" into a single evidence graph rather than two documents that drift.
- **Sign the packet** — sign `packet.manifest.json` with cosign per chapter 05 (or reserve the slot for after exercise 05). A signed packet is what makes "this is the packet at version X" verifiable to the validator without trusting the folder's mtime.

## Deliverable format

A single markdown document `mod-110-ex01-mrm-packet.md` that indexes the packet. The index contains:

1. *System and packet summary* — one paragraph naming the running system, the packet's semantic version, and the mod-103 register version it is bound to.
2. *Pillar coverage* — the four-row pillar table quoted verbatim from `README.md`.
3. *Heading-to-artefact map* — the table quoted verbatim from `README.md`.
4. *Validator memo excerpt* — the disposition line and any material findings from `07-independent-validation-memo.md`.
5. *Gap list* — the gap table with owners and target dates.
6. *Cross-references* — pointers to the mod-103 register rows the packet references and to the placeholder(s) reserved for exercise 02 (third-party components) and exercise 04 (Data Cards / ML-BOM / SLSA).
7. *Open items* — anything blocked on a peer or platform team.

## Submission

Attach the index markdown plus the packet folder (redacted if the running system is under NDA, but preserving the pillar table, heading map, and file-layout structure). The packet is the load-bearing artefact for the module — exercises 02, 04, 05, and 06 all extend it. Treat it as the folder you would hand an examiner on the first day of an on-site review, because that is precisely what SR 11-7 asks it to be.
