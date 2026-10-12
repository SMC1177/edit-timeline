# Code Quality Gates — Global

Universal rules enforced by edit-timeline on every project.

The presence of this file in the install directory is what ARMS the ESLint
quality gate at mark_applied. A project that keeps its own ESLint config is
linted with that config; this file's rules apply only where a project has
none of its own.

**Owner:** the install operator
**Created:** 2026-05-27
**Override:** set `policy.require_quality_gate: false` on a session to skip the gate for that session, or delete this file from the install directory to disarm it entirely.

---

## GATE G1: No Silent Error Swallowing

Every `catch` block must either log the error or rethrow it.

**Banned:**
- `catch (e) {}`
- `catch (_e) { /* ignore */ }`
- `catch (e) { return; }` (without logging)

**Required:** `logger.warn(err)`, `logger.error(err)`, `console.error(err)`, or `throw err`

**ESLint rule:** `no-silent-catch` or equivalent per project

---

## GATE G2: No Garbage Tests

Tests must assert real behavior.

**Banned:**
- `expect(true).toBe(true)`
- Empty `it(...)` blocks
- `.catch(() => {})` in test code
- `it.skip(...)` to make suites pass

---

## GATE G3: Visual Verification on UI Changes

No AI session may declare a UI feature complete without human verification.
After UI changes, the AI must say: "I cannot verify this visually — please check the screen."

---

## GATE G4: No Data Deletion Without User Action

No code path may silently delete, discard, or zero-out user data based on
timers, heuristics, or error recovery. If the app doesn't know what happened,
it keeps the data and asks the user.
