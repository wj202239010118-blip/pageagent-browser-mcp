# Page Agent 分享版 — 让 AI 工具直接操控你的 Chrome 浏览器

> 5 分钟完成配置，之后对 AI 说"帮我打开 XX 网站、点击 XX 按钮"即可直接操作。

---

## 这是什么

**Page Agent** 是阿里巴巴开源的浏览器自动化工具。通过 Chrome 扩展 + MCP 服务器，你的 AI 编程助手（Claude Code、Cursor、Trae、Windsurf、Codex 等）可以**直接操控你正在使用的 Chrome 浏览器**，而不需要你手动运行脚本、安装 Python 或 Playwright。

```
你 → 对 AI 说"帮我登录 XXX 网站，下载最新报表"
AI → 自动调用 browser_navigate / browser_click / browser_type / browser_screenshot
浏览器 → 实际完成操作，AI 看到结果
```

---

## 快速开始（3 步）

### 第 1 步：安装 Chrome 扩展

在 Chrome 网上应用店搜索 **Page Agent Ext** 或直接访问：

```
https://chromewebstore.google.com/detail/page-agent-ext/akldabonmimlicnjlflnapfeklbfemhj
```

安装完成后扩展栏会出现 Page Agent 图标。

---

### 第 2 步：配置 MCP 服务器

将 `mcp-config-template.json` 的内容**填入你 AI 工具的 MCP 配置文件**（填完你自己的 API Key）：

```json
{
  "mcpServers": {
    "page-agent": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@page-agent/mcp"],
      "env": {
        "LLM_BASE_URL": "https://dashscope.aliyuncs.com/compatible-mode/v1",
        "LLM_MODEL_NAME": "qwen-turbo",
        "LLM_API_KEY": "sk-你的key"
      }
    }
  }
}
```

**各工具配置文件位置：**

| 工具 | 配置文件路径 |
|------|------------|
| Claude Code | `~/.claude/mcp.json` |
| Cursor | `.cursor/mcp.json`（项目级）或 `~/.cursor/mcp.json`（全局） |
| Trae | Trae 设置 → MCP，或项目下 `.trae/mcp.json` |
| Windsurf | `~/.codeium/windsurf/mcp_config.json` |
| Claude Desktop | `%APPDATA%\Claude\claude_desktop_config.json` |
| VS Code Copilot | `.vscode/mcp.json` |

> **不需要 LLM API？** `LLM_*` 三个环境变量仅用于 `execute_task`（自主多步任务代理）。  
> 如果只用 `browser_click` / `browser_type` 等基础工具，**不需要填任何 API Key**，删掉 `env` 字段即可。

---

### 第 3 步：授权连接

MCP 服务器启动时会自动在 Chrome 打开 `http://localhost:38401`，扩展会弹出 **"Allow this session?"** 提示。

点击 **Allow** → 连接成功 → AI 现在可以操控浏览器了。

**验证方式**：让 AI 调用 `get_status`，返回 `{"connected": true}` 即表示正常。

---

## 本地运行（可选，适合无网络环境）

如果不想用 `npx`，可以直接运行本仓库的 `mcp-server/` 源码：

```bash
cd mcp-server
npm install
```

然后配置文件中将 `command` 改为 `node`，`args` 改为：
```json
["绝对路径/mcp-server/src/index.js"]
```

---

## 可用工具一览

详见 [AGENTS.md](./AGENTS.md)，里面有完整工具列表、工作流程和排查指南。

**核心工具：**
- `browser_get_map` — 获取当前页面的所有可交互元素（必须先调用）
- `browser_click` / `browser_type` — 点击和输入
- `browser_navigate` / `browser_open_tab` — 导航
- `browser_screenshot` — 截图
- `browser_scroll` / `browser_drag` / `browser_press_key` — 高级操作
- `execute_task` — 把复杂任务交给内置 AI 自主完成

---

## 常见问题

**Q: 我用的是 Trae，它一直想用 Playwright，怎么办？**  
A: 确保项目目录下有 `AGENTS.md` 文件（本仓库已包含）。Trae 会自动读取该文件并遵循其中的指令，优先使用 Page Agent 工具。

**Q: 连接断了怎么办？**  
A: 重启 AI 工具（重新加载 MCP）或直接用 `browser_reload_extension` 工具重置连接。

**Q: 支持哪些 LLM？**  
A: 任何 OpenAI 兼容接口都支持：Qwen、DeepSeek、Moonshot、Groq、OpenAI 等。

**Q: 安全吗？**  
A: MCP 服务器只监听 `localhost`，不对外暴露。不要让 AI 操作密码和 2FA，这些步骤请手动完成。

---

## 来源

- GitHub: https://github.com/alibaba/page-agent
- npm: https://www.npmjs.com/package/@page-agent/mcp
- 文档: https://alibaba.github.io/page-agent/
