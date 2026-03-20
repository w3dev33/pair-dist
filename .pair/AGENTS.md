# AGENTS.md — PaiR

This project uses `pair` for issue tracking. Issues are stored locally in SQLite (`.pair/pair.db`).
The PaiR app monitors AI activity in real-time — every CLI call you make is reflected instantly in the UI.

## First-time setup — Hooks

**On your first session in this project, check if PaiR notification hooks are installed.** If they are not, propose to the user to set them up. Hooks enable real-time notifications in PaiR: sound alerts, visual indicators, and AI activity tracking.

To check: look for `pair notify` entries in `~/.claude/settings.json` (global) or `.claude/settings.json` (project-level) under `"hooks"`.

If missing, offer to install them by adding the hooks configuration listed in the "Claude Code hooks configuration" section below. This is a one-time setup — once installed, every action you take will be visible in the PaiR app.

## How to work with PaiR

**These are not suggestions — they are part of your workflow.** The human using this project expects you to track your work through PaiR, just like you would use git for version control.

### Before starting any task

- Run `pair list -s open` to see existing issues — your task may already be tracked
- If working on a specific issue, run `pair update <id> -s in_progress` before writing any code
- Run `pair show <id>` to read the full context (description, comments, acceptance criteria)

### While working

- **Found a bug?** → `pair create "Bug title" -t bug -p p1 -d "Description"`
- **Spotted a TODO or limitation?** → `pair create "Title" -t task -d "Details"`
- **Hit a blocker?** → `pair update <id> -s blocked` and `pair dep add <id> <blocker-id>`
- **Made progress worth noting?** → `pair comments add <id> "What was done"`

### When finishing work

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

Syncs issues from GitHub or GitLab into PaiR (one-way: external → PaiR).

**Configuration** (`.pair/config.yaml`):
```yaml
sync:
  provider: github          # or "gitlab"
  repo: owner/repo          # Optional: override git remote auto-detection
  token_env: GITHUB_TOKEN   # or GITLAB_TOKEN
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
