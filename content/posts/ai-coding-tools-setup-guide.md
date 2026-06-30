---
title: "7 AI Coding Tools Setup Guide: Use GLM Models with Claude Code, zcode, Copilot & More"
date: 2026-06-30
draft: false
tags: ["AI coding", "Claude Code", "zcode", "Cursor", "GLM", "tutorial"]
description: "Step-by-step guide to connect Claude Code, zcode, GitHub Copilot, Crush(opencode), CodeGPT, Cursor, and Trae to a proxy API and use GLM models. One API key, all tools supported."
---

> **🔥 [Get Your API Key](https://tinyapi.nykjsd.cn/home)** — One API key works across all 7 tools in this guide. Affordable pricing, supports GLM, Claude, GPT, and more.

## General Info

- **API Base URL**: `https://tinyapi.nykjsd.cn`
- **Recommended Models**: `glm-5.2` (best quality), `glm-5` (fast & lightweight)
- **API Key**: [👉 Purchase here](https://tinyapi.nykjsd.cn/home), format: `sk-xxx…xxxx`

---

## 1. Claude Code (⭐ Recommended — Most Powerful AI Coding Assistant)

Claude Code by Anthropic understands your entire codebase, edits files across projects, and runs commands. Available as a terminal CLI, VS Code extension, JetBrains plugin, desktop app, and browser tool.

### 1.1 Installation

Official docs: https://code.claude.com/docs/zh-CN/overview

> ⚠️ Installation requires a VPN with a US region.

**Recommended (requires Node.js 18+):**

```bash
npm install -g @anthropic-ai/claude-code
```

**macOS / Linux / WSL:**

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

**Windows PowerShell:**

```powershell
irm https://claude.ai/install.ps1 | iex
```

**Windows CMD:**

```batch
curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd
```

**Homebrew (macOS):**

```bash
brew install --cask claude-code
```

After installation, restart your terminal and run `claude` to verify.

### 1.2 Configure Proxy API

Open or create the config file:

- macOS / Linux: `~/.claude/settings.json`
- Windows: `C:\Users\{YourUsername}\.claude\settings.json`

> ⚠️ Use your actual username path, NOT Administrator.

Replace `your_zhipu_api_key` with your API key:

```json
{
  "env": {
    "ANTHROPIC_AUTH_TOKEN": "your_zhipu_api_key",
    "ANTHROPIC_BASE_URL": "https://tinyapi.nykjsd.cn",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "glm-5",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "glm-5.2",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "glm-5.2",
    "API_TIMEOUT_MS": "3000000",
    "CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC": 1
  },
  "model": "opus"
}
```

Also add to `~/.claude.json` (Windows: `UserDir/.claude.json`):

```json
{
  "hasCompletedOnboarding": true
}
```

### 1.3 Verify

Restart terminal, navigate to your project, run `claude`. If prompted "Do you want to use this API key?", select **Yes**.

### 1.4 VS Code / JetBrains Plugin

- **VS Code**: Install "Claude Code" from the marketplace. Uses the same `settings.json`.
- **JetBrains IDE**: Install from JetBrains Marketplace. Same config applies.

### 1.5 CCSwitch (Optional)

A visual config manager for Claude Code. Switch between multiple API providers easily.

> 📌 Optional — manual `settings.json` editing works fine.

Steps:
1. Open CCSwitch → Create Anthropic provider config
2. Paste the JSON above and save

---

## 2. zcode (Zhipu AI Coding Assistant)

zcode is Zhipu AI's official coding assistant with native GLM model support. Available as a terminal tool and IDE plugin. Works with proxy API.

### 2.1 Installation

**Option A: One-click helper (Recommended)**

```bash
npx @z_ai/coding-helper
```

Follow the prompts to auto-configure.

**Option B: VS Code Plugin**

Search "zcode" in the VS Code marketplace and install.

### 2.2 Configure Proxy API

zcode natively uses GLM models. With a proxy, configure a custom endpoint.

Create config file:

- macOS / Linux: `~/.zcode/settings.json`
- Windows: `C:\Users\{YourUsername}\.zcode\settings.json`

```json
{
  "api_key": "***",
  "base_url": "https://tinyapi.nykjsd.cn/v1",
  "model": "glm-5.2",
  "timeout": 300
}
```

> ⚠️ Config path may vary by version. Use `npx @z_ai/coding-helper` for auto-setup.

### 2.3 Tips

- Primary model: `glm-5.2` (best quality)
- Quick tasks: `glm-5` (faster response)
- Best GLM compatibility as an official Zhipu tool

---

## 3. GitHub Copilot

GitHub's AI pair programmer, deeply integrated with VS Code, Visual Studio, and JetBrains IDEs.

### 3.1 Installation

- **VS Code**: Search "GitHub Copilot" in extensions marketplace
- **JetBrains**: Settings → Plugins → Search "GitHub Copilot"
- Requires Copilot subscription ($10/month) or Copilot Free

### 3.2 Configure Proxy API

> ⚠️ Custom endpoint support is limited in Copilot. YMMV depending on version.

Add to VS Code `settings.json`:

```json
{
  "github.copilot.openai.endpoint": "https://tinyapi.nykjsd.cn/v1",
  "github.copilot.openai.key": "your-api-key",
  "github.copilot.model": "glm-5.2"
}
```

> 💡 If custom endpoints don't work with Copilot, use Claude Code or Crush instead.

### 3.3 Verify

Open VS Code, trigger Copilot completion via right-click or keyboard shortcut.

---

## 4. opencode / Crush (Terminal AI Coding Assistant)

opencode (now renamed **Crush**) is a terminal-based AI coding assistant built with Go, maintained by the Charm team. Features a TUI, multi-model support, and custom API endpoints.

### 4.1 Installation

```bash
# Homebrew
brew install charmbracelet/tap/crush

# NPM
npm install -g @charmland/crush

# Go
go install github.com/charmbracelet/crush@latest

# Windows (WinGet)
winget install charmbracelet.crush

# Windows (Scoop)
scoop bucket add charm https://github.com/charmbracelet/scoop-bucket.git
scoop install crush
```

### 4.2 Configure Proxy API

Create `~/.config/crush/crush.json` (or `.crush.json` in your project):

```json
{
  "providers": {
    "openai": {
      "id": "openai",
      "name": "GLM Proxy",
      "base_url": "https://tinyapi.nykjsd.cn/v1",
      "type": "openai",
      "api_key": "***",
      "models": [
        { "id": "glm-5.2", "name": "GLM 5.2" },
        { "id": "glm-5", "name": "GLM 5" }
      ]
    }
  },
  "agents": {
    "coder": { "model": "glm-5.2" },
    "task": { "model": "glm-5.2" }
  }
}
```

Or use environment variables:

```bash
export OPENAI_API_KEY="***"
export OPENAI_BASE_URL="https://tinyapi.nykjsd.cn/v1"
```

> 💡 Crush supports switching models mid-session — great for A/B testing.

### 4.3 Usage

```bash
cd your-project
crush
```

---

## 5. CodeGPT (VS Code Plugin)

A VS Code plugin AI coding assistant with multi-model support and custom API endpoints.

### 5.1 Install

Search "CodeGPT" in the VS Code marketplace and install.

### 5.2 Configure API Key

Open `%USERPROFILE%\.codex\` and edit `auth.json`:

```json
{
  "OPENAI_API_KEY": "your-api-key"
}
```

### 5.3 Configure URL & Model

Edit `config.toml` in the same directory:

```toml
model = "glm-5.2"
model_provider = "anyrouter"
preferred_auth_method = "apikey"
personality = "pragmatic"
model_reasoning_effort = "high"

[model_providers.anyrouter]
name = "Any Router"
base_url = "https://tinyapi.nykjsd.cn/v1"
wire_api = "responses"
```

### 5.4 Verify

Restart VS Code and test CodeGPT with a conversation.

---

## 6. Cursor

An AI-native code editor built on VS Code, with built-in AI coding features and custom model support.

> ⚠️ Requires Cursor Pro or higher. Free tier doesn't support custom Base URL.

### 6.1 Install

Download from https://cursor.com

### 6.2 Configure Proxy API

Open Settings → `File → Preferences → Open Cursor Settings`

Find Models section → Override Model / Create New Model:

| Setting | Value |
|---------|-------|
| Model Name | glm-5.2 |
| Base URL | https://tinyapi.nykjsd.cn/v1 |
| API Key | your-api-key |
| Model ID | glm-5.2 |

Or in settings JSON:

```json
{
  "cursor.customModels": [
    {
      "name": "GLM 5.2",
      "baseURL": "https://tinyapi.nykjsd.cn/v1",
      "apiKey": "***",
      "model": "glm-5.2"
    }
  ]
}
```

### 6.3 Verify

Open Cursor, press `Ctrl+K` / `Cmd+K`, select GLM 5.2 and test.

---

## 7. Trae (Pro Version)

An AI IDE by ByteDance, built on VS Code. Pro version supports custom models.

> ⚠️ Requires Pro version with custom model support.

### 7.1 Install

Download from https://www.trae.ai

### 7.2 Configure Proxy API

Edit config file: `%APPDATA%\Trae\config.yaml`

```yaml
model_settings:
  retries: 10
  timeout: 300

custom_models:
  - name: "GLM 5.2"
    provider: "openai"
    base_url: "https://tinyapi.nykjsd.cn/v1"
    api_key: "***"
    model: "glm-5.2"
    max_tokens: 32768
```

### 7.3 Verify

Restart Trae, open AI chat panel, select GLM 5.2 and test.

---

## FAQ

**Q: API timeout?**
A: Set timeout to 300+ seconds. Claude Code: `API_TIMEOUT_MS`. Other tools: adjust `timeout` in settings.

**Q: Claude Code installation failed?**
A: Ensure VPN is on with US region. Run as admin in PowerShell/CMD. Or try `npm install -g @anthropic-ai/claude-code`.

**Q: Which model should I use?**
A: `glm-5.2` for best quality. `glm-5` for fast, lightweight tasks.

**Q: Where to get an API Key?**
A: 👉 [Purchase here](https://tinyapi.nykjsd.cn/home) — supports GLM, Claude, GPT, and more.

**Q: Which tool do you recommend?**
A: Claude Code (⭐) is the most powerful. zcode has best GLM compatibility. Crush (opencode) is great for terminal lovers.

**Q: What API protocol does the proxy support?**
A: Both OpenAI-compatible and Anthropic-compatible. Claude Code / zcode use Anthropic protocol. Cursor / CodeGPT / Crush use OpenAI protocol.

---

> 🔥 **Get Your API Key**: [https://tinyapi.nykjsd.cn/home](https://tinyapi.nykjsd.cn/home) — One key, all tools work!
