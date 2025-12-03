# Python Project Structure

## Directory Layout

All Python projects **MUST** use the `src/` layout with tests parallel:

```
project/
  src/
    myapp/
      __init__.py
      models/
      routes/
      services/
      db/
  tests/
    test_models/
    test_services/
  pyproject.toml
  uv.lock
```

## Package Organization

Use **nested packages by layer**:

- `models/` - Data structures, Pydantic models, database models
- `routes/` - API endpoints (FastAPI routers)
- `services/` - Business logic
- `db/` - Database access, repositories

## Dependency Direction

**One-way dependencies are mandatory:**

- `routes/` may import from `services/` and `models/`
- `services/` may import from `models/` and `db/`
- `models/` **MUST NOT** import from `services/` or `routes/`
- `db/` **MUST NOT** import from `services/` or `routes/`

Break cycles with dependency injection or adapter objects.

## CLI Architecture

**CLI entry points MUST be thin wrappers over library code.**

```python
# src/myapp/cli.py - THIN WRAPPER ONLY
import typer
from myapp.services.processor import process_data

app = typer.Typer()

@app.command()
def process(input_file: Path, output_file: Path) -> None:
    """Process data from input to output."""
    result = process_data(input_file, output_file)
    typer.echo(f"Processed {result.count} records")
```

Rules:
- **No business logic in CLI modules**
- All logic **MUST** be importable and testable without invoking the CLI
- CLI modules only handle argument parsing, output formatting, and exit codes

## External Command Wrappers

When calling external commands (git, docker, etc.), wrap them in typed functions:

```python
from dataclasses import dataclass

@dataclass
class GitStatus:
    branch: str
    modified_files: list[str]
    untracked_files: list[str]

def get_git_status(repo_path: Path) -> GitStatus | GitError:
    """Get git status as structured data."""
    result = subprocess.run(
        ["git", "status", "--porcelain", "-b"],
        capture_output=True,
        text=True,
        cwd=repo_path,
    )
    if result.returncode != 0:
        return GitError(result.stderr)
    return _parse_git_status(result.stdout)
```

**Never pass raw subprocess output through the system.** Parse it into typed containers immediately.
