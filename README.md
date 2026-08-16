# edit-timeline

**Your AI coding agent, with receipts.** edit-timeline is a governance server for AI coding agents — enforced plans, server-verified edits, adversarial review, and a durable audit trail. Not another prompt telling your agent to be careful: a gate it actually has to pass.

Works with any MCP-capable agent; built and battle-tested with Claude Code.

## Why

Agents ship code that *looks* right. edit-timeline exists for the gap between "looks right" and "is right":

- **Plans before edits.** Work is declared as snippets with intents and verification commands *before* any file changes. Off-plan diffs get flagged.
- **Verified, not vibed.** Every applied change must pass its declared verify commands — the server runs them itself and refuses to let stale or missing test runs through.
- **An adversary reviews the diff.** A fresh model is dispatched to hunt wrong-but-green defects — changes that pass the suite but are wrong, including tests that accommodate the bug.
- **Everything is a ledger.** Sessions, edits, test runs, reviews, deploys — append-only, queryable, and there six months later when you ask "why is this file shaped like this?"
- **A suggestion box your agents actually file into**, so the tool improves from real friction, not guesses.

Caught the night before this beta shipped: an adversarial reviewer that green-lit a change it never saw — because the diff exceeded its embed cap. That finding is now a fixed bug and a regression gate. That's the loop, working on itself.

## See it in action

A 60-second true story from the ledger: an AI session got stuck mid-commit at 10pm, used the built-in help line to reach the builder, and the diagnosis, repair, and permanent fix all shipped the same night. [Read the full case study](CASE-STUDY.md).

<!-- VIDEO: drag promo-final.mp4 into this line in the GitHub web editor to embed it natively, or replace with the YouTube link -->


## Install (free beta)

**Claude Code (PowerShell / VS Code / terminal).** Run these as **two separate commands** — enter the first, let it finish, then enter the second (don't paste both at once):

Step 1 — add the marketplace:

```
/plugin marketplace add SMC1177/edit-timeline
```

Step 2 — install the plugin:

```
/plugin install edit-timeline@edit-timeline
```

The bundled MCP server starts automatically once the plugin is enabled (approve it when Claude Code asks). If you run `/plugin marketplace add` with no argument and it prompts "Enter marketplace source," type just `SMC1177/edit-timeline` there — nothing else.

**Claude Desktop (macOS/Windows):** download `edit-timeline-mcp-<version>.mcpb` from the [latest release](../../releases/latest), then **open Claude Desktop and drag the file into Settings → Extensions** (or use the **Install extension** button there and pick the file). That is the whole install.

> **Don’t double-click the .mcpb.** On a fresh Windows machine the file often isn’t associated with Claude yet, so Windows asks which app to open it with — and Claude’s program folder is hidden by default, so hunting for `claude.exe` is a dead end. You never need to find the exe: dragging the file into the Extensions window installs it directly. Verify the download against the `.sha256` sidecar if you like.

**Any other MCP client:** extract the `.mcpb` (it's a zip) and point your MCP config at the bundled server (Node 20+):

```jsonc
// .mcp.json
{
  "mcpServers": {
    "edit-timeline": {
      "command": "node",
      "args": ["<path-to-extracted>/server/index.mjs", "serve", "--http", "39450"]
    }
  }
}
```

## After you install

**First run: smax walks you through setup.** Every install requires a Google account to activate (accounts are enforced). Open the seat console:

```
http://127.0.0.1:39450/seat
```

Smax (your orchestrator advisor) greets you with a guided conversation and walks you through the setup one step at a time — **Sign in with Google**, your DeepSeek API key, adopting a project directory, importing history, and semantic memory. Tap the reply buttons under his messages to answer, just like texting; each step saves for real as you go. When setup is complete the walkthrough clears from the chat, and a **📖 Tutorial** tab stays in the left sidebar so you can replay it any time. You can also paste an account key from another machine instead of signing in — either way the key is saved locally and your account works across machines (each machine keeps its own data).

**Check it's running.** Ask your agent to call the `ping` tool, or open:

```
http://127.0.0.1:39450/health
```

You should get JSON with a `version` and a `git_sha`. If the connection is refused, the server started without `--http` — see the config above.

**Open the console.**

```
http://127.0.0.1:39450/ui
```

This is the human side of edit-timeline: live sessions, the plan each agent is working against, every snippet and whether it verified, and the diff the server actually accepted. While setup is still pending, the dashboard sends you to the seat console so smax can walk you through it. It binds to loopback, so it's reachable from your machine only.

**Your first session.** You don't drive edit-timeline directly — your agent does. Ask it to do a piece of work through edit-timeline and it will open a session, file a plan, and dispatch. Watch the console while it works: the point is that you can see the plan before any edit lands, and see which edits passed the verify commands they declared.

**Where your work is kept.** Sessions, plans, diffs, and the audit trail are written under the server's install directory on your own disk. Removing the extension does not delete that history.

## Setup: model key (for the AI-powered features)

edit-timeline's plan / verify / commit / audit-trail spine works out of the box. The **model-powered** features — subagent-driven implementation, adversarial review, and grounded phone answers — call a model backend (DeepSeek by default), so set your own key once:

- Put `DEEPSEEK_API_KEY=sk-...` in your environment, or in `~/.aider.env`.

It's **your** key — nothing routes through us. Without it, those specific features return "DeepSeek API key not found" and everything else keeps working.

## Status

Public beta. Free during the beta under the [beta license](./LICENSE-BETA.md) — plain English: use it freely (including at work), don't redistribute or resell it, don't reverse engineer it. Commercial terms may follow after the beta; nothing you do in the beta obligates you to them.

## Feedback — the whole point

Found a scenario it handles badly? **That's exactly what this beta exists to find — please file it.** Open an [issue](../../issues) with the version (in the release filename) and what you expected vs. got. Bug reports here feed directly into the suggestion-box pipeline that drives every release; "fixed in vX.Y.Z" replies are how we say thanks.

## Trademarks

edit-timeline is a product of Smack Tax LLC. Claude is a trademark of Anthropic PBC; edit-timeline is not affiliated with or endorsed by Anthropic.
