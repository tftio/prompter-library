# uv Package Manager

**Use [uv](https://docs.astral.sh/uv) exclusively for Python package management.** Legacy tools (pip, setuptools, poetry, pdm) are superseded. Use `uv pip` subcommand only when explicitly required.

**Invoke Python via uv**: Use `uv run` or `uvx` for all Python and Python tool invocations.

## Key Commands

| Command | Purpose |
|---------|---------|
| `uv init` | Initialize new project with `pyproject.toml` |
| `uv add <package>` | Add dependency to project |
| `uv add --dev <package>` | Add development dependency |
| `uv remove <package>` | Remove dependency (including transitives) |
| `uv sync` | Sync environment with lockfile |
| `uv lock` | Update lockfile without syncing |
| `uv run <command>` | Run command in project environment |
| `uvx <tool>` | Run tool without installing |

## Project Structure

uv creates and manages:
- `pyproject.toml` – project metadata and dependencies
- `uv.lock` – cross-platform lockfile (commit to version control)
- `.python-version` – pinned Python version

## pyproject.toml Configuration

```toml
[project]
name = "my-project"
version = "0.1.0"
requires-python = ">=3.13"
dependencies = [
    "fastapi>=0.115.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=8.0",
    "ruff>=0.8.0",
    "basedpyright>=1.22",
]

[tool.uv]
dev-dependencies = [
    "pytest>=8.0",
    "ruff>=0.8.0",
]
```

## Lockfile Management

- `uv.lock` is human-readable TOML – edit only via uv commands
- **Always commit `uv.lock`** to version control for reproducible builds
- `uv run` automatically verifies lockfile is up-to-date before execution