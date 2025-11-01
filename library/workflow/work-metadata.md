# Work Metadata Formalism

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

**Never manually edit** - Tools/agents maintain this file

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

```
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

## Tool Integration

### Reading Metadata

**Discovery process**:
1. Check for `.work-metadata.toml` in current directory
2. If found, parse and extract values
3. If missing or incomplete, fall back to prompting user
4. Optionally offer to create/update file

**Example (pseudocode)**:
```python
import tomllib
from pathlib import Path

def get_work_context():
    metadata_file = Path(".work-metadata.toml")

    if not metadata_file.exists():
        return prompt_user_for_context()

    with open(metadata_file, "rb") as f:
        data = tomllib.load(f)

    return {
        "asana_task": data["work"]["asana_task"],
        "github_project": data["work"]["github_project"],
        "assignee": data["tracking"].get("default_assignee"),
        "labels": data["tracking"].get("default_labels", [])
    }
```

### Writing Metadata

**Automated creation/updates only** - Tools maintain consistency

**Example (pseudocode)**:
```python
import tomli_w
from datetime import datetime, timezone

def create_work_metadata(
    project_name: str,
    github_url: str,
    asana_task: str,
    github_project: str,
    assignee: str = None,
    labels: list[str] = None
):
    metadata = {
        "project": {
            "name": project_name,
            "github_url": github_url,
        },
        "work": {
            "asana_task": asana_task,
            "github_project": github_project,
            "started_at": datetime.now(timezone.utc).isoformat(),
        },
        "tracking": {
            "default_assignee": assignee,
            "default_labels": labels or [],
        }
    }

    with open(".work-metadata.toml", "wb") as f:
        tomli_w.dump(metadata, f)
```

### URL Parsing

**Extract IDs for API calls**:

```python
import re

def parse_asana_task_url(url: str) -> str:
    """Extract task GID from Asana URL or return as-is if already GID."""
    if url.startswith("https://app.asana.com"):
        # URL format: https://app.asana.com/0/{project_gid}/{task_gid}
        match = re.search(r"/(\d+)/?$", url)
        return match.group(1) if match else None
    return url  # Assume already a GID

def parse_github_project_url(url: str) -> tuple[str, str, int]:
    """Extract owner, type, and project number from GitHub Project URL."""
    if isinstance(url, int):
        return None, None, url  # Just a number

    # Org: https://github.com/orgs/{org}/projects/{number}
    org_match = re.match(r"https://github.com/orgs/([^/]+)/projects/(\d+)", url)
    if org_match:
        return org_match.group(1), "org", int(org_match.group(2))

    # User: https://github.com/users/{user}/projects/{number}
    user_match = re.match(r"https://github.com/users/([^/]+)/projects/(\d+)", url)
    if user_match:
        return user_match.group(1), "user", int(user_match.group(2))

    # Repo: https://github.com/{owner}/{repo}/projects/{number}
    repo_match = re.match(r"https://github.com/([^/]+)/([^/]+)/projects/(\d+)", url)
    if repo_match:
        return repo_match.group(1), "repo", int(repo_match.group(3))

    return None, None, None
```

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

## Agent/Tool Responsibilities

### On Invocation

1. **Check for `.work-metadata.toml`** in current directory
2. **If found**: Parse and use for all tracking operations
3. **If missing**: Prompt user or fail with clear error
4. **Validate contents**: Ensure required fields present

### During Work

1. **Read metadata** before any Asana/GitHub operations
2. **Use URLs/IDs** to make correct API calls
3. **Never hardcode** Asana tasks or GitHub projects
4. **Log operations** that use metadata for debugging

### Error Handling

**Missing file**:
```
Error: No .work-metadata.toml found in current directory.

This file links worktree to work tracking systems.

To create it, run:
  work-start --asana <task-url> --github-project <project-number>

Or create manually following the schema in workflow/work-metadata.md
```

**Invalid format**:
```
Error: .work-metadata.toml is malformed.

Missing required field: work.asana_task

Expected format:
[work]
asana_task = "https://app.asana.com/0/123/456"
github_project = "https://github.com/orgs/org/projects/42"
```

**Invalid URLs**:
```
Error: Could not parse Asana task URL from .work-metadata.toml

Got: "invalid-url"
Expected: "https://app.asana.com/0/{project_gid}/{task_gid}"
```

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

## Anti-Patterns

### DON'T: Commit .work-metadata.toml
```
❌ git add .work-metadata.toml

This file is worktree-specific, not repository-wide.
Add to .gitignore instead.
```

### DON'T: Manually Edit (If Using Automated Tools)
```
❌ vim .work-metadata.toml

If using automated work tracking tools, let them maintain this file.
Manual edits may be overwritten or cause inconsistencies.
```

### DON'T: Reuse Across Worktrees
```
❌ cp ../other-worktree/.work-metadata.toml .

Each worktree tracks different work. Create fresh metadata per worktree.
```

### DON'T: Use Generic Values
```
❌ Bad:
[work]
asana_task = "TBD"
github_project = "TODO"

Metadata must contain real, valid URLs/IDs.
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
