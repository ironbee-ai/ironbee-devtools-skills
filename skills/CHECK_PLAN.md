# CLI / Ref / Skill Check Plan (MCP Tool Flags)

CLI flags are derived from MCP `inputSchema` keys via **camelCase → kebab-case** (e.g. `delayMs` → `--delay-ms`). Object/array params become a single `--param <json>`.

---

## 1. Browser tools: Tool → Allowed CLI flags

| Category | Tool (CLI subcommand) | Allowed flags (--kebab-case) |
|----------|------------------------|------------------------------|
| **content** | take-screenshot | `output-path`, `name`, `selector`, `full-page`, `type`, `quality`, `include-base64`, `annotate`, `annotate-content`, `annotate-cursor-interactive` |
| **content** | save-as-pdf | `output-path`, `name`, `format`, `print-background`, `margin` (object) |
| **content** | get-as-text | `selector`, `max-length` |
| **content** | get-as-html | `selector`, `remove-scripts`, `remove-comments`, `remove-styles`, `remove-meta`, `clean-html`, `minify`, `max-length` |
| **content** | start-recording | `output-dir`, `name` |
| **content** | stop-recording | (none) |
| **navigation** | go-to | `url`, `timeout`, `wait-until`, `wait-for-navigation`, `wait-for-timeout-ms`, `include-snapshot`, `snapshot-options` (object), `include-screenshot`, `screenshot-options` (object) |
| **navigation** | go-back-or-forward | `direction`, `timeout`, `wait-until`, `wait-for-navigation`, `wait-for-timeout-ms`, `include-snapshot`, `snapshot-options` (object), `include-screenshot`, `screenshot-options` (object) |
| **navigation** | reload | `timeout`, `wait-until`, `wait-for-navigation`, `wait-for-timeout-ms`, `include-snapshot`, `snapshot-options` (object), `include-screenshot`, `screenshot-options` (object) |
| **interaction** | click | `selector`, `timeout-ms`, `wait-for-navigation`, `wait-for-timeout-ms` |
| **interaction** | hover | `selector`, `timeout-ms` |
| **interaction** | select | `selector`, `value`, `timeout-ms` |
| **interaction** | fill | `selector`, `value`, `timeout-ms` |
| **interaction** | scroll | `mode`, `selector`, `dx`, `dy`, `x`, `y`, `behavior` |
| **interaction** | resize-viewport | `width`, `height` |
| **interaction** | resize-window | `width`, `height`, `state` |
| **interaction** | press-key | `key`, `selector`, `hold-ms`, `repeat`, `repeat-interval-ms`, `timeout-ms` |
| **interaction** | drag | `source-selector`, `target-selector`, `timeout-ms` |
| **sync** | wait-for-network-idle | `timeout-ms`, `idle-time-ms`, `max-connections`, `poll-interval-ms` |
| **stub** | intercept-http-request | `pattern`, `modifications`, `delay-ms`, `times` |
| **stub** | mock-http-response | `pattern`, `response`, `delay-ms`, `times`, `chance` (+ shortcuts: `status`, `headers`, `body`) |
| **stub** | list | (none) |
| **stub** | clear | `stub-id` |
| **o11y** | get-web-vitals | `wait-ms`, `include-debug` |
| **o11y** | get-console-messages | `type`, `search`, `timestamp`, `sequence-number`, `limit` (object) |
| **o11y** | get-http-requests | `resource-type`, `status` (object), `ok`, `timestamp`, `sequence-number`, `limit` (object), `include-request-headers`, `include-response-headers`, `include-response-body` |
| **o11y** | set-trace-context | `trace-id`, `trace-state` |
| **o11y** | get-trace-context | (none) |
| **o11y** | new-trace-id | (none) |
| **a11y** | take-aria-snapshot | `selector`, `interactive-only`, `max-depth`, `compact`, `cursor-interactive` |
| **a11y** | take-ax-tree-snapshot | `roles`, `include-styles`, `include-runtime-visual`, `check-occlusion`, `only-visible`, `only-in-viewport`, `text-preview-max-length`, `style-properties` |
| **debug** | put-tracepoint | `url-pattern`, `line-number`, `column-number`, `condition`, `hit-condition` |
| **debug** | put-logpoint | `url-pattern`, `line-number`, `log-expression`, `column-number`, `condition`, `hit-condition` |
| **debug** | put-exceptionpoint | `state` |
| **debug** | get-probe-snapshots | `types`, `probe-id`, `from-sequence`, `limit` (number), `max-call-stack-depth`, `include-scopes`, `max-variables-per-scope` |
| **debug** | clear-probe-snapshots | `types`, `probe-id` |
| **debug** | list-probes | `types` |
| **debug** | remove-probe | `type`, `id` |
| **debug** | clear-probes | `types` |
| **debug** | add-watch | `expression` |
| **debug** | status | (none) |
| **debug** | resolve-source-location | `url`, `line`, `column` |
| **react** | get-component-for-element | `selector`, `x`, `y`, `max-stack-depth`, `include-props-preview`, `max-props-preview-chars` |
| **react** | get-element-for-component | `anchor-selector`, `anchor-x`, `anchor-y`, `component-name`, `match-strategy`, `file-name-hint`, `line-number`, `max-elements`, `only-visible`, `only-in-viewport`, `text-preview-max-length`, `max-matches` |
| **figma** | compare-page-with-design | `figma-file-key`, `figma-node-id`, `selector`, `full-page`, `figma-scale`, `figma-format`, `weights` (object), `mssim-mode`, `max-dim`, `jpeg-quality` |
| **run** | execute | `code`, `file`, `timeout-ms` |
| **default** | scenario-add | `name`, `description`, `script`, `scope` |
| **default** | scenario-update | `name`, `description`, `script`, `scope` |
| **default** | scenario-delete | `name`, `scope` |
| **default** | scenario-list | `scope` |
| **default** | scenario-search | `query`, `limit` (number) |
| **MCP-only** | scenario-run | `name`, `timeout-ms` (MCP fields) — NOT a direct CLI subcommand |

Scenarios are platform-agnostic shared tools — flat names like `scenario-add` (no `domain_` prefix), so the CLI exposes them under the auto-generated `default` group. They're available on all three CLIs (browser, node, backend). **`scenario-run` is registered in the daemon's tool registry only, not in `cliProvider.tools`** — to run a scenario from the CLI, invoke `run execute` and `await callTool('scenario-run', { name: '...' })` from inside the script.

---

## 2. Node tools: Tool → Allowed CLI flags

| Category | Tool (CLI subcommand) | Allowed flags |
|----------|------------------------|---------------|
| **debug** | connect | `pid`, `process-name`, `container-id`, `container-name`, `host`, `inspector-port`, `ws-url` |
| **debug** | disconnect | (none) |
| **debug** | status | (none) |
| **debug** | put-tracepoint | `url-pattern`, `line-number`, `column-number`, `condition`, `hit-condition` |
| **debug** | put-logpoint | `url-pattern`, `line-number`, `column-number`, `log-expression`, `condition`, `hit-condition` |
| **debug** | put-exceptionpoint | `state` |
| **debug** | get-probe-snapshots | `types`, `probe-id`, `from-sequence`, `limit` (number), `max-call-stack-depth`, `include-scopes`, `max-variables-per-scope` |
| **debug** | clear-probe-snapshots | `types`, `probe-id` |
| **debug** | list-probes | `types` |
| **debug** | remove-probe | `type`, `id` |
| **debug** | clear-probes | `types` |
| **debug** | add-watch | `expression` |
| **debug** | get-logs | `type`, `search`, `timestamp`, `sequence-number`, `limit` (object) |
| **debug** | resolve-source-location | `url`, `line`, `column` |
| **run** | execute | `code`, `file`, `timeout-ms` |
| **default** | scenario-{add,update,delete,list,search} | (see §1) — scenario-run is MCP-only, invoke via `run execute` |

---

## 3. Backend tools: Tool → Allowed CLI flags

Base request fields shared by `request_http`, `request_grpc`, `request_graphql`, `request_replay`, `request_websocket-open`: `timeout-ms`, `retries` (object), `auth` (object), `trace-id`, `tls` (object).

| Category | Tool (CLI subcommand) | Allowed flags |
|----------|------------------------|---------------|
| **request** | http | `url`, `method`, `headers` (object), `query` (object), `body` (object), `follow-redirects`, `max-redirects`, `response-body-max-bytes`, `expect-stream`, `use-cookie-jar`, `use-default-headers`, + base |
| **request** | grpc | `target`, `service`, `method`, `proto-source` (object), `streaming-mode`, `request` (object), `client-messages` (array), `metadata` (object), `use-default-metadata`, `max-receive-bytes`, `max-send-bytes`, `compression`, + base |
| **request** | graphql | `url`, `query`, `variables` (object), `operation-name`, `method`, `persisted-query` (object), `headers` (object), `response-body-max-bytes`, `use-cookie-jar`, `use-default-headers`, + base |
| **request** | replay | `source` (object), `overrides` (object), `new-trace-id`, `use-cookie-jar`, `use-default-headers`, + base |
| **request** | websocket-open | `url`, `subprotocols` (array), `headers` (object), `use-cookie-jar`, `use-default-headers`, `receive-buffer-size`, `buffer-overflow`, `max-frame-bytes`, `include-control-frames`, `idle-timeout-ms`, `max-lifetime-ms`, + base |
| **request** | websocket-send | `connection-id`, `data` (object) |
| **request** | websocket-receive | `connection-id`, `max-count`, `timeout-ms` |
| **request** | websocket-close | `connection-id`, `code`, `reason`, `drain-timeout-ms` |
| **request** | websocket-list | (none) |
| **request** | set-default-headers | `host`, `headers` (object), `replace` |
| **request** | clear-default-headers | `host`, `header-names` (array) |
| **request** | list-default-headers | (none) |
| **request** | set-default-metadata | `target`, `metadata` (object), `replace` |
| **request** | clear-default-metadata | `target`, `metadata-names` (array) |
| **request** | list-default-metadata | (none) |
| **request** | set-cookies | `cookies` (array) |
| **request** | list-cookies | `url`, `include-expired` |
| **request** | clear-cookies | `domain`, `names` (array) |
| **log** | register-source | `name`, `type`, `path`, `container`, `pod`, `kubernetes-container`, `namespace` |
| **log** | unregister-source | `name` |
| **log** | list-sources | (none) |
| **log** | check-source | `name` |
| **log** | read | `source`, `tail`, `since`, `until`, `pattern`, `level`, `limit`, `parse-json`, `json-filter` (object), `coalesce` (object/boolean), `context-before`, `context-after`, `select` (array) |
| **log** | read-multi | `sources` (array), `tail`, `since`, `until`, `pattern`, `level`, `limit`, `parse-json`, `json-filter` (object), `coalesce` (object/boolean), `context-before`, `context-after`, `select` (array), `sort-by-timestamp` |
| **log** | follow | `source`, `max-buffer-lines` |
| **log** | get-followed | `follow-id`, `drain`, `pattern`, `level`, `limit`, `since`, `until`, `parse-json`, `json-filter` (object), `coalesce` (object/boolean), `context-before`, `context-after`, `select` (array) |
| **log** | stop-follow | `follow-id` |
| **db** | connect | `name`, `type`, `connection-string`, `connection-string-env`, `allow-writes` |
| **db** | disconnect | `connection` |
| **db** | list-connections | (none) |
| **db** | list-tables | `connection` |
| **db** | describe-table | `connection`, `table`, `schema` |
| **db** | query | `connection`, `sql`, `params` (array), `row-limit`, `timeout-ms` |
| **db** | snapshot | `connection`, `table`, `schema`, `where`, `params` (array), `order-by`, `row-limit`, `timeout-ms` |
| **db** | diff | `from`, `to` |
| **db** | watch-changes | `connection`, `table`, `schema`, `where`, `params` (array), `poll-interval-ms`, `max-buffer-events`, `row-limit`, `timeout-ms` |
| **db** | get-changes | `watch-id`, `drain` |
| **db** | transaction-begin | `connection`, `writable` |
| **db** | transaction-commit | `connection` |
| **db** | transaction-rollback | `connection` |
| **db** | seed | `connection`, `table`, `schema`, `rows` (array), `allow-auto-commit`, `timeout-ms` |
| **db** | run-script | `connection`, `sql`, `timeout-ms` |
| **o11y** | new-trace-id | (none) |
| **o11y** | set-trace-context | `trace-id`, `trace-state` |
| **o11y** | get-trace-context | (none) |
| **run** | execute | `code`, `file`, `timeout-ms` |
| **default** | scenario-{add,update,delete,list,search} | (see §1) — scenario-run is MCP-only, invoke via `run execute` |

---

## 4. Forbidden patterns (refs + skills must NOT use)

- `--figma-url` (use `--figma-file-key` + `--figma-node-id`)
- `--mssim-weight`, `--image-weight`, `--text-weight` (use `--weights` JSON)
- `--limit-count`, `--limit-from` (use `--limit` JSON with `count`/`from`)
- `--full-page true` or `--full-page false` (boolean = flag only: `--full-page`)
- scroll: `--direction`, `--amount` (use `--mode`, `--dx`, `--dy`)
- navigation: `--snapshot-interactive-only`, `--snapshot-cursor-interactive` (use `--snapshot-options` JSON, e.g. `'{"interactiveOnly":true,"cursorInteractive":true}'`)
- get-http-requests: `--status-min`, `--status-max` (use `--status` JSON, e.g. `'{"min":400,"max":599}'`)
- backend `log` filters: do NOT split `--json-filter` / `--coalesce` / `--select` into multiple flags (each is a single object/array param)
- backend `db` writes: do NOT use `--writable` without an `--allow-writes` connection, and do NOT use `db seed` without either an active writable transaction or `--allow-auto-commit`
- backend cookies / default headers / metadata: do NOT pass `--cookies` / `--headers` / `--metadata` as repeated KV flags — they are single JSON params
- legacy CLI binary names: do NOT use `browser-devtools-cli`, `node-devtools-cli`, or `browser-devtools-mcp` — the binaries are `ironbee-browser-devtools-cli` (alias `ironbee-devtools-cli`), `ironbee-node-devtools-cli`, `ironbee-backend-devtools-cli`
- `default scenario-run`: this CLI subcommand does NOT exist (`scenario-run` is registered in the daemon's tool registry only). To run a scenario from CLI, use `run execute --code "await callTool('scenario-run', { name: '...' })"`
- `--no-<flag>` for backend boolean fields whose Zod schema is `z.boolean().optional()` without `.default(true)` (e.g. `follow-redirects`, `use-cookie-jar`, `use-default-headers`, `use-default-metadata`, `include-control-frames`, `drain`, `allow-writes`, `allow-auto-commit`, `expect-stream`, `new-trace-id`, `replace`, `include-expired`, `sort-by-timestamp`, `writable`, `parse-json`): only `--<flag>` is registered as a switch. To send `false`, call via MCP / `run execute` instead.

---

## 5. Ref files → tools they document

| Ref file | Tools |
|----------|--------|
| ironbee-browser-devtools-cli/references/content.md | take-screenshot, get-as-html, get-as-text, save-as-pdf, start-recording, stop-recording |
| ironbee-browser-devtools-cli/references/navigation.md | go-to, go-back-or-forward, reload |
| ironbee-browser-devtools-cli/references/interaction.md | click, fill, hover, select, press-key, scroll, drag, resize-viewport, resize-window |
| ironbee-browser-devtools-cli/references/sync.md | wait-for-network-idle |
| ironbee-browser-devtools-cli/references/stub.md | intercept-http-request, mock-http-response, list, clear |
| ironbee-browser-devtools-cli/references/o11y.md | get-web-vitals, get-console-messages, get-http-requests, set-trace-context, get-trace-context, new-trace-id |
| ironbee-browser-devtools-cli/references/a11y.md | take-aria-snapshot, take-ax-tree-snapshot |
| ironbee-browser-devtools-cli/references/debug.md | put-tracepoint, put-logpoint, put-exceptionpoint, list-probes, remove-probe, clear-probes, get-probe-snapshots, clear-probe-snapshots, add-watch, status, resolve-source-location |
| ironbee-browser-devtools-cli/references/react.md | get-component-for-element, get-element-for-component |
| ironbee-browser-devtools-cli/references/figma.md | compare-page-with-design |
| ironbee-browser-devtools-cli/references/execute.md | execute |
| ironbee-browser-devtools-cli/references/scenario.md | scenario-add, scenario-update, scenario-delete, scenario-list, scenario-search, scenario-run |
| ironbee-node-devtools-cli/references/debug.md | connect, disconnect, status, put-tracepoint, put-logpoint, put-exceptionpoint, get-probe-snapshots, clear-probe-snapshots, list-probes, remove-probe, clear-probes, add-watch, get-logs, resolve-source-location |
| ironbee-backend-devtools-cli/references/request.md | http, grpc, graphql, replay, websocket-open, websocket-send, websocket-receive, websocket-close, websocket-list, set-default-headers, clear-default-headers, list-default-headers, set-default-metadata, clear-default-metadata, list-default-metadata, set-cookies, list-cookies, clear-cookies |
| ironbee-backend-devtools-cli/references/log.md | register-source, unregister-source, list-sources, check-source, read, read-multi, follow, get-followed, stop-follow |
| ironbee-backend-devtools-cli/references/db.md | connect, disconnect, list-connections, list-tables, describe-table, query, snapshot, diff, watch-changes, get-changes, transaction-begin, transaction-commit, transaction-rollback, seed, run-script |
| ironbee-backend-devtools-cli/references/o11y.md | new-trace-id, set-trace-context, get-trace-context |

---

## 6. Check steps (run repeatedly)

Scope: all files under `skills/` (refs + SKILL.md), **excluding** this file (CHECK_PLAN.md).

1. **Forbidden grep:** Search `skills/` for forbidden patterns; fix any match in refs or SKILL.md.
   - Patterns: `--figma-url`, `--mssim-weight`, `--image-weight`, `--text-weight`, `--limit-count`, `--limit-from`, `--full-page true`, `--full-page false`, `scroll.*--direction`, `scroll.*--amount`, `--snapshot-interactive-only`, `--snapshot-cursor-interactive`, `--status-min`, `--status-max`, legacy binary names (`browser-devtools-cli`, `node-devtools-cli`, `browser-devtools-mcp`) without `ironbee-` prefix.
   - For scroll, only the **interaction scroll** tool is forbidden `--direction`/`--amount`; navigation `go-back-or-forward --direction` is allowed.
2. **§5 vs refs:** Each ref path in §5 must exist under `skills/`. Each ref's content (sections/headings) must cover the tools listed in §5 for that ref (no tool in §5 missing from the ref, no ref file missing from §5).
3. **Ref argument tables:** In each ref under `ironbee-browser-devtools-cli/references/`, `ironbee-node-devtools-cli/references/`, and `ironbee-backend-devtools-cli/references/`, every `| \`--...\` |` must be an allowed flag for the tool(s) that ref documents. Object/array params (limit, weights, margin, response, modifications, headers, query, body, metadata, request, client-messages, proto-source, source, overrides, subprotocols, header-names, metadata-names, cookies, names, json-filter, coalesce, select, sources, params, rows, persisted-query, data, and any other object/array in §1/§2/§3) must be a single `--param`, not split (e.g. no --limit-count, no --limit-from).
4. **Skill examples:** Every `ironbee-browser-devtools-cli` / `ironbee-node-devtools-cli` / `ironbee-backend-devtools-cli` command in SKILL.md files must use only allowed flags; booleans as flag only (e.g. `--full-page` not `--full-page true`). Object params in examples must use single `--param` with JSON (e.g. `--limit '{"count":10,"from":"end"}'`).
5. **Run 3 full passes;** if any pass finds an issue, fix and re-run from step 1.

**Maintenance:** When MCP adds or renames tools, update §1 / §2 / §3 (allowed flags) and §5 (ref → tools) so the plan stays the source of truth for checks.
