# Database Tools

Database verification for Postgres, MySQL, and SQLite. **Readonly by default**; writable transactions are opt-in per connection.

**Boolean flag convention.** Fields typed `z.boolean().optional()` upstream (e.g. `allowWrites`, `writable`, `allowAutoCommit`, `drain`) are exposed as a single CLI switch — passing it sends `true`; omitting it sends `undefined` and the server applies its documented default. **`--no-<flag>` is NOT registered.** To force `false`, call via MCP / `run execute`.

**Readonly enforcement (`query`, `snapshot`):**
- Parser layer: only `SELECT` / `WITH` / `EXPLAIN` / `SHOW` / `DESCRIBE` / `PRAGMA` / `VALUES` / `TABLE` leading keywords accepted; multi-statement rejected; `WITH` / `EXPLAIN` are scanned for embedded `INSERT` / `UPDATE` / `DELETE`.
- Server-side guard (the actual safety): Postgres `default_transaction_read_only=on`, MySQL `SESSION TRANSACTION READ ONLY`, SQLite OS-level readonly file open. The parser is a clean-error layer; the server is the guarantee.

**Column-name redaction** (`BACKEND_DB_QUERY_REDACT_KEYS`, default includes `password`, `token`, `api_key`, `password_hash`, …) is applied to every result path; nested JSON/JSONB column values are walked too.

---

## connect

Open a named database connection. Prefer `--connection-string-env` so the credential stays in `process.env` and never enters the agent's context.

```bash
ironbee-backend-devtools-cli db connect --name <name> --type <enum> (--connection-string <url> | --connection-string-env <var>) [--allow-writes]
```

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--name` | string | Yes | - | Connection name used by every other `db_*` tool. |
| `--type` | enum | Yes | - | `postgres` / `mysql` / `sqlite`. |
| `--connection-string` | string | Conditional | - | Engine connection string. One of `--connection-string` or `--connection-string-env` is required. |
| `--connection-string-env` | string | Conditional | - | Name of the env var holding the connection string (preferred). |
| `--allow-writes` | boolean | No | `false` | Switch — pass to open the connection in writable mode. Required to use writable transactions / `seed` with auto-commit / `run-script`. |

---

## disconnect

Close a connection. Auto-stops any watchers attached to it and drops the snapshots taken against it.

```bash
ironbee-backend-devtools-cli db disconnect --connection <name>
```

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--connection` | string | Yes | - | Connection to close. |

---

## list-connections

List all open connections with their active watcher ids and snapshot counts.

```bash
ironbee-backend-devtools-cli db list-connections
```

No arguments.

---

## list-tables

List all tables and views visible on a connection. `pg_catalog` and `information_schema` are skipped.

```bash
ironbee-backend-devtools-cli db list-tables --connection <name>
```

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--connection` | string | Yes | - | Open connection. |

---

## describe-table

Full schema for a table: columns (name, type, nullable, default, isPrimaryKey), primary key, indexes, foreign keys, and a row count estimate.

```bash
ironbee-backend-devtools-cli db describe-table --connection <name> --table <table> [--schema <schema>]
```

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--connection` | string | Yes | - | Open connection. |
| `--table` | string | Yes | - | Table name. |
| `--schema` | string | No | (engine default) | Schema name (Postgres / MySQL). |

---

## query

Run a parameterized `SELECT`. Readonly-enforced (see top of file). Multi-statement is rejected at parser level. Result-set is clipped to `--row-limit`.

```bash
ironbee-backend-devtools-cli db query --connection <name> --sql <sql> [--params <json>] [--row-limit <n>] [--timeout-ms <n>]
```

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--connection` | string | Yes | - | Open connection. |
| `--sql` | string | Yes | - | SELECT/WITH/EXPLAIN/SHOW/DESCRIBE/PRAGMA/VALUES/TABLE statement. |
| `--params` | array | No | - | JSON positional parameters (e.g. `'["abc-123",42]'`). |
| `--row-limit` | number | No | (server default) | Hard cap on returned rows. |
| `--timeout-ms` | number | No | (server default) | Per-query timeout. |

---

## snapshot

Capture a point-in-time set of rows from a table, keyed by primary key. Used with `diff` for before-vs-after assertions ("did exactly the right row change?").

```bash
ironbee-backend-devtools-cli db snapshot --connection <name> --table <table> [options]
```

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--connection` | string | Yes | - | Open connection. |
| `--table` | string | Yes | - | Table to snapshot. |
| `--schema` | string | No | (engine default) | Schema name. |
| `--where` | string | No | - | WHERE clause (use `$1`, `$2`, ... placeholders). |
| `--params` | array | No | - | JSON positional parameters for `--where`. |
| `--order-by` | string | No | - | Stable ORDER BY (helps diff readability). |
| `--row-limit` | number | No | (server default) | Cap on snapshot rows. |
| `--timeout-ms` | number | No | (server default) | Per-query timeout. |

Returns a `snapshotId` used by `diff`.

---

## diff

Compute added / removed / changed rows between two snapshots. Each changed row enumerates the columns that differ.

```bash
ironbee-backend-devtools-cli db diff --from <snapshotId> --to <snapshotId>
```

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--from` | string | Yes | - | Snapshot id (typically the "before"). |
| `--to` | string | Yes | - | Snapshot id (typically the "after"). |

---

## watch-changes

Start a polling change feed on a table. Each poll cycle's diff is pushed into a ring buffer; drain with `get-changes`.

```bash
ironbee-backend-devtools-cli db watch-changes --connection <name> --table <table> [options]
```

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--connection` | string | Yes | - | Open connection. |
| `--table` | string | Yes | - | Table to watch. |
| `--schema` | string | No | (engine default) | Schema name. |
| `--where` | string | No | - | Optional WHERE clause. |
| `--params` | array | No | - | JSON positional parameters for `--where`. |
| `--poll-interval-ms` | number | No | (server default) | Poll cadence. |
| `--max-buffer-events` | number | No | (server default) | Ring buffer size for diffs. |
| `--row-limit` | number | No | (server default) | Per-poll row cap. |
| `--timeout-ms` | number | No | (server default) | Per-poll timeout. |

Returns a `watchId` used by `get-changes`.

---

## get-changes

Drain (or peek at) buffered change events from a watcher. Each event is one poll cycle's diff (added / removed / changed). Default behavior is **drain** — events are returned and the buffer is cleared.

```bash
ironbee-backend-devtools-cli db get-changes --watch-id <id> [--drain]
```

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--watch-id` | string | Yes | - | Id returned by `watch-changes`. |
| `--drain` | boolean | No | server `true` | Switch — passing it (or omitting it) drains the buffer after returning. **There is no `--no-drain`**; to peek without clearing, call via MCP / `run execute` with `drain: false`. |

---

## transaction-begin

Open a transaction on a connection. Two modes: readonly (default) or writable. Writable mode requires the connection to have been opened with `--allow-writes`.

```bash
ironbee-backend-devtools-cli db transaction-begin --connection <name> [--writable]
```

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--connection` | string | Yes | - | Open connection. |
| `--writable` | boolean | No | `false` | Switch — pass to open a writable transaction (requires `--allow-writes` on `connect`). |

---

## transaction-commit

Commit the active transaction on a connection.

```bash
ironbee-backend-devtools-cli db transaction-commit --connection <name>
```

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--connection` | string | Yes | - | Open connection with an active transaction. |

---

## transaction-rollback

Roll back the active transaction. The canonical "test fixture cleanup" verb.

```bash
ironbee-backend-devtools-cli db transaction-rollback --connection <name>
```

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--connection` | string | Yes | - | Open connection with an active transaction. |

---

## seed

Insert structured rows into a table. Each row is a column-keyed object; the tool generates a parameterized `INSERT` per row. Requires `--allow-writes` on the connection AND either an active writable transaction OR `--allow-auto-commit`.

```bash
ironbee-backend-devtools-cli db seed --connection <name> --table <table> --rows <json> [options]
```

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--connection` | string | Yes | - | Open connection (must allow writes). |
| `--table` | string | Yes | - | Target table. |
| `--schema` | string | No | (engine default) | Schema name. |
| `--rows` | array | Yes | - | JSON array of `{column: value, ...}` row objects. |
| `--allow-auto-commit` | boolean | No | `false` | Switch — pass to permit insert outside an explicit transaction. Default is to require an active writable transaction. |
| `--timeout-ms` | number | No | (server default) | Per-statement timeout. |

---

## run-script

Execute a raw multi-statement SQL script. Bypasses the readonly parser. Required for DDL (`CREATE TABLE`, `ALTER`, `DROP`). Requires `--allow-writes` on the connection.

```bash
ironbee-backend-devtools-cli db run-script --connection <name> --sql <sql> [--timeout-ms <n>]
```

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--connection` | string | Yes | - | Open connection (must allow writes). |
| `--sql` | string | Yes | - | Multi-statement SQL body. |
| `--timeout-ms` | number | No | (server default) | Per-statement timeout. |
