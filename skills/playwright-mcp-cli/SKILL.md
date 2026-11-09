---
name: playwright-mcp-cli
cli_version: "0.12.1"
description: Playwright browser automation — zero-config isolated mode by default (no broker conflict), plus CI-friendly check/watch commands and rich scripting on top of raw @playwright/mcp tool calls.
---

# playwright-mcp-cli

Part of `mcp-broker-cli`. Unlike calling `@playwright/mcp` tools directly through the broker,
`playwright-mcp-cli` provides:

- **Zero conflict by default** — spawns `@playwright/mcp --isolated` directly, giving every CLI
  run its own Chrome profile so it never conflicts with Cursor's IDE session.
- **Recipes** — `read <url>` does navigate + wait + extract in one command vs 3+ raw tool calls.
- **CI-friendly** — `check <url>` returns exit code 0/1; composable via pipes and shell scripts.
- **Monitoring** — `watch <url>` polls for content changes; `network --follow` tails live requests.
- **Multi-step sessions** — `script <file>` and `recipe` share one browser instance across many steps.
- **Error intelligence** — stale-ref recovery, blocked-URL guard, timeout hints, formatted output.

## Prerequisites

1. `mcp-broker-cli` installed via Homebrew (`brew install Shaic/tap/mcp-broker-cli`).
2. Node.js and `npx` on PATH.
3. `@playwright/mcp` and browsers installed (`playwright-mcp-cli setup` guided flow).

> **Transport mode**: By default `playwright-mcp-cli` spawns `@playwright/mcp --isolated` directly
> (no broker). Set `MCP_BROKER_TRANSPORT=stdio` to route through the policy broker instead (adds
> policy enforcement and audit trail).

## Commands

| Command | Description |
|---|---|
| `navigate <url>` | Open a URL in the browser |
| `read <url>` | Navigate + extract clean content in one command |
| `check <url> [--text T] [--title T]` | Health check — exit 0 if loaded + text found, 1 otherwise (CI-friendly) |
| `watch <url> [--interval N] [--text T]` | Periodically poll URL and alert when content changes |
| `back` | Go back in browser history |
| `click <target>` | Click an element |
| `type <target> <text>` | Type text into an element |
| `fill <json\|->` | Fill multiple form fields from JSON |
| `hover <target>` | Hover over an element |
| `drag <source> <target>` | Drag from one element to another |
| `press <key>` | Press a keyboard key (Enter, ArrowDown, Tab…) |
| `select <target> <value>` | Select dropdown option(s) |
| `upload --path <file>` | Upload file(s) to a file input |
| `dialog <accept\|dismiss>` | Accept or dismiss a browser dialog |
| `wait [--text] [--time]` | Wait for a condition or duration |
| `eval <function>` | Run JavaScript in the page |
| `text` | Extract clean page text/Markdown |
| `links` | List every link on the page |
| `find <text-or-role>` | Search page elements by role or text |
| `extract <schema-json>` | Extract named fields via CSS selectors |
| `table <selector>` | Extract an HTML table into rows |
| `forms` | Dump every input/select/textarea field |
| `snapshot` | Get accessibility tree |
| `screenshot` | Take a screenshot |
| `resize [w h] [--device]` | Resize browser window or apply device preset |
| `tabs <list\|new\|close\|select>` | Manage browser tabs |
| `console` | Read browser console output |
| `network` | List network requests since page load |
| `close` | Close the browser |
| `auth save <file>` | Save cookies + localStorage to file |
| `auth load <file>` | Re-inject saved auth state |
| `script <file>` | Run a multi-command script in one browser session |
| `recipe <name>` | Run a built-in step template (login, search, …) |
| `setup [--yes]` | Check and fix Playwright MCP prerequisites |
| `doctor` | Health check: prerequisites + transport/isolation status + live smoke test |

## Key Flags

- `--tab <N>` — target a specific tab before any command
- `--json` — raw JSON output for any command
- `--markdown` / `--text` — control `read`/`text` output format
- `--wait <s>` / `--wait-for-text <t>` / `--wait-for-selector <sel>` — wait conditions on `read`
- `--device <name>` — emulate a mobile device (`resize --device "iPhone 14"`)
- `--dry-run` — show what a `click`/`type` would target without acting
- `--timing` — print elapsed time for any command
- `--trace` — capture full JSON-RPC exchange + HAR-shaped network dump

## Common Patterns

### Quick content read (no broker, no conflict)
```bash
playwright-mcp-cli read https://app.getport.io/... --markdown
playwright-mcp-cli read https://strandsagents.com/
playwright-mcp-cli text                              # active page
```

### CI health check (exit code 0/1)
```bash
playwright-mcp-cli check https://app.example.com --text "Welcome"
playwright-mcp-cli check https://status.example.com --text "All systems operational"
# Combine with CI pipeline:
playwright-mcp-cli check https://deploy.example.com --text "Build complete" && echo "Deploy OK"
```

### Monitor a page for changes
```bash
playwright-mcp-cli watch https://status.example.com
playwright-mcp-cli watch https://deploy.example.com --text "Deploy complete" --interval 15
```

### Multi-step automation via script
```bash
cat > /tmp/login.txt <<'EOF'
navigate https://app.example.com/login
type [name=email] user@example.com
type [name=password] secret --submit
wait --text "Dashboard" --timeout 10
screenshot --filename /tmp/after-login.png
EOF
playwright-mcp-cli script /tmp/login.txt
```

### Use a recipe
```bash
playwright-mcp-cli recipe list
playwright-mcp-cli recipe login --var url=https://example.com --var username=me --var password=secret
```

### Capture and inspect
```bash
playwright-mcp-cli snapshot                          # accessibility tree
playwright-mcp-cli screenshot --full-page --filename /tmp/page.png
playwright-mcp-cli network --url-contains /api --status 500
playwright-mcp-cli console --min-level warn
```

### Auth persistence
```bash
playwright-mcp-cli navigate https://app.example.com
# … perform login in the browser …
playwright-mcp-cli auth save ~/.cache/app-auth.json
# Later, in a fresh session:
playwright-mcp-cli navigate https://app.example.com
playwright-mcp-cli auth load ~/.cache/app-auth.json
```

### Override transport (route through broker instead of direct)
```bash
MCP_BROKER_TRANSPORT=stdio playwright-mcp-cli read https://example.com
```

## Notes

- Default mode (`playwright-direct`): spawns `@playwright/mcp --isolated` directly — no broker
  involved, each run gets its own Chrome profile, never conflicts with Cursor's IDE session.
- Broker mode (`MCP_BROKER_TRANSPORT=stdio`): routes through `mcp-policy-broker` for policy
  enforcement — may conflict with Cursor if the broker policy doesn't include `--isolated`.
- `playwright-mcp-cli doctor` shows the active transport and isolation status.
- Use `--tab <N>` to work across multiple open tabs without losing context.

## Self-Improvement

After completing this skill's objective, do a silent self-assessment:
1. Did the skill fail because of a bug **in the skill itself** (wrong flag, stale API path, missing step) — not a user environment issue (bad credentials, network error, missing local dependency)?
2. If yes — check open issues first: `gh issue list --repo github.cyberng.com/Shaic/mcp-broker-cli --search "[Skill Bug] playwright-mcp-cli" --limit 10`. If no duplicate exists, open one: `gh issue create --repo github.cyberng.com/Shaic/mcp-broker-cli --title "[Skill Bug] playwright-mcp-cli: <what broke>" --body "<brief description of what failed and why>"`. Never include secrets, tokens, or account IDs.
3. If neither — do nothing. Do not mention this check.
