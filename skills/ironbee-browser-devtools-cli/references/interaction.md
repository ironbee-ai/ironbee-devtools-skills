# Interaction Tools

User interaction simulation commands. All tools accept **CSS selector or ref** (e.g. `e1`, `@e1`) from the last `a11y take-aria-snapshot`. Use refs when selectors are fragile or elements lack stable IDs.

## click

Click an element on the page.

```bash
ironbee-browser-devtools-cli interaction click --selector <selector-or-ref>
```

**Arguments:**

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--selector` | string | Yes | - | CSS selector or ref (e1, @e1) for element to click |
| `--timeout-ms` | number | No | 10000 | Timeout when waiting for element |
| `--wait-for-navigation` | boolean | No | `false` | When true, wait for navigation and network idle after click (use when click opens a new page) |
| `--wait-for-timeout-ms` | number | No | 30000 | Timeout for navigation and network idle when wait-for-navigation is true |

**Examples:**

```bash
# Click by ref (from ARIA snapshot)
ironbee-browser-devtools-cli a11y take-aria-snapshot
ironbee-browser-devtools-cli interaction click --selector "e1"

# Click a button
ironbee-browser-devtools-cli interaction click --selector "button#submit"

# Click by data-testid
ironbee-browser-devtools-cli interaction click --selector "[data-testid='login-btn']"
```

---

## fill

Fill out an input field (clears existing content first).

```bash
ironbee-browser-devtools-cli interaction fill --selector <selector-or-ref> --value <value>
```

**Arguments:**

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--selector` | string | Yes | - | CSS selector or ref (e1, @e1) for input field |
| `--value` | string | Yes | - | Value to fill |
| `--timeout-ms` | number | No | 10000 | Timeout when waiting for element |

**Examples:**

```bash
# Fill text input
ironbee-browser-devtools-cli interaction fill --selector "#email" --value "user@example.com"

# Fill password
ironbee-browser-devtools-cli interaction fill --selector "input[type=password]" --value "secret123"

# Fill textarea
ironbee-browser-devtools-cli interaction fill --selector "textarea#message" --value "Hello world"
```

---

## hover

Hover over an element.

```bash
ironbee-browser-devtools-cli interaction hover --selector <selector-or-ref>
```

**Arguments:**

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--selector` | string | Yes | - | CSS selector or ref (e1, @e1) for element to hover |
| `--timeout-ms` | number | No | 10000 | Timeout when waiting for element |

**Examples:**

```bash
# Hover over menu
ironbee-browser-devtools-cli interaction hover --selector ".dropdown-menu"

# Hover to reveal tooltip
ironbee-browser-devtools-cli interaction hover --selector "[data-tooltip]"
```

---

## select

Select an option from a dropdown.

```bash
ironbee-browser-devtools-cli interaction select --selector <selector-or-ref> --value <value>
```

**Arguments:**

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--selector` | string | Yes | - | CSS selector or ref (e1, @e1) for select element |
| `--value` | string | Yes | - | Option value to select |
| `--timeout-ms` | number | No | 10000 | Timeout when waiting for element |

**Examples:**

```bash
# Select by value
ironbee-browser-devtools-cli interaction select --selector "#country" --value "US"

# Select by visible text
ironbee-browser-devtools-cli interaction select --selector "#size" --value "Large"
```

---

## press-key

Press a keyboard key. Supports optional hold and repeat (e.g. for scroll-by-key).

```bash
ironbee-browser-devtools-cli interaction press-key --key <key> [options]
```

**Arguments:**

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--key` | string | Yes | - | Key to press (e.g., Enter, Tab, Escape, ArrowDown, Space) |
| `--selector` | string | No | - | CSS selector or ref (e1, @e1) to focus before pressing |
| `--hold-ms` | number | No | - | Ms between keydown and keyup |
| `--repeat` | boolean | No | `false` | Repeat key while holdMs (e.g. for scroll) |
| `--repeat-interval-ms` | number | No | `50` | Interval between repeated presses when repeat=true (min 10) |
| `--timeout-ms` | number | No | 10000 | Timeout when waiting for selector (if used) |

**Examples:**

```bash
# Press Enter
ironbee-browser-devtools-cli interaction press-key --key "Enter"

# Press Tab to navigate
ironbee-browser-devtools-cli interaction press-key --key "Tab"

# Press Escape to close modal
ironbee-browser-devtools-cli interaction press-key --key "Escape"

# Press on specific element
ironbee-browser-devtools-cli interaction press-key --selector "#search" --key "Enter"
```

---

## scroll

Scroll the page viewport or a scrollable element.

```bash
ironbee-browser-devtools-cli interaction scroll [options]
```

**Arguments:**

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--mode` | enum | No | `by` | `by` (dx/dy), `to` (x/y), `top`, `bottom`, `left`, `right` |
| `--selector` | string | No | - | CSS selector or ref for scroll container (omit = viewport) |
| `--dx` | number | No | - | Delta X for mode=by (pixels) |
| `--dy` | number | No | - | Delta Y for mode=by (pixels) |
| `--x` | number | No | - | Absolute X for mode=to |
| `--y` | number | No | - | Absolute Y for mode=to |
| `--behavior` | enum | No | `auto` | `auto` or `smooth` |

**Examples:**

```bash
# Scroll down by 500px
ironbee-browser-devtools-cli interaction scroll --mode by --dy 500

# Scroll to bottom of page
ironbee-browser-devtools-cli interaction scroll --mode bottom

# Scroll inside a container
ironbee-browser-devtools-cli interaction scroll --selector "#sidebar" --mode bottom

# Scroll up
ironbee-browser-devtools-cli interaction scroll --mode by --dy -200

# Smooth scroll to position
ironbee-browser-devtools-cli interaction scroll --mode to --x 0 --y 1000 --behavior smooth
```

---

## drag

Drag an element to another location.

```bash
ironbee-browser-devtools-cli interaction drag --source-selector <selector-or-ref> --target-selector <selector-or-ref>
```

**Arguments:**

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--source-selector` | string | Yes | - | CSS selector or ref (e1, @e1) for element to drag |
| `--target-selector` | string | Yes | - | CSS selector or ref (e2, @e2) for drop target |
| `--timeout-ms` | number | No | 10000 | Timeout when waiting for elements |

**Examples:**

```bash
ironbee-browser-devtools-cli interaction drag --source-selector ".draggable" --target-selector ".dropzone"

# Using refs from ARIA snapshot
ironbee-browser-devtools-cli interaction drag --source-selector "e1" --target-selector "e2"
```

---

## resize-viewport

Resize the browser viewport.

```bash
ironbee-browser-devtools-cli interaction resize-viewport --width <px> --height <px>
```

**Arguments:**

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--width` | number | Yes | - | Viewport width in pixels |
| `--height` | number | Yes | - | Viewport height in pixels |

**Examples:**

```bash
# Mobile viewport
ironbee-browser-devtools-cli interaction resize-viewport --width 375 --height 667

# Desktop viewport
ironbee-browser-devtools-cli interaction resize-viewport --width 1920 --height 1080

# Tablet viewport
ironbee-browser-devtools-cli interaction resize-viewport --width 768 --height 1024
```

---

## resize-window

Resize the real browser window (OS-level). Best with Chromium and headful mode.

```bash
ironbee-browser-devtools-cli interaction resize-window [options]
```

**Arguments:**

| Argument | Type | Required | Default | Description |
|----------|------|----------|---------|-------------|
| `--width` | number | No* | - | Window width in pixels (*required when state=normal) |
| `--height` | number | No* | - | Window height in pixels (*required when state=normal) |
| `--state` | enum | No | `normal` | `normal`, `maximized`, `minimized`, `fullscreen` (width/height ignored when not normal) |

## Form Automation Example

```bash
SESSION="--session-id form-test"

# Navigate to form
ironbee-browser-devtools-cli $SESSION navigation go-to --url "https://example.com/signup"

# Fill registration form
ironbee-browser-devtools-cli $SESSION interaction fill --selector "#firstName" --value "John"
ironbee-browser-devtools-cli $SESSION interaction fill --selector "#lastName" --value "Doe"
ironbee-browser-devtools-cli $SESSION interaction fill --selector "#email" --value "john@example.com"
ironbee-browser-devtools-cli $SESSION interaction fill --selector "#password" --value "SecurePass123!"

# Select country
ironbee-browser-devtools-cli $SESSION interaction select --selector "#country" --value "US"

# Check terms checkbox
ironbee-browser-devtools-cli $SESSION interaction click --selector "#terms"

# Submit form
ironbee-browser-devtools-cli $SESSION interaction click --selector "button[type=submit]"

# Wait for response
ironbee-browser-devtools-cli $SESSION sync wait-for-network-idle

# Verify success
ironbee-browser-devtools-cli $SESSION content get-as-text --selector ".success-message"
```

## Responsive Testing Example

```bash
SESSION="--session-id responsive-test"

# Navigate
ironbee-browser-devtools-cli $SESSION navigation go-to --url "https://example.com"

# Test mobile
ironbee-browser-devtools-cli $SESSION interaction resize-viewport --width 375 --height 667
ironbee-browser-devtools-cli $SESSION content take-screenshot --name "mobile"

# Test tablet
ironbee-browser-devtools-cli $SESSION interaction resize-viewport --width 768 --height 1024
ironbee-browser-devtools-cli $SESSION content take-screenshot --name "tablet"

# Test desktop
ironbee-browser-devtools-cli $SESSION interaction resize-viewport --width 1920 --height 1080
ironbee-browser-devtools-cli $SESSION content take-screenshot --name "desktop"
```
