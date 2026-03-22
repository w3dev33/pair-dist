![PaiR](assets/logo-dark.png)

<p align="center"><a href="https://pair.w3dev.fr"><img src="https://img.shields.io/badge/Visit_Website-4a90d9?style=for-the-badge&logo=globe&logoColor=white" alt="Visit Website"></a></p>

A lightweight desktop application for managing tasks and issues directly inside your codebase.

PaiR ships with its own CLI (`pair`) — no external tool to install. The CLI manages issues across multiple independent projects, and the application aggregates them all in a single window, updated in real time. The CLI does the heavy lifting — AI agents use it to create, update, and close issues programmatically — while the app gives you a unified view and control over everything. Issues live in your project directory, tracked by git, and visible to both humans and AI agents.

## Why PaiR?

AI-assisted development is moving fast. Multi-agent orchestration, autonomous task management, swarms of AI workers — the tools are impressive, but they're racing ahead of most developers' reality.

We believe the transition to AI autonomy should be progressive. Today, most developers work **with** AI — pair programming, reviewing suggestions, steering decisions. They need to see what's happening, understand it, and stay in control. Jumping straight to full autonomy means losing the ability to learn, verify, and course-correct.

PaiR is built for this transition. Start with pair programming — one human, one AI, full visibility. As trust builds, delegate more. But at every step, you can see what's going on, step in, and redirect. It's not just a workflow — it's a learning process.

The name says it all: **PaiR** — the human and the AI, working side by side.

## Installation

Download PaiR for macOS, Linux and Windows:

[![Download PaiR](https://img.shields.io/badge/Download_PaiR-macOS_·_Linux_·_Windows-28a745?style=for-the-badge&logo=download&logoColor=white)](https://pair.w3dev.fr)

The `pair` CLI is bundled with the application and a symlink is created automatically during installation, making it available from any terminal.

## Features

### Core
- **Multi-project dashboard** — All your projects in one window, with real-time updates, charts, and per-project settings
- **Built-in CLI** — `pair` ships with the app. Full issue lifecycle from the terminal: create, update, close, pin, comment, attach, search
- **Git-synced** — Issues live in `.pair/` inside your repo, tracked by git, visible to humans and AI agents alike

### Issue Management
- **Epics & children** — Parent/child hierarchy with ordered subtasks
- **Dependencies** — Blocks/blocked-by relationships with visual indicators
- **Pinned issues** — Pin important issues for quick access — stored in the DB, synced via git, with undo support (⌘Z)
- **Specs & campaigns** — Dedicated dashboard sections for specifications and ongoing tracking
- **Attachments** — Images, markdown, and PDF files attached to issues, previewed in-app
- **Full-text search** — FTS5-powered search across titles, descriptions, and notes

### Integrated Terminal
- **Multi-session terminals** — Run multiple terminal sessions per project with a tabbed interface, directly inside PaiR. No need to switch to an external terminal
- **tmux mode** — Persistent sessions that survive app restarts. Access your sessions from any external terminal (Zed, iTerm, VS Code) via `tmux attach`. Sessions are automatically named after issue IDs
- **Smart session close** — Detects when a tmux session is attached in another terminal and warns before killing. Defaults to detach when used elsewhere, so your external terminal keeps working
- **Launch AI from issues** — Click Play on any issue to open a terminal tab and start Claude Code with `/run-issue` automatically. Closing an issue automatically closes its terminal session
- **Split view** — Pin tabs to display terminals side by side. A built-in multiplexer experience for monitoring several AI sessions in parallel
- **Smart notifications** — When an AI agent running inside PaiR needs your attention, the terminal tab flashes and turns red with a sound alert. Click the notification to jump directly to the right tab
- **AI Text Transform** — Transform text in any input field using AI: reformulate, translate, or summarize. Works with any Claude subscription

### AI Integration
- **AI-native workflow** — Designed for AI coding assistants (Claude Code, Cursor, Codex, Gemini CLI). AI agents use the CLI to manage issues autonomously
- **Live activity tracking** — AI activity LED per project, events panel (⌘⇧A), focus AI session window (⌘⇧F)
- **Sound alerts** — Per-project notification sounds when AI agents interact with your issues. Internal sessions (PaiR terminal) get visual tab alerts, external sessions (Zed, Cursor, VS Code) get toast notifications
- **Real-time push** — CLI mutations and AI events pushed instantly via Unix socket

### Sync & Collaboration
- **GitHub / GitLab sync** — Bidirectional issue sync with external providers, comment push/pull
- **Conflict resolution** — Detects and resolves merge conflicts when multiple collaborators edit the same issue

### Cross-platform
- macOS, Linux, Windows — with dark, light, flat, and neon themes

## Background & Compatibility

PaiR is inspired by [Beads](https://github.com/steveyegge/beads), the AI-native issue tracker created by Steve Yegge, which stores issues directly in the codebase using a SQLite + JSONL structure. Our first take was [Beads Task-Issue Tracker](https://github.com/w3dev33/beads-task-issue-tracker), a desktop app built as a frontend for the existing Beads CLIs (`bd`, `br`). But as those CLIs evolved in diverging directions — Dolt migration, server mode, breaking changes — depending on external tools became a liability. PaiR was built from scratch with its own CLI, its own schema, and its own features to move at its own pace.

For existing Beads projects, the app handles migration automatically. The migration is **non-destructive** — your `.beads/` directory is left untouched. PaiR creates its own `.pair/` directory alongside it, so you can try PaiR without any risk to your existing data.

- **`bd`** (Go) projects up to version 0.49.x (before the Dolt migration)
- **`br`** ([beads_rust](https://github.com/Dicklesworthstone/beads_rust)) projects up to version 0.1.20+

## For AI agents

PaiR is designed to be driven by AI coding assistants (Claude Code, Cursor, Codex, Gemini CLI, etc.). The `pair` CLI is bundled — agents use it to manage issues, and the app reflects every change in real time.

### Setup

1. **Initialize the project**: from the app (add a folder) or via `pair init`
2. **Read the reference**: an `AGENTS.md` is generated in `.pair/` with the full CLI documentation — commands, flags, workflows, and expected behaviors
3. **Install notification hooks** (Claude Code): add `pair notify --hook` hooks in `.claude/settings.json` so the app gets real-time AI activity notifications (sound, tab flash, toast). The setup instructions are in `AGENTS.md`

### Teaching the agent

Add these rules to your project's `CLAUDE.md` or equivalent agent configuration:

```markdown
## Issue tracking

- Use `pair` CLI for all issue management (create, update, close, comment, search)
- Before starting work: `pair list -s open` to check existing issues, then `pair update <id> -s in_progress`
- Before creating an issue: `pair search "keyword"` to avoid duplicates
- When done: `pair comments add <id> "Summary of what was done"` then `pair close <id>`
- Commit `.pair/issues.jsonl` separately from code: `chore(pair): update issues`
- Never ignore `.pair/` — it is the project's issue tracker
```

The key point: **the agent should treat `pair` like `git`** — not optional, part of the workflow. Every task starts with checking issues, every completion ends with closing one.

### CLI reference

The full command reference is in [`.pair/AGENTS.md`](.pair/AGENTS.md). Key commands:

```bash
pair list -s open              # Find available work
pair show <id>                 # Read full context (description, comments, children)
pair create "Title" -t bug     # Create an issue (-t bug/feature/task/epic, -d, -p, --parent)
pair update <id> -s in_progress
pair comments add <id> "Progress update"
pair close <id>
pair search "keyword"          # Full-text search across all issues
pair attach <id> file.png      # Attach images, markdown, PDFs
```

### Integrated terminal

Instead of running agents in an external terminal, launch them directly from PaiR:

- **Click Play on any issue** — opens a terminal tab named after the issue and starts the AI assistant with the right context
- **tmux mode** — sessions persist across app restarts. Attach from any external terminal via `tmux attach -t pair-<issue-id>` to monitor or intervene while the agent works
- **Real-time notifications** — when the agent needs attention, the tab flashes red with a sound alert. Click to jump to the right session
- **Split view** — pin tabs to monitor multiple agents side by side

### Real-time push

Every CLI mutation triggers a push event — the app refreshes instantly. On macOS and Linux, this uses a Unix socket for real-time push. On Windows, the app polls for changes. Agents running inside PaiR's terminal are automatically detected, and their activity is routed to the correct project and tab.

No special integration needed — agents just use the `pair` CLI, and the app reacts.

## Links

- [Changelog](https://github.com/w3dev33/pair-dist/blob/main/CHANGELOG.md)
- [Website](https://pair.w3dev.fr)
- [Beads Task-Issue Tracker](https://github.com/w3dev33/beads-task-issue-tracker) (previous version)
- [Beads (original project)](https://github.com/steveyegge/beads)
- [beads_rust](https://github.com/Dicklesworthstone/beads_rust)

## Disclaimer

PaiR is free to use. The software is provided as-is, with no warranty. Your data stays local — issues are stored in your project directory — but as with any tool, regular backups are your responsibility. See [LICENSE](LICENSE) for details.

## License

[MIT](LICENSE) — Laurent Chapin
