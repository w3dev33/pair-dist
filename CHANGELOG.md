# Changelog

## [0.4.0] — 2026-03-06

### Features
- **AI session tracking** — Associate AI session PID with projects via push notifications. Walk PPID chain to detect AI CLI (Claude, Cursor, Codex) and editor (Zed, VS Code)
- **Per-project activity LED** — Each project line shows a LED that blinks green on AI notifications (replaces folder icon)
- **Focus AI session window** — Click the focus button or press ⌘⇧F to switch to the editor window where AI is working (uses editor CLI for precise window targeting)
- **AI Events panel** — Renamed from "Claude Events" to "AI Events". Shows AI name badge (claude/cursor/codex) per event. Toggle with ⌘⇧A
- **Keyboard shortcuts** — ⌘⇧A (AI panel), ⌘⇧F (focus AI session) added to app menu
- **Blocked badge** — Visual badge for blocked issues in the issue list
- **README** — Added Features section, merged Background & Compatibility

### Improvements
- AI name (`ai_name`) and editor name (`editor`) fields added to PushEvent for richer notifications
- Focus window uses 3 strategies: editor CLI from notification → detect editor from PID tree → AppleScript fallback
- Session focus button uses neutral gray style (white when project selected)

## [0.3.4] — 2026-03-04

### Features
- **`pair attach` CLI command** — attach files to issues from the command line, with real-time attachment refresh in the app

### Fixes
- Fix toast notification contrast in light theme
- Rename release artifacts + disable unavailable downloads on website

## [0.3.3] — 2026-03-04

### Features
- **Claude Events: native hook integration** — `pair notify --hook <hook_type>` reads JSON from stdin, classifies events (significant vs verbose), and forwards to the app. Replaces the bash/jq bridge script.
- **`pair notify` CLI** — new `--hook` flag for Claude Code hooks, plus existing manual mode (`-t`/`-m`) preserved for backward compatibility
- Self-call detection: Bash commands starting with `pair notify` are automatically skipped to prevent echo loops

### Docs
- Mark `scripts/claude-hook.sh` as legacy (kept as reference)
- Update AGENTS.md with `--hook` mode documentation and simplified Claude Code config

## [0.3.2] — 2026-03-03

### Features
- **Print Markdown** — print button + `Cmd+P` in Markdown preview modal (uses native print dialog via Tauri WebView)
- **Custom relations CLI** — `pair relate <id1> <id2> --type <type>` and `pair unrelate` for non-blocking relations (relates-to, duplicates, supersedes, etc.)
- Custom relations exported/imported in JSONL (`relations` field) — survives sync round-trips

### Fixes
- Fix dynamic window title — add missing `core:window:allow-set-title` permission
- Fix print in Tauri WebView — rewrite from iframe (blocked) to permanent `@media print` styles
- Print page breaks: `break-inside: avoid` on code blocks, tables, headings

### Docs
- Rebrand all docs from Beads to PaiR
- Rewrite README for public distribution + sync automation
- Document `pair relate` / `pair unrelate` in AGENTS.md

## [0.3.1] — 2026-03-03

### Features
- New **Maintenance** section in project settings dialog (separate from Actions)
  - **Repair database** — re-sync SQLite with JSONL (auto-backup before repair)
  - **Fix CLI permissions** — restore +x on the pair binary (resolves symlink)
  - Re-initialize moved to Maintenance section (most destructive = last)
- Dynamic window title — shows "PaiR — ProjectName" when a project is selected
- Parse `.pair/config.yaml` for `issue-prefix` (was hardcoded to defaults)
- Toast notifications on all maintenance actions (success/error)

## [0.3.0] — 2026-03-02

### Features
- Per-project settings dialog — click the cog icon on any project in the sidebar
  - Project info: path, database status, issue count, legacy .beads/ detection
  - Actions: remove from list, re-initialize database, migrate from .beads/
  - All actions require confirmation before executing
  - Read-only settings display (prefix, git tracking) — editing in a future version

### Fixes
- Migration dialog: add "Start fresh" option + skip if .pair/ already exists
- Onboarding: folder select triggers migration + remove "switch without migrating"
- Onboarding: cancel migration no longer creates .pair/ + git tracking choice preserved
- Check-for-updates broken on all platforms

## [0.2.2] — 2026-03-02

### Fixes
- Fix sidecar binary not found on Linux — Tauri uses full target triple (`pair-x86_64-unknown-linux-gnu`) but lookup only checked `pair` and `pair-x86_64`

## [0.2.1] — 2026-03-02

### CLI
- New `pair dep tree <id>` — recursive dependency tree with cycle detection
- New `pair dep list <id>` — direct blocks/blocked_by with issue details
- Both commands support `--json` for structured output

### UI
- App icon now uses dark background (#1E1E2E) for better dock visibility
- Header logo aligned with left sidebar
- Search field now overrides status filters to show matching results

## [0.2.0] — 2026-03-01

### Branding
- New PaiR app icon (blue silhouette) for macOS dock, Windows taskbar, and all platforms
- New PaiR logo with name in app header (theme-aware: light/dark variants)
- New PaiR logo in About dialog (replaces plain icon)
- Chevron separator between logo and project name in header
- Added logo assets: light, dark, mono, and PSD sources

### Cleanup
- Removed legacy bd/br backend code
- Removed unused Rust functions (4 compiler warnings fixed)
- Removed legacy attachment refs migration code
- Removed unused backendMode watcher in index.vue

## [0.1.0] — 2026-02-28

First pre-release of PaiR — standalone issue tracker with built-in Rust engine.

### Core Engine
- SQLite-based issue tracker with full CRUD (create, update, close, delete)
- Schema v1→v5 with automatic migrations
- Full-text search (FTS5) across titles, bodies, and notes
- JSONL export/import for portable data exchange
- Git sync cycle: export → commit → pull → import → push
- Merge conflict detection and resolution

### Child Issues & Ordering
- Sequential child IDs: `parent.1`, `parent.2`, `parent.3`
- Auto-position on child creation
- `reorder_child()` with sequential re-numbering
- `list_children()` ordered by position (legacy fallback on created_at)

### Dependencies
- Block dependencies: `blocked_by` / `blocks`
- `list_ready_issues()` — open issues not blocked by any open issue
- `is_blocked()` — real-time blocking check
- JSONL format: `blocked_by` (replaces legacy `dependencies`)

### CLI (`pair`)
- `pair init/list/show/create/update/close/delete`
- `pair search` — full-text search
- `pair ready` — unblocked open issues
- `pair export` — force DB→JSONL re-export
- `pair import` — JSONL import with merge
- `pair reorder` — reorder child position
- `pair migrate` — unidirectional .beads → .pair migration
- `pair comments add/delete`, `pair label add/remove`, `pair dep add/remove`
- JSON output mode (`--json`)

### Push Notifications
- Unix socket (`/tmp/pair.sock`) for CLI→App instant notifications
- Event batching with selective toasts
- Auto-refresh on push events

### App Features
- Tauri 2 desktop app (macOS)
- Vue 3 + Nuxt 4 frontend with shadcn UI
- Dashboard with KPIs, priority/status charts
- Issue table with filters (status, type, priority, assignee, labels)
- Keyboard navigation, sidebar resize, dark/light theme
- Attachment system (filesystem-based)
- AGENTS.md auto-update on project open

### Testing
- 107 backend tests (Rust)
- 215 frontend tests (Vitest)
- Integration test suite (`scripts/test-suite.py`)

### Project
- Rebranded from Beads Task-Issue Tracker to PaiR
- CLI binary: `pair`, data folder: `.pair/`, prefix: `pair-xxxx`
- App identifier: `com.pair.app`
