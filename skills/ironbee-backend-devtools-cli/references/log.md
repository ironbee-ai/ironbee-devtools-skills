# Log Tools

Log capture from file, Docker, and Kubernetes sources with filtering, multi-line stack coalescing, JSON parsing, and live following.

**Three source types:** `file` (tailable file with rotation handling), `docker` (`docker logs --follow`), `kubernetes` (`kubectl logs --follow`).

**Pipeline (applied in order):** raw lines → `coalesce` (merge multi-line stacks) → `parseJson` (attach `parsed` field) → `pattern` / `level` / `jsonFilter` + `contextBefore` / `contextAfter` → `select` (dot-path projection over `parsed`) → `limit`.

**Common filter fields** (used by `read`, `read-multi`, `get-followed`):
- `--pattern <text>` — substring match, case-sensitive
- `--level <enum>` — `FATAL` / `ERROR` / `WARN` / `INFO` / `DEBUG` / `TRACE` (heuristic match on the line)
- `--limit <number>` — cap on returned lines
- `--parse-json` — switch (boolean optional): pass to `JSON.parse` each line into a `parsed` field
- `--json-filter <json>` — JSON object: `{ "<dot.path>": "<string>" }`. Leaf equality, AND-combined. Values are strings only. Requires `--parse-json`; non-JSON lines are dropped.
- `--coalesce <json|true>` — merge multi-line stack traces. Pass `true` (literal) for defaults. Custom heuristics: `{ "continuationPattern": "<regex>", "boundaryPattern": "<regex>", "maxCoalesce": <number> }`. Default `continuationPattern: ^\\s`, default `maxCoalesce: 100`.
- `--context-before <n>` / `--context-after <n>` — extra surrounding lines per match (requires at least one line-level filter to be set)
- `--select <json>` — JSON array of dot-paths to project from `parsed` (requires `--parse-json`; pruned tree preserves nested structure)

**Boolean flag convention.** Fields typed `z.boolean().optional()` upstream (e.g. `parseJson`, `drain`, `sortByTimestamp`) are exposed as a single CLI switch — passing the switch sends `true`; omitting it sends `undefined` and the server applies its documented default. **`--no-<flag>` is NOT registered.** To force `false`, call via MCP / `run execute`.

---

## register-source

Register a named log source. Existence is NOT validated at register time — use `check-source` if you need to verify.

```bash
ironbee-backend-devtools-cli log register-source --name <name> --type <enum> [options]
```

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--name` | string | Yes | - | Unique source name used in `read` / `follow` / etc. |
| `--type` | enum | Yes | - | `file` / `docker` / `kubernetes`. |
| `--path` | string | Conditional | - | Required when `--type file`. Path to the log file. |
| `--container` | string | Conditional | - | Required when `--type docker`. Container id or name. |
| `--pod` | string | Conditional | - | Required when `--type kubernetes`. Pod name. |
| `--kubernetes-container` | string | No | - | Container name inside the pod (multi-container pods). |
| `--namespace` | string | No | - | Kubernetes namespace. |

---

## unregister-source

Remove a registered source. Any attached followers are stopped.

```bash
ironbee-backend-devtools-cli log unregister-source --name <name>
```

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--name` | string | Yes | - | Source to remove. |

---

## list-sources

List every registered source plus the ids of any active followers attached to it.

```bash
ironbee-backend-devtools-cli log list-sources
```

No arguments.

---

## check-source

Probe a source for reachability. `file` → `fs.stat`. `docker` → `docker logs --tail 1`. `kubernetes` → `kubectl logs --tail 1`. 5s timeout.

```bash
ironbee-backend-devtools-cli log check-source --name <name>
```

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--name` | string | Yes | - | Source to probe. |

---

## read

Point-in-time read with filters. Idempotent — re-running with the same args returns the same lines.

```bash
ironbee-backend-devtools-cli log read --source <name> [filter options]
```

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--source` | string | Yes | - | Registered source name. |
| `--tail` | number | No | - | Number of trailing lines to read. |
| `--since` | string | No | - | ISO-8601 timestamp lower bound. |
| `--until` | string | No | - | ISO-8601 timestamp upper bound. |
| `--pattern` | string | No | - | Substring filter. |
| `--level` | enum | No | - | Heuristic level filter. |
| `--limit` | number | No | - | Cap on returned lines. |
| `--parse-json` | boolean | No | server `false` | Switch — pass to JSON-parse each line. |
| `--json-filter` | object | No | - | JSON: `{ "<dot.path>": "<string>" }` (string values only). |
| `--coalesce` | object/boolean | No | server `false` | `true` for default multi-line coalescing, or `{"continuationPattern":"<regex>","boundaryPattern":"<regex>","maxCoalesce":<n>}` JSON. |
| `--context-before` | number | No | `0` | Lines before each match. |
| `--context-after` | number | No | `0` | Lines after each match. |
| `--select` | array | No | - | JSON array of dot-paths to project from `parsed`. |

---

## read-multi

Fan-out read across several sources. Lines are tagged with their source and (by default) interleaved by timestamp. Per-source failures are isolated — one bad source doesn't fail the whole call.

```bash
ironbee-backend-devtools-cli log read-multi --sources <json> [filter options]
```

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--sources` | array | Yes | - | JSON array of registered source names. |
| `--tail` | number | No | - | Per-source tail. |
| `--since` | string | No | - | ISO-8601 lower bound. |
| `--until` | string | No | - | ISO-8601 upper bound. |
| `--pattern` | string | No | - | Substring filter. |
| `--level` | enum | No | - | Heuristic level filter. |
| `--limit` | number | No | - | Cap on combined returned lines. |
| `--parse-json` | boolean | No | server `false` | Switch — pass to JSON-parse each line. |
| `--json-filter` | object | No | - | JSON: `{ "<dot.path>": "<string>" }` (string values only). |
| `--coalesce` | object/boolean | No | server `false` | `true` or `{"continuationPattern":"<regex>","boundaryPattern":"<regex>","maxCoalesce":<n>}`. |
| `--context-before` | number | No | `0` | Pre-context lines. |
| `--context-after` | number | No | `0` | Post-context lines. |
| `--select` | array | No | - | Dot-path projection. |
| `--sort-by-timestamp` | boolean | No | server `true` | Switch — to disable timestamp interleaving, call via MCP / `run execute` with `sortByTimestamp: false`. |

---

## follow

Start a streaming follower on a source. Returns a `followId`. Lines accumulate in a ring buffer until you drain with `get-followed` or stop with `stop-follow`.

```bash
ironbee-backend-devtools-cli log follow --source <name> [--max-buffer-lines <n>]
```

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--source` | string | Yes | - | Registered source. |
| `--max-buffer-lines` | number | No | (server default) | Ring buffer size. Older lines are dropped first when full. |

---

## get-followed

Drain (or peek at) buffered lines for a follower. Default behavior is **drain** — buffered lines are returned and the buffer is cleared.

```bash
ironbee-backend-devtools-cli log get-followed --follow-id <id> [options]
```

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--follow-id` | string | Yes | - | Id returned by `follow`. |
| `--drain` | boolean | No | server `true` | Switch — passing it (or omitting it) drains the buffer after returning. **There is no `--no-drain`**; to peek without clearing, call via MCP / `run execute` with `drain: false`. |
| `--pattern` | string | No | - | Substring filter. |
| `--level` | enum | No | - | Heuristic level filter. |
| `--limit` | number | No | - | Cap on returned lines. |
| `--since` | string | No | - | ISO-8601 lower bound. |
| `--until` | string | No | - | ISO-8601 upper bound. |
| `--parse-json` | boolean | No | server `false` | Switch — pass to JSON-parse each line. |
| `--json-filter` | object | No | - | JSON: `{ "<dot.path>": "<string>" }` (string values only). |
| `--coalesce` | object/boolean | No | server `false` | `true` or `{"continuationPattern":"<regex>","boundaryPattern":"<regex>","maxCoalesce":<n>}`. |
| `--context-before` | number | No | `0` | Pre-context lines. |
| `--context-after` | number | No | `0` | Post-context lines. |
| `--select` | array | No | - | Dot-path projection. |

---

## stop-follow

Stop a follower, close the watcher / child process, and return the final buffer contents.

```bash
ironbee-backend-devtools-cli log stop-follow --follow-id <id>
```

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--follow-id` | string | Yes | - | Id returned by `follow`. |
