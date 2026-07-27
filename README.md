
# ⚙️ OpenWorker
### Sovereign Local AI Coworker for Desktop Task Automation & Multi-Tool Execution

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg?style=for-the-badge)](LICENSE)
[![Local AI: Ollama](https://img.shields.io/badge/Local%20AI-Ollama%20Support-FF6F00?style=for-the-badge&logo=ollama)](https://ollama.com)
[![Platform: macOS & Windows](https://img.shields.io/badge/Platform-macOS%20%7C%20Windows-0078D4?style=for-the-badge&logo=windows)](https://openworker.com)
[![Engine: aisuite](https://img.shields.io/badge/Engine-aisuite%20(Python)-blue?style=for-the-badge&logo=python)](https://github.com/andrewyng/aisuite)
[![Framework: React + Tauri](https://img.shields.io/badge/Framework-React%20%2B%20Tauri-61DAFB?style=for-the-badge&logo=react)](https://tauri.app)

---

<p align="center">
  <strong>OpenWorker</strong> is an open-source AI coworker that lives on your desktop and delivers <strong>finished work</strong>—polished documents, Slack replies, calendar updates, and triaged inboxes. Connect cloud models or run fully local with Ollama.
</p>

</div>

---

## 📑 Table of Contents
- [💡 What is OpenWorker & Why it Matters](#-what-is-openworker--why-it-matters)
- [📥 macOS & Windows App Download & Installation](#-macos--windows-app-download--installation)
- [🔑 Understanding Local Secret Store & Authentication](#-understanding-local-secret-store--authentication)
- [🌐 Desktop Automation & Multi-Tool Workflow](#-desktop-automation--multi-tool-workflow)
- [🎨 OpenWorker UI & Surface Breakdown](#-openworker-ui--surface-breakdown)
- [🛠️ Building `gemm4-no-think` in Ollama](#️-building-gemm4-no-think-in-ollama)
- [🔌 Connecting Ollama to OpenWorker & Fixing Agent Errors](#-connecting-ollama-to-openworker--fixing-agent-errors)
- [⚡ 3 Quick In-App Tests](#-3-quick-in-app-tests)
- [📂 Project File Architecture](#-project-file-architecture)
- [🚀 5 Real-World Use Cases](#-5-real-world-use-cases)
- [🔮 5 Roadmap & Future Features](#-5-roadmap--future-features)
- [💻 Tech Stack Overview](#-tech-stack-overview)

---

## 💡 What is OpenWorker & Why it Matters

**OpenWorker** is designed to shift AI interaction from basic chat prompts to **autonomous task completion**. Built on top of [**aisuite**](https://github.com/andrewyng/aisuite) by Andrew Ng and the AI Fund team, OpenWorker combines local desktop tools, 25+ third-party integrations, and Model Context Protocol (MCP) support into a single native application.

```
 ┌──────────────────────────────────────────────────────────────────┐
 │           OpenWorker Desktop App (React + Tauri Shell)          │
 └────────────────────────────────┬─────────────────────────────────┘
                                  │ HTTP / Native IPC
                                  ▼
 ┌──────────────────────────────────────────────────────────────────┐
 │            Local Agent Server (Python coworker Engine)           │
 │               (built on aisuite & MCP Protocol)                  │
 └──────┬─────────────────────────┬─────────────────────────┬───────┘
        │                         │                         │
        ▼                         ▼                         ▼
 ┌──────────────┐          ┌──────────────┐          ┌──────────────┐
 │ Desktop Files│          │ 25+ Connectors│          │  Local Model │
 │  & Terminal  │          │Slack/Jira/MCP│          │ (Ollama /    │
 └──────────────┘          └──────────────┘          │ Cloud LLMs)  │
                                                     └──────────────┘
```

### Key Differentiators
- 🎯 **Delivers Finished Deliverables**: Generates Markdown files, spreadsheets, draft reports, and direct Slack responses rather than raw chat lists.
- 🛡️ **Human-in-the-Loop Safeguards**: Actions with consequences (sending emails, modifying calendars, running terminal commands) require explicit approval.
- 🔌 **Extensible via MCP & 25+ Connectors**: Native integration with Slack, Jira, GitHub, Notion, Linear, HubSpot, Gmail, and Google Calendar.
- 🏠 **100% Data Sovereignty & Offline Capability**: Run completely offline using local models via Ollama or connect any provider with zero telemetry leakage.

---

## 📥 macOS & Windows App Download & Installation

### Official Download Links
| Platform | Binary | Link |
| :--- | :--- | :--- |
| **macOS (Apple Silicon)** | `.dmg` | [Download macOS Release](https://download.openworker.com/mac) |
| **Windows 10/11 (x64)** | `.exe` / Installer | [Download Windows Release](https://download.openworker.com/windows) |
| **Source Code** | Monorepo Zip | [OpenWorker GitHub Repository](https://github.com/andrewyng/openworker) |

### Local Source Build & Development Setup

#### Prerequisites
- **Python**: `3.10+`
- **Node.js**: `20+`
- **Rust Toolchain**: `rustup` (required for Tauri shell)

#### 1. Clone & Bootstrap Environment
```bash
git clone https://github.com/andrewyng/openworker
cd openworker

# Run setup script (creates Python venv at .venv)
bash packaging/setup_dev_env.sh
```

#### 2. Launch Local Agent Server
```bash
# Windows PowerShell / Git Bash:
.venv/Scripts/openworker-server.exe --cwd . --port 8765

# macOS / Linux:
.venv/bin/openworker-server --cwd . --port 8765
```

#### 3. Launch UI Surface
```bash
cd surfaces/gui
npm install

# Option A: Vite Browser Dev Surface
npm run dev

# Option B: Full Tauri Desktop Application
npm run tauri dev
```

---

## 🔑 Understanding Local Secret Store & Authentication

OpenWorker operates under a **local-first security model**:

1. **Local Secret Store**: API keys, OAuth tokens, and model keys live exclusively on your local file system, protected by OS-level keyring storage.
2. **Sidecar Security Token**: When running the agent server, a single-use token is generated at `<state-dir>/sidecar-8765.token`. Frontends pass this token in the `X-OpenWorker-Token` header.
3. **No Mandatory Account**: You do not need an account to use OpenWorker. Cloud services are limited strictly to brokering OAuth handshakes for third-party tools (e.g. Slack/Google Calendar).

---

## 🌐 Desktop Automation & Multi-Tool Workflow

```text
 User Task Request ──► Step Decomposition ──► Tool Execution ──► Approval Checkpoint ──► Final Deliverable
```

1. **Task Input**: Enter prompt directly in the UI or mention `@OpenWorker` in a connected Slack channel.
2. **Step Decomposition**: The agent engine uses `aisuite` to sequence steps across terminal, local files, and MCP connectors.
3. **Approval Inbox**: Destructive or outbound actions pause for user confirmation.
4. **Artifact Delivery**: Results land directly as downloadable files or updated workspace records.

---

## 🎨 OpenWorker UI & Surface Breakdown

```text
┌───────────────────┬────────────────────────────────────────────────────────┐
│  OPENWORKER       │  Task: Build Customer Weekly Summary Report           │
├───────────────────┼────────────────────────────────────────────────────────┤
│ WORKSPACE         │  [Step 1/3] Reading local markdown files... COMPLETE   │
│  📁 Project Root  │  [Step 2/3] Fetching recent closed issues from Jira    │
│  ⚙️ Connectors     │                                                        │
│                   │  ⚠️ APPROVAL REQUIRED                                  │
│ TASKS             │  Action: Execute command `git log -n 10`               │
│  • Weekly Report  │  [ Approve ] [ Reject & Redirect ]                    │
│  • Slack Triage   │                                                        │
│                   │  ----------------------------------------------------  │
│ APPROVAL INBOX 📥 │  Deliverable: Customer_Summary_Q3.md generated.       │
│  (1 Pending)      │  [ Open File ]  [ Export PDF ]                        │
└───────────────────┴────────────────────────────────────────────────────────┘
```

| UI Surface | Purpose & Description |
| :--- | :--- |
| **Workspace Bar** | Select working directory, toggle active connectors, manage Ollama provider endpoints. |
| **Tasks Feed** | Displays ongoing and completed autonomous multi-step agent runs. |
| **Approval Inbox** | Centralized queue for gating file writes, outbound emails, and shell commands. |
| **Deliverables View** | Interactive viewer for generated documents, tables, code patches, and reports. |

---

## 🛠️ Building `gemm4-no-think` in Ollama

To provide fast, deterministic performance without unnecessary thinking monologues, we create a custom model named **`gemm4-no-think`** based on **`gemma4:e2b`**.

### 1. The `Modelfile`
The repository includes a custom [`Modelfile`](file:///d:/Ray%20Codes/AG%20Projects/Openworker/Modelfile):

```dockerfile
# Custom Ollama Modelfile for gemm4-no-think
# Built from gemma4:e2b for direct, deterministic, zero-thinking-overhead AI agent execution in OpenWorker

FROM gemma4:e2b

# Model parameters tuned for high-speed, direct tool calls and document generation
PARAMETER temperature 0.2
PARAMETER top_p 0.9
PARAMETER stop "<start_of_turn>"
PARAMETER stop "<end_of_turn>"
PARAMETER num_ctx 8192

# System Prompt for OpenWorker Desktop & Autonomous Agents
SYSTEM """You are gemm4-no-think, a direct, highly capable AI coworker in OpenWorker. You execute desktop tasks, execute local tool calls, write code, interact with APIs (Slack, Jira, GitHub, Notion), and summarize data without unnecessary thinking monologues, chain-of-thought intros, meta-conversational commentary, or verbose filler text."""
```

### 2. Build Steps in Terminal
Execute the following commands in your terminal:

```powershell
# Step 1: Pull base Gemma 4 e2b model
ollama pull gemma4:e2b

# Step 2: Create custom gemm4-no-think model
ollama create gemm4-no-think -f Modelfile

# Step 3: Verify model availability
ollama list
```

---

## 🔌 Connecting Ollama to OpenWorker & Fixing Agent Errors

### Resolving Connection & CORS Errors

When running OpenWorker desktop with Ollama, requests may fail due to **CORS cross-origin restrictions** or missing headers.

#### Configure Ollama Environment Variables
Configure Ollama to accept cross-origin requests from desktop shells:

```powershell
# Windows PowerShell
$env:OLLAMA_ORIGINS="*"
$env:OLLAMA_HOST="0.0.0.0"
ollama serve
```

### OpenWorker Settings Configuration

1. Open **OpenWorker Settings** -> **Models & Providers**.
2. Select **Ollama / OpenAI Compatible**.
3. Set fields:
   - **Base URL**: `http://localhost:11434/v1`
   - **API Key**: `ollama`
   - **Model Name**: `gemm4-no-think`
4. Click **Save & Test Connection**.

---

## ⚡ 3 Quick In-App Tests

Try these 3 brief, high-value tests inside the OpenWorker app to verify `gemm4-no-think` performance:

### Task 1: Executive Workspace Audit
```text
Summarize the current project directory structure and git status into a 3-bullet summary.
```
> **What it proves**: Tests instant desktop directory reading and structured output without chain-of-thought delay.

### Task 2: Automated Slack Standup Draft
```text
Draft a concise 2-sentence Slack update for @channel on today's release deliverables.
```
> **What it proves**: Demonstrates rapid multi-connector text formatting designed for direct team delivery.

### Task 3: Fast Shell Command Execution
```text
Find all files modified in the last 24 hours and list their filenames.
```
> **What it proves**: Verifies fast local tool invocation gated by OpenWorker's approval UI.

---

## 📂 Project File Architecture

```
d:/Ray Codes/AG Projects/Openworker/
├── Modelfile                     # Custom Ollama definition (optional gemm4-no-think model)
├── README.md                     # Comprehensive GitHub-ready documentation (this file)
├── README (69).md                # Upstream OpenWorker technical specification reference
├── coworker/                     # Python backend: agent engine, connectors, MCP client, memory
├── surfaces/gui/                 # React UI + Tauri desktop app shell
├── stt/                          # Speech-to-text Rust sidecar for voice commands
├── packaging/                    # Desktop build scripts (macOS DMG, Windows PS1)
└── tests/                        # Backend pytest suite & GUI e2e tests
```

| File / Folder | Purpose & Function |
| :--- | :--- |
| [`Modelfile`](file:///d:/Ray%20Codes/AG%20Projects/Openworker/Modelfile) | Custom Ollama specification for fast, direct, zero-thought agent task execution. |
| [`README.md`](file:///d:/Ray%20Codes/AG%20Projects/Openworker/README.md) | Standardized documentation following `ainews` skill guidelines. |
| [`coworker/`](file:///d:/Ray%20Codes/AG%20Projects/Openworker/coworker/) | Core Python agent harness built on `aisuite`. |
| [`surfaces/gui/`](file:///d:/Ray%20Codes/AG%20Projects/Openworker/surfaces/gui/) | React + Vite frontend wrapped inside a Tauri Rust desktop container. |

---

## 🚀 5 Real-World Use Cases

1. 🔒 **Air-Gapped Codebase Analysis**: Inspect local repositories, refactor components, and generate technical documentation without leaking source code to external cloud providers.
2. 📊 **Automated Daily Standups & Jira Sync**: Summarize completed tickets and git commits into formatted daily update drafts.
3. 🧹 **Scheduled System Diagnostics & Cleanup**: Run scheduled desktop tasks to identify stale logs, temp files, and unused dev containers.
4. 🛠️ **Safe Shell Script Automation**: Execute complex multi-step build scripts with strict human approval gates at every step.
5. 🔌 **Custom Enterprise MCP Integration**: Connect proprietary enterprise databases and microservices via standard Model Context Protocol servers.

---

## 🔮 5 Roadmap & Future Features

1. 🤖 **Multi-Agent Swarm Orchestration**: Support delegating sub-tasks across multiple local Ollama instances simultaneously.
2. ⚡ **Dynamic Model Routing**: Automatically send light tool tasks to local models and complex code reasoning to larger frontier models.
3. 🎙️ **Voice Huddle & Real-Time Audio Engine**: Enhanced Rust speech-to-text sidecar with real-time voice response capability.
4. 🔐 **Enterprise Role-Based Access Control (RBAC)**: Fine-grained permissions for multi-user shared workspace setups.
5. 📱 **Mobile Approval Companion App**: Receive push notifications to review and approve pending desktop agent actions on mobile devices.

---

## 💻 Tech Stack Overview

- **Agent Engine**: Python 3.10+ (`aisuite`, AsyncIO, Pydantic)
- **Desktop UI**: React 18 + TypeScript + Vite + Tailwind CSS
- **App Shell**: Tauri 2.0 (Rust)
- **Local AI Engine**: Ollama (Optional custom models e.g. `gemm4-no-think`) / Cloud LLMs
- **Integration Protocol**: Model Context Protocol (MCP) + 25 Native Connectors
- **Voice Sidecar**: Rust STT (Whisper binding)

---

## 🏷️ Keywords & Tags

`Andrew Ng OpenWorker` `OpenWorker` `aisuite` `Ollama` `Desktop AI Coworker` `Local AI Agent` `Model Context Protocol` `MCP` `Tauri AI App` `React Desktop Automation` `Sovereign AI` `Offline AI` `Autonomous Workflows` `Free AI Setup` `Local LLM`

---

<div align="center">
  <sub>Built with ⚙️ by Andrew Ng & the OpenWorker Team.</sub>
</div>
