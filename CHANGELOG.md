# Changelog

## [0.36.0] - 2026-08-19

### Features
- **Connection rails in the Sessions list** - the sidebar SESSIONS section now draws a colored rail between cabled sessions, so you see which sessions are wired together without opening the graph. Rails are grouped and colored per origin (the same stable color as the graph's cables, shared via `originColorMap`), the initiator end is a dot and the receiver end an arrow, and a rail animates green while a message travels the cable.

### Fixes
- **Cables no longer wiped on a window refresh** - the cable reconcile ran in the graph's `onMounted` before the terminal sessions were restored, so a refresh saw an empty live set and dropped every cable from the catalog. `reconcile` now bails on an empty live set and runs once from `index.vue` after the restore completes, with the true full live set.
- **Graph edges stay anchored at any zoom** - the graph container is kept out of the app zoom factor (counter-zoom), so cable/binding edges anchor on their session handles instead of drifting when the app zoom isn't 100%. Cable edges also route around vertically-stacked projects with a distinct tower curve instead of crossing the node.
- **Projects and Favorites collapse independently** - the Projects section reused the Favorites collapse key, so folding one folded the other in both directions. Projects now has its own `projectsCollapsed` key.
- **`pair cable list --json` exposes the cable initiator** - the CLI JSON omitted `initiator` (and the project paths / `createdAt`) while the catalog and the app already had them; the output now matches the full cable shape.

## [0.35.0] - 2026-08-14

### Features
- **Unified agent attention across list, graph and terminal** - a session waiting for you now surfaces the same attention signal everywhere: the session list, the orchestration graph node, and the terminal tab share one state instead of each computing its own.

### Fixes
- **`pair` CLI now resolves in the terminal on Windows** - after install, the `pair` command is found both in PaiR's embedded terminal and in external shells. The sidecar was silently found only by the app itself (it sits next to the app binary, which Windows searches first), so the boot repair skipped copying the CLI to the PATH location terminals actually look in.
- **Clearer graph readability** - improved node and text contrast in the graph view, and session nodes now visually distinguish their inserts from their outgoing links.

## [0.34.0] - 2026-08-13

### Features
- **Auto-open a session on an empty project** - `pair journal --push --to <project> --auto-open` creates a session in a project that has none, launches Claude and delivers the message as its opening prompt. The session is created through PaiR (not a raw tmux session) so it registers and becomes routable, and a reply target is stashed so the new agent's answer returns to the opener. Explicit flag on purpose: starting an agent is never a silent side effect of a plain push.
- **`pair whoami`** - print the current session identity (id, name, project, runtime) in one call, with `--json` for scripts. Resolves even when the shell exposes neither `PAIR_TERMINAL` nor `$TMUX` (an agent's Bash tool) by matching the process ancestors against tmux pane pids.
- **Remote-session cabling parity** - cable to LAN peer sessions with the same drag gesture as local ones. A cable to a peer that is offline stays visible in a degraded state (dimmed container, grey antenna, dashed edge) instead of disappearing, and becomes live again when the peer returns.
- **Constrained-height Sessions panel** - the cross-project Sessions list gets the same constrained-height + expand toggle as the other sidebar lists (Projects, Favorites, Recent).

### Fixes
- **Session-to-session replies target the exact sender** - the delivered message is labelled with its emitter session (`[<session> · <type>]`), and a bare `pair journal --push` reply returns only to the session that messaged you, never fanning out to the other sessions of its project.
- **Reliable Enter on injected messages** - the submit keystroke (`\r`) is now sent as a distinct event after a short delay across all injection paths (tmux, PTY, peer), so a message pushed to another session is validated instead of left sitting unsubmitted in a paste-aware TUI.
- **Perceptible round-trip animation** - a reply arriving right after the outbound message now animates back distinctly on the cable, each direction held a minimum time instead of an imperceptible mid-loop flip.
- **Favorites section keeps its collapsed state** across relaunches - its collapse key was per-project while registered global, so it was wiped on every boot and reopened each launch.

## [0.33.0] - 2026-08-11

### Features
- **Session-to-session cables** - A cable wires two local agent sessions together, across projects, and message routing (`pair journal --push --to`) follows the cables instead of project associations. In the graph a cable is a single edge from the initiator to its destination (blue at rest), and each delivered message animates it green in the direction it travelled (forward from the initiator, reversed for the answer). Cabled on the fly when you name a recipient that isn't wired yet; drag two sessions together in the graph or use `pair cable add/rm/list`.
- **herdr runtime** - Per-session runtime choice via `RuntimeMode` (pty | tmux | herdr). herdr is a single global multiplexed session shown as one tab in every project, restored at boot, with hook identity resolution and inbound/outbound semantic bridges so PaiR observes the agents running in it.
- **Workspace display pins** - A second layer over workspace membership: pin which workspace sessions show in the split view (`workspaceDisplayPins`), so a large multi-project workspace stays manageable. The tab pin is mode-aware (project vs workspace); the graph shows no display pin.
- **Resizable split panes + maximize mode** - Per-project and per-workspace pane widths, and a maximized terminal mode.
- **Cross-project Sessions panel + footer KPIs** - New Sessions section in the left panel across projects; KPI counters moved to the footer.
- **AI events filtering** - Filter the AI panel by session with a per-tab badge.
- **Graph grid snapping** - Nodes snap to a 16px grid on drag (`snap-to-grid`).
- **Journal merged view follows the workspace** - The merged journal view now follows the projects present in the workspace instead of catalog associations.

### Changes
- **Project associations removed** - The project-to-project association concept (UI, `catalog.rs` API, `pair associate/dissociate/associations` CLI, docs) is retired; its only functional role (message routing) is served by cables. The hook context injection (`inject_journal_context`) now reads the journals of cabled projects.
- **`PAIR_LINKED` / session-linking removed** - Dead since delivery moved to cables; the whole linked-session mechanism (env sync, `.pair/.linked-sessions`, link buttons) is gone. Workspace pinning is a pure display choice.

## [0.32.0] - 2026-07-09

### Features
- **LAN peer collaboration** - Pair PaiR with teammates on the same local network and see who is working on what in real time. Discovery works across macOS, Linux and Windows, including Docker, OrbStack and VPN setups.
- **Peer messaging** - Chat with all paired peers at once (broadcast) or in dedicated 1-to-1 conversations. Send a message, a snippet or a command straight into a teammate's terminal session, with manual or automatic accept.
- **Session linking** - Connect your terminal sessions to your peers' through a visual graph; links replay automatically on reconnect.
- **Live session states** - The panel shows each session's real state (running, waiting, blocked), with a notification sound when a session is waiting for you.
- **Inline images in markdown previews** - Embedded images now render directly in the preview, with per-image zoom and pan.
- **External blockers** - Mark an issue as blocked by an external reference (e.g. Redmine) and follow the link directly.
- **Settings redesign** - Configuration dialogs are grouped by domain and load on demand.
- **`pair update --branch`** - Reassign an issue's branch from the CLI without recreating it.

### Fixes
- **Windows** - Fixed hundreds of console windows opening on project init.
- **Linux** - Fixed accented character input in the built-in terminal.
- **Workspace terminal** - Associated projects group correctly, including names with underscores.
- **Migration & network** - More robust worktree cleanup, reconnection and state convergence.

## [0.26.0] - 2026-05-04

### Features
- **Tracker branch configurable per project** - Branch name is no longer hardcoded to `pair-tracker`; users can pick a custom name at init or rename it later from Settings (with assisted rename across worktree + remote)
- **Tracker user combobox in Settings** - User combobox with auto-resolution of git author / `.pair` journal authors
- **Auto onboarding for new contributors** - First-time contributors on a migrated project get the tracker branch + worktree set up automatically without manual steps

### Fixes
- **Cascade close guard** - Closing an issue with unfinished children now prompts before cascading; prevents accidental mass-close of an epic's open children (pair-a5og)

## [0.25.0] - 2026-04-29

### Features
- **Tracker on a dedicated branch** - `.pair/` migrated from per-branch versioning to a worktree of an orphan `pair-tracker` branch. Tickets persist across `git checkout` and no longer appear in code-branch `git status`. One-click in-app migration with backup tag for rollback. Cross-branch divergence detection merges tickets scattered across legacy branches into the orphan during migration. Auto-trigger dialog at project switch when legacy state is detected
- **Branch filter & column** - New `branch` column in IssueTable (visible by default, after favorite). New BranchFilterDropdown with 3-tier layout: "Current branch [HEAD]" pseudo-entry mapping to `@head` sentinel that follows `git checkout` automatically, alphabetic by-name pins. Backend `.git/HEAD` watcher emits `git-head-changed` for live UI sync
- **"Track issues in git" toggle at init** - Opt-out of git versioning for tickets; the toggle conditions whether `pair-tracker` is created at all
- **Workflow rule update** - On migrated projects, the legacy "two separate commits" rule (code + `.pair/`) becomes "two separate pushes": `git push` then `git -C .pair push`. Auto-deployed to every PaiR-managed project via `.pair/AGENTS.md`

### Fixes
- **Dashboard Last Edit** - Consistent cross-project ordering
- **Orchestration** - Internal session flash propagates correctly to SessionNode
- **Update Download & Quit** - Uses `destroy()` to bypass the quit-confirmation dialog
- **Tracker migration** - Multiple hardenings: SQLite cache rebuilt from JSONL post-migration, tombstones excluded from issue count, untracked files no longer block working-tree-clean check, Engine cache invalidated after migration/rollback, porcelain leading byte preserved, `.pair/.gitignore` self-healed at every Engine open
- **Migration on fresh repo** - `git symbolic-ref --short HEAD` resolves the branch even on a 0-commit repo; backup tag silently skipped when there's nothing to roll back to
- **Filters legacy schema** - `useFilters` now re-normalizes `FilterState` on every project switch, fixing a silent throw when projects had a stored `filters` predating later fields (e.g. `branch === undefined`)

## [0.24.0] - 2026-04-21

### Features
- **Paused status** - New `paused` status in the issue workflow for work temporarily set aside
- **Search field highlight** - Populated search inputs now stand out with a neon yellow style
- **Table scrollToIssue** - Programmatic navigation to any issue, with automatic section expand, page pagination, and parent epic unfold
- **Workspace orchestration coverage** - Workspace mode graph now includes projects associated with workspace-pinned projects, even when those associates have no pinned session (bidirectional)
- **Release scripts** - Auto-detect `CLAUDECODE` to trim verbose output during release workflows

### Fixes
- **Search reliability** - `external_ref` is now indexed in FTS5; silent search errors are surfaced
- **Preview sidebar** - Sidebar stays open when navigating to a closed child issue
- **Terminal detach** - Cross-window sync for active tab, workspace mode, and current project between main app and detached terminal panels. Single-location invariant: the internal section hides whenever any terminal is detached, and clicking the internal header re-attaches unconditionally. AI notifications and focus actions target the detached window when present. Detached header now surfaces tmux/Claude status badges

## [0.23.0] - 2026-04-12

### Features
- **Orchestration view** - New graphical canvas (VueFlow) showing projects and their terminal sessions as interactive nodes with puzzle-piece connectors for cross-session communication
- **Bidirectional session sync** - Clicking a session in the graph focuses the terminal tab (and switches project); clicking a terminal tab highlights the session in the graph. Powered by a new `useActiveSession` global store
- **Workspace mode in graph** - When workspace is active, the graph shows all projects with workspace-pinned sessions alongside the current project and associations
- **Cross-session journal push** - Linked tmux sessions can broadcast journal entries to each other, including PTY sessions. Link/unlink events are notified
- **Cross-project favorites** - Star issues from any project; favorites are stored globally in the catalog and displayed in the dashboard
- **Auto-fit graph** - ResizeObserver detects container changes (terminal open/close, sidebar resize) and auto-fits the viewport. Toggle button in bottom-right corner
- **Issue title in graph** - Sessions launched via Play show the issue title below the session name
- **In-app documentation** - New "Orchestration View" section in terminal docs (EN + FR)

### Fixes
- **tmux issue ID persistence** - `PAIR_ISSUE_ID` was never written to tmux env for root issue IDs. Sessions now survive app restarts with their issue association intact
- **Session click cross-project** - Clicking a session in a non-active project now correctly switches both the project and the terminal tab
- **VueFlow fitView timing** - Replaced `nextTick` with `setTimeout(150ms)` for fitView after node changes
- **Handle border masking** - Added missing masking rects on source handles that caused border bleed
- **Edge z-index** - Edges now render above project nodes instead of being hidden behind backgrounds
- **Session badge count** - Corrected session badge count and cleanup logic

## [0.22.1] - 2026-04-10

### Fixes
- **Read-only directory handling** - FolderPicker now detects write permissions and adapts the UI: on read-only directories, clicking "Init as PaiR" opens a terminal with `pair init` pre-filled instead of failing silently
- **Real error messages** - Tauri invoke errors now display the actual error message instead of a generic "Failed to initialize tracker"
- **Catalog registration on init** - `tracker_init` (app) now calls `catalog::auto_register` like the CLI does, ensuring the project appears in the global catalog
- **CLI path resolution** - Fixed CLI subprocess path resolution in production app (sidecar + extended PATH)

## [0.22.0] - 2026-04-07

### Features
- **Video demo on website** - Interactive demo player on the landing page: play/pause/stop overlay on the screenshot, theme-aware (dark/light), auto-returns to image on end or theme/locale change
- **CLI as single source of truth** - All mutations (status changes, attachments, issue creation) now route through the CLI, ensuring consistent journal tracking and push notifications
- **Enriched journal** - Journal now tracks `status_changed`, `attachment_added`, and `attachment_removed` events in addition to issue creation/closing
- **Pagination indicator** - Dashboard shows a visible count of displayed vs total issues above the table, with critical issues always visible regardless of pagination

### Fixes
- **Dashboard refresh on CLI mutations** - Status changes and attachments from CLI/agents now trigger immediate dashboard refresh
- **Screenshot automation** - Tmux sessions are cleaned before capture to prevent stale session badges

### Automation
- **Video capture system** - New `capture-video.py` script with scenario-based recording, journal polling for timing, variable interpolation (`--var theme=light`), pre-layout setup, and automatic speed-up (`--speed 2`)
- **Remote control commands** - Added `terminal-type`, `terminal-close`, `terminal-switch`, `journal-open/close`, `pomodoro-open/start/reset`, `toast-dismiss-all` for screenshot and video automation

## [0.21.5] - 2026-04-06

### Features
- **Last Edit section** - New collapsible section showing the 10 most recently modified issues. Populated automatically from journal entries and live push events (create, close, update, comment)

### Fixes
- **Dashboard push event tracking** - Issue create/update events from CLI and agents now correctly trigger a poll refresh and appear in Last Edit
- **Cross-platform paths** - Replaced hardcoded `$HOME` paths with `dirs::` crate for proper cross-platform support (Windows, Linux, macOS)
- **Terminal auto-start default** - Auto-start is now disabled by default for new users, preventing unexpected Claude launches
- **Terminal multi-tab race condition** - Replaced singleton callbacks (shellReadyResolve, claudeReadyResolve, pendingCommand) with per-session Maps, fixing race conditions when launching multiple terminal tabs rapidly

## [0.21.3] - 2026-04-02

### Features
- **Terminal session naming** - Sessions now use the project folder name as prefix (e.g., `MyProject 1`, `MyProject zsju.2`). Visible in both PaiR tabs and `tmux ls`, making it easy to identify which project a session belongs to - especially useful in workspace mode
- **Hooks auto-journal** - Git commits and session stops are automatically logged to the project journal via Claude Code hooks
- **Journal context injection** - Journal entries are auto-injected as context on Edit/Write tool calls via PreToolUse hooks
- **Journal auto-refresh** - Journal panel refreshes automatically on push events from external agents

### Fixes
- **First keypress lost in tmux** - Removed a stray Escape character sent during tmux initialization that was consumed by the shell, causing the first keystroke to be silently dropped
- **Isolated tmux socket** - PaiR sessions now run on a dedicated tmux socket (`pair`), preventing interference with user's personal tmux sessions
- **Terminal section auto-open** - Terminal panel no longer opens unexpectedly on hot-reload
- **Comments panel visibility** - Comments panel stays visible and resizes correctly in split screen
- **Hook deduplication** - Removed session_stop noise and deduplicated auto_commit journal entries

### Performance
- **Debounced resize** - Terminal resize events are debounced, keydown handlers consolidated, and deep watchers replaced with targeted ones

## [0.20.0] - 2026-03-29

### Features
- **Activity journal** - Every project gets a live activity feed. Issues created, changes made, decisions taken - logged automatically and manually via `pair journal`. Collapsible panel in the footer with scroll-to-bottom, project name header, and auto-export to JSONL
- **Project catalog & associations** - Global project catalog stored in Application Support. Link related projects together via `pair associate` or the app settings. Bidirectional associations with UUID-based catalog
- **Cross-project orchestration** - AI agents automatically read associated projects' journals at session start. Reply-to protocol enables asynchronous agent-to-agent communication across codebases
- **Workspace mode** - Pin terminal sessions from multiple projects side by side. Workspace toggle in terminal header, smart "+" button to attach existing sessions, chain link indicator between associated project tabs
- **Workspace journal view** - Merged chronological view of journal entries from all associated projects in the journal panel
- **Detachable terminal window** - Pop out terminal sessions into independent windows (PTY mode). Per-project geometry persistence, dynamic window title, and proper session sync
- **Epics section** - Dedicated Epics section in the issue table with automatic in-progress status when children are active
- **Full-text search on comments & labels** - FTS5 index now covers comments and labels, not just titles and descriptions
- **Terminal session count badge** - Sidebar shows the number of active terminal sessions per project
- **Storage adapter abstraction** - New storage layer for future migration to centralized catalog-based settings

### Fixes
- **Journal panel** - Entries now display in chronological order (oldest first) with UTC-to-local time conversion
- **Terminal tmux rendering** - Resize-bounce workaround for reliable rendering on project switch
- **Log timestamps** - Application logs now use local timezone instead of UTC
- **Search on poll** - FTS search re-runs on poll instead of silently overwriting results
- **Detached window focus** - AI notifications focus the detached window instead of reopening internally
- **Catalog self-association** - Prevented linking a project to itself

## [0.12.2] - 2026-03-23

### Fixes
- **App freeze on project switch** - Column config watcher (`deep: true`) triggered an infinite reactive loop when switching projects, causing `Maximum recursive updates exceeded`. Replaced with a stable ID-based watcher
- **Thread starvation on startup** - `TRACKER_ENGINES` global mutex blocked all concurrent project access. Replaced with per-project `Arc<Mutex<Engine>>` locking so different projects don't block each other
- **Startup overload** - `initialScan` fired `cliList` for all projects in parallel via `Promise.allSettled`, saturating the Tauri thread pool. Serialized to sequential calls
- **Migration dialog crash** - `useIssues()` was called inside event handlers (`switchWithoutMigration`, `migrate`) instead of at setup level, causing `useI18n` to throw
- **Filter watcher race** - `reloadProjectStorage` updated filters during project switch, triggering a redundant `fetchIssues` call that raced with `doPathChange`. Added `isSwitchingProject` guard

## [0.12.1] - 2026-03-23

### Fixes
- **Play button missing** - Run column now re-syncs when switching projects (no more Cmd+R needed)
- **Shift+Enter in terminal** - Works in both PTY and tmux modes (kitty keyboard protocol + tmux send-keys bypass)
- **Claude Code icons** - MesloLGS NF font loaded for Nerd Font glyphs (no more red rectangles)
- **Accents in tmux** - UTF-8 locale forced on all terminal sessions (LANG/LC_ALL + tmux -u flag)

## [0.12.0] - 2026-03-23

### Features
- **Pomodoro timer** - Built-in timer in the footer bar (heart icon) with work/break cycles, persistent notifications with contextual actions, and a popover with play/pause/stop/skip controls
- **"Start with app" setting** - Optionally auto-launch a Pomodoro cycle when the app starts
- **State persistence** - Timer state survives app refreshes and restarts, resuming where it left off
- **Healthy settings** - New collapsible "Healthy" section in Settings with configurable work/break durations, cycle count, auto-start, and notification sound
- **Website "Takes care of you"** - New feature card combining sound alerts and Pomodoro, hero section mentions developer well-being

### Fixes
- **Vue warnings** - Fixed missing `isDev` in SettingsDialog and missing `rendered` emit in IssueTable
- **Silent notifications** - Added `silent` option to `useNotification` to prevent double sounds when the caller plays its own

### UI
- **Settings uniformization** - All settings sections (Language, Theme, Sound, AI Agent, Terminal, Healthy, Onboarding) now have consistent bordered containers

## [0.11.2] - 2026-03-23

### Fixes
- **"Visit Website" button** - Clicking the button in the Check for Updates dialog caused an unhandled error on all platforms. The shell plugin was missing the `open` scope configuration in `tauri.conf.json`.
- **tmux terminal display** - CLI tools (e.g. Claude Code) rendered degraded ASCII logos inside tmux sessions. Now forces `TERM=xterm-256color` inside tmux for consistent Unicode rendering.

## [0.11.1] - 2026-03-23

### Fixes
- **tmux 3.6+ compatibility** - Terminal tmux mode failed with immediate EOF on systems with tmux 3.6+. The PTY attach command was missing the `TERM` environment variable, causing tmux to refuse the connection.

## [0.11.0] - 2026-03-22

### Features
- **tmux mode** - Persistent terminal sessions that survive app restarts and can be accessed from external terminals (Zed, iTerm, VS Code) via `tmux attach`
- **Issue-named sessions** - tmux sessions are automatically named after the issue ID (e.g., `pair-xein-16` in `tmux ls`)
- **Smart session close** - Detects when a tmux session is attached in another terminal and warns before killing. Switch defaults to "detach only" when session is used elsewhere
- **Auto-close on issue close** - Closing an issue automatically closes its associated terminal session, with persistent toast notification for poll-detected closes
- **Natural language Run prompt** - "Work on bug pair-xein.23: Fix login" instead of generic commands
- **Mode switch confirmation** - Changing terminal mode (PTY ↔ tmux) shows a confirmation when sessions are open
- **Kill switch on tab close** - Toggle to also terminate the tmux session when closing a tab (enabled by default)

### Fixes
- **tmux path resolution** - Apps launched from Finder (not terminal) now correctly find the tmux binary
- **tmux session restore** - Fixed input routing, resize, and status bar on restored sessions
- **tmux prompt detection** - Fixed latency in Claude prompt detection and stale state
- **Split border** - Hidden border in single-tab mode
- **tmux prefix key** - Disabled tmux prefix to avoid conflicts with PaiR shortcuts
- **Active tab persistence** - Active tab is correctly persisted when closing an issue session
- **Windows PTY PATH** - CLI `pair` is now in the PTY shell PATH on Windows
- **Parent validation** - Creating a child issue on a non-existent parent is now rejected

### Docs
- tmux mode documented in in-app help (EN/FR)
- tmux mode mentioned in website features and FAQ
- Issue comments traçability guidelines added to CLAUDE.md

## [0.10.2] - 2026-03-21

### Fixes
- **Windows CLI PATH** - CLI install directory is now automatically added to user PATH on Windows
- **Windows/Linux update dialog** - "Download & Quit" button only shown on macOS; Windows/Linux show "Visit website" instead
- **Linux PATH fix message** - Onboarding PATH instructions now shown on Linux too (was macOS-only)
- **Cross-platform PATH resolution** - `get_extended_path()` now uses correct separator and paths on Windows

## [0.10.1] - 2026-03-21

### Features
- **Windows terminal support** - Terminal now detects and uses PowerShell Core, Windows PowerShell, or cmd.exe automatically
- **Review onboarding** - New button in Settings to review the getting started steps at any time

### Fixes
- **Onboarding scroll on small screens** - Onboarding panel now scrolls properly on low-resolution displays (Windows/small screens)
- **CLI symlink auto-repair** - App automatically repairs broken CLI symlink at startup
- **French locale** - Replaced "issues" with "tickets" throughout French locale for consistency

### Docs
- Windows unsigned app notice on website download page (SmartScreen instructions)
- Updated in-app docs: hooks-setup, sync-conflicts, terminal - clarify Unix socket vs Windows polling
- Updated terminal shortcut to mention Linux/Windows

## [0.10.0] - 2026-03-20

### Features
- **Integrated multi-session terminals** - Run multiple terminal sessions per project with tabbed interface. Launch Claude Code directly from issues, monitor AI agents in real time with visual and sound notifications when they need attention
- **Terminal split view** - Pin tabs to display terminals side by side. Built-in multiplexer experience with per-project persistence
- **AI notification routing** - Internal sessions flash and turn red, external sessions (Zed, Cursor, VS Code) show toast + sound only
- **AI Text Transform** - Transform text in any input field using AI. Smart interpretation: reformulate, translate, summarize. Works with any Claude subscription

### Fixes
- **Markdown zoom** - Zoom now scales all elements (headings, code, tables), not just normal text
- **Dashboard charts** - Charts expanded by default on first launch

### Docs
- New "AI Text Transform" dedicated help page (EN/FR)
- Updated terminal docs (split view, pins, notifications, shortcuts) - EN/FR
- Added terminal section to keyboard shortcuts doc

## [0.9.3] - 2026-03-17

### Features
- **Issue focus borders** - Active issue row highlighted with a visible focus border for better accessibility
- **Project path navigation** - PathSelector now supports keyboard navigation and direct project switching
- **Keyboard shortcuts docs** - Added keyboard shortcuts documentation page (EN/FR)

### Fixes
- **Attachment list refresh** - Attachment list in preview now refreshes correctly on add/delete

### Dependencies
- Updated 15 dependencies (Nuxt, Vue, Tailwind, Vitest, Reka UI, etc.)

## [0.9.2] - 2026-03-16

### Features
- **Panel keyboard shortcuts** - Navigate between panels with `1`/`2`/`3` (+ Numpad, F13/F14/F15), `Escape` closes details preview, `Cmd/Ctrl+R` refreshes issues
- **Mobile auto-focus** - Switching tabs in mobile mode auto-focuses the container for immediate arrow key navigation
- **Focused panel indicator** - Subtle border-top highlight shows which panel is active on desktop
- **Markdown preview refresh** - Refresh button and external change detection for attached markdown files
- **Blocked icon** - Replaced blocked status icon with LockKeyhole padlock for better clarity

### Cleanup
- Internal code cleanup and legacy naming removal

## [0.9.1] - 2026-03-14

### Features
- **Pinned issues in DB** - Pin state moved from localStorage to SQLite, synced via git, visible from CLI and AI agents
- **CLI pin/unpin** - `pair pin <id>`, `pair unpin <id>`, `pair list --pinned`
- **Undo on unpin** - Toast with undo button + Cmd/Ctrl+Z keyboard shortcut (8s window)
- **Notification action buttons** - Toast notifications now support visible action buttons

### Improvements
- **Campaign badge color** - Changed from amber to pink to avoid confusion with P2 priority
- **README rewrite** - Features section restructured into themed subsections

### Cleanup
- **Removed `hooked` status** - Dead status type never used in production, fully removed from types, CSS, i18n, and tests
- **Removed `pinned` status** - Pin is now a boolean flag, not a status value
- **Removed pinned sort modes** - Simplified to updatedAt sort, removed drag-and-drop reorder
- **Updated in-app docs** - Dashboard, CLI reference, and AGENTS.md updated for pin/unpin

## [0.9.0] - 2026-03-13

### Features
- **Resizable comments panel** - Comments split into dedicated resizable panel with source badges
- **Unified zoom controls** - New ZoomControl component, compact header/footer, sound toggle moved to footer
- **Uniformized section buttons** - Consistent collapsible section UI across all table sections
- **Sync integration tests** - Full roundtrip sync test coverage

### Fixes
- **CLI sidecar lookup** - Look for sidecar in exe dir instead of resource dir (GitHub #1)
- **Linux Wayland** - Auto-disable WebKitGTK compositing on Wayland
- **Resync updates in-place** - No longer deletes and recreates issues on resync
- **Sync-repo config** - Fix sync-repo detection without explicit sync-provider
- **Reopen clears closed_at** - Reopening an issue properly clears the closed date

## [0.8.0] - 2026-03-12

### Features
- **GitHub issue sync** - One-way GitHub → PaiR issue synchronization with single-issue resync
- **GitLab provider** - GitLab issue sync support via API, auto-detected from git remote
- **Spec issue type** - Dedicated spec type with its own table section and `pair children` command
- **Comments fullscreen dialog** - Expand comments in a full dialog with search (Cmd+F) and print (Cmd+P)
- **AI session count** - Dashboard shows active AI session count per project
- **Specs dashboard section** - New Specs panel in the project dashboard

### Improvements
- **Issue title in all preview dialogs** - Image, Markdown, PDF, and Comments dialogs now show the issue title in the header
- **Shared search/print utilities** - Extracted duplicated DOM search and print code into reusable modules
- **Search highlight colors** - Blue-themed highlights with proper dark/light mode contrast
- **In-app help** - Documented single-issue resync and spec type

### Fixes
- **Sound alerts persistence** - Fixed sound alerts saving to the wrong project path
- **Dashboard AI column** - Hidden when no AI sessions detected

## [0.7.6] - 2026-03-11

### Fixes
- **App freeze on rapid project switching** - Prevent the app from freezing when switching between projects quickly

## [0.7.5] - 2026-03-11

### Features
- **Attachment downloads** - Download button added to image and PDF preview modals
- **PDF attachments** - Full PDF support with preview, migrated to Tauri asset protocol
- **In-progress badge** - Sidebar project list shows count of in-progress issues

### Fixes
- **Search regression** - Fixed crash when searching (useI18n called outside setup), also resets pagination on search

## [0.7.4] - 2026-03-11

### Features
- **Deferred section** - Issues with "deferred" status now appear in their own collapsible table section, placed after "Ready to Work"

## [0.7.3] - 2026-03-10

### Features
- **Bilingual interface (EN/FR)** - Full i18n support with language switcher, all UI labels and table section headers translated
- **Automated screenshot capture** - Screenshots generated for the website with theme and locale variants (dark/light x en/fr)
- **ConflictDialog visual testing** - Mock conflict dialog for design iteration

### Improvements
- **Website screenshots by locale** - Website now serves screenshots matching the visitor's language and theme
- **Screenshot deployment checks** - Deploy and verify scripts validate all 4 theme/locale combinations

### Fixes
- **Duplicate log entries** - Eliminated redundant log lines in the native logger
- **Table section headers** - Section headers now properly translated in all locales
- **Linux platform info** - Corrected Linux installation instructions in FAQ
- **Git sync safety** - Skip git operations when `.pair` is gitignored

## [0.7.2] - 2026-03-10

### Features
- **User-facing changelog page** - Bilingual (EN/FR) changelog page on the website at `/changelog`, with version badges and grouped features/fixes
- **Scroll spy navigation** - Header nav links highlight based on visible section when scrolling the homepage
- **Website-first approach** - Release notes and changelog now live on the website, no longer synced to GitHub pair-dist

### Improvements
- **Check for updates** - Now uses release body directly instead of fetching CHANGELOG.md from GitHub. "View on GitHub" replaced by "Visit website"
- **Centralized website URL** - `WWW_BASE_URL` from `.env` used as single source of truth across frontend (runtimeConfig) and scripts (changelog_utils.py)
- **Frontend constants** - Added `app/utils/constants.ts` for shared GitHub distribution repo references
- **Release notes generation** - New `scripts/changelog_utils.py` generates user-facing release bodies from `www/app/data/changelog.ts`
- **FAQ updated** - Added "Is PaiR open source?" and "Does PaiR work offline?" questions, reordered logically, clarified AI hooks support and Beads migration

### Cleanup
- Removed CHANGELOG.md from pair-dist sync (SYNC_FILES in sync-releases.py and publish-local.py)
- Removed "All releases available on GitHub" mention from download section
- Removed GitHub link from header navigation (kept in footer)

## [0.7.1] - 2026-03-10

### Features
- **Soft delete by default** - Deleting an issue now archives it (tombstone) instead of permanently removing it. Check "Delete permanently" in the confirmation dialog for hard delete. Dialog wording adapts contextually ("Archive" vs "Delete permanently").

### Documentation
- Updated in-app help docs (issue workflow, issue details) to reflect soft delete as default behavior

## [0.7.0] - 2026-03-09

### Features
- **Embedded help documentation** - 10-page help system accessible via `?` shortcut or Help menu, with table of contents navigation, cross-page search, and print support
- **Clickable AI toast notifications** - Click on an AI notification toast to focus the agent's session window (macOS only)

### Improvements
- **Help viewer UX** - Wider dialog, resizable TOC sidebar, scroll spy, arrow key navigation between pages and sections
- **Hidden internal statuses** - Internal statuses (pinned, hooked) no longer appear in filters and edit form

### Fixes
- **Mute icon zoom** - Mute icon no longer scales disproportionately when zooming the interface
- **Help viewer overflow** - Content no longer overflows on narrow windows
- **Website screenshot** - Website now uses the most recent screenshot by date instead of a version-based path that could break on patch releases

### Documentation
- Why PaiR? - Project philosophy and manifesto
- Getting Started, Dashboard, Issue Details, Issue Workflow
- AI Agent Hooks, Sync & Conflicts, CLI Reference
- Settings (global + per-project), Keyboard Shortcuts

## [0.6.1] - 2026-03-09

### Features
- **AI agent hooks settings** - Configure AI agent hooks (Claude Code, Cursor, Codex) directly from the global settings page

### Fixes
- **Notification sound fallback** - Fall back to global sound preset when a project has no notification config
- **Linux audio crash** - Prevent audio playback error from blocking toast notifications on Linux

## [0.6.0] - 2026-03-09

### Features
- **AI sound notifications** - Per-project sound alerts when AI agents need attention, with 5 sound presets (Bell, Double beep, Rising, Soft chime, Alert), per-project volume override, and global mute toggle in header

### Improvements
- **Centralized localStorage** - All keys managed through `storage-keys.ts` with active registries and automatic cleanup of stale keys
- **Non-destructive storage migration** - `beads:` → `pair:` key migration at boot, old keys preserved for backward compatibility
- **Path normalization** - `hashPath()` now strips trailing slashes, fixing per-project settings mismatches
- **Legacy rename** - `useBeadsPath` → `useCliPath` across the entire codebase
- Cargo cache added to all CI builds (Windows, macOS, Linux)

### Tests
- Added 21 tests for hash normalization, storage key registry, and migration logic

## [0.5.0] - 2026-03-07

### Features
- **Collapsible sections** - Issue list organized into In Progress, Pinned, and Current sections with collapsible headers
- **AI panel: unknown project/agent detection** - Flag events from unrecognized projects or AI agents

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

## [0.4.4] - 2026-03-07

### Features
- **Project pinning** - Pin/unpin projects in sidebar, toggle "pinned only" filter to focus on starred projects. Drag-and-drop and sort hidden when filtered
- **AI panel: brand icons** - Per-event SVG icons for Claude, Cursor, Codex, Gemini, Vibe (+ generic fallback)
- **AI panel: project name** - Display project name (last path segment) after AI icon
- **AI panel: Copy button** - Copy filtered events to clipboard as text
- **AI panel: cost & tokens on Stop** - Show total cost and token counts when AI session ends
- **AI panel: tool error detection** - Highlight failed tool calls in red via `tool_error` event type
- **AI panel: tool_use_id via meta** - Pass tool_use_id on all tool events for future duration tracking
- **SubagentStop hook** - Capture agent completion events (`agent_stop` in AI panel)
- **PreCompact hook** - Capture context compaction events (`compact` in AI panel)
- **`--ai-name` flag** - `pair notify --ai-name claude` for testing AI icons without actual AI process

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

## [0.4.3] - 2026-03-06

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

## [0.4.2] - 2026-03-06

### Docs
- Add `notify` command and Claude Code hooks configuration to AGENTS.md template

## [0.4.1] - 2026-03-06

### Fixes
- Fix stale data when external process (CLI script) modifies the database - invalidate cached SQLite connection on mtime change
- Add `relate`/`unrelate` commands to AGENTS.md template

## [0.4.0] - 2026-03-06

### Features
- **AI session tracking** - Associate AI session PID with projects via push notifications. Walk PPID chain to detect AI CLI (Claude, Cursor, Codex) and editor (Zed, VS Code)
- **Per-project activity LED** - Each project line shows a LED that blinks green on AI notifications (replaces folder icon)
- **Focus AI session window** - Click the focus button or press ⌘⇧F to switch to the editor window where AI is working (uses editor CLI for precise window targeting)
- **AI Events panel** - Renamed from "Claude Events" to "AI Events". Shows AI name badge (claude/cursor/codex) per event. Toggle with ⌘⇧A
- **Keyboard shortcuts** - ⌘⇧A (AI panel), ⌘⇧F (focus AI session) added to app menu
- **Blocked badge** - Visual badge for blocked issues in the issue list
- **README** - Added Features section, merged Background & Compatibility

### Improvements
- AI name (`ai_name`) and editor name (`editor`) fields added to PushEvent for richer notifications
- Focus window uses 3 strategies: editor CLI from notification → detect editor from PID tree → AppleScript fallback
- Session focus button uses neutral gray style (white when project selected)

## [0.3.4] - 2026-03-04

### Features
- **`pair attach` CLI command** - attach files to issues from the command line, with real-time attachment refresh in the app

### Fixes
- Fix toast notification contrast in light theme
- Rename release artifacts + disable unavailable downloads on website

## [0.3.3] - 2026-03-04

### Features
- **Claude Events: native hook integration** - `pair notify --hook <hook_type>` reads JSON from stdin, classifies events (significant vs verbose), and forwards to the app. Replaces the bash/jq bridge script.
- **`pair notify` CLI** - new `--hook` flag for Claude Code hooks, plus existing manual mode (`-t`/`-m`) preserved for backward compatibility
- Self-call detection: Bash commands starting with `pair notify` are automatically skipped to prevent echo loops

### Docs
- Mark `scripts/claude-hook.sh` as legacy (kept as reference)
- Update AGENTS.md with `--hook` mode documentation and simplified Claude Code config

## [0.3.2] - 2026-03-03

### Features
- **Print Markdown** - print button + `Cmd+P` in Markdown preview modal (uses native print dialog via Tauri WebView)
- **Custom relations CLI** - `pair relate <id1> <id2> --type <type>` and `pair unrelate` for non-blocking relations (relates-to, duplicates, supersedes, etc.)
- Custom relations exported/imported in JSONL (`relations` field) - survives sync round-trips

### Fixes
- Fix dynamic window title - add missing `core:window:allow-set-title` permission
- Fix print in Tauri WebView - rewrite from iframe (blocked) to permanent `@media print` styles
- Print page breaks: `break-inside: avoid` on code blocks, tables, headings

### Docs
- Rebrand all docs from Beads to PaiR
- Rewrite README for public distribution + sync automation
- Document `pair relate` / `pair unrelate` in AGENTS.md

## [0.3.1] - 2026-03-03

### Features
- New **Maintenance** section in project settings dialog (separate from Actions)
  - **Repair database** - re-sync SQLite with JSONL (auto-backup before repair)
  - **Fix CLI permissions** - restore +x on the pair binary (resolves symlink)
  - Re-initialize moved to Maintenance section (most destructive = last)
- Dynamic window title - shows "PaiR - ProjectName" when a project is selected
- Parse `.pair/config.yaml` for `issue-prefix` (was hardcoded to defaults)
- Toast notifications on all maintenance actions (success/error)

## [0.3.0] - 2026-03-02

### Features
- Per-project settings dialog - click the cog icon on any project in the sidebar
  - Project info: path, database status, issue count, legacy .beads/ detection
  - Actions: remove from list, re-initialize database, migrate from .beads/
  - All actions require confirmation before executing
  - Read-only settings display (prefix, git tracking) - editing in a future version

### Fixes
- Migration dialog: add "Start fresh" option + skip if .pair/ already exists
- Onboarding: folder select triggers migration + remove "switch without migrating"
- Onboarding: cancel migration no longer creates .pair/ + git tracking choice preserved
- Check-for-updates broken on all platforms

## [0.2.2] - 2026-03-02

### Fixes
- Fix sidecar binary not found on Linux - Tauri uses full target triple (`pair-x86_64-unknown-linux-gnu`) but lookup only checked `pair` and `pair-x86_64`

## [0.2.1] - 2026-03-02

### CLI
- New `pair dep tree <id>` - recursive dependency tree with cycle detection
- New `pair dep list <id>` - direct blocks/blocked_by with issue details
- Both commands support `--json` for structured output

### UI
- App icon now uses dark background (#1E1E2E) for better dock visibility
- Header logo aligned with left sidebar
- Search field now overrides status filters to show matching results

## [0.2.0] - 2026-03-01

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

## [0.1.0] - 2026-02-28

First pre-release of PaiR - standalone issue tracker with built-in Rust engine.

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
- `list_ready_issues()` - open issues not blocked by any open issue
- `is_blocked()` - real-time blocking check
- JSONL format: `blocked_by` (replaces legacy `dependencies`)

### CLI (`pair`)
- `pair init/list/show/create/update/close/delete`
- `pair search` - full-text search
- `pair ready` - unblocked open issues
- `pair export` - force DB→JSONL re-export
- `pair import` - JSONL import with merge
- `pair reorder` - reorder child position
- `pair migrate` - unidirectional .beads → .pair migration
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
