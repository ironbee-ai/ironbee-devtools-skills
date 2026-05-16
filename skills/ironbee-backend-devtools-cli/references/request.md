# Request Tools

HTTP / gRPC / GraphQL / WebSocket protocol surface plus session-scoped state (cookie jar, default headers, default gRPC metadata) and replay primitives.

**Base request fields shared by `http`, `grpc`, `graphql`, `replay`, `websocket-open`:** `--timeout-ms`, `--retries` (object), `--auth` (object), `--trace-id`, `--tls` (object). Object/array params are passed as a single `--<flag> <json>`, never split.

**4xx / 5xx / gRPC non-OK status are normal results, not errors** — only transport-level failures (DNS, TLS, timeout, abort) populate the response `error` field.

**Boolean flag convention (read first).** Many tool fields are typed `z.boolean().optional()` upstream. The CLI registers each of them as a single switch (e.g. `--follow-redirects`) — passing the switch sends `true`; omitting it sends `undefined`, and the server applies its default. The server-side default is shown in the `Default` column below. **`--no-<flag>` is NOT registered for these.** To explicitly send `false` (e.g. to disable cookie jar, default headers, or `parseJson`), call via MCP or `run execute` and pass the field as `false` in the JSON arguments.

---

## http

Full HTTP/1.1 and HTTP/2 (ALPN auto-negotiation). Cookie jar, default headers, and W3C trace context are wired in transparently.

```bash
ironbee-backend-devtools-cli request http --url <url> [options]
```

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--url` | string | Yes | - | Request URL (include scheme). |
| `--method` | enum | No | `GET` | `GET` / `POST` / `PUT` / `PATCH` / `DELETE` / `HEAD` / `OPTIONS`. |
| `--headers` | object | No | - | JSON header map. Merged on top of session default headers for the host. |
| `--query` | object | No | - | JSON key/value query string params. |
| `--body` | object | No | - | JSON: `{kind:'json'\|'text'\|'form'\|'multipart'\|'raw', ...}`. See body shapes below. |
| `--follow-redirects` | boolean | No | server `true` | Follow 3xx. Switch only — to send `false`, use MCP / `run execute`. |
| `--max-redirects` | number | No | `5` | Cap on followed redirects. |
| `--response-body-max-bytes` | number | No | `1048576` (1 MiB) | Truncate large response bodies. Larger bodies spill to disk as artifacts. |
| `--expect-stream` | boolean | No | `false` | Stream the response (useful for SSE / chunked endpoints). |
| `--use-cookie-jar` | boolean | No | server `true` | Use the session cookie jar. Switch only — to send `false`, use MCP / `run execute`. |
| `--use-default-headers` | boolean | No | server `true` | Merge in `request_set-default-headers` for the host. Switch only — to send `false`, use MCP / `run execute`. |
| `--timeout-ms` | number | No | (session default) | Per-call timeout. |
| `--retries` | object | No | - | JSON: `{attempts:n, delayMs:n, retryOn:['network','5xx',...]}`. |
| `--auth` | object | No | - | JSON: `{kind:'bearer'\|'basic'\|'apiKey', ...}`. For `apiKey` include `placement`. |
| `--trace-id` | string | No | (session pin or random) | Override the W3C trace id for this call only. |
| `--tls` | object | No | - | JSON: `{insecure:true}` to bypass TLS verification (testing only). |

**Body shapes (exactly one `kind`):**
- `{"kind":"json","value":<any>}`
- `{"kind":"text","value":"<string>"}`
- `{"kind":"form","fields":{"<name>":"<value>",...}}`
- `{"kind":"multipart","parts":[{"kind":"field","name":"...","value":"..."},{"kind":"file","name":"...","filePath":"...","fileName":"...","contentType":"..."}]}`
- `{"kind":"raw","contentType":"<mime>","dataBase64":"<base64>"}`

---

## grpc

gRPC with all four streaming modes (unary, client-stream, server-stream, bidi). JSON ↔ protobuf marshaling via inline descriptor or `.proto` file path.

```bash
ironbee-backend-devtools-cli request grpc --target <host:port> --service <fqn> --method <name> --proto-source <json> [options]
```

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--target` | string | Yes | - | gRPC server `host:port`. |
| `--service` | string | Yes | - | Fully-qualified service name (e.g. `orders.v1.OrderService`). |
| `--method` | string | Yes | - | RPC method name. |
| `--proto-source` | object | Yes | - | JSON: `{kind:'protoFile', path:'...'}` or `{kind:'descriptor', dataBase64:'...'}`. |
| `--streaming-mode` | enum | No | `unary` | `unary` / `clientStream` / `serverStream` / `bidi`. |
| `--request` | object | No | - | Single JSON message (for unary / server-stream). |
| `--client-messages` | array | No | - | JSON array of messages (for client-stream / bidi). |
| `--metadata` | object | No | - | gRPC metadata map. Merged on top of session default metadata for the target. |
| `--use-default-metadata` | boolean | No | server `true` | Merge in `request_set-default-metadata` for the target. Switch only — to send `false`, use MCP / `run execute`. |
| `--max-receive-bytes` | number | No | - | Per-message receive cap. |
| `--max-send-bytes` | number | No | - | Per-message send cap. |
| `--compression` | enum | No | - | `identity` / `gzip` / `deflate`. |
| `--timeout-ms`, `--retries`, `--auth`, `--trace-id`, `--tls` | | | | Base request fields. |

---

## graphql

GraphQL query / mutation via HTTP. Wraps the HTTP transport — accepts persisted queries (GET) and standard JSON POST.

```bash
ironbee-backend-devtools-cli request graphql --url <url> --query <gql> [options]
```

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--url` | string | Yes | - | GraphQL endpoint. |
| `--query` | string | Yes | - | GraphQL document (query or mutation). |
| `--variables` | object | No | - | JSON variables map. |
| `--operation-name` | string | No | - | Operation name when the document defines multiple. |
| `--method` | enum | No | `POST` | `GET` (persisted) or `POST` (default). |
| `--persisted-query` | object | No | - | JSON: `{version:1,sha256Hash:'...'}` for APQ. |
| `--headers` | object | No | - | Extra request headers. |
| `--response-body-max-bytes` | number | No | `1048576` (1 MiB) | Truncate large responses. |
| `--use-cookie-jar` | boolean | No | server `true` | Cookie jar control. Switch only. |
| `--use-default-headers` | boolean | No | server `true` | Default header merge. Switch only. |
| `--timeout-ms`, `--retries`, `--auth`, `--trace-id`, `--tls` | | | | Base request fields. |

---

## replay

Replay a previously-captured HTTP call from either a `curl` command line or a HAR entry. Parsed, then delegated to the HTTP transport.

```bash
ironbee-backend-devtools-cli request replay --source <json> [options]
```

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--source` | object | Yes | - | JSON: `{kind:'curl', value:'curl ...'}` or `{kind:'har', value:{...}}`. |
| `--overrides` | object | No | - | JSON: shallow per-field overrides (e.g. `{"headers":{"Authorization":"Bearer NEW"}}`). |
| `--new-trace-id` | boolean | No | `false` | Switch — pass to generate and pin a fresh trace id for this replay. |
| `--use-cookie-jar` | boolean | No | server `true` | Cookie jar control. Switch only. |
| `--use-default-headers` | boolean | No | server `true` | Default header merge. Switch only. |
| `--timeout-ms`, `--retries`, `--auth`, `--trace-id`, `--tls` | | | | Base request fields. |

---

## websocket-open

Open a WebSocket and register it under a session-scoped `connectionId`. Server-pushed frames are buffered until you drain them with `websocket-receive`.

```bash
ironbee-backend-devtools-cli request websocket-open --url <ws-url> [options]
```

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--url` | string | Yes | - | `ws://` or `wss://` URL. |
| `--subprotocols` | array | No | - | JSON array of subprotocols, e.g. `'["v1.events"]'`. |
| `--headers` | object | No | - | JSON header map for the upgrade request. |
| `--use-cookie-jar` | boolean | No | server `true` | Cookie jar control. Switch only. Applied to the upgrade request only. |
| `--use-default-headers` | boolean | No | server `true` | Default header merge. Switch only. Applied to the upgrade request only. |
| `--receive-buffer-size` | number | No | `1000` | Ring-buffer capacity for buffered server frames. |
| `--buffer-overflow` | enum | No | `drop-oldest` | `drop-oldest` or `drop-newest`. |
| `--max-frame-bytes` | number | No | `1048576` (1 MiB) | Reject frames over this size. Larger frames spill to disk. |
| `--include-control-frames` | boolean | No | `false` | Switch — pass to include ping / pong / close frames in receive results. |
| `--idle-timeout-ms` | number | No | `300000` (5 min) | Close the connection after this much idle time. `0` disables. |
| `--max-lifetime-ms` | number | No | `3600000` (1 h) | Force-close after this many ms regardless of activity. Pass `null` via MCP to disable; not directly settable to null from CLI. |
| `--timeout-ms`, `--retries`, `--auth`, `--trace-id`, `--tls` | | | | Base request fields. |

---

## websocket-send

Send a single frame on an open WebSocket.

```bash
ironbee-backend-devtools-cli request websocket-send --connection-id <id> --data <json>
```

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--connection-id` | string | Yes | - | Id returned by `websocket-open`. |
| `--data` | object | Yes | - | JSON: `{kind:'text', value:'...'}` / `{kind:'json', value:<any>}` / `{kind:'binary', dataBase64:'...'}`. |

---

## websocket-receive

Drain (or peek at) buffered server frames. Optionally wait until `maxCount` is reached or `timeoutMs` elapses.

```bash
ironbee-backend-devtools-cli request websocket-receive --connection-id <id> [options]
```

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--connection-id` | string | Yes | - | Id returned by `websocket-open`. |
| `--max-count` | number | No | - | Cap on returned frames. |
| `--timeout-ms` | number | No | - | Wait up to this long for `maxCount` frames. |

---

## websocket-close

Close the WebSocket, return the close frame and session totals.

```bash
ironbee-backend-devtools-cli request websocket-close --connection-id <id> [options]
```

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--connection-id` | string | Yes | - | Id returned by `websocket-open`. |
| `--code` | number | No | `1000` | WebSocket close code. |
| `--reason` | string | No | - | Close reason string. |
| `--drain-timeout-ms` | number | No | - | Wait up to this long for buffered frames before closing. |

---

## websocket-list

List all session-scoped WebSocket connections (open, opening, closing, and closed-within-linger).

```bash
ironbee-backend-devtools-cli request websocket-list
```

No arguments.

---

## set-default-headers

Pin host-scoped default headers (exact host match, scheme-independent). Case-insensitive merge or replace.

```bash
ironbee-backend-devtools-cli request set-default-headers --host <host> --headers <json> [--replace]
```

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--host` | string | Yes | - | Host the headers apply to (e.g. `api.example.com`). |
| `--headers` | object | Yes | - | JSON header map. |
| `--replace` | boolean | No | `false` | Switch — pass to replace all headers for the host; omit to merge. |

---

## clear-default-headers

Drop host-scoped default headers. Granular by host and header names.

```bash
ironbee-backend-devtools-cli request clear-default-headers [--host <host>] [--header-names <json>]
```

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--host` | string | No | (all hosts) | Restrict the clear to one host. |
| `--header-names` | array | No | (all headers) | JSON array of header names to drop (rest stay). |

---

## list-default-headers

List every host with default headers configured.

```bash
ironbee-backend-devtools-cli request list-default-headers
```

No arguments.

---

## set-default-metadata

Pin target-scoped (host:port) default gRPC metadata. Case-insensitive merge or replace.

```bash
ironbee-backend-devtools-cli request set-default-metadata --target <host:port> --metadata <json> [--replace]
```

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--target` | string | Yes | - | gRPC target (`host:port`). |
| `--metadata` | object | Yes | - | JSON metadata map. |
| `--replace` | boolean | No | `false` | Switch — pass to replace; omit to merge. |

---

## clear-default-metadata

Drop target-scoped default gRPC metadata.

```bash
ironbee-backend-devtools-cli request clear-default-metadata [--target <host:port>] [--metadata-names <json>]
```

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--target` | string | No | (all targets) | Restrict to one target. |
| `--metadata-names` | array | No | (all names) | JSON array of metadata keys to drop. |

---

## list-default-metadata

List every target with default gRPC metadata configured.

```bash
ironbee-backend-devtools-cli request list-default-metadata
```

No arguments.

---

## set-cookies

Seed cookies into the session jar. RFC 6265 scoped (domain + path + secure + sameSite).

```bash
ironbee-backend-devtools-cli request set-cookies --cookies <json>
```

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--cookies` | array | Yes | - | JSON array of cookie objects: `{name, value, domain, path?, expires?, httpOnly?, secure?, sameSite?}`. |

---

## list-cookies

Inspect the session cookie jar.

```bash
ironbee-backend-devtools-cli request list-cookies [--url <url>] [--include-expired]
```

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--url` | string | No | (all) | Filter to cookies that would be sent to this URL. |
| `--include-expired` | boolean | No | `false` | Switch — pass to include expired cookies in the result. |

---

## clear-cookies

Drop cookies from the session jar. Granular by domain and name.

```bash
ironbee-backend-devtools-cli request clear-cookies [--domain <domain>] [--names <json>]
```

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--domain` | string | No | (all) | Restrict to one domain. |
| `--names` | array | No | (all) | JSON array of cookie names to drop. |
