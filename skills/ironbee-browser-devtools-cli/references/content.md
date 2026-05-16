# Content Tools

Content extraction and capture commands.

## take-screenshot

Take a screenshot of the current page or a specific element.

The screenshot is always saved to the file system and the file path is returned.
By default, the image data is NOT included in the response to reduce payload size.
Use `--include-base64` when the AI assistant cannot access the MCP server's file system.

```bash
ironbee-browser-devtools-cli content take-screenshot [options]
```

**Arguments:**

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--output-path` | string | No | OS temp dir | Directory to save screenshot |
| `--name` | string | No | `screenshot` | Screenshot name (file: `{name}-{time}.{type}`) |
| `--selector` | string | No | - | CSS selector for element screenshot |
| `--full-page` | boolean | No | `false` | Capture full scrollable page |
| `--type` | enum | No | `png` | Image format: `png` or `jpeg` |
| `--quality` | number | No | `100` | JPEG quality (0-100, ignored for PNG) |
| `--include-base64` | boolean | No | `false` | Include image data in response (for remote MCP servers) |
| `--annotate` | boolean | No | `false` | Overlay numbered labels [1],[2] on interactive elements (from last ARIA snapshot refs). Labels map to e1,e2,... |
| `--annotate-content` | boolean | No | `false` | With annotate: also include content elements (headings, list items) in overlay |
| `--annotate-cursor-interactive` | boolean | No | `false` | With annotate: include cursor-interactive elements (clickable without ARIA role). Takes fresh snapshot |

**When to use `--include-base64`:**
- Remote MCP server (different machine)
- Containerized/sandboxed environment
- AI assistant cannot access the file system where MCP server runs

**Examples:**

```bash
# Basic screenshot (file path only)
ironbee-browser-devtools-cli content take-screenshot --name "homepage"

# Full page screenshot
ironbee-browser-devtools-cli content take-screenshot --name "full" --full-page

# Element screenshot
ironbee-browser-devtools-cli content take-screenshot --selector "#main-content" --name "content"

# JPEG with quality
ironbee-browser-devtools-cli content take-screenshot --type jpeg --quality 80 --name "compressed"

# Custom output path
ironbee-browser-devtools-cli content take-screenshot --output-path "/tmp/screenshots" --name "test"

# Include image data in response (for remote access)
ironbee-browser-devtools-cli --json content take-screenshot --name "test" --include-base64

# JSON output for parsing file path
ironbee-browser-devtools-cli --json content take-screenshot --name "test"

# Annotated screenshot (overlay ref labels) - call a11y take-aria-snapshot first
ironbee-browser-devtools-cli a11y take-aria-snapshot
ironbee-browser-devtools-cli content take-screenshot --annotate --name "annotated"

# Include content elements (headings) in annotation
ironbee-browser-devtools-cli content take-screenshot --annotate --annotate-content --name "full-annotated"
```

**Output (JSON) - Default (without `--include-base64`):**

```json
{
  "filePath": "/tmp/screenshot-20240115-143022.png"
}
```

**Output (JSON) - With `--include-base64`:**

```json
{
  "filePath": "/tmp/screenshot-20240115-143022.png",
  "image": {
    "data": "<base64-encoded-image-data>",
    "mimeType": "image/png"
  }
}
```

**Output (JSON) - With `--annotate`:**

```json
{
  "filePath": "/tmp/screenshot-20240115-143022.png",
  "annotations": [
    {
      "ref": "e1",
      "number": 1,
      "role": "button",
      "name": "Submit",
      "box": { "x": 100, "y": 200, "width": 80, "height": 32 }
    }
  ]
}
```

---

## save-as-pdf

Generate a PDF of the current page.

```bash
ironbee-browser-devtools-cli content save-as-pdf [options]
```

**Arguments:**

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--output-path` | string | No | OS temp dir | Directory to save PDF |
| `--name` | string | No | `page` | PDF filename base (file: `{name}-{time}.pdf`) |
| `--format` | enum | No | `A4` | Paper size: Letter, Legal, Tabloid, Ledger, A0–A6 |
| `--print-background` | boolean | No | `false` | Print background graphics |
| `--margin` | object | No | `1cm` all sides | Margins: top, right, bottom, left (e.g. `"1cm"`) |

**Examples:**

```bash
# Basic PDF
ironbee-browser-devtools-cli content save-as-pdf --name "report"

# Letter format with background
ironbee-browser-devtools-cli content save-as-pdf --format Letter --print-background --name "slides"

# JSON output
ironbee-browser-devtools-cli --json content save-as-pdf --name "document"
```

---

## get-as-html

Get HTML content of the page or an element. By default scripts are removed; use options to include or clean content.

```bash
ironbee-browser-devtools-cli content get-as-html [options]
```

**Arguments:**

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--selector` | string | No | - | CSS selector or ref (omit for full document) |
| `--remove-scripts` | boolean | No | `true` | Remove `<script>` tags from output |
| `--remove-comments` | boolean | No | `false` | Remove HTML comments |
| `--remove-styles` | boolean | No | `false` | Remove `<style>` tags |
| `--remove-meta` | boolean | No | `false` | Remove meta tags |
| `--clean-html` | boolean | No | `false` | Normalize/clean HTML |
| `--minify` | boolean | No | `false` | Minify output |
| `--max-length` | number | No | `50000` | Truncate after this many characters |

**Examples:**

```bash
# Full page HTML (scripts removed by default)
ironbee-browser-devtools-cli content get-as-html

# Specific element
ironbee-browser-devtools-cli content get-as-html --selector "#main"

# Include scripts and minify
ironbee-browser-devtools-cli content get-as-html --selector "#content" --remove-scripts false --minify
```

---

## get-as-text

Get text content of the page or an element.

```bash
ironbee-browser-devtools-cli content get-as-text [options]
```

**Arguments:**

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--selector` | string | No | - | CSS selector or ref (omit for full page) |
| `--max-length` | number | No | `50000` | Truncate text after this many characters |

**Examples:**

```bash
# Full page text
ironbee-browser-devtools-cli content get-as-text

# Specific element text
ironbee-browser-devtools-cli content get-as-text --selector "article"

# Get heading text
ironbee-browser-devtools-cli content get-as-text --selector "h1"
```

---

## start-recording

Begin recording a video of the current browser session using Playwright's native screencast. The video is written when `content_stop-recording` is called.

```bash
ironbee-browser-devtools-cli content start-recording [--output-dir <dir>] [--name <basename>]
```

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--output-dir` | string | No | OS temp dir | Directory where the video file will be saved. |
| `--name` | string | No | `recording` | Video file basename (without extension). |

Returns `{ message, startTimestamp? }`. `startTimestamp` is wall-clock ms since epoch of the first frame — use it to align video time with other timestamps (logs, network events): `(eventTimestampMs - startTimestamp) / 1000` seconds.

---

## stop-recording

Stop the current recording and finalize the video file.

```bash
ironbee-browser-devtools-cli content stop-recording
```

No arguments. Returns `{ filePath? }` with the saved video's full path.

---

## Workflow Example

```bash
SESSION="--session-id content-test --json"

# Navigate
ironbee-browser-devtools-cli $SESSION navigation go-to --url "https://example.com"

# Capture screenshot
SCREENSHOT=$(ironbee-browser-devtools-cli $SESSION content take-screenshot --name "page")
echo "Screenshot saved: $SCREENSHOT"

# Get page title/heading
ironbee-browser-devtools-cli $SESSION content get-as-text --selector "h1"

# Get specific content
ironbee-browser-devtools-cli $SESSION content get-as-html --selector "article"

# Save as PDF
ironbee-browser-devtools-cli $SESSION content save-as-pdf --name "report"

# Record a flow as a video
ironbee-browser-devtools-cli $SESSION content start-recording --name "checkout-flow"
# ... interact with the page ...
ironbee-browser-devtools-cli $SESSION content stop-recording
```
