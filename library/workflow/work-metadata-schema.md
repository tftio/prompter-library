# Work Metadata Schema

## Overview

**`.work-metadata.toml`** is the single source of truth linking a worktree to external work tracking systems (Asana + GitHub Projects).

**Location**: Root of each worktree (not committed, worktree-specific)

**Purpose**:
- Bridge between local development and work tracking systems
- Enable automated tools/agents to update correct Asana tasks and GitHub projects
- Provide context for what work is being done in this worktree

## File Schema

### Minimal Required Format

```toml
# Repository information (static per repo)
[project]
name = "my-api"
github_url = "https://github.com/org/my-api"

# Current work context (specific to this worktree)
[work]
asana_task = "https://app.asana.com/0/1234567890/9876543210"
github_project = "https://github.com/orgs/org/projects/42"

# Tracking defaults (static per repo)
[tracking]
default_assignee = "github-username"
default_labels = ["backend", "api"]
```

### Field Definitions

#### `[project]` Section
**Purpose**: Static repository information

- `name` (required) - Short repository/project name
- `github_url` (required) - Full GitHub repository URL
- `description` (optional) - Brief project description

#### `[work]` Section
**Purpose**: Current work context for this worktree

- `asana_task` (required) - Full Asana task URL
  - Format: `https://app.asana.com/0/{project_gid}/{task_gid}`
  - Or task GID only: `"1234567890123456"`
- `github_project` (required) - GitHub Project URL or number
  - Format: `https://github.com/orgs/{org}/projects/{number}` (org projects)
  - Format: `https://github.com/users/{user}/projects/{number}` (user projects)
  - Format: `https://github.com/{owner}/{repo}/projects/{number}` (repo projects)
  - Or project number only: `42`

**Optional fields**:
- `started_at` (ISO 8601 timestamp) - When work began in this worktree
- `branch` (string) - Associated git branch (for reference)

#### `[tracking]` Section
**Purpose**: Default values for issue/PR creation

- `default_assignee` (string) - GitHub username for auto-assignment
- `default_labels` (array) - Default labels for issues/PRs
- `default_milestone` (string, optional) - Default milestone name
