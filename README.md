# IronBee DevTools Skills

Skills for [IronBee DevTools](https://github.com/ironbee-ai/ironbee-devtools) — AI agent skills for browser automation, Node.js debugging, and backend service verification (HTTP/gRPC/GraphQL/WS + log capture + database).

## Installation

Install IronBee DevTools as a skill for AI coding agents (Claude Code, Cursor, Windsurf, etc.) using the [skills.sh](https://skills.sh) ecosystem:

```bash
npx skills add ironbee-ai/ironbee-devtools-skills
```

Then install the CLI binary itself:

```bash
npm install -g @ironbee-ai/devtools
```

## Available Skills

### CLI Skills

| Skill | Description |
|-------|-------------|
| [ironbee-browser-devtools-cli](skills/ironbee-browser-devtools-cli/SKILL.md) | Browser platform CLI (Playwright-based; navigation, content, interaction, a11y, o11y, debug, stub, sync, react, figma, scenario, execute) |
| [ironbee-node-devtools-cli](skills/ironbee-node-devtools-cli/SKILL.md) | Node.js platform CLI (Inspector-protocol debugging: tracepoints, logpoints, exceptionpoints, watches) |
| [ironbee-backend-devtools-cli](skills/ironbee-backend-devtools-cli/SKILL.md) | Backend platform CLI (HTTP/gRPC/GraphQL/WS request, log capture, Postgres/MySQL/SQLite verification) |

**ironbee-browser-devtools-cli Domain References:**
- [navigation](skills/ironbee-browser-devtools-cli/references/navigation.md) - Page navigation (go-to, go-back-or-forward, reload)
- [content](skills/ironbee-browser-devtools-cli/references/content.md) - Content extraction (screenshot, PDF, HTML, text)
- [interaction](skills/ironbee-browser-devtools-cli/references/interaction.md) - User interactions (click, fill, hover, scroll)
- [a11y](skills/ironbee-browser-devtools-cli/references/a11y.md) - Accessibility snapshots (ARIA, AX tree)
- [o11y](skills/ironbee-browser-devtools-cli/references/o11y.md) - Observability (Web Vitals, console, HTTP, traces)
- [debug](skills/ironbee-browser-devtools-cli/references/debug.md) - Non-blocking debugging (tracepoints, logpoints, exceptions)
- [stub](skills/ironbee-browser-devtools-cli/references/stub.md) - HTTP mocking (intercept, mock, clear)
- [sync](skills/ironbee-browser-devtools-cli/references/sync.md) - Synchronization (wait for network idle)
- [react](skills/ironbee-browser-devtools-cli/references/react.md) - React DevTools integration
- [figma](skills/ironbee-browser-devtools-cli/references/figma.md) - Figma design comparison
- [scenario](skills/ironbee-browser-devtools-cli/references/scenario.md) - Reusable JS scripts (add, update, delete, list, search, run)
- [execute](skills/ironbee-browser-devtools-cli/references/execute.md) - Batch JavaScript execution (run execute; CLI and MCP)

**ironbee-node-devtools-cli References:**
- [debug](skills/ironbee-node-devtools-cli/references/debug.md) - Connection, tracepoints, logpoints, exceptionpoints, snapshots

**ironbee-backend-devtools-cli References:**
- [request](skills/ironbee-backend-devtools-cli/references/request.md) - HTTP / gRPC / GraphQL / WebSocket + cookies + default headers/metadata + curl/HAR replay
- [log](skills/ironbee-backend-devtools-cli/references/log.md) - File / Docker / Kubernetes log capture (read, follow, multi-source, filter, coalesce)
- [db](skills/ironbee-backend-devtools-cli/references/db.md) - Postgres / MySQL / SQLite verification (query, snapshot/diff, watch-changes, transactions, seed, run-script)
- [o11y](skills/ironbee-backend-devtools-cli/references/o11y.md) - Session-pinned W3C trace context

### Task-Specific Skills

| Skill | Description |
|-------|-------------|
| [browser-testing](skills/browser-testing/SKILL.md) | Browser automation, interaction, and form testing |
| [backend-testing](skills/backend-testing/SKILL.md) | Backend API testing (HTTP/gRPC/GraphQL/WS) with correlated log + db verification |
| [debugging](skills/debugging/SKILL.md) | Console, network, tracepoints, logpoints, and exception monitoring |
| [visual-testing](skills/visual-testing/SKILL.md) | Screenshots, responsive testing, and Figma comparison |
| [performance-audit](skills/performance-audit/SKILL.md) | Web Vitals and performance analysis |
| [accessibility-audit](skills/accessibility-audit/SKILL.md) | WCAG compliance and ARIA validation |
| [api-testing](skills/api-testing/SKILL.md) | Browser-side API mocking and request interception (see `backend-testing` for server-side API testing) |
| [react-debugging](skills/react-debugging/SKILL.md) | React component inspection |
| [observability](skills/observability/SKILL.md) | Distributed tracing and monitoring |

## Quick Start

```bash
# Install globally
npm install -g @ironbee-ai/devtools

# Browser automation
ironbee-browser-devtools-cli navigation go-to --url "https://example.com"
ironbee-browser-devtools-cli content take-screenshot --name "homepage"
ironbee-browser-devtools-cli content get-as-text

# Node.js backend debugging (requires PLATFORM=node daemon; use --port 2021 if browser daemon is on 2020)
PLATFORM=node ironbee-node-devtools-cli daemon start --port 2021
ironbee-node-devtools-cli --port 2021 debug connect --pid 12345
ironbee-node-devtools-cli --port 2021 debug put-tracepoint --url-pattern "server.js" --line-number 42

# Backend service verification (use --port 2022 if other daemons are running)
PLATFORM=backend ironbee-backend-devtools-cli daemon start --port 2022
ironbee-backend-devtools-cli --port 2022 --json request http --url "https://api.example.com/health"
ironbee-backend-devtools-cli --port 2022 db connect --name main --type postgres --connection-string-env DATABASE_URL
```

`ironbee-browser-devtools-cli` and `ironbee-devtools-cli` are aliases — same binary, browser platform.

## License

Elastic License 2.0 (ELv2)
