# AGENTS.md — PaiR

This project uses `pair` for issue tracking. Issues are stored locally in SQLite (`.pair/pair.db`).

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
pair list -t bug                 # Filter by type (task, bug, feature, epic)
pair list -p p0                  # Filter by priority (p0, p1, p2, p3)
pair list --assignee "Name"      # Filter by assignee
pair list --limit 10             # Limit results
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
| `--type` | `-t` | Issue type: `task`, `bug`, `feature`, `epic` |
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

To connect Claude Code to PaiR, add hooks to `.claude/settings.json` in your project:

```json
{
  "hooks": {
    "PreToolUse": [
      { "matcher": "", "hooks": [{ "type": "command", "command": "pair notify --hook PreToolUse" }] }
    ],
    "PostToolUse": [
      { "matcher": "", "hooks": [{ "type": "command", "command": "pair notify --hook PostToolUse" }] }
    ],
    "Notification": [
      { "matcher": "", "hooks": [{ "type": "command", "command": "pair notify --hook Notification" }] }
    ],
    "Stop": [
      { "matcher": "", "hooks": [{ "type": "command", "command": "pair notify --hook Stop" }] }
    ],
    "SubagentStop": [
      { "matcher": "", "hooks": [{ "type": "command", "command": "pair notify --hook SubagentStop" }] }
    ],
    "PreCompact": [
      { "matcher": "", "hooks": [{ "type": "command", "command": "pair notify --hook PreCompact" }] }
    ]
  }
}
```

**Important:** Claude Code hooks run in a minimal shell where `PATH` may be incomplete.
Always use the absolute path to the `pair` binary in hook commands (run `which pair` to find it):

- **macOS:** `/usr/local/bin/pair` (or your symlink location)
- **Linux:** `/usr/bin/pair`

This enables real-time AI activity tracking in the PaiR app: per-project activity LED,
AI events panel, and session focus (switch to the editor window where AI is working).

## Workflow: Work on an issue

```bash
pair list -s open              # Pick an issue
pair update <id> -s in_progress
# ... do the work ...
pair comments add <id> "Done: implemented the feature"
pair close <id>
```
