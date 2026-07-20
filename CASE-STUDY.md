# Case study: a stuck AI session phoned for help — and was fixed the same evening

*A true story from the edit-timeline ledger, July 19, 2026. Every timestamp below
is a real record you can audit in the session files.*

## The problem

An AI coding agent was finishing a feature in a production web app — a new
review-chain component, fully written, **118 of 118 tests passing**, type checks
clean. But its edit-timeline session refused to commit: the ledger showed two
phantom "pending" work items that no tool could reach. Every command the agent
tried resolved to the wrong record. It was wedged — correct code on disk,
a ledger that wouldn't accept it.

Most AI coding setups end here with a human copy-pasting diffs out of a chat
window and hoping.

## What happened instead

**22:13** — The stuck agent used edit-timeline's built-in help line
(`phone_a_friend`) and described the problem: duplicate work-item ids, the
pending twins unreachable, "how do I recover WITHOUT reverting verified work?"

**22:15** — Not satisfied with the automated first answer, it escalated to the
human-attended builder channel with a full reproduction: session id, the exact
tool calls that led to the wedge, and what it had already verified.

**Minutes later** — The builder session picked the thread up, and because
edit-timeline keeps an **append-only ledger of every session**, the diagnosis
took one read: a file-retarget had left two records with the same id — a
verified twin carrying the full proof-of-work, and a bare pending copy no tool
could address. The builder repaired the session record surgically (backup
saved first — the ledger never loses history), replied on the same thread with
exactly three commands to finish the commit, and told the agent: **do not
revert anything, your work is intact.**

**Same night** — The underlying bug was filed in edit-timeline's public
suggestion box with the full repro, and a permanent fix shipped: the session
store now enforces that a work-item id can exist only once, healing any
violation at the moment of writing and stamping a visible anomaly record —
so this entire class of wedge can never happen silently again. The fix
landed with regression tests reproducing the exact incident shape.

## Why this is the product

- **Nothing was lost.** The agent's verified work, its test evidence, and the
  whole history of the wedge survived — because sessions are an append-only
  ledger, not a chat scrollback.
- **The user's agent could ask for help in-band.** The phone channel is part
  of the tool, with an automated first responder and a human-attended
  escalation path — near-real-time follow-up from edit-timeline itself.
- **The incident improved the product the same night.** Friction and bugs are
  captured in a suggestion box with reproduction detail, and the fix shipped
  through edit-timeline's own governed pipeline: planned session, subagent
  implementation, independent verification, adversarial review, then commit.
- **Everything above is auditable.** The thread, the session records, the
  suggestion-box entry, and the fixing commit are all on disk with timestamps.

edit-timeline is an MCP server that makes AI coding accountable: every change
planned, verified, reviewed, and recorded — and when something does go wrong,
recoverable.
