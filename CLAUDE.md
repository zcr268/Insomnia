# Insomnia

Lightweight Windows app (Electron) that prevents the PC from sleeping while AI coding tools (Claude Code, Cursor, Aider, Codex, Ollama) or watched apps are running. Tray icon + manual toggle. Hook-based detection for Claude Code and Codex; process-based for the rest.

## Agent Memory Protocol

Use this file as the standing project memory shared between Claude Code and Codex.

Rules:

1. Any meaningful code change, architecture decision, bug fix, regression, or workflow adjustment should be recorded here.
2. Each note should be prefixed with the agent name in brackets, for example `[Claude]` or `[Codex]`.
3. Prefer concise entries that explain what changed, why it changed, and any follow-up risk.
4. Keep older entries unless they are clearly obsolete; append instead of rewriting history unless the file needs cleanup.
5. If the file's encoding gets damaged, normalize it back to plain ASCII or valid UTF-8 while preserving meaning.

Suggested note format:

```text
### YYYY-MM-DD
- [AgentName] What changed. Why it changed. Any follow-up note.
```

## Agent Log

### 2026-04-26
- [Claude] Stable base: v1.3.0 shipped and tagged. Core detection logic uses SESSION_TIMEOUT_MS=3min (hook inactivity), SESSION_PROCESS_GRACE_MS=5min (process alive + last hook within window), PROCESS_GRACE_MS=30s (process-based integrations). Claude Code is hook-based via `~/.claude/settings.json`; Cursor/Aider/Codex/Ollama were process-based. `code.exe` removed from Claude Code processNames (always running in VS Code, caused permanent purple). `pending_response` flag and child-process detection both reverted — both caused cascading bugs. Keep solutions simple.
- [Claude] Tooltip: changed "Manual mode" → "Manually triggered" to match plain-English expectation.
- [Claude] SEO pass: optimized `docs/index.html` (landing page) with Open Graph tags, Twitter Cards, JSON-LD structured data, canonical URL, favicon link, and keyword-rich title/description targeting "keep windows awake", "prevent sleep Claude Code", "caffeine alternative". Expanded GitHub repo topics to 14. Blog post published on Hashnode; Reddit posts on r/ClaudeAI, r/SideProject, r/electronjs.
- [Claude] SEO pass: optimized `README.md` — H1 now includes target keywords, "90 seconds" timeout references updated to "3 minutes" (matching actual code), "Manual mode" → "Manually triggered" in tooltip example, Ollama use-case added, "Why Not Just Change Power Settings?" section added to capture that search intent.
- [Claude] v1.4.0: Codex integration converted from process-based to hook-based. Old behavior kept the PC awake the entire time `codex.exe` was alive — including idle sessions in the VS Code terminal — so the user walked away and the PC never slept until the battery died. New behavior writes `notify = ["node", "<agent-hook.js>", "stay-awake", "codex-cli"]` to `~/.codex/config.toml`, which fires on every Codex turn-complete event regardless of interface (CLI, VS Code extension, desktop app). Hook setup is re-applied on app startup for already-enabled hook-based integrations. Renamed integration from "OpenAI Codex CLI" → "OpenAI Codex" with a startup migration that updates the stored display name. Removed the bare `'codex'` string from `processNames` (over-broad substring). Generalized IPC dispatch (`setupHooksForIntegration` / `removeHooksForIntegration`).
- [Claude] v1.4.1: removed the "synthetic activity on process newly detected" logic added in v1.4.0. Intent was to cover Codex's blind spot before the first `notify` fires. In practice the ChatGPT VS Code extension keeps `codex.exe` app-server alive for the entire VS Code session, so synthetic fired on every install and produced a 5-min false-positive "active" window — the "stays awake when nothing is running" bug. Also added startup cleanup that strips `:auto-detected` entries from `~/.insomnia/agent-sessions.json` left by v1.4.0.

### 2026-05-02
- [Claude] v1.4.2: discovered that the Codex VS Code extension (`openai.chatgpt`) runs as `codex.exe app-server` — a persistent daemon that does NOT fire the `notify` hook from `config.toml` (that only works in interactive CLI mode). Diagnosed by confirming zero `codex-cli` entries ever appeared in `agent-sessions.json` after VS Code Codex usage. Fix: watch `~/.codex/logs_2.sqlite-wal` directly — verified it gets written on every processed Codex request but stays untouched during idle (confirmed across hours of idle app-server running). When the WAL file changes, Insomnia writes a `codex-cli:vscode` activity entry to `agent-sessions.json`; the existing 3-min / 5-min timeout logic handles the rest. The CLI `notify` hook remains in place as a second signal for terminal users. Watcher starts on app launch if codex-cli is enabled, and is started/stopped on integration enable/disable.

### 2026-05-03
- [Codex] Hardened Codex activity detection without changing Claude behavior. Root cause in the local install: the running installed app was still a v1.4.1-era build from 2026-04-26, so it did not include the v1.4.2 WAL watcher even though the source/dist had moved on. Source fix: Codex now watches `~/.codex` plus `logs_2.sqlite-wal` and `logs_2.sqlite`, polls those files every 5s as a Windows `fs.watch` fallback, handles WAL creation/recreation, and immediately re-evaluates wake state after writing `codex-cli:vscode` activity. This is intended to cover VS Code Codex and the standalone Codex app surfaces that use `codex.exe app-server`, while the CLI `notify` hook remains in place.
- [Codex] Emergency correction: the SQLite log/state signal is too noisy and updates while Codex is merely open/idle, causing a permanent false-positive "staying awake for Codex" state. Removed SQLite DB watching and switched app-server detection to polling `~/.codex/sessions/**/*.jsonl` transcript mtime/size changes every 5s. Added startup cleanup for the bad `codex-cli:vscode` session entry. Claude hook behavior remains untouched.

### 2026-05-09
- [Codex] Repo is public at `https://github.com/stanley-projects/Insomnia` with GitHub Pages at `https://stanley-projects.github.io/Insomnia/`. Public promotion so far includes the DEV post "I Built a Tiny Electron App to Stop Windows From Killing My Claude Code Sessions"; earlier memory also records posts on Reddit in r/ClaudeAI, r/SideProject, and r/electronjs. Note: the DEV post still describes Codex as process-watched, which matches the public v1.3 era but not the local v1.4.2 Codex activity work.
- [Codex] Committing and pushing local v1.4.2 source changes to `master`: package version bump, UI/footer version bump, README/docs updates, Codex notify hook setup, Codex session-transcript activity watcher for VS Code/standalone app-server surfaces, startup migration/cleanup, and this shared memory file. Claude Code hook setup is intentionally unchanged.

## Current State (as of 2026-05-09)

- **Version**: Source is v1.4.2 on `master` after push. Local installer exists in `dist/` and a copy exists in user Downloads.
- **Detection**: Claude Code = hook-based (`~/.claude/settings.json`). Codex = hook-based dual-path: `notify` in `~/.codex/config.toml` for CLI + session transcript watcher/poller for VS Code and standalone app-server surfaces. Cursor/Aider/Ollama = process-based with 30s grace.
- **Known limitation**: If user's very first Codex CLI prompt takes >3 min before any `notify` fires, PC may sleep mid-generation. Subsequent turns are covered by 5-min process grace. Acceptable trade-off.
- **Remaining release work**: Create/publish a GitHub release for v1.4.2 with the installer, then update package channels if needed (Scoop/winget). Update the DEV post or add a note if mentioning current Codex behavior matters.
