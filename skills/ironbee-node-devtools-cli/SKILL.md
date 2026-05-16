---
name: ironbee-node-devtools-cli
description: CLI for debugging Node.js backend processes with non-blocking inspection. Use when the user needs to connect to Node.js processes (by PID, name, Docker, or port), set tracepoints/logpoints/exceptionpoints, capture call stacks and local variables, inspect console logs, or run reusable scenarios. Requires daemon; connect before other debug commands.
allowed-tools: Bash(ironbee-node-devtools-cli:*)
---

# IronBee Node DevTools CLI

Command-line interface for non-blocking debugging of Node.js backend processes. Part of [IronBee DevTools](https://github.com/ironbee-ai/ironbee-devtools). Connects via the Inspector Protocol (Chrome DevTools Protocol) and provides tracepoints, logpoints, exceptionpoints, and watch expressions without pausing execution.

## Installation

```bash
# Install this skill (skills.sh)
npx skills add ironbee-ai/ironbee-devtools-skills

# Install the CLI binary (same package as the rest of IronBee DevTools)
npm install -g @ironbee-ai/devtools
```

## Port note

The browser CLI (`ironbee-devtools-cli` / `ironbee-browser-devtools-cli`) and the node CLI both default to daemon port `2020`. If you want both daemons running at once, start the node daemon on a different port and pass `--port` to every node CLI call:

```bash
PLATFORM=node ironbee-node-devtools-cli daemon start --port 2021
ironbee-node-devtools-cli --port 2021 debug connect --pid 12345
```

## Quick Start

```bash
# 1. Start daemon (if not running)
ironbee-node-devtools-cli daemon start

# 2. Connect to a Node.js process (by PID)
ironbee-node-devtools-cli --session-id my-debug debug connect --pid 12345

# 3. Set a tracepoint on server.js line 42
ironbee-node-devtools-cli --session-id my-debug debug put-tracepoint \
  --url-pattern "server.js" \
  --line-number 42

# 4. Trigger the code path (e.g., make API request to your app)
# 5. Get captured snapshots
ironbee-node-devtools-cli --session-id my-debug --json debug get-probe-snapshots
```

## Global Options

| Option | Description | Default |
|--------|-------------|---------|
| `--port <number>` | Daemon server port | `2020` |
| `--session-id <string>` | Session for Node connection persistence | auto |
| `--json` | Output as JSON (recommended for AI) | `false` |
| `--quiet` | Suppress log messages | `false` |
| `--verbose` | Enable debug output | `false` |
| `--timeout <ms>` | Operation timeout | `30000` |

**AI Agent Recommended:**

```bash
ironbee-node-devtools-cli --json --quiet --session-id "debug-session" <command>
```

## Tool Domains

| Domain | Description | Reference |
|--------|-------------|-----------|
| debug | Connection, tracepoints, logpoints, exceptionpoints, watch, snapshots | [debug](./references/debug.md) |
| scenario | Reusable JS scripts (add, update, delete, list, search, run) — see browser CLI [scenario](../ironbee-browser-devtools-cli/references/scenario.md) reference; same surface |
| execute | Batch JavaScript execution (run execute; CLI and MCP) — see browser CLI [execute](../ironbee-browser-devtools-cli/references/execute.md) reference. **Note:** the `page` binding is browser-only; on node only `callTool` is available inside the VM. |

## Connection Methods

Connect via `debug connect` with one of:

| Method | Option | Example |
|--------|--------|---------|
| PID | `--pid <number>` | `--pid 12345` |
| Process name | `--process-name <pattern>` | `--process-name "server.js"` |
| Docker container | `--container-id` or `--container-name` | `--container-name my-api` |
| Inspector port | `--inspector-port <number>` | `--inspector-port 9229` |
| WebSocket URL | `--ws-url <url>` | `--ws-url "ws://127.0.0.1:9229/abc"` |

If the process doesn't have `--inspect` active, the CLI activates it via SIGUSR1 (no code changes). For Docker: expose port 9229 and use `--inspect=0.0.0.0:9229`.

## CLI Management Commands

### Daemon

```bash
ironbee-node-devtools-cli daemon status
ironbee-node-devtools-cli daemon start
ironbee-node-devtools-cli daemon stop
ironbee-node-devtools-cli daemon restart
ironbee-node-devtools-cli daemon info
```

### Session

```bash
ironbee-node-devtools-cli session list
ironbee-node-devtools-cli session info <session-id>
ironbee-node-devtools-cli session delete <session-id>
```

### Tools

```bash
ironbee-node-devtools-cli tools list
ironbee-node-devtools-cli tools search <query>
ironbee-node-devtools-cli tools info <tool-name>
```

### Config & Updates

```bash
ironbee-node-devtools-cli config
ironbee-node-devtools-cli update --check
```

## Examples

### Connect by PID

```bash
SESSION="--session-id api-debug"

# Connect
ironbee-node-devtools-cli $SESSION debug connect --pid $(pgrep -f "node server.js")

# Set tracepoint on route handler
ironbee-node-devtools-cli $SESSION debug put-tracepoint \
  --url-pattern "routes/api.ts" \
  --line-number 25

# Trigger: curl http://localhost:3000/api/users
# Get snapshots
ironbee-node-devtools-cli $SESSION --json debug get-probe-snapshots
```

### Connect by Process Name

```bash
ironbee-node-devtools-cli debug connect --process-name "api"
```

### Docker Container

```bash
# App runs in container with -p 9229:9229
ironbee-node-devtools-cli debug connect \
  --container-name my-node-app \
  --host host.docker.internal \
  --inspector-port 9229
```

### Exception Catching

```bash
SESSION="--session-id exc-debug"

ironbee-node-devtools-cli $SESSION debug connect --pid 12345
ironbee-node-devtools-cli $SESSION debug put-exceptionpoint --state uncaught

# Trigger error in app
# Check snapshots
ironbee-node-devtools-cli $SESSION --json debug get-probe-snapshots --types exceptionpoint
```

### Batch with execute

```bash
# Run JavaScript in the session VM — node platform only exposes callTool (no `page`)
ironbee-node-devtools-cli run execute --code "await callTool('debug_status', {}, true); await callTool('debug_list-probes', {}, true);"
```

## Interactive Mode

```bash
ironbee-node-devtools-cli interactive
```

| Command | Description |
|---------|-------------|
| `help` | Show commands |
| `exit`, `quit` | Exit |
| `debug connect` | Connect to process |
| `debug status` | Connection status |
| `<domain> <tool>` | Execute tool |

## Shell Completions

```bash
eval "$(ironbee-node-devtools-cli completion bash)"
eval "$(ironbee-node-devtools-cli completion zsh)"
```
