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
- If the project has associated projects (`pair associations`), follow the cross-project reading protocol from AGENTS.md
```

## How to work with PaiR

**These are not suggestions — they are part of your workflow.** The human using this project expects you to track your work through PaiR, just like you would use git for version control.

### Before starting any task

- Run `pair list -s open` to see existing issues — your task may already be tracked
- If working on a specific issue, run `pair update <id> -s in_progress` before writing any code
- Run `pair show <id>` to read the full context (description, comments, acceptance criteria)
- **Check associated projects:** run `pair associations` — if associations exist, read their recent journal: `pair journal --from <prefix> --since 4h` for each one. Adapt your plan if breaking changes or related work is detected.

### While working

- **Found a bug?** → `pair create "Bug title" -t bug -p p1 -d "Description"`
- **Spotted a TODO or limitation?** → `pair create "Title" -t task -d "Details"`
- **Hit a blocker?** → `pair update <id> -s blocked` and `pair dep add <id> <blocker-id>`
- **Made progress worth noting?** → `pair comments add <id> "What was done"`

### Session journal — leave a trail

The journal is not just an audit log — it is the **communication channel between agents working on linked projects.** Write journal entries at key moments so that other sessions (on associated projects) can understand what happened here without reading your code or commits.

**When to write:**

| Moment | Example |
|--------|---------|
| Starting a significant task | `pair journal "Starting: implement POST /foo endpoint" --tags task` |
| Key technical decision | `pair journal "Decision: use WebSocket instead of polling for sync" --tags decision` |
| Completing a unit of work | `pair journal "Done: POST /foo endpoint with validation and tests" --tags progress` |
| Blocked or unexpected issue | `pair journal "Blocked: dependency X v3 incompatible with our auth layer" --tags blocker` |

**Standard tags:** `task`, `decision`, `progress`, `blocker` — these enable filtering and aggregation.

**Default bias: write.** When in doubt about whether a journal entry is worth writing, write it. The cost of an extra entry is near zero; the cost of a missing one is a blind spot for the associated project. Only skip entries that are clearly internal with no cross-session value (formatting, imports, typos). Ask yourself: *"would an agent on an associated project — or a future session on this project — benefit from knowing this?"* If the answer isn't a clear "no", write it.

### Cross-project awareness — stay in sync

If this project has associated projects, **don't just check the journal at session start and forget about it.** The other session may be working in parallel — **you MUST re-read at these specific moments:**

| When | What to do | Why |
|------|-----------|-----|
| **Before starting any new task** | `pair journal --from <prefix> --since 2h` | The other session may have changed something that affects your plan |
| **Before editing a shared interface** (API, schema, config, types) | Re-read the other project's journal | Avoid implementing against an outdated contract |
| **Before every commit** | `pair journal --from <prefix> --since 1h` | Catch last-minute changes before locking in your work |
| **Before closing an issue** | Final check for all associated prefixes | Don't close with undetected conflicts |

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

- **If associated projects exist:** run `pair journal --from <prefix> --since 2h` for a final coherence check before closing
- Add a summary comment: `pair comments add <id> "Done: what was implemented"`
- Close the issue: `pair close <id>`
- Reference the issue ID in your commit message

### Always

- **Never ignore `.pair/`** — it is the project's issue tracker, not a temp folder
- **Commit `.pair/issues.jsonl` separately** from code changes (e.g., `chore(pair): update issues`)
- **Check for related issues** before creating duplicates: `pair search "keyword"`

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

### `list` — List issues

```bash
pair list                        # Open issues (default)
pair list -s open                # Filter by status
pair list -s in_progress
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

```bash
pair update <id> -s in_progress
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

### `dep` — Manage dependencies

```bash
pair dep add <id> <blocker-id>              # blocker blocks id
pair dep add <id> <blocker-id> --type blocks
pair dep remove <id> <other-id>
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
pair journal --today                    # Today's entries
pair journal --last 10                  # Last 10 entries
pair journal --tag api                  # Filter by tag
pair journal --since 2h                 # Since 2 hours ago
pair journal --from scripteasy-v4      # Read another project's journal (read-only)
pair journal --export                   # Export to .pair/journal.jsonl
```

The journal is **auto-populated** on every `pair create`, `pair close`, and `pair comments add`. Manual entries are for decisions, milestones, and notes.

After reading another project's journal (`--from`), the next manual write is automatically tagged with `reply-to:<project>:<id>` to trace cross-project exchanges.

### `catalog` — Global project catalog

```bash
pair catalog                           # List all known projects (prefix, path, name)
```

Projects are auto-registered on any CLI use. The catalog lives in `~/Library/Application Support/com.pair.app/catalog.db`.

### `associate` / `dissociate` / `associations` — Project associations

```bash
pair associate project-a project-b --reason "Project B consumes Project A's API"
pair dissociate project-a project-b
pair associations                      # List active associations
```

Associations are bidirectional. They enable cross-project journal reading and visual indicators in the workspace.

## Cross-project communication protocol

### Why this exists

When two projects are associated, it means they depend on each other — a change in one can break or require adaptation in the other. But each project has its own AI agent, working in its own session, with no direct communication channel. The journal is that channel: **an asynchronous message bus between agents working on linked projects.**

Without this protocol, an agent changes an API contract on Project A, and the agent on Project B has no idea until something breaks. With it, the agent on Project B reads the journal on session start, sees the change, and can adapt proactively.

### What happens automatically
- Every `pair create`, `pair close`, and `pair comments add` is logged in the project's journal — no extra action needed.
- After reading another project's journal (`pair journal --from <prefix>`), your next manual journal write is auto-tagged with `reply-to:<prefix>:<id>`. This creates a traceable conversation thread across projects.
- **The PreToolUse hook automatically injects new journal entries** (local + associated projects) into your context before every Edit/Write — you don't need to manually re-read during the session. See "Automatic journal context injection" in the hooks section.

### Session start protocol — MANDATORY

**This is the FIRST thing you do when you start a session. No exceptions. Do not skip any step. Do not start working on anything before completing this protocol.**

**Step 1 — Read the local journal** (always, unconditionally):

```bash
pair journal --last 10
```

This tells you what happened in recent sessions on this project: decisions made, tasks completed, blockers hit, direction changes. Without this, you are working blind — you might redo work, contradict a decision, or miss critical context.

**Step 2 — Check for associated projects** (always, unconditionally):

```bash
pair associations
```

**Step 3 — If associations exist, read each associated project's journal** (mandatory for each one):

```bash
pair journal --from <prefix> --since 4h    # Run this for EVERY associated project
```

Do not skip this. Do not defer it. The other project's agent may have changed an API, a schema, a shared contract — if you don't read their journal now, you will build on stale assumptions.

**Step 4 — Act on what you read:**

- **Breaking change detected** (API contract, shared schema, data format) → **warn the user immediately**, before starting any work. Example: "Project B changed the `/items` endpoint response format 2h ago — this may affect our client code."
- **Related work in progress** (feature that touches a shared boundary) → factor it into your plan. Don't duplicate effort or make conflicting changes.
- **No relevant activity** → proceed normally. Don't mention it.

**This protocol is not a one-time action.** You must re-read journals at key moments during the session — see "Cross-project awareness — stay in sync" above.

A detailed step-by-step procedure with classification rules and report format is available in `.pair/commands/cross-project.md`.

### When to write a manual journal entry

**General session entries** — use the standard tags (`task`, `decision`, `progress`, `blocker`) as described in the "Session journal" section above. These entries track your work for cross-session awareness: starting a task, making a decision, finishing a unit of work, or hitting a blocker.

**Cross-project entries** — when your work directly affects an associated project, tag with the project prefix:

| Trigger | Example |
|---------|---------|
| You change a shared interface (API, schema, config format) | `pair journal "Changed GET /items response: added pagination wrapper" --tags api,project-b` |
| You discover a bug that originates in the other project | `pair journal "Auth token refresh fails — the issue is in Project B's token endpoint, not here" --tags bug,project-b` |
| You make an architecture decision that constrains the other project | `pair journal "Switching to WebSocket for real-time sync — REST polling deprecated" --tags architecture,project-b` |
| You complete work that the other project was waiting on | `pair journal "Export endpoint is live — Project B can start integration" --tags api,project-b` |

**Only skip journal entries for:**
- Things already captured by auto-logging (create/close/comment) — no need to duplicate
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
- **Never** run `pair associate` or `pair dissociate` without the user's explicit request — associations are deliberate decisions.
- **Never** write in another project's journal — cross-project access is **read-only**. You write in your own journal, tagged with the other project's prefix.
- **Never** spam the journal — if it doesn't cross the project boundary, it doesn't belong here.

## Claude Code hooks configuration

To connect Claude Code to PaiR, add hooks to `~/.claude/settings.json` (global) or `.claude/settings.json` (per-project):

```json
{
  "hooks": {
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

This enables real-time AI activity tracking in the PaiR app: per-project activity LED,
AI events panel, and session focus (switch to the editor window where AI is working).

### Automatic journal context injection

The `PreToolUse` hook automatically injects recent journal entries into Claude's context before every **Edit** or **Write** tool call. This means:

- **Local journal**: entries from the last hour (or since last check) are shown
- **Associated projects**: new entries since last check are shown
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

Scenario: you work on **Project A** (associated with **Project B**). Project B changed an API endpoint.

```bash
# 1. Session start — check associated projects
pair associations                                      # → project-b is linked
pair journal --from project-b --since 4h               # → sees: "Changed GET /items to paginated response"

# 2. You warn the user
# "Project B switched /items to paginated responses — our client code needs updating."

# 3. You adapt the code, then log the cross-project impact
pair journal "Adapted API client for paginated /items endpoint" --tags api,project-b

# 4. Check the reply-to was added
pair journal --last 1                                  # → reply-to:project-b:42

# 5. Later, the agent on Project B reads Project A's journal and sees the confirmation
```
