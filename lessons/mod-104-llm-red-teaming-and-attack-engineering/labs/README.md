# Labs for mod-104-llm-red-teaming-and-attack-engineering (LLM and Agent Red-Team Engineering)

Three short guided drills. Each builds one atomic skill the exercises assume, on the same running `PolicyPal` example used across the mod-102 and mod-103 labs so the running system does not switch between labs. The labs do not duplicate the module's exercises — the exercises build the production artifacts; the labs drill the discipline those artifacts rely on.

- [`lab-01-coverage-matrix-cell-marking-drill.md`](lab-01-coverage-matrix-cell-marking-drill.md) — 45 minutes. Fill one 12-technique × 4-surface coverage matrix end-to-end with every cell marked `covered:<suite-id>`, `n/a`, `deferred:<reason>`, or `escalated:L40`, every `covered:*` traced, and no blank cells. Chapter 01 skill feeding exercise 01.
- [`lab-02-garak-triage-five-step-walk.md`](lab-02-garak-triage-five-step-walk.md) — 45 minutes. Walk five fabricated garak candidate rows through chapter 02's five-step triage (dedup within run → false-positive review → dedup across runs → severity + register attachment → regression fixture), producing a dispositioned triage log with the correct per-detector-kind FP review and the correct downstream handoffs. Chapter 02 skill feeding exercise 02.
- [`lab-03-out-of-repo-payload-manifest-drill.md`](lab-03-out-of-repo-payload-manifest-drill.md) — 40 minutes. Author the in-repo artifacts (`MANIFEST.md`, `MANIFEST.yaml`, `README.md`, `precommit-check-stub.md`) that make the chapter 01 / chapter 05 harmful-payload-out-of-repo discipline operational, using fabricated payload metadata only. Chapters 01 + 05 skill feeding exercises 03 and 05.

Run them in order — each lab's output is a shape exercise 01 (and the downstream exercises) will fill with real decisions.
