![PaiR](assets/logo-dark.png)

A lightweight desktop application for managing tasks and issues directly inside your codebase.

PaiR ships with its own CLI (`pair`) — no external tool to install. The CLI manages issues across multiple independent projects, and the application aggregates them all in a single window, updated in real time. The CLI does the heavy lifting — AI agents use it to create, update, and close issues programmatically — while the app gives you a unified view and control over everything. Issues live in your project directory, tracked by git, and visible to both humans and AI agents.

## Why PaiR?

AI-assisted development is moving fast. Multi-agent orchestration, autonomous task management, swarms of AI workers — the tools are impressive, but they're racing ahead of most developers' reality.

We believe the transition to AI autonomy should be progressive. Today, most developers work **with** AI — pair programming, reviewing suggestions, steering decisions. They need to see what's happening, understand it, and stay in control. Jumping straight to full autonomy means losing the ability to learn, verify, and course-correct.

PaiR is built for this transition. Start with pair programming — one human, one AI, full visibility. As trust builds, delegate more. But at every step, you can see what's going on, step in, and redirect. It's not just a workflow — it's a learning process.

The name says it all: **PaiR** — the human and the AI, working side by side.

## Installation

Download PaiR for macOS and Linux: **[pair.w3dev.fr](https://pair.w3dev.fr)**

> Windows support is coming soon — we're currently limited by CI build quotas but it's on the roadmap.

The `pair` CLI is bundled with the application and a symlink is created automatically during installation, making it available from any terminal.

## Features

- **Multi-project dashboard** — Aggregate issues from all your projects in a single window with real-time updates
- **Built-in CLI** — `pair` ships with the app. Create, update, close issues, manage dependencies, labels, and comments from the terminal
- **AI-native** — Designed for AI coding assistants (Claude Code, Cursor, Codex). AI activity LED per project, AI events panel (⌘⇧A), focus AI session window (⌘⇧F) — *experimental, macOS only for now*
- **Real-time push notifications** — CLI mutations and AI events are pushed instantly to the app via Unix socket
- **Git-synced issues** — Issues live in `.pair/` inside your repo, tracked by git, visible to humans and AI agents alike
- **Dependencies & blocking** — Link issues with blocks/blocked-by relationships, visualize blockers
- **Epics & child issues** — Organize work with parent/child hierarchy, drag-and-drop reordering
- **Attachments** — Attach images and markdown files to issues, previewed directly in the app
- **Full-text search** — FTS5-powered search across titles, descriptions, and notes
- **Dark & light themes** — Automatic theme detection with manual override
- **Cross-platform** — macOS, Linux (Windows coming soon)

## Background & Compatibility

PaiR is inspired by [Beads](https://github.com/steveyegge/beads), the AI-native issue tracker created by Steve Yegge, which stores issues directly in the codebase using a SQLite + JSONL structure. Our first take was [Beads Task-Issue Tracker](https://github.com/w3dev33/beads-task-issue-tracker), a desktop app built as a frontend for the existing Beads CLIs (`bd`, `br`). But as those CLIs evolved in diverging directions — Dolt migration, server mode, breaking changes — depending on external tools became a liability. PaiR was built from scratch with its own CLI, its own schema, and its own features to move at its own pace.

For existing Beads projects, the app handles migration automatically:

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

## License

[MIT](LICENSE) — Laurent Chapin
