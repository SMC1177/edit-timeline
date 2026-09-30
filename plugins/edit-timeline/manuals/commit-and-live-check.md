---
name: reference-dsh-funnel-7-commit
description: "FUNNEL PHASE 7 — commit in depth: self_test and friction shapes, what commit_session actually stages, and BOOT_CHECK_REQUIRED — which fires on a brand-new lib/ file that NOTHING imports, because boot-loaded is a property of the PATH, not the import graph."
metadata:
  type: reference
---

# PHASE 7 — COMMIT

## HOW IT WORKS

```
commit_session({session_id, message, self_test:{q1..q5}, friction})
```

- **`self_test` q1–q5**: each ≥30 chars, **distinct**, each carrying a confidence marker
  (`[verified]` / `[inferred]` / `[assumed]`). A placeholder is refused.
- **`friction`**: `'none'` (an explicit no-friction declaration) **or** an array of ≥20-char
  suggestions, auto-filed to the suggestion box. Omitting it entirely is `FRICTION_REQUIRED`.
- **Staging**: the plan's paths plus `extra_paths[]` (ADDS). `paths[]` REPLACES the whole set — use it
  for exact control, and any dropped plan file comes back loudly as `omitted_plan_paths`.
- `dry_run: true` previews `would_stage_files` **and** reports every remaining blocker in one call.
  It is the authoritative read on a detached gauntlet's terminal state when a status tool lags.
- Push only if the project has a remote — **edit-timeline is `local_only`**.

**Unrelated dirty paths are excluded by default.** Measured 2026-09-21: 3 source files staged,
**105 excluded**. Do not hand-manage them; check `excluded_count` and move on.

## 🔴 `BOOT_CHECK_REQUIRED` — "BOOT-LOADED" IS A PROPERTY OF THE PATH, NOT THE IMPORT GRAPH

**MEASURED 2026-09-17:** `lib/probe-runs.js` was created *that session*, had **ZERO importers** (its
own test was the only referrer), and `commit_session` still refused, naming it boot-loaded.

> **The cost was a scope decision taken on a false premise.** That session had EXCLUDED a genuinely
> boot-loaded change *specifically to avoid* this gate, believing a zero-consumer module could not
> trip it. **Never scope a session on the import graph's behalf: a new file under `lib/` inherits the
> live check and the restart obligation.**

**The cure, verbatim:**

```
run_live_check({session_id, probe_command:"node scripts/live-probes/live-verify-gate.mjs"})
→ poll get_live_check_status({session_id}) until status "done" and ok   (~90s)
→ commit_session again
```

- **`died`** means the runner exited without finishing — a REAL failed check, not a formality.
- **`running`** means keep polling; a runner that dies is now reported as `died`, so a long `running`
  is not stuck.
- ⚠️ **Traps:** `probe_command` rejects shell operators (so never an inline `node -e` with an arrow
  function), and `scripts/smoke.js` is NOT a valid probe — it starts its own throwaway server and
  asserts a stale tool list.
- **Ordering wrinkle, and it is benign:** the live check runs AFTER the breaker and BEFORE the commit
  — that is the gate's own sequence, so it does **not** stale the breaker.

**The second arm — `set_session_policy({session_id, policy:{live_verify:"not_needed"}})` — is
self-certification about your own new file.** It is faster. The probe arm is the honest one and is
cheap (one call + one poll; measured `done`/`ok` on the FIRST poll). Use `not_needed` only when the
change genuinely has no live behaviour.

**Read `running_server` in the result.** It reports whether the LONG-RUNNING server is on the shipped
sha — `status:"current"` vs `"stale"`. A green probe says the code boots; it does **not** say the
running server is on it. See [[live-check-vs-verify-gate]].

## 🔴 `ready_to_commit: true` IS EVIDENCE ABOUT THE GATES, NOT ABOUT THE CODE

A gauntlet returned it with **331/331 green and both reviews clean** while the landed bytes held a
SILENT under-reach. What caught it was **quoting the region that had changed and READING it**, plus a
reviewer's HIGH finding naming the same fixture.

> **Before commit, read the landed bytes of your own change and ask whether they do what the plan
> says.** Behavioural green and byte-level correctness are two different claims.

## THE CLAIM DISCIPLINE IN THE MESSAGE

> **State a claim only as strongly as it was exercised.**

A structural fact and a behavioural claim are DIFFERENT claims. Never write "fixes X" when X was never
exercised. The commit message is a permanent record and the funnel will not catch an overclaim —
`self_test` q4 exists precisely to carry *what was NOT proven*.

## WHAT COMMIT DOES NOT DO

**It does not make the change live.** A committed module that `server.js` imports at boot is still
serving the OLD bytes until `restart_server`, because Node caches the module. Phase 8 owns that, and
skipping it is how a "shipped" fix sits inert.
