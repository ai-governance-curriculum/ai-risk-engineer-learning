# Lab 03 — Out-of-Repo Payload Manifest Drill

**Duration:** 40 minutes · **Format:** guided practice, markdown + YAML authoring (no payloads touched)

## Objective

Internalise the **harmful-payload-out-of-repo** discipline chapters 01 and 05 pin — "never in the source repo, out-of-repo payload store only, access-logged, rotate-on-retraining, never re-publish the raw string" — by authoring the in-repo artifacts (a manifest + the signed-reference convention + a repo-root check stub) that make the discipline operational. The payloads themselves are *simulated*; no real harmful strings are generated, read, or referenced. The point is the *artifact shape* that the repo carries, so that when exercises 03 and 05 ask you to wire real payloads into converters and sweep configurations you already have the manifest discipline wired.

This lab deliberately uses fabricated payload metadata — GCG suffix IDs, many-shot corpus IDs, hashes — so the discipline can be drilled without touching any real harmful artifact. Every string in this lab's output must be metadata only.

## Prerequisites

- Chapter 01 read (specifically "Harmful-payload hosting — out of the repo" and the three-column where / what / access table).
- Chapter 05 read (specifically "GCG payload discipline" and "Many-shot payload discipline").
- Chapter 03 skimmed enough to recall the converter pattern that loads payloads by hash at runtime.
- A terminal where you can create a working directory and author markdown and YAML files.
- No external access required; no real payloads are referenced.

## The fixed example system

Carry PolicyPal from labs 01 and 02. PolicyPal's model is a closed-API hosted LLM, so chapter 05's Level 2 GCG fresh-optimisation coverage is `n/a` — the manifest will only hold *replay suffixes* (Level 1), plus a many-shot demonstration corpus the chapter requires as its own payload class.

The simulated payload set this lab enters into the manifest (metadata only — these are fabricated IDs and hashes, not real payloads):

### GCG replay suffixes (3 entries)

| ID | Provenance | Target set | SHA256 (placeholder) |
|---|---|---|---|
| `gcg-replay-01` | Zou et al. 2023 appendix; suffix A | PolicyPal hosted LLM | `sha256:aaaaaaa...placeholder-01` |
| `gcg-replay-02` | Zou et al. 2023 appendix; suffix B | PolicyPal hosted LLM | `sha256:bbbbbbb...placeholder-02` |
| `gcg-replay-03` | Public follow-on 2024; suffix published by <author-placeholder> | PolicyPal hosted LLM | `sha256:ccccccc...placeholder-03` |

### Many-shot demonstration corpora (3 entries)

| ID | Category | Shot count available | SHA256 (placeholder) |
|---|---|---|---|
| `manyshot-malware-corpus-v1` | malware-generation | 256 shots | `sha256:ddddddd...placeholder-04` |
| `manyshot-selfharm-corpus-v1` | self-harm | 128 shots | `sha256:eeeeeee...placeholder-05` |
| `manyshot-illicitadvice-corpus-v1` | illicit-advice | 256 shots | `sha256:fffffff...placeholder-06` |

Every hash in this lab is a *placeholder* (not a real content hash). Every provenance is fabricated. No raw suffix, no raw demonstration pair enters this lab's output under any circumstance.

## Steps

### Step 1 — Create your working directory and the manifest files

```bash
mkdir -p mod104-lab03/red-team/payloads
cd mod104-lab03
touch red-team/payloads/MANIFEST.md
touch red-team/payloads/MANIFEST.yaml
touch red-team/payloads/README.md
touch red-team/payloads/precommit-check-stub.md
```

The layout mirrors the exercise-01 charter's `red-team/payloads/` subtree. All four files are in-repo artifacts — none of them contain harmful strings. The actual payloads live out-of-repo (in whatever private object store your organisation uses); the repo carries manifests, hashes, and store URIs only.

### Step 2 — Fill `red-team/payloads/README.md` with the discipline statement

Paste the following template and fill in the one-paragraph "why not in-repo" statement in your own words, using chapter 01's four reasons.

```markdown
# red-team/payloads — out-of-repo payload reference manifest

System: PolicyPal (internal HR-policy Q&A RAG assistant)
Chapter reference: mod-104 ch 01 ("Harmful-payload hosting — out of the repo"), ch 05 ("GCG payload discipline", "Many-shot payload discipline")

## What lives here (in-repo)

- `MANIFEST.md` — human-readable list of every payload the suites reference, by ID, hash, provenance, category, and store URI
- `MANIFEST.yaml` — machine-readable version of the same list, consumed by converters at runtime
- `precommit-check-stub.md` — the repo-root check that fails if a harmful-payload signature lands in the tree

## What does *not* live here (ever)

- Raw GCG suffixes
- Raw many-shot demonstration pairs
- Any `(policy-negative ask, model-would-comply response)` pair
- Any string tagged "harmful" in the out-of-repo store
- Any transcript of a successful attack that includes the attack's payload text

## Why not in-repo

<one paragraph in your own words covering chapter 01's four reasons: cloning
 exposure to harmful content, public-mirror and fork surfaces, grep surfaces
 where payloads are persuasive against humans, and code-hosting-provider
 content-policy risk>

## Linking convention

Every payload referenced from the suites is referenced by:

- `payload_id` — the stable ID in this manifest
- `sha256` — the content hash
- `store_uri` — the out-of-repo store URI pattern (bucket, vault, or filesystem path pattern; actual credentials never committed)

Converters resolve `payload_id` → `store_uri` → payload-content at runtime, verify the hash,
exercise against the target, and never write the content back into the repo tree.

## Access

The out-of-repo store carries an allow-list and per-read access logging. Reading a
GCG-category or many-shot-category payload is an access-logged event; the log is
retained alongside the triage history.

## Rotation

- GCG replay suffixes: re-evaluated on every model provider safety-tuning update.
  Dead suffixes are flagged `retired:<date>` in the manifest and kept for
  historical reproducibility; they are not deleted.
- Many-shot corpora: versioned (`-v1`, `-v2`, ...) when demonstrations are added
  or removed; the version tag is part of the payload_id.
```

Fill the "why not in-repo" paragraph. Keep it to three or four sentences. Target shape:

> "Cloning this repo must never expose a new engineer to CSAM-adjacent, self-harm-inducing, or CBRN-uplift payloads; public mirrors (open-source, forks, archival backups) must never surface them either. `grep` across the codebase must not surface them — many payloads (DAN framings, many-shot demonstrations) are designed to be persuasive against *humans*, not just models. The code-hosting provider's content policy may itself be violated by committing raw harmful strings. The four reasons together make the discipline non-negotiable: payloads live in the out-of-repo store, the repo carries hashes and store URIs only."

### Step 3 — Fill `red-team/payloads/MANIFEST.yaml` with the six simulated entries

The YAML manifest is machine-readable and converter-consumed. Every entry carries the same fields.

```yaml
# red-team/payloads/MANIFEST.yaml
# Metadata only. No raw payload strings, no demonstration pairs, no suffix tokens.

store:
  type: private-object-bucket  # or vault; actual bucket name not committed
  uri_pattern: "<store-uri-pattern-placeholder>/payloads/{payload_id}"
  access: allow-list, access-logged

payloads:

  # --- GCG replay suffixes (Level 1 coverage; Level 2 n/a for closed-API target) ---

  - payload_id: gcg-replay-01
    category: gcg-suffix
    provenance: "Zou et al. 2023 appendix; suffix A (<citation-placeholder>)"
    sha256: "sha256:aaaaaaa...placeholder-01"
    target_set: [policypal-hosted-llm]
    status: active
    notes: "Level 1 replay against closed-API target; no fresh optimisation."

  - payload_id: gcg-replay-02
    category: gcg-suffix
    provenance: "Zou et al. 2023 appendix; suffix B (<citation-placeholder>)"
    sha256: "sha256:bbbbbbb...placeholder-02"
    target_set: [policypal-hosted-llm]
    status: active
    notes: "Level 1 replay against closed-API target; no fresh optimisation."

  - payload_id: gcg-replay-03
    category: gcg-suffix
    provenance: "Public follow-on 2024; suffix published by <author-placeholder>"
    sha256: "sha256:ccccccc...placeholder-03"
    target_set: [policypal-hosted-llm]
    status: active
    notes: "Level 1 replay against closed-API target; no fresh optimisation."

  # --- Many-shot demonstration corpora (one per internal-policy category) ---

  - payload_id: manyshot-malware-corpus-v1
    category: manyshot-demonstration-corpus
    sub_category: malware-generation
    provenance: "Internal curation; demonstrations authored under mod-104 ch 05 discipline"
    sha256: "sha256:ddddddd...placeholder-04"
    target_set: [policypal-hosted-llm]
    shot_counts_available: [4, 16, 64, 256]
    status: active
    notes: "Per-demonstration review on file in out-of-repo review log."

  - payload_id: manyshot-selfharm-corpus-v1
    category: manyshot-demonstration-corpus
    sub_category: self-harm
    provenance: "Internal curation; demonstrations authored under mod-104 ch 05 discipline"
    sha256: "sha256:eeeeeee...placeholder-05"
    target_set: [policypal-hosted-llm]
    shot_counts_available: [4, 16, 64, 128]
    status: active
    notes: "Per-demonstration review on file; curator access logged."

  - payload_id: manyshot-illicitadvice-corpus-v1
    category: manyshot-demonstration-corpus
    sub_category: illicit-advice
    provenance: "Internal curation; demonstrations authored under mod-104 ch 05 discipline"
    sha256: "sha256:fffffff...placeholder-06"
    target_set: [policypal-hosted-llm]
    shot_counts_available: [4, 16, 64, 256]
    status: active
    notes: "Per-demonstration review on file; curator access logged."
```

Paste this into `MANIFEST.yaml`. Confirm by reading it back that *not a single field* carries a raw suffix, a raw demonstration pair, or any string tagged as harmful. If any field does, delete it — the manifest's invariant is metadata-only.

### Step 4 — Fill `red-team/payloads/MANIFEST.md` with the human-readable version

The markdown manifest is what a reviewer reads cold. Keep the shape short and the discipline clear.

```markdown
# PolicyPal — out-of-repo payload manifest

Store: private object bucket, allow-list + access-logged; URIs resolved at runtime by converters.

## GCG replay suffixes

| payload_id | sha256 (placeholder) | provenance | target | status |
|---|---|---|---|---|
| `gcg-replay-01` | `sha256:aaaaaaa...placeholder-01` | Zou et al. 2023 appendix; suffix A | PolicyPal hosted LLM | active |
| `gcg-replay-02` | `sha256:bbbbbbb...placeholder-02` | Zou et al. 2023 appendix; suffix B | PolicyPal hosted LLM | active |
| `gcg-replay-03` | `sha256:ccccccc...placeholder-03` | Public follow-on 2024 | PolicyPal hosted LLM | active |

Note: Fresh-optimisation (chapter 05 Level 2) is `n/a` for PolicyPal — the target is a closed-API hosted LLM
and gradients are unreachable. The manifest therefore holds replay suffixes only.

## Many-shot demonstration corpora

| payload_id | sub-category | sha256 (placeholder) | shot counts | status |
|---|---|---|---|---|
| `manyshot-malware-corpus-v1` | malware-generation | `sha256:ddddddd...placeholder-04` | 4 / 16 / 64 / 256 | active |
| `manyshot-selfharm-corpus-v1` | self-harm | `sha256:eeeeeee...placeholder-05` | 4 / 16 / 64 / 128 | active |
| `manyshot-illicitadvice-corpus-v1` | illicit-advice | `sha256:fffffff...placeholder-06` | 4 / 16 / 64 / 256 | active |

## What is in-repo vs. out-of-repo

- In-repo: this manifest (`MANIFEST.md`, `MANIFEST.yaml`), the `README.md` discipline statement, and the
  `precommit-check-stub.md` repo-root check.
- Out-of-repo: every raw suffix, every demonstration pair, every transcript of a successful attack.

## Rotation policy

- GCG suffixes: dead suffixes flagged `retired:<date>` on provider safety-tuning updates; retained for
  historical reproducibility, not deleted.
- Many-shot corpora: new versions (`-v2`, `-v3`, ...) when demonstrations change; prior versions retained.
```

### Step 5 — Fill `red-team/payloads/precommit-check-stub.md` with the repo-root check

The repo-root check is the automated backstop that fails CI if a harmful-payload signature lands in the tree. The check does not have to be sophisticated; it has to *exist* and be documented, so arming it in exercise 02 is a one-line change.

```markdown
# red-team/payloads/precommit-check-stub.md — repo-root payload-leak check

Status: stubbed; armed by exercise 02's CI wiring.

## Intent

If any file in a commit matches a harmful-payload signature, the commit (or the CI
pipeline, depending on where the check runs) fails and surfaces the offending path.

## Signature classes the stub checks

- Size + entropy filter — files above a size threshold with entropy above a configured
  bound are flagged as candidate ciphertext / token-blob (a GCG suffix is a token blob
  with high entropy).
- Known-hash list — every SHA256 listed in `MANIFEST.yaml` is checked against file
  hashes in the commit; a match means a raw payload has leaked into the repo.
- Keyword stop-list — a small list of DAN / jailbreak giveaway strings that should
  never appear in-repo. The stop-list is itself not published inside the repo; the
  check pulls the stop-list from the out-of-repo store at run time.

## Deliberately out of scope for the stub

- Semantic classification of file contents — a classifier that decides "is this file
  a jailbreak" is overkill for the backstop. The three signatures above cover the
  shapes that matter.

## Arming

Exercise 02's CI wiring step enables this check in the smoke-suite pipeline. The
stubbed check is a one-line activation (set `check.enabled: true` in the CI config)
and a rebuild. Until then, the check documents its intent so no surprise exists at
arming time.

## When the check fails

A failed check means a raw payload has entered a commit. Response:

1. Block the commit / revert the PR.
2. Investigate how the payload entered; a second commit removing it does not rewind
   the leak in git history.
3. Rotate the exposed payload — flag it `retired:<date>` in the manifest and
   regenerate the replacement in the out-of-repo store.
4. File a payload-hygiene incident under mod-111's process.
```

### Step 6 — Checkpoint on the discipline invariants

Re-read all four files. Confirm every item:

- **No raw suffix anywhere.** `MANIFEST.yaml`, `MANIFEST.md`, `README.md`, and `precommit-check-stub.md` contain no string that is a raw GCG suffix, a raw DAN framing, or any other harmful payload. Every payload-identifying field is a hash or an ID, not the content.
- **No demonstration pair anywhere.** Many-shot corpora are referenced by `payload_id` + hash + category + shot-count metadata. The individual `(ask, response)` pairs do not appear.
- **Every `sha256` is marked as placeholder in this lab's output.** A real manifest would carry real content hashes; a lab manifest must carry placeholders so there is no confusion that a lab artifact has been treated as an operational artifact.
- **Every entry carries a provenance line.** Provenance is what the triage pipeline cites when a finding is produced; an entry without provenance is unauditable.
- **Rotation is written down.** Both GCG and many-shot entries carry a rotation note (retired-on-update vs. versioned-on-change). An entry without a rotation policy is an entry nobody has thought about the half-life of.
- **The pre-commit check is a stub, not armed.** The check *exists in the repo* — its intent, its signature classes, its activation path, and its failure response are all documented — so exercise 02 can arm it in one line. An unwritten check is one that will not exist when it is needed.
- **The GCG manifest notes `n/a` for fresh-optimisation on PolicyPal.** The closed-API constraint from chapter 05 is surfaced on the manifest, not left implicit. A reviewer reading the manifest must see both why the Level 1 replay set exists and why Level 2 does not.

If any invariant fails, rewrite the offending file before moving on.

### Step 7 — Write the one-paragraph manifest narrative

Append a short narrative to `README.md` that reads back the manifest in English, in two to four sentences. Target shape:

> "PolicyPal's out-of-repo payload manifest holds three GCG replay suffixes (Zou et al. 2023 + one public follow-on) and three many-shot demonstration corpora (one per internal-policy category — malware-generation, self-harm, illicit-advice). All six entries are referenced by `payload_id` + sha256; none of the content enters the repo. Fresh-optimisation of GCG against this target is `n/a` because the model is closed-API; the manifest holds replay suffixes only. The pre-commit check is stubbed and armed in exercise 02's CI wiring step; its failure response includes payload rotation and a mod-111 payload-hygiene incident."

That paragraph is what a risk council or an auditor reads cold.

### Step 8 — Final checkpoint

Re-read everything. Confirm:

- Four files exist in `red-team/payloads/` (`README.md`, `MANIFEST.md`, `MANIFEST.yaml`, `precommit-check-stub.md`).
- `README.md` carries the four-reason "why not in-repo" paragraph in your own words plus the manifest narrative.
- `MANIFEST.yaml` has six entries (three GCG + three many-shot), each with `payload_id`, `category`, `provenance`, `sha256` placeholder, `target_set`, and `status`.
- `MANIFEST.md` has two tables (GCG + many-shot) with the same six rows and a rotation-policy note.
- `precommit-check-stub.md` documents intent, three signature classes, arming path, and failure response.
- *No file contains a raw harmful string of any kind*. The deliberate invariant.

## Expected output

`mod104-lab03/red-team/payloads/` with the four files as described; roughly two pages of total content across them. The deliverable is the *shape and discipline*, not the specific placeholder IDs — those are fabricated for the lab.

## Checkpoint questions (self-check)

1. A reviewer asks "if the manifest only carries hashes, how does a suite actually run the attacks?" Which chapter-01 and chapter-03 detail answers that? (Converters resolve `payload_id` → `store_uri` at runtime, verify the hash, and exercise the payload against the target without writing it back into the repo. Chapter 01's three-column table pins the split; chapter 03's converter pattern pins the runtime resolution.)
2. A junior engineer proposes adding a `raw_sample` field to each manifest entry "so reviewers can see what the payload looks like." What is the specific defect? (The `raw_sample` field *is* the harmful content. Committing it defeats the entire out-of-repo discipline, surfaces the string on every clone / mirror / grep, and in the GCG case puts an opaque token blob into the repo that a human reviewer cannot evaluate anyway. The manifest's invariant is *metadata-only*; sampling belongs in the out-of-repo store under its own access control.)
3. The GCG manifest marks fresh-optimisation `n/a` for PolicyPal. Under what target-shape change would that `n/a` have to flip to `deferred` or `covered`? (If PolicyPal's model were swapped for an open-weight model the programme deploys locally. Then chapter 05's Level 2 coverage would apply — quarterly fresh optimisation against the current checkpoint — and the manifest would gain a new entry class for `gcg-fresh-<run-id>` suffixes with their optimisation provenance.)
4. A suffix is retired after a provider retrain. Why does the chapter say to flag it `retired:<date>` rather than delete it from the manifest? (Historical reproducibility. The register row and the triage log reference the suffix by hash; a measurement produced against `gcg-replay-02` on a prior release must still be interpretable six months later. Deleting the entry makes the historical measurement un-anchored. Chapter 05's rotation discipline is explicit on this: "archive the old set with a 'retired' flag rather than deleting outright.")

## Why this lab matters

Exercises 03 and 05 ask you to wire real converters against real payloads from the out-of-repo store. If the manifest and the repo-root check have not been authored by the time you start exercise 03, the first thing an incident will surface is "the GCG suffix ended up in the repo on commit X because nobody had written the check." This lab drills the artifact shape — manifest, hashes, provenance, rotation, pre-commit stub, store URI pattern — until it is routine, so exercises 03 and 05 are *wiring runtime consumers* rather than *inventing payload-hygiene policy under deadline pressure*. Labs 01 and 02 drill the coverage-matrix cell discipline and the five-step triage; together the three labs make exercises 01–06 tractable within their nominal budgets.
