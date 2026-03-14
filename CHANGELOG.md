# Changelog

## [0.9.1] — 2026-03-14

### Features
- **Pinned issues in DB** — Pin state moved from localStorage to SQLite, synced via git, visible from CLI and AI agents
- **CLI pin/unpin** — `pair pin <id>`, `pair unpin <id>`, `pair list --pinned`
- **Undo on unpin** — Toast with undo button + Cmd/Ctrl+Z keyboard shortcut (8s window)
- **Notification action buttons** — Toast notifications now support visible action buttons

### Improvements
- **Campaign badge color** — Changed from amber to pink to avoid confusion with P2 priority
- **README rewrite** — Features section restructured into themed subsections

### Cleanup
- **Removed `hooked` status** — Dead status type never used in production, fully removed from types, CSS, i18n, and tests
- **Removed `pinned` status** — Pin is now a boolean flag, not a status value
- **Removed pinned sort modes** — Simplified to updatedAt sort, removed drag-and-drop reorder
- **Updated in-app docs** — Dashboard, CLI reference, and AGENTS.md updated for pin/unpin

## [0.9.0] — 2026-03-13

### Features
- **Resizable comments panel** — Comments split into dedicated resizable panel with source badges
- **Unified zoom controls** — New ZoomControl component, compact header/footer, sound toggle moved to footer
- **Uniformized section buttons** — Consistent collapsible section UI across all table sections
- **Sync integration tests** — Full roundtrip sync test coverage

### Fixes
- **CLI sidecar lookup** — Look for sidecar in exe dir instead of resource dir (GitHub #1)
- **Linux Wayland** — Auto-disable WebKitGTK compositing on Wayland
- **Resync updates in-place** — No longer deletes and recreates issues on resync
- **Sync-repo config** — Fix sync-repo detection without explicit sync-provider
- **Reopen clears closed_at** — Reopening an issue properly clears the closed date

## [0.8.0] — 2026-03-12

### Features
- **GitHub issue sync** — One-way GitHub → PaiR issue synchronization with single-issue resync
- **GitLab provider** — GitLab issue sync support via API, auto-detected from git remote
- **Spec issue type** — Dedicated spec type with its own table section and `pair children` command
- **Comments fullscreen dialog** — Expand comments in a full dialog with search (Cmd+F) and print (Cmd+P)
- **AI session count** — Dashboard shows active AI session count per project
- **Specs dashboard section** — New Specs panel in the project dashboard

### Improvements
- **Issue title in all preview dialogs** — Image, Markdown, PDF, and Comments dialogs now show the issue title in the header
- **Shared search/print utilities** — Extracted duplicated DOM search and print code into reusable modules
- **Search highlight colors** — Blue-themed highlights with proper dark/light mode contrast
- **In-app help** — Documented single-issue resync and spec type

### Fixes
- **Sound alerts persistence** — Fixed sound alerts saving to the wrong project path
- **Dashboard AI column** — Hidden when no AI sessions detected

## [0.7.6] — 2026-03-11

### Fixes
- **App freeze on rapid project switching** — Prevent the app from freezing when switching between projects quickly

## [0.7.5] — 2026-03-11

### Features
- **Attachment downloads** — Download button added to image and PDF preview modals
- **PDF attachments** — Full PDF support with preview, migrated to Tauri asset protocol
- **In-progress badge** — Sidebar project list shows count of in-progress issues

### Fixes
- **Search regression** — Fixed crash when searching (useI18n called outside setup), also resets pagination on search

## [0.7.4] — 2026-03-11

### Features
- **Deferred section** — Issues with "deferred" status now appear in their own collapsible table section, placed after "Ready to Work"

## [0.7.3] — 2026-03-10

### Features
- **Bilingual interface (EN/FR)** — Full i18n support with language switcher, all UI labels and table section headers translated
- **Automated screenshot capture** — Screenshots generated for the website with theme and locale variants (dark/light x en/fr)
- **ConflictDialog visual testing** — Mock conflict dialog for design iteration

### Improvements
- **Website screenshots by locale** — Website now serves screenshots matching the visitor's language and theme
- **Screenshot deployment checks** — Deploy and verify scripts validate all 4 theme/locale combinations

### Fixes
- **Duplicate log entries** — Eliminated redundant log lines in the native logger
- **Table section headers** — Section headers now properly translated in all locales
- **Linux platform info** — Corrected Linux installation instructions in FAQ
- **Git sync safety** — Skip git operations when `.pair` is gitignored

## [0.7.2] — 2026-03-10

### Features
- **User-facing changelog page** — Bilingual (EN/FR) changelog page on the website at `/changelog`, with version badges and grouped features/fixes
- **Scroll spy navigation** — Header nav links highlight based on visible section when scrolling the homepage
- **Website-first approach** — Release notes and changelog now live on the website, no longer synced to GitHub pair-dist

### Improvements
- **Check for updates** — Now uses release body directly instead of fetching CHANGELOG.md from GitHub. "View on GitHub" replaced by "Visit website"
- **Centralized website URL** — `WWW_BASE_URL` from `.env` used as single source of truth across frontend (runtimeConfig) and scripts (changelog_utils.py)
- **Frontend constants** — Added `app/utils/constants.ts` for shared GitHub distribution repo references
- **Release notes generation** — New `scripts/changelog_utils.py` generates user-facing release bodies from `www/app/data/changelog.ts`
- **FAQ updated** — Added "Is PaiR open source?" and "Does PaiR work offline?" questions, reordered logically, clarified AI hooks support and Beads migration

### Cleanup
- Removed CHANGELOG.md from pair-dist sync (SYNC_FILES in sync-releases.py and publish-local.py)
- Removed "All releases available on GitHub" mention from download section
- Removed GitHub link from header navigation (kept in footer)

## [0.7.1] — 2026-03-10

### Features
- **Soft delete by default** — Deleting an issue now archives it (tombstone) instead of permanently removing it. Check "Delete permanently" in the confirmation dialog for hard delete. Dialog wording adapts contextually ("Archive" vs "Delete permanently").

### Documentation
- Updated in-app help docs (issue workflow, issue details) to reflect soft delete as default behavior

## [0.7.0] — 2026-03-09

### Features
- **Embedded help documentation** — 10-page help system accessible via `?` shortcut or Help menu, with table of contents navigation, cross-page search, and print support
- **Clickable AI toast notifications** — Click on an AI notification toast to focus the agent's session window (macOS only)

### Improvements
- **Help viewer UX** — Wider dialog, resizable TOC sidebar, scroll spy, arrow key navigation between pages and sections
- **Hidden internal statuses** — Internal statuses (pinned, hooked) no longer appear in filters and edit form

### Fixes
- **Mute icon zoom** — Mute icon no longer scales disproportionately when zooming the interface
- **Help viewer overflow** — Content no longer overflows on narrow windows
- **Website screenshot** — Website now uses the most recent screenshot by date instead of a version-based path that could break on patch releases

### Documentation
- Why PaiR? — Project philosophy and manifesto
- Getting Started, Dashboard, Issue Details, Issue Workflow
- AI Agent Hooks, Sync & Conflicts, CLI Reference
- Settings (global + per-project), Keyboard Shortcuts

## [0.6.1] — 2026-03-09

### Features
- **AI agent hooks settings** — Configure AI agent hooks (Claude Code, Cursor, Codex) directly from the global settings page

### Fixes
- **Notification sound fallback** — Fall back to global sound preset when a project has no notification config
- **Linux audio crash** — Prevent audio playback error from blocking toast notifications on Linux

## [0.6.0] — 2026-03-09

### Features
- **AI sound notifications** — Per-project sound alerts when AI agents need attention, with 5 sound presets (Bell, Double beep, Rising, Soft chime, Alert), per-project volume override, and global mute toggle in header

### Improvements
- **Centralized localStorage** — All keys managed through `storage-keys.ts` with active registries and automatic cleanup of stale keys
- **Non-destructive storage migration** — `beads:` → `pair:` key migration at boot, old keys preserved for backward compatibility
- **Path normalization** — `hashPath()` now strips trailing slashes, fixing per-project settings mismatches
- **Legacy rename** — `useBeadsPath` → `useCliPath` across the entire codebase
- Cargo cache added to all CI builds (Windows, macOS, Linux)

### Tests
- Added 21 tests for hash normalization, storage key registry, and migration logic

## [0.5.0] — 2026-03-07

### Features
- **Collapsible sections** — Issue list organized into In Progress, Pinned, and Current sections with collapsible headers
- **AI panel: unknown project/agent detection** — Flag events from unrecognized projects or AI agents

### Improvements
- Compact header layout and refined table typography
- Harmonized primary color and simplified neon default theme
- Epic borders restored with child in_progress group promotion
- Centered KPI card content, neutral label badges
- Backend split into command modules for maintainability
- Frontend refactoring: extracted reusable utils, composables, and badge configs
- Linux build script now produces both amd64 and arm64

### Tests
- Added tests for partition helpers, toggle/enrich utilities, and short ID generation

## [0.4.4] — 2026-03-07

### Features
- **Project pinning** — Pin/unpin projects in sidebar, toggle "pinned only" filter to focus on starred projects. Drag-and-drop and sort hidden when filtered
- **AI panel: brand icons** — Per-event SVG icons for Claude, Cursor, Codex, Gemini, Vibe (+ generic fallback)
- **AI panel: project name** — Display project name (last path segment) after AI icon
- **AI panel: Copy button** — Copy filtered events to clipboard as text
- **AI panel: cost & tokens on Stop** — Show total cost and token counts when AI session ends
- **AI panel: tool error detection** — Highlight failed tool calls in red via `tool_error` event type
- **AI panel: tool_use_id via meta** — Pass tool_use_id on all tool events for future duration tracking
- **SubagentStop hook** — Capture agent completion events (`agent_stop` in AI panel)
- **PreCompact hook** — Capture context compaction events (`compact` in AI panel)
- **`--ai-name` flag** — `pair notify --ai-name claude` for testing AI icons without actual AI process

### Improvements
- Throttle activity LED re-renders (200ms) to reduce DOM updates
- Increase AI panel MAX_EVENTS from 200 to 500
- Dual-theme colors for all event types (light + dark mode)
- Zoom support for AI and Debug panels (`.zoomable-panel`)
- Soften verbose event opacity (60% → 80%)
- Expand AI detection: add Gemini and Vibe to PPID chain walker

### Docs
- Add disclaimer section to README
- Clarify non-destructive migration in README
- Add Features section and improve hero layout on website

## [0.4.3] — 2026-03-06

### Features
- Add `pair --version` / `-V` to CLI (clap version flag)

### Fixes
- Fix CLI binary missing execute permission in Linux `.deb` package
- Fix log file path for Linux (`~/.local/share/com.pair.app/logs/pair.log`)
- Rename log file from `beads.log` to `pair.log`
- Fix push socket stale detection (non-blocking connect check)
- Set socket permissions to world-writable so CLI can always connect
- Auto-fix CLI permissions on app startup (Linux `.deb` workaround)

### Docs
- README: add "Why PaiR?" section, mark AI features as experimental (macOS only), update platform availability (Windows coming soon)
- Claude Code hooks: recommend absolute path to `pair` binary on all platforms
- Add Linux-specific note to AGENTS.md template

## [0.4.2] — 2026-03-06

### Docs
- Add `notify` command and Claude Code hooks configuration to AGENTS.md template

## [0.4.1] — 2026-03-06

### Fixes
- Fix stale data when external process (CLI script) modifies the database — invalidate cached SQLite connection on mtime change
- Add `relate`/`unrelate` commands to AGENTS.md template

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
