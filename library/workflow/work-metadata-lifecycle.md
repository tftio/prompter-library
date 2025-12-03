# Work Metadata Lifecycle

## File Lifecycle

### Creation

**When**: Starting work on an Asana task in a new worktree

**How**: Use the `work-start` CLI tool to create the file:

```bash
# Interactive mode (recommended)
work-start --interactive

# With explicit values
work-start \
  --asana-ticket "https://app.asana.com/0/123/456" \
  --github-project 42 \
  --assignee jsmith \
  --labels "backend,auth"

# Create new GitHub Project
work-start \
  --asana-ticket "https://app.asana.com/0/123/456" \
  --create-project "Authentication Feature"
```

**What the tool does**:
1. Extracts repo info from git remote
2. Creates `.work-metadata.toml` with provided values
3. Optionally creates GitHub Project via `gh` CLI
4. Validates and displays the created file

### Updates

**When**: Tools/agents need to interact with work tracking systems

**How**:
- Read `.work-metadata.toml` to determine which systems to update
- Parse URLs to extract IDs for API calls
- Update Asana tasks and GitHub issues automatically

**Use tools exclusively** - Tools/agents maintain this file

### Deletion

**When**: Work complete, worktree no longer needed

**How**: Remove worktree (file goes with it)

## Worktree Integration

### Why Per-Worktree?

Each worktree represents discrete work:
- One Asana task per worktree
- One GitHub Project per worktree
- Parallel work = parallel worktrees with separate metadata

### Example Structure

```text
~/Projects/my-api/                    # Main repository
├── main/                             # Main branch worktree
│   └── .work-metadata.toml           # (optional, for main branch work)
├── feature-auth/                     # Feature worktree
│   ├── .work-metadata.toml           # Links to auth Asana task + GH project
│   └── src/...
└── fix-database-bug/                 # Bugfix worktree
    ├── .work-metadata.toml           # Links to bugfix Asana task + GH project
    └── src/...
```

Each worktree is self-contained with its own work context.

## Integration with Issue Tracking Workflow

### Checkpoint 1: Work Start
```bash
# Tool reads Asana task
# Tool creates/finds GitHub Project
# Tool creates .work-metadata.toml
# Tool updates Asana task → "In Progress"
```

### Checkpoint 2: Issues Created
```bash
# Tool reads .work-metadata.toml
# Tool creates GitHub issues in specified project
# Tool comments on Asana task with issue links
```

### Checkpoint 3: PRs Opened
```bash
# Tool reads .work-metadata.toml
# Tool creates PR with proper references
# Tool comments on Asana task with PR links
```

### Checkpoint 4: Work Complete
```bash
# Tool reads .work-metadata.toml
# Tool verifies all GitHub issues closed
# Tool updates Asana task → "Complete" with summary
# Worktree can be removed (metadata goes with it)
```
