# Scenario Tools

Reusable JavaScript scripts stored on disk and runnable by name. They share the same VM sandbox as the [`execute`](./execute.md) tool — `callTool(...)` and (on the browser platform) `page` are both available inside a scenario's script. Scenarios are **platform-agnostic**: the same tools are exposed by the browser, node, and backend CLIs.

## Storage

- **Project scope** (default): `${WORKING_DIR}/.ironbee-devtools-mcp/scenarios.json`
- **Global scope**: `~/.ironbee-devtools-mcp/scenarios.json`
- `WORKING_DIR` defaults to `process.cwd()` and is configurable via env var.
- Resolution order for `scenario-run`: project first, then global.

## CLI form

The five scenario CRUD tools have flat names (`scenario-add`, `scenario-update`, `scenario-delete`, `scenario-list`, `scenario-search`) — they have no `domain_` prefix, so the CLI exposes them under the auto-generated `default` group:

```bash
ironbee-browser-devtools-cli default <scenario-tool> [options]
```

**`scenario-run` is not registered as a CLI subcommand** (it lives in the daemon's tool registry only, not in `cliProvider.tools`). To run a scenario from the CLI, invoke `run execute` and call it from there:

```bash
ironbee-browser-devtools-cli run execute --code "await callTool('scenario-run', { name: 'login-flow' }, true);"
```

From an MCP client, all six tools are available directly under the flat names: `scenario-add`, `scenario-update`, `scenario-delete`, `scenario-list`, `scenario-search`, `scenario-run`.

---

## scenario-add

Add a new scenario at the chosen scope.

```bash
ironbee-browser-devtools-cli default scenario-add \
  --name <name> \
  --description <description> \
  --script <javascript> \
  [--scope project|global]
```

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--name` | string | Yes | - | Unique scenario name (used as the lookup key). |
| `--description` | string | Yes | - | What the scenario does. Used by `scenario-search` ranking. |
| `--script` | string | Yes | - | JavaScript body. Same sandbox as `execute`: `await callTool(name, input, returnOutput?)` and (browser) `page`. |
| `--scope` | enum | No | `project` | `project` (per-repo) or `global` (per-user). |

---

## scenario-update

Update an existing scenario's description and/or script. At least one of `--description` or `--script` must be supplied.

```bash
ironbee-browser-devtools-cli default scenario-update \
  --name <name> \
  [--description <description>] \
  [--script <javascript>] \
  [--scope project|global]
```

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--name` | string | Yes | - | Name of the scenario to update. |
| `--description` | string | No | (unchanged) | New description. |
| `--script` | string | No | (unchanged) | New script body. |
| `--scope` | enum | No | `project` | Which scope to update. |

---

## scenario-delete

Delete a scenario by name from the chosen scope.

```bash
ironbee-browser-devtools-cli default scenario-delete --name <name> [--scope project|global]
```

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--name` | string | Yes | - | Scenario to delete. |
| `--scope` | enum | No | `project` | Scope to delete from. |

---

## scenario-list

List available scenarios. When `--scope` is omitted, both scopes are returned (project entries override global entries with the same name).

```bash
ironbee-browser-devtools-cli default scenario-list [--scope project|global]
```

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--scope` | enum | No | (both) | Filter by `project` or `global`. Omit to list all. |

---

## scenario-search

Search scenarios by name/description using the configured search strategy (`SEARCH_STRATEGY` env var, or `SCENARIO_SEARCH_STRATEGY` override — `SIMPLE` default, `FTS5` optional with `better-sqlite3`). Results are ranked by relevance.

```bash
ironbee-browser-devtools-cli default scenario-search --query <text> [--limit <number>]
```

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--query` | string | Yes | - | Search text matched against scenario name and description. |
| `--limit` | number | No | `10` | Max number of ranked results. |

---

## scenario-run

Run a saved scenario by name. The scenario's JS body runs inside the same VM as `execute`. Scenarios can compose other scenarios via `await callTool('scenario-run', { name: '...' })`. Max recursion depth: `5`.

`scenario-run` is **not** a direct CLI subcommand. Invoke it through `run execute` from the CLI, or call it directly from any MCP client:

```bash
# CLI (via execute VM)
ironbee-browser-devtools-cli run execute --code "await callTool('scenario-run', { name: 'login-flow' }, true);"

# Or with a custom timeout
ironbee-browser-devtools-cli run execute --code "await callTool('scenario-run', { name: 'login-flow', timeoutMs: 60000 }, true);"
```

**Input fields** (MCP tool name `scenario-run`):

| Field | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `name` | string | Yes | - | Scenario to run. Resolved project-first, then global. |
| `timeoutMs` | number | No | `30000` | Wall-clock timeout for the whole run, in ms. |

The response shape mirrors `execute`: `toolOutputs` (when `callTool(..., true)` is used inside the script), `logs`, an optional `result` return value, and on failure `error` / `failedTool`.

---

## Examples

```bash
# Register a login flow scenario at the project scope
ironbee-browser-devtools-cli default scenario-add \
  --name "login-flow" \
  --description "Log in with test creds and assert dashboard loads" \
  --script "await callTool('navigation_go-to', { url: 'https://app.example.com/login' }); await callTool('interaction_fill', { selector: '#email', value: 'user@example.com' }); await callTool('interaction_fill', { selector: '#password', value: 'pw' }); await callTool('interaction_click', { selector: 'button[type=submit]' }); await callTool('sync_wait-for-network-idle', {});"

# Discover and search
ironbee-browser-devtools-cli default scenario-list
ironbee-browser-devtools-cli default scenario-search --query "login"

# Run via the execute VM (scenario-run has no direct CLI subcommand)
ironbee-browser-devtools-cli run execute --code "await callTool('scenario-run', { name: 'login-flow' }, true);"

# Compose: call one scenario from inside another (inside the --script body)
# await callTool('scenario-run', { name: 'login-flow' });
```
