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
- **Epics & children** — Parent/child hierarchy with drag-and-drop reordering
- **Dependencies** — Blocks/blocked-by relationships with visual indicators
- **Pinned issues** — Pin important issues for quick access — stored in the DB, synced via git, with undo support (⌘Z)
- **Specs & campaigns** — Dedicated dashboard sections for specifications and ongoing tracking
- **Attachments** — Images, markdown, and PDF files attached to issues, previewed in-app
- **Full-text search** — FTS5-powered search across titles, descriptions, and notes

### AI Integration
- **AI-native workflow** — Designed for AI coding assistants (Claude Code, Cursor, Codex, Gemini CLI). AI agents use the CLI to manage issues autonomously
- **Live activity tracking** — AI activity LED per project, events panel (⌘⇧A), focus AI session window (⌘⇧F)
- **Sound alerts** — Per-project notification sounds when AI agents interact with your issues
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

PaiR is designed to be driven by AI coding assistants (Claude Code, Cursor, Copilot, etc.). Projects can be initialized from the desktop app (just add a folder) or via `pair init` from the terminal. Either way, an `AGENTS.md` file is generated in `.pair/` with the full CLI reference — commands, flags, and workflows.

Agents can use the CLI to manage the full issue lifecycle:

```bash
pair list -s open              # Find available work
pair update <id> -s in_progress
# ... do the work ...
pair comments add <id> "Done: implemented the feature"
pair close <id>
```

See [`.pair/AGENTS.md`](.pair/AGENTS.md) for the complete reference.

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
