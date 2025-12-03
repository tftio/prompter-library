# Work Metadata Tool Integration

## Reading Metadata

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

## Writing Metadata

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

## URL Parsing

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

## Agent/Tool Responsibilities

### On Invocation

1. **Check for `.work-metadata.toml`** in current directory
2. **If found**: Parse and use for all tracking operations
3. **If missing**: Prompt user or fail with clear error
4. **Validate contents**: Ensure required fields present

### During Work

1. **Read metadata** before any Asana/GitHub operations
2. **Use URLs/IDs** to make correct API calls
3. **Read from metadata** for Asana tasks and GitHub projects
4. **Log operations** that use metadata for debugging

### Error Handling

**Missing file**:
```text
Error: No .work-metadata.toml found in current directory.

This file links worktree to work tracking systems.

To create it, run:
  work-start --asana <task-url> --github-project <project-number>

Or create manually following the schema in workflow/work-metadata.md
```

**Invalid format**:
```text
Error: .work-metadata.toml is malformed.

Missing required field: work.asana_task

Expected format:
[work]
asana_task = "https://app.asana.com/0/123/456"
github_project = "https://github.com/orgs/org/projects/42"
```

**Invalid URLs**:
```text
Error: Could not parse Asana task URL from .work-metadata.toml

Got: "invalid-url"
Expected: "https://app.asana.com/0/{project_gid}/{task_gid}"
```
