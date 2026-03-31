---
name: beads-feature-workflow
description: Specialized workflow for implementing a new feature using Beads. Use when creating epics, managing subtasks with dependencies, and extracting the highest priority ready task for a specific feature.
---

# Beads Feature Workflow

This skill provides a structured approach to building features using `beads` (bd), focusing on setup, epic-driven decomposition, and priority execution.

## Project Setup

### Initialize Beads
Before managing features, you must initialize `beads` in your project root.
```bash
# Basic initialization (uses directory name as prefix)
bd init

# Initialize with a custom prefix
bd init myproject

# Non-interactive initialization (best for automation)
bd init --quiet --stealth
```

### Choose Your Role
The initialization process establishes how issues are routed based on your role:
- **Maintainer (`bd init --team`)**: Best for repo owners or teams with push access. Issues live in the project's `.beads/` directory.
- **Contributor (`bd init --contributor`)**: Best for fork contributors. Routes issues to a separate planning repo (`~/.beads-planning`) to avoid polluting upstream PRs.

## Feature Implementation Lifecycle

### 1. Create the Feature Epic
Start by defining the high-level feature as an epic.
```bash
bd create "Feature Name" -t epic -p 1
```
*Note the generated ID (e.g., `bd-a1b2`).*

### 2. Decompose into Tasks
Break the epic down into implementable tasks. Use `--parent` to link them automatically.
```bash
bd create "Task 1" -t task --parent bd-a1b2 -p 1 -d "Description for Task 1"
bd create "Task 2" -t task --parent bd-a1b2 -p 1 -d "Description for Task 2"
bd create "Task 3" -t task --parent bd-a1b2 -p 2 -d "Description for Task 3"
```

### 3. Define Dependencies
Order the work by adding blocking relationships.
```bash
bd dep add <task-2-id> <task-1-id> --type blocks
bd dep add <task-3-id> <task-2-id> --type blocks
```

### 4. Execute Tasks by Priority
To find the most important task to work on next for a specific feature:
1.  **Identify Epic ID**: `bd list --type epic --title-contains "Feature" --json`
2.  **Filter Ready Tasks**: Use the following logic to get the highest priority child task that is ready:
    ```bash
    # Concept: Get all ready work, filter by parent epic, and sort by priority (P0 first)
    bd ready --json | jq -r '.[] | select(.parent_id == "EPIC_ID") | [.priority, .id, .title] | @tsv' | sort -k1,1n | head -1
3.  **Claim and Work**: Once you have the task ID, claim it and start working:
    ```bash
    bd update <task-id> --claim
    ```
4. **Read the task details**: Always review the task description and any linked dependencies before starting work:
    ```bash
    bd show <task-id>
    ```
5. **Complete the Task**: Implement the task.After finishing the work, close the task with a completion message:
    ```bash
    bd close <task-id> --reason "Completed implementation of Task X for Feature Y"
    ```
6. **Continue**: After closing, check for the next ready task in the same epic and repeat the process until all tasks are completed.


# CLI Command Reference

## Basic Operations

### Check Status

```bash
# Check database path and server status
bd info --json

# Example output:
# {
#   "database_path": "/path/to/.beads/beads.db",
#   "issue_prefix": "bd",
#   "agent_mail_enabled": false
# }
```

### Find Work

```bash
# Find ready work (no blockers, not already claimed)
bd ready --json

# Atomically claim an issue from the ready queue
bd update <id> --claim --json               # Fails if already claimed

# Find stale issues (not updated recently)
bd stale --days 30 --json                    # Default: 30 days
bd stale --days 90 --status in_progress --json  # Find abandoned claims
bd stale --limit 20 --json                   # Limit results
```

## Issue Management

### Create Issues

```bash
# Basic creation
# IMPORTANT: Always quote titles and descriptions with double quotes
bd create "Issue title" -t bug|feature|task -p 0-4 -d "Description" --json

# Create with explicit ID (for parallel workers)
bd create "Issue title" --id worker1-100 -p 1 --json

# Create with labels (--labels or --label work)
bd create "Issue title" -t bug -p 1 -l bug,critical --json
bd create "Issue title" -t bug -p 1 --label bug,critical --json

# Examples with special characters (all require quoting):
bd create "Fix: auth doesn't validate tokens" -t bug -p 1 --json
bd create "Add support for OAuth 2.0" -d "Implement RFC 6749 (OAuth 2.0 spec)" --json
bd create "Implement auth" --spec-id "docs/specs/auth.md" --json

# Create multiple issues from markdown file
bd create -f feature-plan.md --json

# Create with description from file (avoids shell escaping issues)
bd create "Issue title" --body-file=description.md --json
bd create "Issue title" --body-file description.md -p 1 --json

# Read description from stdin
echo "Description text" | bd create "Issue title" --stdin --json
cat description.md | bd create "Issue title" --stdin -p 1 --json
# --body-file=- also works:
echo "Description text" | bd create "Issue title" --body-file=- --json

# Create epic with hierarchical child tasks
bd create "Auth System" -t epic -p 1 --json                     # Returns: bd-a3f8e9
bd create "Login UI" -p 1 --parent bd-a3f8e9 --json             # Auto-assigned: bd-a3f8e9.1
bd create "Backend validation" -p 1 --parent bd-a3f8e9 --json   # Auto-assigned: bd-a3f8e9.2
bd create "Tests" -p 1 --parent bd-a3f8e9 --json                # Auto-assigned: bd-a3f8e9.3

# Create and link discovered work (one command)
bd create "Found bug" -t bug -p 1 --deps discovered-from:<parent-id> --json

# Create with external reference (v0.9.2+)
bd create "Fix login" -t bug -p 1 --external-ref "gh-123" --json  # Short form
bd create "Fix login" -t bug -p 1 --external-ref "https://github.com/org/repo/issues/123" --json  # Full URL
bd create "Jira task" -t task -p 1 --external-ref "jira-PROJ-456" --json  # Custom prefix
```

### Update Issues

```bash
# Update one or more issues
bd update <id> [<id>...] --claim --json
bd update <id> [<id>...] --priority 1 --json
bd update <id> [<id>...] --spec-id "docs/specs/auth.md" --json

# Update external reference (v0.9.2+)
bd update <id> --external-ref "gh-456" --json           # Short form
bd update <id> --external-ref "jira-PROJ-789" --json    # Custom prefix

# Atomically claim an issue for work (prevents race conditions)
# Sets assignee to you and status to in_progress in one atomic operation
# Fails if already claimed (assignee is not empty)
bd update <id> --claim --json

# Edit issue fields in $EDITOR (HUMANS ONLY - not for agents)
# NOTE: This command is intentionally NOT exposed via the MCP server
# Agents should use 'bd update' with field-specific parameters instead
bd edit <id>                    # Edit description
bd edit <id> --title            # Edit title
bd edit <id> --design           # Edit design notes
bd edit <id> --notes            # Edit notes
bd edit <id> --acceptance       # Edit acceptance criteria
```

### Close/Reopen Issues

```bash
# Complete work (supports multiple IDs)
bd close <id> [<id>...] --reason "Done" --json

# Reopen closed issues (supports multiple IDs)
bd reopen <id> [<id>...] --reason "Reopening" --json
```

### View Issues

```bash
# Show dependency tree
bd dep tree <id>

# Get issue details (supports multiple IDs)
bd show <id> [<id>...] --json

# Show the currently active issue (in-progress, hooked, or last touched)
bd show --current
```

## Dependencies & Labels

### Dependencies

```bash
# Link discovered work (old way - two commands)
bd dep add <discovered-id> <parent-id> --type discovered-from

# Create and link in one command (new way - preferred)
bd create "Issue title" -t bug -p 1 --deps discovered-from:<parent-id> --json
```

### Labels

```bash
# Label management (supports multiple IDs)
bd label add <id> [<id>...] <label> --json
bd label remove <id> [<id>...] <label> --json
bd label list <id> --json
bd label list-all --json
```

### State (Labels as Cache)

For operational state tracking on role beads. Uses `<dimension>:<value>` label convention.
See [LABELS.md](LABELS.md#operational-state-pattern-labels-as-cache) for full pattern documentation.

```bash
# Query current state value
bd state <id> <dimension>                    # Output: value
bd state agent-abc patrol                  # Output: active
bd state --json agent-abc patrol           # {"issue_id": "...", "dimension": "patrol", "value": "active"}

# List all state dimensions on an issue
bd state list <id> --json
bd state list agent-abc                    # patrol: active, mode: normal, health: healthy

# Set state (creates event + updates label atomically)
bd set-state <id> <dimension>=<value> --reason "explanation" --json
bd set-state agent-abc patrol=muted --reason "Investigating stuck worker"
bd set-state agent-abc mode=degraded --reason "High error rate"
```

**Common dimensions:**
- `patrol`: active, muted, suspended
- `mode`: normal, degraded, maintenance
- `health`: healthy, warning, failing
- `status`: idle, working, blocked

**What `set-state` does:**
1. Creates event bead with reason (source of truth)
2. Removes old `<dimension>:*` label if exists
3. Adds new `<dimension>:<value>` label (cache)

## Filtering & Search

### Basic Filters

```bash
# Filter by status, priority, type
bd list --status open --priority 1 --json               # Status and priority
bd list --assignee alice --json                         # By assignee
bd list --type bug --json                               # By issue type
bd list --id bd-123,bd-456 --json                       # Specific IDs
bd list --spec "docs/specs/" --json                     # Spec prefix
```

### Label Filters

```bash
# Labels (AND: must have ALL)
bd list --label bug,critical --json

# Labels (OR: has ANY)
bd list --label-any frontend,backend --json
```

### Text Search

```bash
# Title search (substring)
bd list --title "auth" --json

# Pattern matching (case-insensitive substring)
bd list --title-contains "auth" --json                  # Search in title
bd list --desc-contains "implement" --json              # Search in description
bd list --notes-contains "TODO" --json                  # Search in notes

# Find beads issue by external reference
bd list --json | jq -r '.[] | select(.external_ref == "gh-123") | .id'
```

### Date Range Filters

```bash
# Date range filters (YYYY-MM-DD or RFC3339)
bd list --created-after 2024-01-01 --json               # Created after date
bd list --created-before 2024-12-31 --json              # Created before date
bd list --updated-after 2024-06-01 --json               # Updated after date
bd list --updated-before 2024-12-31 --json              # Updated before date
bd list --closed-after 2024-01-01 --json                # Closed after date
bd list --closed-before 2024-12-31 --json               # Closed before date
```

### Empty/Null Checks

```bash
# Empty/null checks
bd list --empty-description --json                      # Issues with no description
bd list --no-assignee --json                            # Unassigned issues
bd list --no-labels --json                              # Issues with no labels
```

### Priority Ranges

```bash
# Priority ranges
bd list --priority-min 0 --priority-max 1 --json        # P0 and P1 only
bd list --priority-min 2 --json                         # P2 and below
```

### Combine Filters

```bash
# Combine multiple filters
bd list --status open --priority 1 --label-any urgent,critical --no-assignee --json
```

## Global Flags

Global flags work with any bd command and must appear **before** the subcommand.

### Sandbox Mode

**Auto-detection (v0.21.1+):** bd automatically detects sandboxed environments and enables sandbox mode.

When detected, you'll see: `ℹ️  Sandbox detected, using direct mode`

**Manual override:**

```bash
# Explicitly enable sandbox mode
bd --sandbox <command>
```

**What it does:**
- Uses embedded database mode (no server needed)
- Disables auto-sync operations

**When to use:** Sandboxed environments where the Dolt server can't be controlled (permission restrictions), or when auto-detection doesn't trigger.

### Other Global Flags

```bash
# JSON output for programmatic use
bd --json <command>

# Disable auto-sync
bd --no-auto-flush <command>    # Disable auto-flush
bd --no-auto-import <command>   # Disable auto-import

# Custom database path
bd --db /path/to/.beads/beads.db <command>

# Custom actor for audit trail
bd --actor alice <command>
```

## Advanced Operations

### Cleanup

```bash
# Clean up closed issues (bulk deletion)
bd admin cleanup --force --json                                   # Delete ALL closed issues
bd admin cleanup --older-than 30 --force --json                   # Delete closed >30 days ago
bd admin cleanup --dry-run --json                                 # Preview what would be deleted
bd admin cleanup --older-than 90 --cascade --force --json         # Delete old + dependents
```

### Orphan Detection

Find issues referenced in git commits that were never closed:

```bash
# Basic usage - scan current repo
bd orphans

# Cross-repo: scan CODE repo's commits against external BEADS database
cd ~/my-code-repo
bd orphans --db ~/my-beads-repo/.beads/beads.db

# JSON output
bd orphans --json
```

**Use case**: When your beads database lives in a separate repository from your code, run `bd orphans` from the code repo and point `--db` to the external database. This scans commits in your current directory while checking issue status from the specified database.

### Duplicate Detection & Merging

```bash
# Find and merge duplicate issues
bd duplicates                                          # Show all duplicates
bd duplicates --auto-merge                             # Automatically merge all
bd duplicates --dry-run                                # Preview merge operations

# Merge specific duplicate issues
bd merge <source-id...> --into <target-id> --json      # Consolidate duplicates
bd merge bd-42 bd-43 --into bd-41 --dry-run            # Preview merge
```


### Rename Prefix

```bash
# Rename issue prefix (e.g., from 'knowledge-work-' to 'kw-')
bd rename-prefix kw- --dry-run  # Preview changes
bd rename-prefix kw- --json     # Apply rename
```

### Reset

Remove all local beads data and return to uninitialized state.

```bash
# Preview what would be removed (dry-run)
bd admin reset

# Actually perform the reset
bd admin reset --force
```

**What gets removed:**
- `.beads/` directory (database, config)
- Git hooks installed by bd
- Merge driver configuration
- Sync branch worktrees (`.git/beads-worktrees/`)

**What does NOT get removed:**
- Remote sync branch (if configured)
- Remote Dolt repository data
- Historical git commits


## Database Management

### Export / Backup / Bootstrap

```bash
# Export issues to issue JSONL
bd export -o issues.jsonl

# Dolt-native backup (preserves full commit history)
bd backup init /path/to/backup    # Register a backup destination
bd backup sync                   # Push to backup destination
bd backup restore [path]         # Restore from a backup
bd backup remove                 # Remove backup destination
bd backup status                 # Show backup status

# Bootstrap a new database from an issue export
bd init --from-jsonl                            # Reads .beads/issues.jsonl

# Configure orphan handling for pulls and bootstrapping
bd config set import.orphan_handling "resurrect"
bd dolt pull  # Respects import.orphan_handling setting
```

**Orphan handling modes** (apply to `bd dolt pull` and `bd init --from-jsonl`):

- **`allow` (default)** - Import orphaned children without parent validation. Most permissive, ensures no data loss even if hierarchy is temporarily broken.
- **`resurrect`** - Search for deleted parents and recreate them as tombstones (Status=Closed, Priority=4). Preserves hierarchy with minimal data.
- **`skip`** - Skip orphaned children with warning. Partial import succeeds but some issues are excluded.
- **`strict`** - Fail immediately if a child's parent is missing. Use when database integrity is critical.

See [CONFIG.md](CONFIG.md#example-import-orphan-handling) and [TROUBLESHOOTING.md](TROUBLESHOOTING.md#import-fails-with-missing-parent-errors) for more details.

### Sync Operations

```bash
# Manual sync (push changes to remote)
bd dolt push

# Pull changes from remote
bd dolt pull

# What these do:
# bd dolt push - Commit pending changes to Dolt and push to remote
# bd dolt pull - Pull from remote and merge any updates
```

## Issue Statuses

### Built-in Statuses

- `open` - Ready to be worked on
- `in_progress` - Currently being worked on
- `blocked` - Cannot proceed (waiting on dependencies)
- `deferred` - Deliberately put on ice for later
- `closed` - Work completed
- `tombstone` - Deleted issue (suppresses resurrections)
- `pinned` - Stays open indefinitely (used for hooks, anchors)

**Note:** The `pinned` status is used by orchestrators for hook management and persistent work items that should never be auto-closed or cleaned up.

**Categories:**

| Category | `bd ready` | Default `bd list` | Icon | Description |
|----------|-----------|-------------------|------|-------------|
| `active` | ✓ included | ✓ included | ○ | Ready for work (like `open`) |
| `wip` | ✗ excluded | ✓ included | ◐ | Work-in-progress (like `in_progress`) |
| `done` | ✗ excluded | ✗ excluded | ✓ | Terminal state (like `closed`) |
| `frozen` | ✗ excluded | ✗ excluded | ❄ | On hold (like `deferred`) |
| *(none)* | ✗ excluded | ✓ included | ◇ | No category — backward compatible default |

**List all statuses** (built-in and custom):

```bash
bd statuses          # Human-readable table
bd statuses --json   # JSON output for programmatic use
```

**Rules:**
- Status names must match `[a-z][a-z0-9_-]*` (lowercase, letter-first)
- Cannot collide with built-in status names (case-insensitive)
- Maximum 50 custom statuses
- All custom statuses work with `bd list --status <name>`

## Priorities

- `0` - Critical (security, data loss, broken builds)
- `1` - High (major features, important bugs)
- `2` - Medium (nice-to-have features, minor bugs)
- `3` - Low (polish, optimization)
- `4` - Backlog (future ideas)

## Dependency Types

- `blocks` - Hard dependency (issue X blocks issue Y)
- `related` - Soft relationship (issues are connected)
- `parent-child` - Epic/subtask relationship
- `discovered-from` - Track issues discovered during work

Only `blocks` dependencies affect the ready work queue.

**Note:** When creating an issue with a `discovered-from` dependency, the new issue automatically inherits the parent's `source_repo` field.

## External References

The `--external-ref` flag (v0.9.2+) links beads issues to external trackers:

- Supports short form (`gh-123`) or full URL (`https://github.com/...`)
- Portable via Dolt - survives sync across machines
- Custom prefixes work for any tracker (`jira-PROJ-456`, `linear-789`)

## Output Formats

### JSON Output (Recommended for Agents)

Always use `--json` flag for programmatic use:

```bash
# Single issue
bd show bd-42 --json

# List of issues
bd ready --json

# Operation result
bd create "Issue" -p 1 --json
```

### Human-Readable Output

Default output without `--json`:

```bash
bd ready
# ○ bd-42 [P1] [bug] - Fix authentication bug
# ○ bd-43 [P2] [feature] - Add user settings page
```

**Dependency visibility:** When issues have blocking dependencies, they appear inline:

```bash
bd list --parent epic-123
# ○ bd-123.1 [P1] [task] - Design API (blocks: bd-123.2, bd-123.3)
# ○ bd-123.2 [P1] [task] - Implement endpoints (blocked by: bd-123.1, blocks: bd-123.3)
# ○ bd-123.3 [P1] [task] - Add tests (blocked by: bd-123.1, bd-123.2)
```

This makes blocking relationships visible without running `bd show` on each issue.

## Common Patterns for AI Agents

### Claim and Complete Work

```bash
# 1. Find available work
bd ready --json

# 2. Claim issue
bd update bd-42 --claim --json

# 3. Work on it...

# 4. Close when done
bd close bd-42 --reason "Implemented and tested" --json
```

### Discover and Link Work

```bash
# While working on bd-100, discover a bug

# Old way (two commands):
bd create "Found auth bug" -t bug -p 1 --json  # Returns bd-101
bd dep add bd-101 bd-100 --type discovered-from

# New way (one command):
bd create "Found auth bug" -t bug -p 1 --deps discovered-from:bd-100 --json
```


### Session Workflow

```bash
# Start of session
bd ready --json  # Find work

# During session
bd create "..." -p 1 --json
bd update bd-42 --claim --json
# ... work ...

# End of session (IMPORTANT!)
bd dolt push  # Force immediate sync, bypass debounce
```

**ALWAYS run `bd dolt push` at end of agent sessions** to ensure changes are committed/pushed immediately.
