---
name: reference-dsh-funnel-6-reviews
description: "FUNNEL PHASE 6 — the reviews (sweep, adversarial, breaker) in depth: how run_gauntlet actually behaves, every blocker it can leave standing, and the measured failure modes including the one that costs the most — a sweep red authored by ANOTHER RUNNING SESSION rather than by your change. The index line that points here is part of the injected funnel index, which arrives automatically every session; this file is the deep companion to the measured record that gauntlet sweep reds are shared state."
metadata:
  type: reference
---

# PHASE 6 — THE REVIEWS

> **The index entry is injected every session — it arrives automatically with the funnel index and is
> never deliberately fetched. THIS file is the deep copy: read it when phase 6 refuses and the index
> line is not enough.**
> Companion rule, and the single highest-value line on this page: a sweep red can be authored by
> ANOTHER running session rather than by your change, because this install's own suite asserts
> against its `sessions/` directory. Re-run the named suite alone before you believe it.

## HOW IT WORKS

`run_gauntlet({session_id})` runs the canonical order in one call:

```
verify → sweep → adversarial → breaker → done
```

- It is **`mirror:false`, in-process** — it dies with the server, so **never restart while one runs.**
- It **auto-detaches** and returns `{subagent_id, run_pending:true}`. Poll
  `get_subagent_status({subagent_id, session_id})`, or `get_gauntlet_job({job_id})` when it queued.
- **Gauntlets are SERIALISED.** Behind another session's run it returns `{queued:true, job_id}` and
  **starts by itself** — do NOT re-initiate.
- A durable record is appended to `sessions/<id>/gauntlet-runs.jsonl`, which is how a run is read back
  after the in-memory entry is evicted or the server restarts.

**Measured durations, so the wait is budgeted rather than feared:** 692 s, 868 s, 956 s and 963 s on
four separate runs. Budget ~12–16 minutes for a cold sweep.

## THE FOUR STAGES, AND WHAT EACH ONE CAN LEAVE STANDING

| stage | what it proves | what it does NOT prove |
|---|---|---|
| verify | the snippets' own `verifies_with` ran green | nothing about the rest of the repo |
| sweep | all 334 suites pass | nothing about WHO authored a red — see below |
| adversarial | a reasoning pass found no defect | nothing, if it hit its time cap |
| breaker | pattern scan found no bad shapes | its "new export" detector keys on LINE NUMBERS |

## 🔴 FAILURE 1 — A SWEEP RED IS A HYPOTHESIS, NOT A FINDING

**The single most expensive failure in this phase.** A red can be authored by another running
session, because suites that read shared install state (`sessions/`, `data/`) have more than one
writer. `caused_by_session` will still name YOU.

**THE FIRST MOVE IS ALWAYS: run the named suite ALONE.**
`node --test <suite>` from the install root. Green in isolation ⇒ environmental, not your change.
Measured 2026-09-21: `lib/file-lifecycle.test.js` red in sweep, **58/58 green alone.**

**And NEVER write to the workdir while a gauntlet runs** — session artifacts land under `sessions/`
*inside* the workdir, so you can red your own sweep. Self-inflicted and avoidable.

A sweep red is therefore a hypothesis about your change, never a finding, until the named suite has
been re-run on its own.

## 🔴 FAILURE 2 — THE ADVERSARIAL TIME CAP BLOCKS THE COMMIT

Measured twice in one session: the adversarial stage hit `status:"partial"`,
`incomplete_reason:"time_cap"` at **543 s and 594 s**, and `commit_session` then refused:

> *"zero findings from a review that ran out of budget is not evidence of zero defects."*

**The gate is right.** The escape is NOT to weaken it: run `request_adversarial_review({session_id})`
**standalone** — the same review completed in 307 s outside the gauntlet. Then breaker, then commit.

## 🔴 FAILURE 3 — `stalled: true` IS A FALSE POSITIVE

`get_subagent_status` raises the flag at **~500–586 s**. Four gauntlets measured this session and the
prior one completed normally at **692 s, 868 s, 956 s and 963 s**, every one with `stalled:false` at
the end. **Killing on the flag alone destroys a full 334-suite sweep.** Verify real progress on a
second check before ever calling `kill_subagent`.

## 🔴 FAILURE 4 — `GAUNTLET_REQUIRED` IS ITS OWN BLOCKER

It is **separate** from `adversarial-review-stale` and `breaker-review-stale`. Hand-firing
`request_adversarial_review` + `request_breaker_review` clears those two and leaves this one standing.
**Only a PASSING gauntlet clears it**, and its prescribed fix is literally `run_gauntlet` again.

## 🔴 FAILURE 5 — AN UNCOMMITTED CHANGE FORCES A FULL RERUN

The sweep cache key includes `dirty` and `head`. Any uncommitted edit yields
`cache:{hit:false, reason:"key-mismatch", checkpoint:{resumed:0, rerun:334}}` — a full rerun, not a
replay of the one failure. Budget accordingly; do not edit mid-gauntlet expecting a cheap re-run.

**The stronger form of this is already measured:** the key is a
WHOLE-TREE hash, so **any commit by any agent** invalidates it — five consecutive sessions in one
night with three agents in one workdir, never a single hit. Under concurrency, plan for a cold sweep
every time rather than treating a miss as a surprise.

## 🔴 FAILURE 6 — THE BREAKER'S "NEW EXPORT" WARNING KEYS ON COORDINATES

Measured 2026-09-21: the breaker reported
`semantic-dup: New export "seatRecallBlock" closely resembles "buildRecallBlock"` — but `git grep`
showed `seatRecallBlock` at `:97` in **committed** code and `git diff` showed it **absent from the
change**. An insert above it had shifted line numbers and the detector read a pre-existing export as
new. **Disprove this class with `git diff`, not with argument**, and record the disproof.

## DISPOSITIONS — THE RULE THAT IS EASY TO GET WRONG

Every adversarial finding needs `disposition_adversarial_finding` with
`will_fix` | `acknowledged_risk` | `false_positive`, keyed on **`fingerprint` + `reason`** (NOT
`finding_id`/`disposition_reason`).

> **`false_positive` requires a `counsel_seq` from a seat reply that POSTDATES the review.** Without
> one, the honest bin is `acknowledged_risk` with a reason — *even when the reviewer itself recommends
> false_positive.*

**Never edit verified code to satisfy a finding about code your diff does not touch** — editing
re-stales the whole review chain and you loop.

⚠️ **A finding can be RIGHT and under-rated.** 2026-09-21: a finding filed `low` was a SILENT
stale-read affecting every seat under 8 MB. Read the mechanism, not the severity label.

## THE ORDERING TRAP

> **Any verify (step 2) run AFTER a review (step 3/4) invalidates that review.**

Finish EVERY verify before the reviews, and keep **breaker → commit back-to-back with nothing in
between.** `get_required_verifies` lists the whole set up front so this is plannable.

## AFTER A GREEN GAUNTLET

`ready_to_commit:true` with `remaining_blockers:[]` is NOT the end — `BOOT_CHECK_REQUIRED` can still
fire **at commit**, because it keys on the diff touching a boot-loaded file. Its cure is verbatim and
is rendered in the refusal itself — read it there rather than reconstructing it.

**Manually confirm a detached run's terminal state with `commit_session({dry_run:true})`** — a status
tool can lag a run that has already finished (the operator, 2026-09-21). The dry run reports the gauntlet
stamp AND every remaining blocker in one call, and previews `would_stage_files`.
