# Insomnia — Keep Windows Awake During AI Coding Sessions

**Prevent your PC from sleeping while Claude Code, Cursor, Aider, or Codex are working.**

Insomnia is a free, lightweight Windows app that keeps your computer awake — automatically when AI coding agents are running, when specific apps are active, or manually with a toggle. A smarter caffeine alternative for developers: no more losing progress to an untimely screen timeout.

![Windows](https://img.shields.io/badge/platform-Windows-blue)
![Electron](https://img.shields.io/badge/built%20with-Electron-47848f)
![License](https://img.shields.io/badge/license-MIT-green)

---

## The Problem

You're running Claude Code, Aider, or another AI coding tool. It's churning through a complex task. You step away for a few minutes — and your PC goes to sleep, killing the process. You come back to a broken session and lost work.

Setting your screen timeout to 2 hours works, but then your PC never sleeps when you actually want it to.

**Insomnia solves this.** It keeps your PC awake only when something is actively running that needs it, and gets out of the way the rest of the time.

## Features

### Smart Integrations

Insomnia has built-in support for AI coding tools with two types of monitoring:

| Integration | Type | How It Works |
|---|---|---|
| **Claude Code** | Hook-based | Knows when Claude is *actively working* (not just open). Wakes on tool use, sleeps when idle. |
| **Cursor** | Process-based | Keeps awake while Cursor is running |
| **Aider** | Process-based | Keeps awake while `aider.exe` is running |
| **OpenAI Codex** | Hook-based | Knows when Codex is *actively working* (CLI, VS Code, or desktop). Uses Codex's `notify` config plus Codex session activity to track active work. |
| **Ollama** | Process-based | Keeps awake during local AI model inference |

The Claude Code integration is **hook-based** — it hooks directly into Claude Code's event system so your PC stays awake only while Claude is actively running tools and generating code, not when it's sitting idle waiting for your next prompt. Once Claude finishes, Insomnia releases within 3 minutes of inactivity.

### App Watching

Add any application as a trigger. Insomnia auto-discovers all installed apps on your system:

- Windows Start Menu programs
- Registry-installed apps
- Microsoft Store apps
- Desktop shortcuts
- Or browse manually for any `.exe`

When a watched app is running, your PC stays awake. Close it, and Insomnia steps back.

### Manual Toggle

A simple on/off switch for when you just need your PC to stay awake right now. No questions asked.

### System Tray

Insomnia lives in your system tray, out of your way. The tray icon tells you at a glance:

- **Purple owl (eyes open)** — Staying Awake (keeping your PC on)
- **Grey owl (eyes closed)** — Inactive (normal sleep behavior)
- **Hover tooltip** — Shows exactly why it's awake: *"Staying awake for — Claude Code"*, *"Staying awake for — Manually triggered + Chrome"*

Close the window and it keeps running in the tray. Right-click for quick controls.

## Installation

### Download

Head to the [Releases](https://github.com/stanley-projects/Insomnia/releases) page and download the latest `.exe` installer.

### winget

```
winget install StanleyProjects.Insomnia
```

### Scoop

```
scoop bucket add stanley-projects https://github.com/stanley-projects/scoop-stanley
scoop install stanley-projects/insomnia
```

### Build from Source

Requires [Node.js](https://nodejs.org/) (v18+).

```bash
git clone https://github.com/stanley-projects/Insomnia.git
cd Insomnia
npm install
npm start
```

## How It Works

Insomnia uses Electron's `powerSaveBlocker` API to prevent Windows from entering sleep mode. It evaluates whether to stay awake based on three signals:

1. **Manual toggle** — User explicitly wants the PC awake
2. **Process monitoring** — Polls `tasklist` every 10 seconds to check if watched apps are running
3. **Hook-based sessions** — For tools like Claude Code, lightweight hooks signal activity to a shared session file (`~/.insomnia/agent-sessions.json`). Sessions expire after 3 minutes of inactivity.

If *any* trigger is active, the PC stays awake. When *all* triggers go inactive, normal sleep behavior resumes.

### Claude Code Integration Details

When you enable the Claude Code integration, Insomnia adds hooks to `~/.claude/settings.json` that fire on:

- `UserPromptSubmit`, `PreToolUse`, `PostToolUse`, `PermissionRequest`, `Notification` → signal activity (stay awake)
- `SessionEnd` → signal idle (allow sleep)

This means your PC stays awake precisely while Claude is doing work — reading files, running commands, writing code — and goes back to normal the moment it stops. No wasted power, no interrupted sessions.

### OpenAI Codex Integration Details

When you enable the OpenAI Codex integration, Insomnia adds a `notify` hook to `~/.codex/config.toml` for Codex CLI activity. For Codex surfaces that run through the persistent app server, including the VS Code extension and standalone app, Insomnia also watches Codex's local session transcript activity in `~/.codex/sessions`. Codex updates those transcripts during turns, and Insomnia treats those writes as active work without keeping the PC awake just because `codex.exe` is open.

## Configuration

Settings are stored in your Electron user data directory and persist across restarts:

- **Watched apps** — list of executables to monitor, each with an enable/disable toggle
- **Watched integrations** — AI tools with their monitoring type and enabled state
- **Manual awake state** — remembered across restarts

## Screenshots

| Main Window | Add Trigger — Integrations | Add Trigger — Apps | System Tray |
|---|---|---|---|
| ![Main window](docs/screenshot.png) | ![Integrations](docs/screenshot-integrations.png) | ![Apps](docs/screenshot-apps.png) | ![Tray](docs/screenshot-tray.png) |

---

## Use Cases

- **AI-assisted coding** — Prevent sleep while Claude Code, Cursor, Aider, or Codex work on long tasks
- **Long downloads** — Watch your browser or download manager
- **Video rendering** — Watch your editing software so your PC doesn't sleep mid-render
- **Presentations** — Manual toggle before you present, toggle off when done
- **Game updates** — Watch Steam or your game launcher during large updates
- **Compiling** — Watch your IDE during long builds
- **3D printing** — Watch your slicer or printer software
- **Local AI inference** — Keep awake while Ollama runs large language models

## Tech Stack

- **Electron** — Cross-platform desktop framework (Windows support)
- **Plain HTML/CSS/JS** — No React, no bundlers, no build tools
- **Zero external dependencies** — Only Electron itself
- **Windows APIs** — `tasklist` for process detection, PowerShell for app discovery, `powerSaveBlocker` for sleep prevention

## Contributing

Pull requests welcome. If you'd like to add an integration for another AI coding tool or improve app discovery, feel free to open a PR.

## Why Not Just Change Power Settings?

Setting your screen timeout to "Never" or 2+ hours works, but then your PC never sleeps when you actually want it to — wasting power and wearing your hardware. Insomnia keeps your PC awake **only** when something needs it, and automatically steps back when it doesn't. It's like caffeine for your PC, but smarter.

## License

MIT
