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

## Install (free beta)

**Claude Code (PowerShell / VS Code / terminal) — the two-command way:**

```
/plugin marketplace add SMC1177/edit-timeline
/plugin install edit-timeline@edit-timeline
```

That's it — the bundled MCP server starts automatically when the plugin is enabled (approve it when Claude Code asks).

**Claude Desktop (macOS/Windows):** download `edit-timeline-mcp-<version>.mcpb` from the [latest release](../../releases/latest), then in the Claude Desktop app go to **Settings → Extensions → Install extension** and pick the file (or drag the file into that window — double-clicking the file only works when your Claude Desktop version has registered the `.mcpb` association). Verify the download against the `.sha256` sidecar if you like.

**Any other MCP client:** extract the `.mcpb` (it's a zip) and point your MCP config at the bundled server (Node 20+):

```jsonc
// .mcp.json
{
  "mcpServers": {
    "edit-timeline": {
      "command": "node",
      "args": ["<path-to-extracted>/server/index.mjs", "serve"]
    }
  }
}
```

## Status

Public beta. Free during the beta under the [beta license](./LICENSE-BETA.md) — plain English: use it freely (including at work), don't redistribute or resell it, don't reverse engineer it. Commercial terms may follow after the beta; nothing you do in the beta obligates you to them.

## Feedback — the whole point

Found a scenario it handles badly? **That's the product working as intended — please file it.** Open an [issue](../../issues) with the version (in the release filename) and what you expected vs. got. Bug reports here feed directly into the suggestion-box pipeline that drives every release; "fixed in vX.Y.Z" replies are how we say thanks.

## Trademarks

edit-timeline is a product of Smack Tax LLC. Claude is a trademark of Anthropic PBC; edit-timeline is not affiliated with or endorsed by Anthropic.
