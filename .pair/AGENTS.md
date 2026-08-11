# AGENTS.md — PaiR

This project uses `pair` for issue tracking. Issues are stored locally in SQLite (`.pair/pair.db`).
The PaiR app monitors AI activity in real-time — every CLI call you make is reflected instantly in the UI.

## First-time setup

**On your first session in this project, check these two things:**

### 1. Hooks
Check if PaiR notification hooks are installed. They enable real-time notifications in PaiR: sound alerts, visual indicators, and AI activity tracking.

To check: look for `pair notify` entries in `~/.claude/settings.json` (global) or `.claude/settings.json` (project-level) under `"hooks"`.

If missing, offer to install them by adding the hooks configuration listed in the "Claude Code hooks configuration" section below. This is a one-time setup — once installed, every action you take will be visible in the PaiR app.

### 2. AGENTS.md awareness
Check that `~/.claude/CLAUDE.md` (global) contains a reference to `.pair/AGENTS.md`. This ensures every Claude session on any PaiR project reads this file and follows the workflow — especially the cross-project communication protocol.

To check: look for "AGENTS.md" in `~/.claude/CLAUDE.md`. If missing, propose adding:
```markdown
## PaiR — Cross-project awareness
- If `.pair/AGENTS.md` exists in the project, read it at session start
- If your session is cabled to sessions in other projects (`pair cable list`), follow the cross-project reading protocol from AGENTS.md
```

## How to work with PaiR

**These are not suggestions — they are part of your workflow.** The human using this project expects you to track your work through PaiR, just like you would use git for version control.

### Before starting any task

- Run `pair list -s open` to see existing issues — your task may already be tracked
- If working on a specific issue, run `pair update <id> -s in_progress` before writing any code
- Run `pair show <id>` to read the full context (description, comments, acceptance criteria)
- **Check cabled sessions:** run `pair cable list` — for each project on the far end of a cable, read its recent journal: `pair journal --from <prefix> --since 4h`. Adapt your plan if breaking changes or related work is detected.

### While working

- **Found a bug?** → `pair create "Bug title" -t bug -p p1 -d "Description"`
- **Spotted a TODO or limitation?** → `pair create "Title" -t task -d "Details"`
- **Hit a blocker?** → `pair update <id> -s blocked` and `pair dep add <id> <blocker-id>`
- **Pausing work temporarily** (waiting on external response, context switch)? → `pair update <id> -s paused`
- **Work deferred to later** (not started, postponed)? → `pair update <id> -s deferred`
- **Made progress worth noting?** → `pair comments add <id> "What was done"`

### Session journal — leave a trail

The journal is not just an audit log — it is the **communication channel between agents working on cabled sessions.** Write journal entries at key moments so that other sessions (cabled to yours) can understand what happened here without reading your code or commits.

**When to write:**

| Moment | Example |
|--------|---------|
| Starting a significant task | `pair journal "Starting: implement POST /foo endpoint" --tags task` |
| Key technical decision | `pair journal "Decision: use WebSocket instead of polling for sync" --tags decision` |
| Completing a unit of work | `pair journal "Done: POST /foo endpoint with validation and tests" --tags progress` |
| Blocked or unexpected issue | `pair journal "Blocked: dependency X v3 incompatible with our auth layer" --tags blocker` |

**Standard tags:** `task`, `decision`, `progress`, `blocker` — these enable filtering and aggregation.

**Default bias: write.** When in doubt about whether a journal entry is worth writing, write it. The cost of an extra entry is near zero; the cost of a missing one is a blind spot for the cabled project. Only skip entries that are clearly internal with no cross-session value (formatting, imports, typos). Ask yourself: *"would an agent on a cabled session — or a future session on this project — benefit from knowing this?"* If the answer isn't a clear "no", write it.

### Cross-project awareness — stay in sync

If your session is cabled to sessions in other projects, **don't just check the journal at session start and forget about it.** The other session may be working in parallel — **you MUST re-read at these specific moments:**

| When | What to do | Why |
|------|-----------|-----|
| **Before starting any new task** | `pair journal --from <prefix> --since 2h` | The other session may have changed something that affects your plan |
| **Before editing a shared interface** (API, schema, config, types) | Re-read the cabled project's journal | Avoid implementing against an outdated contract |
| **Before every commit** | `pair journal --from <prefix> --since 1h` | Catch last-minute changes before locking in your work |
| **Before closing an issue** | Final check for every cabled project (`pair cable list`) | Don't close with undetected conflicts |

**This is not optional.** If you skip these checks, the other session works blind and the human has to manually bridge the gap — which defeats the purpose of the journal.

**If nothing new appears, move on silently** — don't mention you checked. Only surface relevant findings.

### Multi-session same-project awareness

When multiple Claude sessions work on the **same project** simultaneously (e.g., two terminals open on the same repo), they risk conflicting edits. The journal is also the coordination channel for this case.

**At session start**, always check if another session is already active:
```bash
pair journal --last 10
```
Look for recent `task` or `progress` entries — if another session wrote "Starting: fix JournalPanel layout" 30 minutes ago, **you know someone else is in that area**.

**Before editing a file**, check the journal for recent activity on the same area:
- If you see another session working on the same component/module, **write a journal entry** signaling your intent before starting: `pair journal "Starting: auto-refresh journal on push events — touching JournalPanel.vue and index.vue" --tags task`
- This gives the other session a chance to see the overlap on their next journal check

**When you finish a unit of work**, always write a progress entry listing the files you modified:
```bash
pair journal "Done: added lastPushAt prop to JournalPanel — modified JournalPanel.vue, index.vue" --tags progress
```

**The goal:** each session leaves enough breadcrumbs that the other can avoid stepping on the same files. This doesn't prevent all conflicts, but it makes them visible early.

### When finishing work

- **If your session has cables (`pair cable list`):** run `pair journal --from <prefix> --since 2h` for each cabled project as a final coherence check before closing
- Add a summary comment: `pair comments add <id> "Done: what was implemented"`
- Close the issue: `pair close <id>`
- Reference the issue ID in your commit message

### Closing a parent issue — children inventory (mandatory)

This rule applies to **any issue that has children** — epic, campaign, spec, or any other type used as a parent. Not just epics.

Before closing any parent issue, **inventory its children first** — never close a parent silently while children are still open.

1. Run `pair children <parent-id>` to list them.
2. Identify children whose status is **not `closed`** (any of `open`, `in_progress`, `paused`, `blocked`, `deferred` triggers the prompt).
3. If any unfinished children exist, surface the list to the user and ask explicitly — three options, mirroring the delete-cascade dialog model:
   - **Close parent + cascade-close children** — iterate `pair close` on each child sequentially (no batch-close command exists; if a tracker-specific close skill is available, prefer it per child to preserve Redmine/GitHub sync and closing comments)
   - **Close only the parent, leave children open** — children remain attached to the now-closed parent
   - **Cancel**
4. If all children are already `closed`, or if the issue has no children, close it normally without prompting.

**Blockers are handled automatically.** Closing a blocker auto-releases its blocked issues — `blocked_by` excludes closed blockers. No manual unblock step needed.

### Always

- **Never ignore `.pair/`** — it is the project's issue tracker, not a temp folder
- **Commit `.pair/issues.jsonl` separately** from code changes (legacy mode only — see "Storage modes" below)
- **Check for related issues** before creating duplicates: `pair search "keyword"`

### Storage modes

PaiR supports two storage layouts for `.pair/`. The mode is auto-detected at boot; both behave the same from a CLI standpoint.

**Legacy mode** — `.pair/` is a plain directory tracked on the current code branch.
- Tickets visible in `git status` whenever they change.
- Switching code branches changes the tracker content (tickets appear / disappear).
- Tracker conflicts can occur when merging code branches.
- **Workflow rule**: always commit `.pair/` changes in a separate commit from code (`chore(pair): update issues`). Don't mix.

**Migrated mode** — `.pair/` is a worktree of the tracker orphan branch (`pair-tracker` by default; the name is configurable per project via `pair config set tracker.branch <name>`).
- The tracker lives on a dedicated branch with no code; the worktree mounts it transparently at `.pair/`.
- `.pair/` is gitignored on every code branch and never appears in code-branch `git status`.
- Switching code branches no longer affects tickets.
- The separate-commit workflow rule no longer applies — there's nothing to commit on the code branch.
- **Workflow rule** (translated from "two separate commits" to "two separate pushes"): if the user opted into "Track issues in git" (i.e. `.pair/` is a real git worktree, not just a local SQLite store), the tracker branch must be pushed **before** the code branch so tickets are backed up to the remote and GitHub proposes the code branch (the last one pushed) by default when opening a PR. Two commands, in order:
  ```bash
  git -C .pair push          # tracker branch FIRST (best-effort; resolve the configured name with `pair config get tracker.branch` if unsure; first time: git -C .pair push -u origin <tracker-branch>)
  git push                   # code branch LAST — stays the most recently pushed so GitHub proposes it for PRs
  ```
  Detection: if `git -C .pair rev-parse HEAD` fails or `.pair/` is not a git worktree, the user did not opt into git tracking — skip the tracker push silently and just `git push` the code branch. Otherwise, if the tracker push fails (no remote, network, etc.), report and continue with the code push — the code push is the source of truth.
- Migration is one-way through the in-app dialog (or `pair migrate-tracker`); a backup tag is created so the migration can be rolled back in-session.

**How to tell which mode the current project uses**: at boot, the native log (`~/Library/Logs/com.pair.app/pair.log`) prints `worktree mounted` (migrated) or `staying in legacy mode` (legacy). Identical in dev and prod.

---

## CLI Binary

`pair` — if not in PATH, check the project's build output.

## Global Flags

| Flag | Description |
|------|-------------|
| `-C <path>` / `--project <path>` | Project directory (default: current directory) |
| `--json` | Output as JSON |
| `--actor <name>` | Actor name for authoring (default: git user.name) |

## Commands

### `init` — Initialize tracker

```bash
pair init
```

### `config` — Per-project configuration

Project-local configuration values stored in the global catalog (machine-local, never shared).

```bash
pair config get tracker.branch        # Show resolved value (with "(default)" tag if fallback)
pair config set tracker.branch <name> # Override the tracker branch name for this project
pair config unset tracker.branch      # Revert to the default
pair config list                      # Show all keys for the current project
pair config get tracker.branch --json # JSON output
```

Supported keys:

| Key | Default | Description |
|-----|---------|-------------|
| `tracker.branch` | `pair-tracker` | Name of the orphan branch hosting `.pair/` content. Set this **before** `pair migrate-tracker` (or before pushing the orphan branch for the first time) so the right branch is created. Useful when several contributors will push their tracker on the same repo — each user can keep their tracker on a personal branch (`pair-tracker-<username>`) without colliding. |

Validation: branch names must match `^[A-Za-z0-9._/-]+$`, no leading `-` or `/`, no `..`, no `//`, no spaces. Empty values are rejected (use `unset` to revert).

If a remote `origin` is configured at migration time and the resolved branch is still the default, PaiR logs a hint with the suggested personalized name (`pair-tracker-<owner>` parsed from the remote URL).

### `list` — List issues

```bash
pair list                        # Open issues (default)
pair list -s open                # Filter by status (open, in_progress, paused, blocked, deferred, closed)
pair list -s in_progress
pair list -s paused
pair list -s blocked
pair list -s deferred
pair list -s closed
pair list -a                     # All issues (shorthand for -s all)
pair list -t bug                 # Filter by type (task, bug, feature, epic, chore, spec, campaign)
pair list -p p0                  # Filter by priority (p0, p1, p2, p3)
pair list --assignee "Name"      # Filter by assignee
pair list --limit 10             # Limit results
pair list --pinned               # Only pinned issues
pair list --json                 # JSON output
```

### `show <id>` — Show issue details

Displays full issue detail including children, comments, labels, dependencies.

```bash
pair show <id>
pair show <id> --json
```

### `create <title>` — Create a new issue

```bash
pair create "Fix login bug"

pair create "Add dark mode" \
  -d "Description here" \
  -t feature \
  -p p1 \
  --assignee "Name" \
  -l "ui,theme" \
  --parent <parent-id> \
  --estimate 120 \
  --design "Design notes" \
  --acceptance "Acceptance criteria" \
  --notes "Additional notes" \
  --external-ref "https://example.com/issue/42" \
  --spec-id "SPEC-001"
```

| Flag | Short | Description |
|------|-------|-------------|
| `--description` | `-d` | Issue body/description |
| `--type` | `-t` | Issue type: `task`, `bug`, `feature`, `epic`, `chore`, `spec`, `campaign` |
| `--priority` | `-p` | Priority: `p0`, `p1`, `p2`, `p3` |
| `--assignee` | | Assignee name |
| `--labels` | `-l` | Comma-separated labels |
| `--parent` | | Parent issue ID (for sub-tasks) |
| `--estimate` | | Estimate in minutes |
| `--design` | | Design notes |
| `--acceptance` | | Acceptance criteria |
| `--notes` | | Additional notes |
| `--external-ref` | | External reference (URL, Redmine ID) |
| `--spec-id` | | Spec ID |

### `update <id>` — Update an issue

Status values: `open`, `in_progress`, `paused`, `blocked`, `deferred`, `closed`. Semantics:
- `paused` — work started then temporarily halted (waiting on external response/team, context switch)
- `blocked` — work that cannot start (distinct from `paused`)
- `deferred` — work not yet started, postponed

```bash
pair update <id> -s in_progress
pair update <id> -s paused
pair update <id> -s blocked
pair update <id> -s deferred
pair update <id> --title "New title"
pair update <id> -d "Updated description"
pair update <id> -t bug -p p0
pair update <id> --assignee "Name"
pair update <id> --assignee ""          # Clear assignee
pair update <id> -l "ui,urgent"         # Replace all labels
pair update <id> --parent <parent-id>
pair update <id> --parent ""             # Clear parent
pair update <id> --estimate 60
pair update <id> --estimate 0            # Clear estimate
pair update <id> --metadata '{"key":"value"}'  # Set metadata JSON
pair update <id> --metadata ""           # Clear metadata
pair update <id> --branch feature/x      # Fix associated branch
pair update <id> --branch ""             # Clear branch
```

Use empty string `""` to clear optional fields, `0` to clear estimate.

### `close <id>` — Close an issue

```bash
pair close <id>
```

### `pin <id>` / `unpin <id>` — Pin or unpin an issue

```bash
pair pin <id>                # Pin for quick access
pair unpin <id>              # Remove pin
pair list --pinned           # List pinned issues only
```

### `delete <id>` — Delete an issue

```bash
pair delete <id>              # Soft delete
pair delete <id> --hard       # Permanent removal
```

### `search <query>` — Full-text search

```bash
pair search "login bug"
pair search "query" --limit 10
```

### `ready` — List unblocked open issues

```bash
pair ready
```

### `children <id>` — List child issues

```bash
pair children <id>
```

Lists all child issues of a parent issue (e.g., an epic).

### `reorder` — Reorder a child issue within its parent

```bash
pair reorder <parent-id> <child-id> <new-position>
```

Position is 1-based. Re-numbers all siblings sequentially.

### `export` — Force re-export database to JSONL

```bash
pair export
```

Useful after schema migrations to update the JSONL format (e.g., `blocked_by`, `position` fields).

### `import <path>` — Import issues from a JSONL file

```bash
pair import path/to/issues.jsonl
```

Merge strategy: last-write-wins by `updated_at`. Comments use append-only merge.

### `comments` — Manage comments

```bash
pair comments add <id> "Comment body"
pair comments delete <comment-id>
pair comments push <comment-id>         # Push a local comment to the external provider (GitHub/GitLab)
```

### `label` — Manage labels

```bash
pair label add <id> "label-name"
pair label remove <id> "label-name"
```

### `attach` — Attach files to an issue

```bash
pair attach <id> screenshot.png              # Attach one file
pair attach <id> img1.png notes.md           # Attach multiple files
pair attach <id> "file with spaces.png"      # Quoted paths
```

Supported: images (png, jpg, jpeg, gif, webp, bmp, svg, ico, tiff) and markdown (md, markdown).
Files are copied to `.pair/attachments/{short-id}/`, sanitized (kebab-case, no accents), with duplicate handling.
The app returns **absolute paths** for attachments (it manages multiple projects simultaneously, so paths must be resolvable regardless of the current working directory).
Emits a push notification so the app refreshes the attachment preview in real-time.

**Updating an existing attachment** (replace, not duplicate): `pair attach` does NOT overwrite — re-attaching a file with the same name copies it as `<name>-1.md` (`-2`, `-3`...). When you update a document attached to an issue (timeline.md, plan.md, design.png, etc.) and want the new version to *replace* the old one, always:

```bash
pair detach <id> <filename>     # remove the old version first
pair attach <id> <path>          # then attach the new version
```

Otherwise the issue accumulates stale duplicates the user has to clean up by hand.

### `detach` — Remove an attachment from an issue

```bash
pair detach <id> filename.png            # Remove a specific attachment
```

Deletes the file from `.pair/attachments/{short-id}/`. Cleans up the directory if empty.
Emits a push notification so the app refreshes.

### `dep` — Manage dependencies

```bash
pair dep add <id> <blocker-id>              # blocker blocks id
pair dep add <id> <blocker-id> --type blocks
pair dep add <id> --external redmine-1234 --label "Backend task"   # external blocker (other tracker, no PaiR issue)
pair dep remove <id> <other-id>
pair dep remove <id> redmine-1234 --external                       # remove an external blocker
pair dep tree <id>                          # Recursive dependency tree
pair dep list <id>                          # Direct dependencies only
```

### `relate` / `unrelate` — Manage relations between issues

Relations are non-blocking links between issues (unlike `dep` which is for blockers).

```bash
pair relate <id1> <id2>                        # Default type: relates-to
pair relate <id1> <id2> --type relates-to      # Explicit type
pair unrelate <id1> <id2>                      # Remove relation
```

### `sync-external` — Sync issues from an external provider

```bash
pair sync-external                    # Incremental sync (since last sync)
pair sync-external --full             # Full sync (ignore last sync timestamp)
pair sync-external --dry-run          # Preview what would be synced
```

Syncs issues from GitHub, GitLab, or Redmine into PaiR (one-way: external → PaiR).

**Configuration** (`.pair/config.yaml`):
```yaml
sync:
  provider: github          # or "gitlab" or "redmine"
  repo: owner/repo          # Optional: override git remote auto-detection
  token_env: GITHUB_TOKEN   # or GITLAB_TOKEN or REDMINE_API_KEY
```

The provider is auto-detected from the git remote URL if not specified.
Use `sync-repo` to point to a different repo than the git remote (e.g., a public issues repo).

| Flag | Description |
|------|-------------|
| `--full` | Force full sync (ignore last sync timestamp) |
| `--dry-run` | Preview what would be synced without writing |

### `migrate` — Migrate from .beads to .pair

```bash
pair migrate                    # Import issues from .beads/ into .pair/
pair migrate --force            # Overwrite existing .pair/ data
```

### `notify` — Send notifications to the PaiR app

```bash
pair notify --hook <hook_type>              # Claude Code hook mode (reads JSON from stdin)
pair notify -t agent_start -m "Starting"    # Manual mode
pair notify -t test -m "Hello"              # Test notification
```

| Flag | Description |
|------|-------------|
| `--hook <type>` | Claude Code hook mode: reads JSON payload from stdin, classifies and forwards |
| `-t, --type` | Notification type (idle, permission, agent_start, agent_stop, test, etc.) |
| `-m, --message` | Message content |
| `--session` | Session ID (from Claude Code hook payload) |
| `--actor` | Actor name (default: git user.name) |
| `--ai-name` | Override AI name for testing (claude, cursor, codex, gemini, vibe) |

### `journal` — Project journal (audit trail)

```bash
pair journal "Decision: use REST API, not GraphQL" --tags architecture,api   # Write
pair journal "API endpoint ready" --push info --to "PaiR 2"      # Write + push to a cabled recipient
pair journal "Waiting for frontend integration" --push attente --to scripteasy-v4  # "waiting" intent
pair journal "Run integration tests now" --push action --to "PaiR 2"   # "action" intent
pair journal --today                    # Today's entries
pair journal --last 10                  # Last 10 entries
pair journal --tag api                  # Filter by tag
pair journal --since 2h                 # Since 2 hours ago
pair journal --from scripteasy-v4      # Read another project's journal (read-only)
pair journal --export                   # Export to .pair/journal.jsonl
```

The journal is **auto-populated** on every `pair create`, `pair close`, `pair comments add`, `pair update` (status changes → `status_changed`), `pair attach` (`attachment_added`), and `pair detach` (`attachment_removed`). Manual entries are for decisions, milestones, and notes.

**Cross-session push** (`--push`): delivers the journal entry in real-time to a **cabled** recipient (named with `--to`) via tmux send-keys. Types: `info` (FYI), `attente` (waiting for the recipient), `action` (instruction to act). `--to` is required; a recipient that isn't cabled yet is cabled on the fly. Without `--push`, the entry is written to the journal only. Cables are created with `pair cable add` or by dragging sessions together in the graph view.

After reading another project's journal (`--from`), the next manual write is automatically tagged with `reply-to:<project>:<id>` to trace cross-project exchanges.

#### Natural-language messaging — translate user requests to the right `journal --push`

When the user asks you (in any language) to communicate something to other sessions/peers, **do not** ask them how to phrase it — translate directly to the right command. Recognise these intents (the user may phrase them in English, French, or any other language — interpret the intent, not the surface form):

| Intent | Run |
|--------|-----|
| "send a message to X that …" / "tell X that …" / "let X know that …" | `pair journal --push info "…" --to X` |
| "ask X to …" / "I'm waiting on X for …" | `pair journal --push attente "…" --to X` |
| "tell X to do …" / "have X run …" | `pair journal --push action "…" --to X` |
| "tell everyone on project P …" | `pair journal --push info "…" --to P` |
| "tell X and Y …" | `pair journal --push info "…" --to X --to Y` |
| "log that …" / "write in your journal that …" / "note that …" | `pair journal "…" --tags status` (no push) |

| "connect with X" / "cable yourself to X" / "wire us to X" | `pair cable add <X>` |
| "disconnect from X" / "cut the cable with X" / "we're done with X" | `pair cable rm <X>` |
| "who are we connected to?" / "list the cables" | `pair cable list` |

Notes:
- **`--to` is required** (pair-jbuz.4). `<X>` is a session name, a session id, or a
  **project name** meaning every cabled session of that project. Repeat `--to` for
  several recipients. A push with no recipient is **refused**: without one it used
  to reach every session of every associated project, which is exactly the message
  duplication this removed.
- **The recipient must be cabled to you.** The cable is the permission, the `--to`
  is the address. If it isn't, the command says so and tells you to run
  `pair cable add <X>` — it never delivers silently to someone else.
- The journal entry is recorded even when delivery is refused: the local trace is
  kept, only the delivery failed.
- `info` is the safe default. Use `attente` only when the user is explicitly waiting on something. Use `action` only when they're asking the recipient to actively do something.
- This is distinct from your own automatic state reporting (which also uses `journal --push` at key milestones — see the "Session journal" section above). The difference is **who initiated** : here it's the user asking you to convey a message ; in the automatic case it's you proactively reporting your state.

### `cable` — Wire your session to another one

```bash
pair cable add scripteasy-v4        # Connect to that project's session
pair cable add "PaiR 1"             # …or name the session precisely
pair cable rm scripteasy-v4         # Disconnect
pair cable list                     # Your cables + the sessions you can cable
pair cable list --all               # Every cable, not just yours
```

A cable wires **your session** to another one, across projects. It is
**bidirectional** (connecting one way means both talk) and persisted in
`catalog.db`, so it survives a restart and shows up in the app's graph view.

Naming the target: a **project name** is the usual form (`scripteasy-v4`) and
works when that project has a single active session. If it has several, the
command refuses and lists them — name the session instead (`"scripteasy-v4 2"`).
A session id also works.

Scope: sessions running under tmux or herdr. A plain PTY session exists only in
the running app and cannot be cabled from the CLI.

**Do not confuse the three channels:**

| Tool | What it does | Reaches the other agent's context? |
|------|--------------|-----------------------------------|
| `pair cable` | Opens the line between two sessions | No — it wires, it does not speak |
| `pair journal --push` | Sends a message to cabled sessions / peers | Yes |
| `pair notify` | Shows a toast in the PaiR app | **No** — UI only |

Cabling does **not** start a conversation. It declares who may talk to whom;
saying something still goes through `journal --push`.

### `catalog` — Global project catalog

```bash
pair catalog                           # List all known projects (prefix, path, name)
```

Projects are auto-registered on any CLI use. The catalog lives in `~/Library/Application Support/com.pair.app/catalog.db`.

## Cross-project communication protocol

### Why this exists

When your session is cabled to a session in another project, the two depend on each other — a change in one can break or require adaptation in the other. Each project has its own AI agent, working in its own session, with no direct communication channel. The journal is that channel: **an asynchronous message bus between agents working on cabled sessions.**

Without this protocol, an agent changes an API contract on Project A, and the agent on the cabled session in Project B has no idea until something breaks. With it, the agent on Project B reads the journal on session start, sees the change, and can adapt proactively.

### What happens automatically
- Every `pair create`, `pair close`, and `pair comments add` is logged in the project's journal — no extra action needed.
- After reading another project's journal (`pair journal --from <prefix>`), your next manual journal write is auto-tagged with `reply-to:<prefix>:<id>`. This creates a traceable conversation thread across projects.
- **The PreToolUse hook automatically injects new journal entries** (local + the projects cabled to your session) into your context before every Edit/Write — you don't need to manually re-read during the session. See "Automatic journal context injection" in the hooks section.

### Session start protocol — MANDATORY

**This is the FIRST thing you do when you start a session. No exceptions. Do not skip any step. Do not start working on anything before completing this protocol.**

**Step 1 — Read the local journal** (always, unconditionally):

```bash
pair journal --last 10
```

This tells you what happened in recent sessions on this project: decisions made, tasks completed, blockers hit, direction changes. Without this, you are working blind — you might redo work, contradict a decision, or miss critical context.

**Step 2 — Check your cables** (always, unconditionally):

```bash
pair cable list
```

**Step 3 — For each project on the far end of a cable, read its journal** (mandatory for each one):

```bash
pair journal --from <prefix> --since 4h    # Run this for EVERY cabled project
```

Do not skip this. Do not defer it. The other session's agent may have changed an API, a schema, a shared contract — if you don't read their journal now, you will build on stale assumptions.

**Step 4 — Act on what you read:**

- **Breaking change detected** (API contract, shared schema, data format) → **warn the user immediately**, before starting any work. Example: "Project B changed the `/items` endpoint response format 2h ago — this may affect our client code."
- **Related work in progress** (feature that touches a shared boundary) → factor it into your plan. Don't duplicate effort or make conflicting changes.
- **No relevant activity** → proceed normally. Don't mention it.

**This protocol is not a one-time action.** You must re-read journals at key moments during the session — see "Cross-project awareness — stay in sync" above.

A detailed step-by-step procedure with classification rules and report format is available in `.pair/commands/cross-project.md`.

### When to write a manual journal entry

**General session entries** — use the standard tags (`task`, `decision`, `progress`, `blocker`) as described in the "Session journal" section above. These entries track your work for cross-session awareness: starting a task, making a decision, finishing a unit of work, or hitting a blocker.

**Cross-project entries** — when your work directly affects a cabled project, tag with the project prefix:

| Trigger | Example |
|---------|---------|
| You change a shared interface (API, schema, config format) | `pair journal "Changed GET /items response: added pagination wrapper" --tags api,project-b` |
| You discover a bug that originates in the other project | `pair journal "Auth token refresh fails — the issue is in Project B's token endpoint, not here" --tags bug,project-b` |
| You make an architecture decision that constrains the other project | `pair journal "Switching to WebSocket for real-time sync — REST polling deprecated" --tags architecture,project-b` |
| You complete work that the other project was waiting on | `pair journal "Export endpoint is live — Project B can start integration" --tags api,project-b` |

**Only skip journal entries for:**
- Things already captured by auto-logging (create/close/comment/status change/attach/detach) — no need to duplicate
- Purely internal micro-steps with zero cross-session value (formatting, imports, typos)

### The reply-to mechanism

When you read another project's journal with `--from`, PaiR remembers the last entry you saw. Your next manual journal write is automatically tagged `reply-to:<prefix>:<id>`, creating a conversation thread:

```
[project-b journal]  #42  "Changed /items response format — now paginated"
        ↓ (agent on project-a reads this)
[project-a journal]  #18  "Adapted client to paginated /items endpoint"  reply-to:project-b:42
        ↓ (agent on project-b reads this)
[project-b journal]  #55  "Confirmed: pagination contract works end-to-end"  reply-to:project-a:18
```

This gives both agents (and the human) a traceable chain of decisions across projects.

### Rules
- **Never** run `pair cable add` or `pair cable rm` without the user's explicit request — cables are deliberate decisions.
- **Never** write in another project's journal — cross-project access is **read-only**. You write in your own journal, tagged with the other project's prefix.
- **Never** spam the journal — if it doesn't cross the project boundary, it doesn't belong here.

## Claude Code hooks configuration

To connect Claude Code to PaiR, add hooks to `~/.claude/settings.json` (global) or `.claude/settings.json` (per-project):

```json
{
  "hooks": {
    "SessionStart": [
      { "matcher": "", "hooks": [{ "type": "command", "command": "/usr/local/bin/pair notify --hook SessionStart || true" }] }
    ],
    "PreToolUse": [
      { "matcher": "", "hooks": [{ "type": "command", "command": "/usr/local/bin/pair notify --hook PreToolUse || true" }] }
    ],
    "PostToolUse": [
      { "matcher": "", "hooks": [{ "type": "command", "command": "/usr/local/bin/pair notify --hook PostToolUse || true" }] }
    ],
    "Notification": [
      { "matcher": "", "hooks": [{ "type": "command", "command": "/usr/local/bin/pair notify --hook Notification || true" }] }
    ],
    "Stop": [
      { "matcher": "", "hooks": [{ "type": "command", "command": "/usr/local/bin/pair notify --hook Stop || true" }] }
    ],
    "SubagentStop": [
      { "matcher": "", "hooks": [{ "type": "command", "command": "/usr/local/bin/pair notify --hook SubagentStop || true" }] }
    ],
    "UserPromptSubmit": [
      { "matcher": "", "hooks": [{ "type": "command", "command": "/usr/local/bin/pair notify --hook UserPromptSubmit || true" }] }
    ],
    "PreCompact": [
      { "matcher": "", "hooks": [{ "type": "command", "command": "/usr/local/bin/pair notify --hook PreCompact || true" }] }
    ]
  }
}
```

**Important:**

- **Absolute path required:** Claude Code hooks run in a minimal shell where `PATH` may be incomplete. Always use the absolute path to the `pair` binary (run `which pair` to find it):
  - **macOS:** `/usr/local/bin/pair` (or your symlink location)
  - **Linux:** `/usr/bin/pair`
- **Resilience pattern (`|| true`):** If the `pair` binary is out of sync with the app (e.g., after a version bump or failed rebuild), hooks can error. The `|| true` absorbs any non-zero exit code — so Claude Code keeps working even if PaiR notifications are broken.
- **`SessionStart` hook:** fires when Claude Code opens a new session (startup), resumes one (`/resume`), or clears it (`/clear`). The payload includes `session_id` (also exposed as `CLAUDE_CODE_SESSION_ID` in the environment) and `source` (`startup` / `resume` / `clear`). This gives PaiR a clean session-boundary marker for journal correlation instead of inferring it from the first `UserPromptSubmit`.

This enables real-time AI activity tracking in the PaiR app: per-project activity LED,
AI events panel, and session focus (switch to the editor window where AI is working).

### Automatic journal context injection

The `PreToolUse` hook automatically injects recent journal entries into Claude's context before every **Edit** or **Write** tool call. This means:

- **Local journal**: entries from the last hour (or since last check) are shown
- **Cabled projects**: new entries since last check are shown, for every project on the far end of a cable from your session
- **Silent when empty**: no output if there's nothing new — zero noise

This replaces the need for manual `pair journal --from <prefix>` reads during a session. The hook state is tracked in `.pair/.journal-hook-state` to only show the delta (new entries since last check).

You should still manually read the journal at session start (`pair journal --from <prefix> --since 4h`) for initial context, but during the session the hook keeps you automatically up to date.

## Workflow examples

### Work on an existing issue

```bash
pair list -s open                                    # Pick an issue
pair show <id>                                       # Read full context
pair update <id> -s in_progress                      # Signal you're working on it
# ... do the work ...
pair comments add <id> "Done: implemented X, Y, Z"  # Summarize what was done
pair close <id>                                      # Close when complete
```

### Report issues discovered during work

```bash
pair create "Login form rejects valid emails" -t bug -p p1 -d "Emails with + are rejected"
pair create "Refactor auth middleware" -t task -p p2 -d "Extract token validation"
pair create "Add dark mode support" -t feature -p p3 -d "User requested in #42"
```

### Track sub-tasks of a larger effort

```bash
pair create "API redesign" -t epic
# Returns: epic-a1b2
pair create "Design new endpoints" -t task --parent epic-a1b2
pair create "Write migration script" -t task --parent epic-a1b2
pair create "Update client SDK" -t task --parent epic-a1b2
pair children epic-a1b2                              # View progress
```

### Cross-project communication (end-to-end)

Scenario: you work on **Project A**, cabled to a session in **Project B**. Project B changed an API endpoint.

```bash
# 1. Session start — check your cables
pair cable list                                        # → cabled to a session in project-b
pair journal --from project-b --since 4h               # → sees: "Changed GET /items to paginated response"

# 2. You warn the user
# "Project B switched /items to paginated responses — our client code needs updating."

# 3. You adapt the code, then log the cross-project impact
pair journal "Adapted API client for paginated /items endpoint" --tags api,project-b

# 4. Check the reply-to was added
pair journal --last 1                                  # → reply-to:project-b:42

# 5. Later, the agent on Project B reads Project A's journal and sees the confirmation
```
