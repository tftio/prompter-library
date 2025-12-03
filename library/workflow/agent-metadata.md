# Agent Metadata Configuration (DEPRECATED)

**This fragment is deprecated.** Use `prompter workflow.work-metadata` instead.

## Migration

`.agent-metadata.toml` has been renamed to `.work-metadata.toml` with an updated schema that better supports worktree-based workflows.

**Key changes**:
- File name: `.agent-metadata.toml` → `.work-metadata.toml`
- New `[work]` section for worktree-specific context (Asana task, GitHub project)
- `tracking.github_project` (name) → `work.github_project` (URL)
- `tracking.asana_project` (project board) → `work.asana_task` (specific task)
- `default_tags` → `default_labels`

See `prompter workflow.work-metadata` for complete documentation.

## Legacy Schema (for reference)

The old `.agent-metadata.toml` format:

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

## Fallback Behavior

**Use hierarchical fallback** - If `.agent-metadata.toml` is missing or incomplete:

1. Check for file in current working directory first
2. If file exists but specific key is missing, request only that value from operator
3. If file doesn't exist, inform operator that metadata file is missing and request all required values
4. Offer to create `.agent-metadata.toml` in the current working directory with provided values for future workflows

**Surface missing metadata explicitly** - If metadata is required but unavailable, ask the operator rather than making assumptions or skipping tracking steps.

## Example Usage

```bash
# Agent checks for metadata
if [ -f .agent-metadata.toml ]; then
  # Parse and use values
  github_project=$(toml get .agent-metadata.toml tracking.github_project)
else
  # Fall back to prompting operator
  echo "No .agent-metadata.toml found. Which GitHub project should I use?"
fi
```
