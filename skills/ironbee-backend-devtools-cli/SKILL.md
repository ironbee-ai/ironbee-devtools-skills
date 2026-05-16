---
name: ironbee-backend-devtools-cli
description: CLI for backend service verification — drive HTTP/1.1+HTTP/2, gRPC, GraphQL, and WebSocket endpoints; capture logs from file, Docker, and Kubernetes sources; and verify state against Postgres, MySQL, or SQLite databases with snapshot/diff and change-feed primitives. Use when the user needs to call backend APIs over any of those protocols, assert on log lines or database rows, replay captured curl/HAR requests, manage session-scoped cookies/default headers, or correlate flows with W3C trace ids.
allowed-tools: Bash(ironbee-backend-devtools-cli:*)
---

# IronBee Backend DevTools CLI

Command-line interface for the **backend platform** of [IronBee DevTools](https://github.com/ironbee-ai/ironbee-devtools). Talks to backend services over their wire protocols (HTTP, gRPC, GraphQL, WebSocket), captures logs from file / Docker / Kubernetes sources, and verifies state against Postgres / MySQL / SQLite databases. Runtime- and language-agnostic — it binds to protocols, not frameworks.

For browser automation use [`ironbee-browser-devtools-cli`](../ironbee-browser-devtools-cli/SKILL.md). For Node.js inspector-based debugging use [`ironbee-node-devtools-cli`](../ironbee-node-devtools-cli/SKILL.md).

## Installation

```bash
# Install this skill (skills.sh)
npx skills add ironbee-ai/ironbee-devtools-skills

# Install the CLI binary (same package as the rest of IronBee DevTools)
npm install -g @ironbee-ai/devtools
```

## Port note

All three CLIs (`ironbee-devtools-cli` (browser), `ironbee-node-devtools-cli`, and `ironbee-backend-devtools-cli`) default to daemon port `2020`. If you run more than one daemon at a time, give the backend daemon its own port:

```bash
PLATFORM=backend ironbee-backend-devtools-cli daemon start --port 2022
ironbee-backend-devtools-cli --port 2022 request http --url "https://api.example.com/health"
```

## Quick Start

```bash
# 1. Start daemon (if not running)
ironbee-backend-devtools-cli daemon start

# 2. Call an HTTP endpoint
ironbee-backend-devtools-cli --json request http --url "https://api.example.com/health"

# 3. Capture logs around a request (file source)
ironbee-backend-devtools-cli log register-source --name app --type file --path /var/log/app.log
ironbee-backend-devtools-cli --json log read --source app --tail 100 --level ERROR

# 4. Verify a database row changed after the operation
ironbee-backend-devtools-cli db connect --name main --type postgres --connection-string-env DATABASE_URL
ironbee-backend-devtools-cli --json db query --connection main --sql "SELECT id, status FROM orders WHERE id = $1" --params '["abc-123"]'
```

## Global Options

| Option | Description | Default |
|--------|-------------|---------|
| `--port <number>` | Daemon server port | `2020` |
| `--session-id <string>` | Session id (cookies, default headers, default gRPC metadata, WS connections, log followers, db connections / snapshots / watchers, trace pin all live in the session) | auto |
| `--json` | Output as JSON (recommended for AI) | `false` |
| `--quiet` | Suppress log messages | `false` |
| `--verbose` | Enable debug output | `false` |
| `--timeout <ms>` | Operation timeout | `30000` |

**AI Agent Recommended:**

```bash
ironbee-backend-devtools-cli --json --quiet --session-id "verify-session" <command>
```

## Tool Domains

| Domain | Description | Reference |
|--------|-------------|-----------|
| request | HTTP / gRPC / GraphQL / WebSocket calls + cookies + default headers/metadata + curl/HAR replay | [request](./references/request.md) |
| log | File / Docker / Kubernetes log capture: read, follow, multi-source, filtering, JSON parsing, coalescing | [log](./references/log.md) |
| db | Postgres / MySQL / SQLite verification: query, snapshot/diff, watch-changes, transactions, seed, run-script | [db](./references/db.md) |
| o11y | Session-pinned W3C trace context (`traceparent` auto-injection) | [o11y](./references/o11y.md) |
| scenario | Reusable JS scripts (add, update, delete, list, search, run) — see browser CLI [scenario](../ironbee-browser-devtools-cli/references/scenario.md) reference; identical surface |
| execute | Batch JavaScript execution (run execute; CLI and MCP) — see browser CLI [execute](../ironbee-browser-devtools-cli/references/execute.md) reference. **Note:** the `page` binding is browser-only; on backend only `callTool` is available inside the VM. |

## Key concepts

**4xx / 5xx and gRPC non-OK status are normal results, not errors.** Only transport-level failures (DNS, TLS, timeout, abort) populate the response `error` field. The agent decides whether a given result counts as a failure for its task.

**Cookie jar** is shared across HTTP requests within a session (RFC 6265 host/path scoping). Login flows work transparently — call `request set-cookies` to seed, `request list-cookies` to inspect, `request clear-cookies` to drop.

**Host-scoped default headers / target-scoped gRPC metadata** are set once per session and merge into every matching call: `request set-default-headers`, `request set-default-metadata`. Auth tokens stay scoped to the host/target you pinned them to.

**W3C trace propagation** auto-injects `traceparent` on every `request_*` call. Pin a session trace id once with `o11y new-trace-id` (or `o11y set-trace-context`) and every subsequent request uses it as the correlation root. Per-call `--trace-id` overrides the pin; `_metadata.traceId` from the MCP client outranks the pin too.

**Readonly database access by default.** `db query` and `db snapshot` only accept SELECT-like statements (parser + server-side `READ ONLY`). To write, open the connection with `--allow-writes` and use `db transaction-begin --writable` + `db seed` / `db run-script` + `db transaction-rollback` (or `commit`). Column-name redaction (`password`, `token`, `api_key`, …) is applied to every result path; nested JSON values are walked too.

## CLI Management Commands

### Daemon

```bash
ironbee-backend-devtools-cli daemon status
ironbee-backend-devtools-cli daemon start
ironbee-backend-devtools-cli daemon stop
ironbee-backend-devtools-cli daemon restart
ironbee-backend-devtools-cli daemon info
```

### Session

```bash
ironbee-backend-devtools-cli session list
ironbee-backend-devtools-cli session info <session-id>
ironbee-backend-devtools-cli session delete <session-id>
```

### Tools

```bash
ironbee-backend-devtools-cli tools list
ironbee-backend-devtools-cli tools search <query>
ironbee-backend-devtools-cli tools info <tool-name>
```

### Config & Updates

```bash
ironbee-backend-devtools-cli config
ironbee-backend-devtools-cli update --check
```

## Examples

### Simple HTTP call with JSON body

```bash
ironbee-backend-devtools-cli --json request http \
  --url "https://api.example.com/orders" \
  --method POST \
  --body '{"kind":"json","value":{"sku":"ABC","qty":2}}' \
  --headers '{"Authorization":"Bearer $TOKEN"}'
```

### Login flow with shared cookie jar

```bash
SESSION="--session-id login-flow"

# Log in — server sets cookies, jar captures them
ironbee-backend-devtools-cli $SESSION request http \
  --url "https://api.example.com/login" \
  --method POST \
  --body '{"kind":"form","fields":{"email":"u@example.com","password":"pw"}}'

# Subsequent calls re-use the jar automatically (useCookieJar defaults to true)
ironbee-backend-devtools-cli $SESSION --json request http --url "https://api.example.com/me"
```

### gRPC unary call (via .proto file)

```bash
ironbee-backend-devtools-cli --json request grpc \
  --target "api.example.com:443" \
  --service "orders.v1.OrderService" \
  --method "GetOrder" \
  --proto-source '{"kind":"protoFile","path":"./protos/orders.proto"}' \
  --request '{"orderId":"abc-123"}' \
  --metadata '{"authorization":"Bearer $TOKEN"}'
```

### GraphQL query

```bash
ironbee-backend-devtools-cli --json request graphql \
  --url "https://api.example.com/graphql" \
  --query 'query($id: ID!) { order(id: $id) { id status } }' \
  --variables '{"id":"abc-123"}' \
  --operation-name "OrderById"
```

### WebSocket session

```bash
SESSION="--session-id ws-test"

# Open connection — returns connectionId
ironbee-backend-devtools-cli $SESSION --json request websocket-open \
  --url "wss://api.example.com/stream" \
  --subprotocols '["v1.events"]'

# Send a frame
ironbee-backend-devtools-cli $SESSION request websocket-send \
  --connection-id "<id>" \
  --data '{"kind":"json","value":{"subscribe":"orders"}}'

# Drain buffered messages (wait up to 5s, max 50)
ironbee-backend-devtools-cli $SESSION --json request websocket-receive \
  --connection-id "<id>" --max-count 50 --timeout-ms 5000

# Close
ironbee-backend-devtools-cli $SESSION request websocket-close --connection-id "<id>"
```

### Replay a captured curl command

```bash
ironbee-backend-devtools-cli --json request replay \
  --source '{"kind":"curl","value":"curl -X POST https://api.example.com/orders -d ..."}' \
  --new-trace-id
```

### Log capture around a request (correlated by trace id)

```bash
SESSION="--session-id correlated-flow"

# Pin a fresh trace id on the session
TRACE=$(ironbee-backend-devtools-cli $SESSION --json o11y new-trace-id | jq -r .traceId)

# Register the app log file and start following BEFORE the request
ironbee-backend-devtools-cli $SESSION log register-source --name app --type file --path /var/log/app.log
FOLLOW=$(ironbee-backend-devtools-cli $SESSION --json log follow --source app --max-buffer-lines 1000 | jq -r .followId)

# Trigger the operation
ironbee-backend-devtools-cli $SESSION request http --url "https://api.example.com/orders" --method POST --body '{"kind":"json","value":{}}'

# Drain just the lines tagged with our trace id
ironbee-backend-devtools-cli $SESSION --json log get-followed \
  --follow-id "$FOLLOW" --drain --pattern "$TRACE" --parse-json --select '["timestamp","level","msg"]'

ironbee-backend-devtools-cli $SESSION log stop-follow --follow-id "$FOLLOW"
```

### Database verification via snapshot + diff

```bash
SESSION="--session-id db-verify"

# Connect with creds in env (preferred — never enters agent context)
ironbee-backend-devtools-cli $SESSION db connect --name main --type postgres --connection-string-env DATABASE_URL

# Snapshot the affected rows BEFORE the request
BEFORE=$(ironbee-backend-devtools-cli $SESSION --json db snapshot \
  --connection main --table orders --where "customer_id = $1" --params '["cust-1"]' | jq -r .snapshotId)

# Trigger
ironbee-backend-devtools-cli $SESSION request http --url "https://api.example.com/orders" --method POST --body '{"kind":"json","value":{"customer_id":"cust-1"}}'

# Snapshot AFTER
AFTER=$(ironbee-backend-devtools-cli $SESSION --json db snapshot \
  --connection main --table orders --where "customer_id = $1" --params '["cust-1"]' | jq -r .snapshotId)

# Diff — added / removed / changed rows keyed by PK
ironbee-backend-devtools-cli $SESSION --json db diff --from "$BEFORE" --to "$AFTER"
```

### Writable transaction (test fixtures, always rollback)

```bash
SESSION="--session-id fixtures"

ironbee-backend-devtools-cli $SESSION db connect --name main --type postgres --connection-string-env TEST_DATABASE_URL --allow-writes

ironbee-backend-devtools-cli $SESSION db transaction-begin --connection main --writable
ironbee-backend-devtools-cli $SESSION db seed --connection main --table users \
  --rows '[{"id":"u1","email":"a@example.com"},{"id":"u2","email":"b@example.com"}]'
# ... run requests / verifications ...
ironbee-backend-devtools-cli $SESSION db transaction-rollback --connection main
```

## Interactive Mode

```bash
ironbee-backend-devtools-cli interactive
```

## Shell Completions

```bash
eval "$(ironbee-backend-devtools-cli completion bash)"
eval "$(ironbee-backend-devtools-cli completion zsh)"
```
