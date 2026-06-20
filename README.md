# mcp-keep-integrations

Integration packs for [mcp-keep](https://github.com/exetorius/mcp-keep) — the lifecycle/resilience relay that fronts your MCP server(s) on one local port and keeps their tools surfaced even while an upstream is offline.

A pack customises the relay for a **specific upstream** — adding tool hints, synthetic tools, and agent instructions — without touching the relay itself. Packs are optional: an upstream works fine without one. A pack just makes its tools nicer to use.

The relay fetches packs from this repo's **`main`** branch on demand. Nothing is bundled into mcp-keep.

## Using a pack

Drive it through the relay's MCP tools (chat is the UI):

```
keep_install_pack                  → list available packs
keep_install_pack name='vibeue'    → download + install a pack
keep_remove_pack   name='vibeue'   → detach from any upstream + delete it
```

`keep_install_pack` downloads the pack into `~/.mcp-keep/integrations/<name>/`, sets the upstream's `integration`, and surfaces any companion-server suggestion for your explicit consent (see [`post_install.json`](#post_installjson)). Re-running it **updates** the pack (see [Versioning & updates](#versioning--updates)).

**Manual install** (fallback): download a pack folder into `~/.mcp-keep/integrations/<name>/`, set `"integration": "<name>"` on the upstream in `~/.mcp-keep/config.json`, and call `keep_reload`.

## Available packs

| Pack | Upstream server | Description |
|------|-----------------|-------------|
| `vibeue` | [VibeUE](https://github.com/kevinpbuckley/VibeUE) | Unreal Engine MCP plugin — tool hints, a dynamic skills index, and agent instructions |

## Pack format

A pack is a folder containing any combination of the files below. Folder names are lowercase, hyphenated, and match the upstream's common name. (Don't commit a `.cache/` folder — the relay uses `<pack>/.cache/` for the upstream's captured tool cache.)

### `pack.json`
Pack manifest. **Required for update detection.**
```json
{ "name": "vibeue", "version": "1.1.0" }
```
`version` is what `keep_status` compares against this repo's `main` to flag "update available". **Bump it whenever you change a pack**, or users won't be told an update exists. Use `MAJOR.MINOR.PATCH`.

### `config.example.json`
Pre-filled relay config for this integration — a starting point showing the upstream's host/port/path. Fill in `bearer_token` and adjust ports as needed.

### `hints.json`
Tool-description appends, injected into `tools/list` at the relay. Keyed by the **exact tool name** (upstream or synthetic):
```json
{ "tool_name": "\n\nExtra guidance appended to this tool's description." }
```
Hints are paid as tokens on every session the upstream's tools are surfaced. Keep them focused.

### `synthetic_tools.json`
Extra tools served by the relay itself (not forwarded upstream) — e.g. a manager/dispatcher tool. Standard MCP tool shape:
```json
[
  { "name": "my_tool", "description": "What it does.",
    "inputSchema": { "type": "object", "properties": {}, "required": [] } }
]
```

### `instructions.md`
Agent instructions for working with this upstream — routing rules, API patterns, "discover before you execute" guidance. Surfaced to the client so the agent reads it before it starts.

### `post_install.json`
Optional steps the relay processes after install. Today the only step type is `mcp_server` — a **suggested companion MCP server** the pack pairs with:
```json
[
  { "type": "mcp_server",
    "server_name": "unreal-engine-skills",
    "server_config": { "type": "http", "url": "https://www.unrealengineskills.com/api/mcp" },
    "target": "~/.claude/.mcp.json" }
]
```
**The relay does NOT write this automatically.** Adding an MCP server — especially an outward-facing one — is a privileged effect, so `keep_install_pack` only **surfaces** the suggestion (name, URL, that it's outward-facing) and the exact `mcpServers` snippet; the user adds it to their client config only on an explicit, separate yes. Keep companions genuinely optional — the pack must work without them.

### `README.md`
Human-facing notes for the pack.

## Versioning & updates

- The relay caches each installed pack's latest `version` from `main` (refreshed on a slow background cadence) and `keep_status` flags `update available: <pack> X → Y`, showing how stale the check is. `keep_status check_updates=true` forces a live re-check.
- Updating is just re-running `keep_install_pack name='<pack>'`: it re-downloads `main`, records the new `version`, and **prunes** any files removed upstream (your `.cache/` is preserved).
- So: **bump `pack.json`'s `version` on every change.** Without a bump, the relay can't tell users an update is available.

## Contributing a pack

1. Fork this repo.
2. Add a folder named after the upstream (lowercase, hyphenated), with whichever files apply — all are optional except `pack.json` if you want update detection.
3. Open a PR against the **`contrib`** branch, not `main` (`contrib` is the staging branch; it's promoted to `main` once verified). The relay serves packs from `main`, so a change is live to users only after it reaches `main`.
4. Bump `pack.json`'s `version` so existing installs are told to update.
