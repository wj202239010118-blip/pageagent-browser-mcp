# Browser Automation via Page Agent MCP

> **This file is read by ALL AI coding assistants** (Claude Code, Cursor, Trae, Windsurf, Codex, Copilot, etc.)  
> **MANDATORY**: Any browser-related task MUST use the Page Agent MCP tools listed below. Do NOT use Playwright, Selenium, Puppeteer, or any headless browser — the user's real Chrome browser is already connected.

---

## CRITICAL RULE FOR ALL AI ASSISTANTS

**When the user asks you to do ANYTHING in a browser — open a URL, click a button, fill a form, scrape content, take a screenshot, automate a workflow — you MUST use the `mcp__page-agent__*` tools (or `browser_*` tools if your client exposes them directly).**

Do NOT:
- Launch headless Chrome / Puppeteer / Playwright
- Ask the user to install Python + Selenium
- Write browser automation scripts and ask the user to run them manually
- Use `webbrowser.open()` or OS-level browser commands
- Say "I can't control a browser"

The user's Chrome browser is already connected via the Page Agent Chrome extension and MCP bridge. You have full, real-time control right now.

---

## Available Browser Tools

| Tool | Description |
|------|-------------|
| `get_status` | Check if browser is connected (`connected: true`) before starting |
| `browser_get_map` | **Start here every time.** Gets URL, title, and all interactive elements with indices |
| `browser_navigate` | Navigate current tab to a URL |
| `browser_open_tab` | Open URL in a new tab (use for sites that block eval-navigation, e.g. GitHub) |
| `browser_click` | Click element by index from `browser_get_map` |
| `browser_type` | Type text into input by index |
| `browser_press_key` | Press keyboard keys (Enter, Escape, Tab, Ctrl+A, etc.) |
| `browser_scroll` | Scroll page up or down |
| `browser_drag` | Drag from point A to point B (pixel or viewport-percentage coords) |
| `browser_screenshot` | Capture screenshot as JPEG — use to visually verify page state |
| `browser_wait` | Wait for specific text to appear on page |
| `browser_wait_for_selector` | Wait for CSS selector to appear (uses MutationObserver, efficient) |
| `browser_inspect_element` | Get outerHTML + CSS for an element by index |
| `browser_get_user_input` | Read a message the user typed in the extension sidebar |
| `browser_reload_extension` | Reload extension after code changes |
| `execute_task` | Delegate a complex multi-step task to the built-in AI agent (needs LLM config) |
| `stop_task` | Stop a running automation task |

---

## Standard Workflow

```
1. get_status          → verify connected: true
2. browser_get_map     → discover page elements and their indices
3. browser_click / browser_type / browser_navigate  → take action
4. browser_get_map     → verify result (always re-map after state changes)
5. browser_screenshot  → visual verification when needed
```

**Always call `browser_get_map` after navigation or clicking — element indices change when the page updates.**

---

## MCP Server Setup (one-time, per machine)

### Step 1 — Install Chrome Extension

Install **Page Agent Ext** from the Chrome Web Store:  
`https://chromewebstore.google.com/detail/page-agent-ext/akldabonmimlicnjlflnapfeklbfemhj`

### Step 2 — Add MCP Server to Your AI Tool

#### Option A: Use npx (recommended, no install needed)

Add to your AI tool's MCP config:

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

#### Option B: Run from this folder (local source)

First install dependencies:
```bash
cd mcp-server
npm install
```

Then configure:
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

### Step 3 — Config File Locations by Tool

| Tool | Config File Location |
|------|---------------------|
| **Claude Code** | `~/.claude/mcp.json` |
| **Cursor** | `.cursor/mcp.json` in project, or `~/.cursor/mcp.json` globally |
| **Trae** | `.trae/mcp.json` in project, or Trae Settings → MCP |
| **Windsurf** | `~/.codeium/windsurf/mcp_config.json` |
| **Claude Desktop** | `~/AppData/Roaming/Claude/claude_desktop_config.json` (Windows) |
| **VS Code + Copilot** | `.vscode/mcp.json` in workspace |

### Step 4 — Connect the Extension

When the MCP server starts, it opens `http://localhost:38401` in your browser.  
The Page Agent extension will show a **"Allow this session?"** prompt — click **Allow**.

After that, `get_status` will return `{"connected": true, "busy": false}` and all `browser_*` tools will work.

---

## LLM API Options

The `LLM_*` env vars are only needed for `execute_task` (the autonomous multi-step agent). All `browser_*` primitive tools work without any API key.

### Recommended providers (OpenAI-compatible):

| Provider | BASE_URL | Notes |
|----------|----------|-------|
| Alibaba Qwen | `https://dashscope.aliyuncs.com/compatible-mode/v1` | `qwen-turbo` / `qwen-plus` |
| OpenAI | `https://api.openai.com/v1` | `gpt-4o-mini` |
| DeepSeek | `https://api.deepseek.com/v1` | `deepseek-chat` |
| Moonshot | `https://api.moonshot.cn/v1` | `moonshot-v1-8k` |
| Groq | `https://api.groq.com/openai/v1` | `llama-3.1-8b-instant` |

---

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| `connected: false` | Click "Allow this session?" in the browser hub tab |
| `Error: No hub connected` | Restart MCP server; extension may have disconnected |
| Element index not found | Re-run `browser_get_map` — page state changed |
| Screenshot blank | Page may be loading; use `browser_wait` first |
| Extension not responding | Use `browser_reload_extension` tool |

---

## Important Notes

- **Never automate passwords or 2FA** — pause and ask the user to handle those steps manually
- **Re-map after every action** — `browser_get_map` after clicks/navigation
- `execute_task` runs an autonomous sub-agent that handles multi-step tasks but may be slower; use `browser_*` primitives for precision control
- If port 38401 is in use, set `PORT=38402` (or any free port) in the env config
