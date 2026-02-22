# OpenClaw + Ollama + Telegram Bot (Local macOS Setup)

This repository documents the exact steps used to configure a **local OpenClaw agent** powered by **Ollama (gpt-oss:20b)** and connected to a **Telegram bot** on macOS.

The goal of this setup is to run a fully local AI agent with:

* Local LLM via Ollama
* OpenClaw automation gateway
* Telegram Bot interface
* Web UI + TUI control

---

# Architecture Overview

```
Telegram → OpenClaw Gateway → Agent → Ollama → Local Model
```

Components:

* **Ollama** → runs the local language model
* **OpenClaw Gateway** → routes messages + manages sessions
* **Telegram Bot** → user-facing chat interface
* **TUI / Web UI** → local control interface

---

# Prerequisites

* macOS
* Node.js installed
* Homebrew installed
* Telegram account

---

# Step 1 — Verify system + Ollama installation

Check architecture and Ollama binary:

```bash
uname -m
sw_vers
which ollama
ollama --version
file "$(which ollama)"
```

Expected:

* `x86_64` (Intel Mac) OR `arm64`
* Ollama binary present at `/usr/local/bin/ollama`

Note:

On Intel Macs, MLX warnings like:

```
Failed to load MLX dynamic library
```

are harmless and can be ignored.

---

# Step 2 — Start Ollama and download model

Run:

```bash
ollama run gpt-oss:20b
```

This downloads ~13GB model and launches interactive prompt.

Test with:

```
Send me a short summary of what DevSecOps is.
```

Exit:

```
/bye
```

---

# Step 3 — Create OpenClaw config directory

```bash
mkdir ~/.openclaw
cd ~/.openclaw
```

If directory already exists, continue.

---

# Step 4 — Create OpenClaw config file

```bash
touch openclaw.json
nano openclaw.json
```

Example configuration used:

```json
{
  "models": {
    "providers": {
      "ollama": {
        "baseUrl": "http://localhost:11434/v1",
        "apiKey": "ollama-local",
        "api": "openai-completions",
        "models": [
          {
            "id": "gpt-oss:20b",
            "name": "gpt-oss:20b",
            "reasoning": false,
            "input": ["text"],
            "cost": {
              "input": 0,
              "output": 0,
              "cacheRead": 0,
              "cacheWrite": 0
            },
            "contextWindow": 131072,
            "maxTokens": 8192
          }
        ]
      }
    }
  },
  "agents": {
    "defaults": {
      "model": {
        "primary": "ollama/gpt-oss:20b"
      },
      "workspace": "~/.openclaw/workspace",
      "maxConcurrent": 4,
      "subagents": {
        "maxConcurrent": 8
      }
    }
  },
  "tools": {
    "web": {
      "search": { "enabled": false },
      "fetch": { "enabled": true }
    }
  }
}
```

---

# Step 5 — Run OpenClaw onboarding

```bash
openclaw onboard
```

Selections used:

* Continue after security warning → **Yes**
* Mode → **QuickStart**
* Config → **Update values**
* Provider → **Skip**
* Default model → **Keep ollama/gpt-oss:20b**

---

# Step 6 — Create Telegram Bot

Inside Telegram:

1. Search **@BotFather**
2. Press **Start**
3. Run:

```
/newbot
```

4. Enter:

* Bot name
* Bot username

5. Copy the generated token:

```
123456:ABC-XYZ...
```

---

# Step 7 — Configure Telegram inside OpenClaw

During onboarding:

* Select channel → **Telegram (Bot API)**
* Paste BotFather token

Example:

```
Enter Telegram bot token
XXXXXXXXXX:XXXXXXXXXXXX
```

---

# Step 8 — Fix workspace permission issue (if occurs)

You may see:

```
Error: EACCES: permission denied, mkdir '/Users/...'
```

Fix by editing the workspace path inside:

```
~/.openclaw/openclaw.json
```

Ensure it points to:

```
~/.openclaw/workspace
```

Then rerun:

```bash
openclaw onboard
```

---

# Step 9 — Install Gateway Service

OpenClaw installs a macOS LaunchAgent:

```
~/Library/LaunchAgents/ai.openclaw.gateway.plist
```

Gateway logs stored at:

```
~/.openclaw/logs/gateway.log
```

---

# Step 10 — Start TUI

```bash
openclaw tui - ws://127.0.0.1:18789 - agent main - session main
```

---

# Step 11 — Approve pairing request

If prompted:

```
Pairing required
```

Run in another terminal:

```bash
openclaw devices list
```

Approve:

```bash
openclaw devices approve <REQUEST_ID>
```

Reconnect TUI.

---

# Step 12 — Access Web Dashboard

Local dashboard:

```
http://127.0.0.1:18789/
```

Token stored in:

```
~/.openclaw/openclaw.json
```

Retrieve:

```bash
openclaw config get gateway.auth.token
```

---

# Verify Telegram Bot Works

Send a message to your Telegram bot.

If configured correctly:

* Gateway receives message
* Agent processes prompt
* Ollama model responds

---

# Useful Commands

## Check Ollama

```bash
ollama list
ollama ps
```

## Check OpenClaw

```bash
openclaw agents list
openclaw sessions list
openclaw gateway status
```

## View logs

```bash
tail -f ~/.openclaw/logs/gateway.log
```

---

# Troubleshooting Notes From Real Setup

## MLX library error on Intel Mac

Safe to ignore:

```
Failed to load MLX dynamic library
```

Intel Macs run CPU inference only.

---

## Ollama port already in use

Means server already running.

Do NOT start another `ollama serve`.

---

## Pairing loop in TUI

Always approve device first:

```
openclaw devices list
openclaw devices approve <id>
```

---

## Telegram configured but plugin disabled

Run onboarding again and keep existing values.

---

# Security Notes

OpenClaw agents can:

* Read files
* Execute tools
* Access network

Always:

* Keep secrets outside workspace
* Avoid exposing gateway publicly without auth
* Run periodic audit:

```bash
openclaw security audit --deep
```

---

# Result

Working system:

* Local AI model via Ollama
* OpenClaw gateway running as service
* Telegram bot connected
* TUI + Web UI operational

---

## License

This project is licensed under the MIT License — see the LICENSE file for details.
