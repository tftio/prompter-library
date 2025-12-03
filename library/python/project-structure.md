# Python Project Structure

## Directory Layout

All Python projects **MUST** use the `src/` layout with tests parallel:

```text
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

Parse subprocess output into typed containers immediately at the call site.

## File Organization

**Split files liberally** - Python allows splitting modules across files.

- **Class as a unit**: One class per file is a reasonable default
- **Size threshold**: ~300 lines is a signal to look for decomposition opportunities
- **Private modules**: Use `_` prefix for internal files (`_helpers.py`, `_internal.py`)
- **Re-exports**: Use `__init__.py` to maintain clean public API:

```python
# mymodule/__init__.py
from .user import User
from .account import Account
__all__ = ["User", "Account"]
```

This allows internal restructuring without breaking importers.

## Package Depth

**4 levels is the refactoring threshold.**

```text
myapp.services.billing.stripe.webhooks  # Too deep - refactor
myapp.services.billing.webhooks         # Better
```

When nesting exceeds 4 levels, flatten or restructure.

## Function Parameters

**Many optional parameters is a code smell** in library code.

Acceptable in CLI (Typer) where parameters map to command-line flags. In library code, it usually indicates the function should be split.

**Fix in order of preference:**
1. Split into multiple focused functions
2. Introduce a builder pattern or config object

```python
# SMELL - too many optional params
def process(data, format=None, validate=True, strict=False,
            encoding="utf-8", timeout=30, retries=3): ...

# BETTER - split by concern
def process(data: Data, options: ProcessOptions) -> Result: ...

# OR - multiple functions
def process(data: Data) -> Result: ...
def process_with_validation(data: Data, strict: bool = False) -> Result: ...
```

## Classes vs Functions

**Use classes only when you need state across method calls.**

Otherwise, prefer free functions.

```python
# UNNECESSARY CLASS
class DataProcessor:
    def process(self, data):
        return transform(data)

# BETTER - just a function
def process_data(data: Data) -> Result:
    return transform(data)

# JUSTIFIED CLASS - maintains state
class ConnectionPool:
    def __init__(self, max_connections: int):
        self._connections: list[Connection] = []
        self._max = max_connections

    def acquire(self) -> Connection: ...
    def release(self, conn: Connection): ...
```

## Import Conventions

**Relative imports within the package, absolute for external.**

```python
# Inside myapp/services/auth.py

# Relative for intra-package
from .utils import hash_password
from ..models import User

# Absolute for external dependencies
from sqlalchemy import select
from pydantic import BaseModel
```

**Avoid deep relative imports** - `...` and beyond becomes confusing.

## Circular Import Prevention

1. **Move shared types to a separate module** that both sides can import
2. **Use `TYPE_CHECKING` blocks aggressively** for type-only imports:

```python
from typing import TYPE_CHECKING

if TYPE_CHECKING:
    from .user import User  # Only imported for type hints

def get_user_name(user: "User") -> str:  # Forward reference
    return user.name
```

## Where Things Live

| What | Where |
|------|-------|
| Domain models, dataclasses | `models/` package with submodules |
| Type aliases, NewTypes | `models/types.py` |
| Protocols, ABCs | Near implementations, in `protocols.py` |
| Constants | `<module>/constants.py` (centralized per module) |

