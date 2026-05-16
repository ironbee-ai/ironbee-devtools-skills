# Node.js Debug Commands

Non-blocking debugging for Node.js backend processes. Connect first, then set probes.

**Unified API:** Use `debug list-probes`, `debug remove-probe`, `debug clear-probes`, `debug get-probe-snapshots`, and `debug clear-probe-snapshots` for tracepoints, logpoints, and (where applicable) watches/snapshots.

## Connection

### Connect

MCP parameters: `pid`, `processName`, `containerId`, `containerName`, `host`, `inspectorPort`, `wsUrl`.

```bash
# By PID
ironbee-node-devtools-cli debug connect --pid 12345

# By process name
ironbee-node-devtools-cli debug connect --process-name "server.js"

# By Docker container
ironbee-node-devtools-cli debug connect --container-name my-app --inspector-port 9229
ironbee-node-devtools-cli debug connect --container-id <id> --host host.docker.internal --inspector-port 9229

# By inspector port (already --inspect)
ironbee-node-devtools-cli debug connect --inspector-port 9229

# Direct WebSocket URL
ironbee-node-devtools-cli debug connect --ws-url "ws://127.0.0.1:9229/abc-123"
```

### Disconnect

```bash
ironbee-node-devtools-cli debug disconnect
```

### Status

```bash
ironbee-node-devtools-cli --json debug status
```

Returns: `connected`, `enabled`, `pid`, `hasSourceMaps`, `exceptionBreakpoint`, `tracepointCount`, `logpointCount`, `watchExpressionCount`, `snapshotStats`.

## Tracepoints

`urlPattern` matches **script file paths** (e.g., `server.js`, `routes/api.ts`). Auto-escaped.

```bash
# Put tracepoint
ironbee-node-devtools-cli debug put-tracepoint \
  --url-pattern "server.js" \
  --line-number 42

# With condition
ironbee-node-devtools-cli debug put-tracepoint \
  --url-pattern "routes/users.ts" \
  --line-number 15 \
  --condition "req.user.id === 1"

# List / Remove / Clear (unified)
ironbee-node-devtools-cli debug list-probes --types tracepoint
ironbee-node-devtools-cli debug remove-probe --type tracepoint --id "tp_abc123"
ironbee-node-devtools-cli debug clear-probes --types tracepoint
```

## Logpoints

Optional: `--column-number`, `--condition`, `--hit-condition` (same semantics as tracepoint).

```bash
ironbee-node-devtools-cli debug put-logpoint \
  --url-pattern "utils.ts" \
  --line-number 10 \
  --log-expression "`Processing: ${item}`"

# With condition / hit condition
ironbee-node-devtools-cli debug put-logpoint \
  --url-pattern "utils.ts" --line-number 10 --log-expression "item" \
  --condition "item.active" --hit-condition "> 5"

# List / Remove / Clear (unified)
ironbee-node-devtools-cli debug list-probes --types logpoint
ironbee-node-devtools-cli debug remove-probe --type logpoint --id "lp_xyz"
ironbee-node-devtools-cli debug clear-probes --types logpoint
```

## Exceptionpoints

```bash
ironbee-node-devtools-cli debug put-exceptionpoint --state uncaught
ironbee-node-devtools-cli debug put-exceptionpoint --state all
ironbee-node-devtools-cli debug put-exceptionpoint --state none
```

## Snapshots (unified)

```bash
# Get all or by type
ironbee-node-devtools-cli --json debug get-probe-snapshots
ironbee-node-devtools-cli --json debug get-probe-snapshots --types tracepoint,logpoint,exceptionpoint
ironbee-node-devtools-cli --json debug get-probe-snapshots --probe-id "tp_123"
ironbee-node-devtools-cli --json debug get-probe-snapshots --from-sequence 50 --limit 20

# Clear snapshots
ironbee-node-devtools-cli debug clear-probe-snapshots
ironbee-node-devtools-cli debug clear-probe-snapshots --types tracepoint --probe-id "tp_123"
```

## Watch Expressions

```bash
ironbee-node-devtools-cli debug add-watch --expression "req.body"
ironbee-node-devtools-cli debug add-watch --expression "this.state"

# List / Remove / Clear (unified)
ironbee-node-devtools-cli debug list-probes --types watch
ironbee-node-devtools-cli debug remove-probe --type watch --id "w_abc"
ironbee-node-devtools-cli debug clear-probes --types watches
```

## Resolve Source Location

For source map resolution (generated → original source). Input: generated script URL, line, optional column (1-based).

```bash
ironbee-node-devtools-cli debug resolve-source-location \
  --url "file:///path/to/dist/app.js" \
  --line 100

# With column
ironbee-node-devtools-cli debug resolve-source-location --url "file:///dist/app.js" --line 100 --column 5
```

## Console Logs (get-logs)

Same filtering as browser `o11y get-console-messages`: single `type` (level or higher), `search`, optional `timestamp`, `sequenceNumber`, `limit` (count + from).

```bash
ironbee-node-devtools-cli --json debug get-logs
ironbee-node-devtools-cli --json debug get-logs --search "error"
ironbee-node-devtools-cli --json debug get-logs --type error
ironbee-node-devtools-cli --json debug get-logs --type warning
```

**Optional:** `--timestamp`, `--sequence-number`, `--limit` (JSON object: `{"count":100,"from":"end"}`; count 0 = no limit). `--type` values: `all`, `debug`, `info`, `warning`, `error` (filter by this level or higher).
