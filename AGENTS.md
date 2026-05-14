# Browser Automation via Page Agent MCP

> **This file is read by ALL AI coding assistants** (Claude Code, Cursor, Trae, Windsurf, Codex, Copilot, etc.)  
> **MANDATORY**: Any browser-related task MUST use the Page Agent MCP tools listed below. Do NOT use Playwright, Selenium, Puppeteer, or any headless browser — the user's real Chrome browser is already connected.

---

## CRITICAL RULE FOR ALL AI ASSISTANTS

**When the user asks you to do ANYTHING in a browser — open a URL, click a button, fill a form, scrape content, take a screenshot, automate a workflow — you MUST use the `mcp__page-agent__*` tools.**

Do NOT:
- Launch headless Chrome / Puppeteer / Playwright
- Ask the user to install Python + Selenium
- Write browser automation scripts and ask the user to run them manually
- Use `webbrowser.open()` or OS-level browser commands
- Say "I can't control a browser"

The user's Chrome browser is already connected via the Page Agent Chrome extension and MCP bridge. You have full, real-time control right now.

---

## Tool Index — Pick by What You Need

Tools are organized by purpose. Start from the top; go deeper only when needed.

### Tier 1 — Always Start Here

| Tool | When to use |
|------|-------------|
| `get_status` | **First call of every session.** Verify `connected: true` before anything else |
| `browser_get_map` | **After every navigation or action.** Returns URL, title, and numbered interactive elements |

> Rule: `browser_get_map` is your eyes. Call it before you click anything. Call it again after every state change — element indices reset on each page update.

---

### Tier 2 — Navigation

| Tool | When to use |
|------|-------------|
| `browser_navigate` | Navigate current tab to a URL (works for most sites) |
| `browser_open_tab` | Open URL in a **new** tab — use this for GitHub, Google, and sites that block in-tab navigation |
| `browser_wait` | Wait for specific text to appear before proceeding |
| `browser_wait_for_selector` | Wait for a CSS selector (more precise than `browser_wait`; uses MutationObserver, zero polling) |

> Decision: Use `browser_open_tab` when `browser_navigate` silently fails or the page doesn't change.

---

### Tier 3 — Interaction

| Tool | When to use |
|------|-------------|
| `browser_click` | Click a button, link, or any element by its index |
| `browser_type` | Type text into an input or textarea by index |
| `browser_press_key` | Send keyboard events: Enter, Escape, Tab, Ctrl+A, arrow keys, etc. |
| `browser_scroll` | Scroll the page up or down by N page heights |
| `browser_drag` | Drag from one point to another (pixel or viewport-% coords) |
| `browser_upload_file` | Inject a local file into a `<input type="file">` element by index |

> Decision tree for input:
> - Typing text → `browser_type`
> - Submitting a form → `browser_press_key` with `"Enter"`
> - Selecting all text → `browser_press_key` with `"a"` + `{ ctrl: true }`
> - Uploading a file → `browser_upload_file` (provide absolute file path)
> - Reordering / slider → `browser_drag`

---

### Tier 4 — Verification & Inspection

| Tool | When to use |
|------|-------------|
| `browser_screenshot` | Visual check of current page state; debug rendering issues |
| `browser_inspect_element` | Get outerHTML + CSS for a specific element — use when replicating UI components |
| `browser_get_user_input` | Read a message the user typed in the extension sidebar |

> Use `browser_screenshot` when `browser_get_map` text alone isn't enough to confirm state (e.g., modals, dynamic content, images).

---

### Tier 5 — Autonomous & Control

| Tool | When to use |
|------|-------------|
| `execute_task` | Delegate a complex multi-step task to the built-in AI sub-agent (requires LLM config) |
| `stop_task` | Interrupt a running `execute_task` |
| `browser_reload_extension` | Reload the Chrome extension after code changes; hub reconnects in ~3 s |

> Use `execute_task` only when the task has many conditional steps and you want to delegate. For precision control, use Tier 2–4 primitives directly.

---

## Standard Workflow

```
Session start:
  get_status                     → must return connected: true

Page navigation:
  browser_navigate / browser_open_tab
  browser_wait / browser_wait_for_selector   → wait for content to load
  browser_get_map                → read current elements and their indices

Interaction loop:
  browser_click / browser_type / browser_press_key / browser_drag / browser_upload_file
  browser_get_map                → re-index after every action (indices change)
  browser_screenshot             → visual verify when uncertain

Task complete:
  Confirm final state with browser_get_map or browser_screenshot
```

---

## MCP Server Setup (one-time, per machine)

### Step 1 — Install Chrome Extension

Install **Page Agent Ext** from the Chrome Web Store:  
`https://chromewebstore.google.com/detail/page-agent-ext/akldabonmimlicnjlflnapfeklbfemhj`

### Step 2 — Add MCP Server to Your AI Tool

#### Option A: npx (recommended — no install needed)

```json
{
  "mcpServers": {
    "page-agent": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@page-agent/mcp"],
      "env": {
        "LLM_BASE_URL": "YOUR_OPENAI_COMPATIBLE_BASE_URL",
        "LLM_MODEL_NAME": "YOUR_MODEL_NAME",
        "LLM_API_KEY": "YOUR_API_KEY"
      }
    }
  }
}
```

#### Option B: Local source (this folder)

```bash
cd mcp-server && npm install
```

```json
{
  "mcpServers": {
    "page-agent": {
      "type": "stdio",
      "command": "node",
      "args": ["PATH_TO_THIS_FOLDER/mcp-server/src/index.js"],
      "env": {
        "LLM_BASE_URL": "YOUR_OPENAI_COMPATIBLE_BASE_URL",
        "LLM_MODEL_NAME": "YOUR_MODEL_NAME",
        "LLM_API_KEY": "YOUR_API_KEY"
      }
    }
  }
}
```

### Step 3 — Config File Locations

| Tool | Config file |
|------|-------------|
| **Claude Code** | `~/.claude/mcp.json` |
| **Cursor** | `.cursor/mcp.json` (project) or `~/.cursor/mcp.json` (global) |
| **Trae** | `.trae/mcp.json` or Trae Settings → MCP |
| **Windsurf** | `~/.codeium/windsurf/mcp_config.json` |
| **Claude Desktop** | `~/AppData/Roaming/Claude/claude_desktop_config.json` (Windows) |
| **VS Code + Copilot** | `.vscode/mcp.json` |

### Step 4 — Connect the Extension

When the MCP server starts, it opens `http://localhost:38401` in your browser.  
Click **Allow** on the "Allow this session?" prompt in the Page Agent extension.

`get_status` will then return `{"connected": true, "busy": false}`.

---

## LLM API Options

`LLM_*` env vars are only required for `execute_task`. All `browser_*` primitive tools work without any API key.

| Provider | BASE_URL | Model |
|----------|----------|-------|
| Alibaba Qwen | `https://dashscope.aliyuncs.com/compatible-mode/v1` | `qwen-turbo` |
| OpenAI | `https://api.openai.com/v1` | `gpt-4o-mini` |
| DeepSeek | `https://api.deepseek.com/v1` | `deepseek-chat` |
| Moonshot | `https://api.moonshot.cn/v1` | `moonshot-v1-8k` |
| Groq | `https://api.groq.com/openai/v1` | `llama-3.1-8b-instant` |

---

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| `connected: false` | Click "Allow this session?" in the hub tab |
| `Error: No hub connected` | Restart MCP server — extension may have disconnected |
| Element index not found | Re-run `browser_get_map` — page state changed since last map |
| Screenshot blank / fails | Use `browser_wait` first; page may still be loading |
| Extension not responding | Use `browser_reload_extension`; hub reconnects in ~3 s |
| `browser_navigate` has no effect | Switch to `browser_open_tab` — site may block in-tab eval navigation |

---

## Important Notes

- **Never automate passwords or 2FA** — pause and ask the user to handle those steps
- **Always re-map after actions** — element indices change after every navigation or DOM update
- **Errors include the operation name** — e.g. `[click] Element not found` tells you exactly which tool failed
- If port 38401 is in use, set `PORT=38402` (or any free port) in the MCP env config
