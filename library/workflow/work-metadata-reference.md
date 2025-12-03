# Work Metadata Reference

## Examples

### Example 1: Backend Feature Work

```toml
[project]
name = "customer-api"
github_url = "https://github.com/company/customer-api"

[work]
asana_task = "https://app.asana.com/0/1234567890/9876543210"
github_project = "https://github.com/orgs/company/projects/15"
started_at = "2025-11-01T10:30:00Z"
branch = "feature/jwt-authentication"

[tracking]
default_assignee = "jsmith"
default_labels = ["backend", "authentication", "api"]
```

### Example 2: Bugfix Work

```toml
[project]
name = "customer-api"
github_url = "https://github.com/company/customer-api"

[work]
asana_task = "https://app.asana.com/0/1234567890/1111111111"
github_project = "https://github.com/orgs/company/projects/16"
started_at = "2025-11-02T14:15:00Z"
branch = "fix/token-expiry-calculation"

[tracking]
default_assignee = "jsmith"
default_labels = ["backend", "bugfix", "p0"]
default_milestone = "2025-Q4"
```

### Example 3: Minimal Format

```toml
[project]
name = "customer-api"
github_url = "https://github.com/company/customer-api"

[work]
asana_task = "https://app.asana.com/0/1234567890/9876543210"
github_project = 15

[tracking]
default_assignee = "jsmith"
default_labels = []
```

## Migration from .agent-metadata.toml

### Old Format (agent-metadata.toml)
```toml
[project]
name = "project-name"
github_url = "https://github.com/owner/repo"

[tracking]
github_project = "Project Board Name"
asana_project = "https://app.asana.com/0/projectid/board"
default_assignee = "github-username"
default_tags = ["tag1", "tag2"]
```

### New Format (work-metadata.toml)
```toml
[project]
name = "project-name"
github_url = "https://github.com/owner/repo"

[work]
asana_task = "https://app.asana.com/0/{project_gid}/{task_gid}"
github_project = "https://github.com/orgs/org/projects/{number}"

[tracking]
default_assignee = "github-username"
default_labels = ["tag1", "tag2"]
```

**Key differences**:
- Renamed file: `.agent-metadata.toml` → `.work-metadata.toml`
- New `[work]` section: Specific task/project for current work
- `tracking.github_project`: Project name → `work.github_project`: Project URL
- `tracking.asana_project`: Asana project board → `work.asana_task`: Specific task
- `default_tags` → `default_labels` (consistent with GitHub terminology)

## Best Practices

### Keep .work-metadata.toml Out of Version Control
```text
# .gitignore
.work-metadata.toml

This file is worktree-specific, not repository-wide.
```

### Use Tools for Updates (When Using Automated Tracking)
```text
# Let work-start and tracking agents maintain this file
work-start --asana <task> --github-project <number>

Automated tools keep the file consistent.
```

### Create Fresh Metadata Per Worktree
```text
# Each worktree tracks different work
work-start --asana <task> --github-project <number>

Different work contexts require separate metadata.
```

### Use Real Values
```text
# Always use actual URLs/IDs
[work]
asana_task = "https://app.asana.com/0/123/456"  # Real task
github_project = 15                               # Real project

Metadata must contain valid, resolvable references.
```

## .gitignore Configuration

**Always ignore `.work-metadata.toml`**:

```gitignore
# Work tracking metadata (worktree-specific)
.work-metadata.toml
```

## The work-start Tool

**Location**: `~/.local/bin/work-start` (installed with plugin)

**Purpose**: Create `.work-metadata.toml` files for work tracking

**Usage**:
```bash
# Show help
work-start --help

# Interactive mode (prompts for all values)
work-start --interactive

# With explicit values
work-start \
  --asana-ticket <url-or-gid> \
  --github-project <url-or-number> \
  --assignee <username> \
  --labels "label1,label2"

# Create new GitHub Project
work-start \
  --asana-ticket <url> \
  --create-project "Project Name"

# Custom output path
work-start --interactive --output /path/to/.work-metadata.toml
```

**Requirements**:
- Must be run from within a git repository
- `gh` CLI must be installed and authenticated
- `uv` must be installed (uses inline script dependencies)

**What it does**:
1. Validates git repository context
2. Extracts project name and GitHub URL from git remote
3. Accepts Asana task (URL or GID)
4. Accepts or creates GitHub Project (URL, number, or create new)
5. Optionally accepts default assignee and labels
6. Generates properly formatted `.work-metadata.toml`
7. Displays created file contents

## Summary

**`.work-metadata.toml`** provides structured context for automated work tracking:

- **Lives in worktree root** - Per-worktree context
- **Minimal information** - Just URLs/IDs needed for automation
- **Created by `work-start` tool** - Not manually written
- **Updated by agents** - Tools maintain, humans rarely touch
- **Links systems** - Bridge between code and tracking (Asana + GitHub)

**Workflow**: User runs `work-start` → Tool creates metadata → Agent reads metadata for all tracking operations → Work completes → Worktree removed with metadata
