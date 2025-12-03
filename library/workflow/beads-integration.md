# Beads Work Tracking

**Beads is the memory system for coding agents.** All work MUST be tracked in beads, not ephemeral markdown plans.

## Why Beads

- **Survives context window exhaustion** - Issues persist across sessions
- **Survives session interruptions** - Work state is always recoverable
- **Syncs across machines** - Git-backed distributed database
- **Provides audit trail** - Full history of work and decisions
- **Enables coordination** - Multiple agents can work on same project
- **Prevents lost work** - Discovered issues are automatically captured

## Session Startup

Every session **MUST** begin with orientation:

```bash
# 1. See what's ready to work on
bd ready

# 2. Claim work
bd update <issue-id> --status in_progress

# 3. Review issue context
bd show <issue-id>
bd dep tree <issue-id>  # See dependencies
```

## During Work

### Discovering New Work

As you work, you'll find bugs, TODOs, and related tasks. **File them immediately:**

```bash
# Create the issue
bd create "Found null pointer in auth flow" -t bug -p 1

# Link it to your current work
bd dep add <new-id> <current-id> --type discovered-from
```

### Tracking Progress

Update status as you go:

```bash
bd update <issue-id> --status in_progress
bd update <issue-id> --notes "Completed API layer, starting tests"
```

## Session End

Before ending a session:

```bash
# 1. Close completed work
bd close <issue-id> --reason "Completed: implemented user authentication"

# 2. Update in-progress work
bd update <issue-id> --notes "Stopped at: database migration pending"

# 3. File issues for remaining work
bd create "Remaining: add rate limiting" -t task

# 4. Sync to git
bd sync
```

## Task Decomposition

### When to Decompose

- Task is too large for a single session
- Task has multiple independent pieces
- Task exceeds context window capacity

### How to Decompose

```bash
# Create epic for the overall feature
bd create "User authentication system" -t epic

# Create child tasks
bd create "Add password hashing" -t task
bd create "Create login endpoint" -t task
bd create "Add session management" -t task

# Set dependencies (login needs password hashing)
bd dep add <login-id> <hashing-id> --type blocks

# Link children to epic
bd dep add <hashing-id> <epic-id> --type parent-child
bd dep add <login-id> <epic-id> --type parent-child
```

### Asking for Guidance

If unsure how to decompose:

1. Create a draft breakdown
2. Show it to the operator
3. Ask which task to tackle first

## Issue Types

| Type | Use For |
|------|---------|
| `bug` | Something broken that needs fixing |
| `feature` | New functionality |
| `task` | Implementation work |
| `epic` | Large feature with child tasks |
| `chore` | Maintenance, refactoring, cleanup |

## Priority Levels

| Level | Meaning |
|-------|---------|
| 0 | Critical - drop everything |
| 1 | High - do soon |
| 2 | Medium - normal priority (default) |
| 3 | Low - when time permits |
| 4 | Backlog - someday/maybe |

## Dependency Types

| Type | Meaning |
|------|---------|
| `blocks` | Hard dependency - must complete first |
| `related` | Soft link - related but independent |
| `parent-child` | Epic/subtask relationship |
| `discovered-from` | Found during work on another issue |

## Commands Reference

```bash
bd init              # Initialize beads in project
bd ready             # Show unblocked work
bd list              # List all issues
bd show <id>         # Show issue details
bd create "title"    # Create issue
bd update <id>       # Update issue
bd close <id>        # Close issue
bd dep add/rm        # Manage dependencies
bd dep tree <id>     # Visualize dependencies
bd sync              # Sync with git
```
