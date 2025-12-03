# Python Linting

- **Run approved lint tools and resolve all errors before committing** – ask the operator for guidance if blocked.
- **Request operator approval for ignore directives** – blanket ignores require explicit approval.
- **Treat every lint issue as blocking** unless the operator designates a specific exception.

## Ignore Directives

Per-line ignores (`# noqa`, `# type: ignore`) are **last resort only**.

When an ignore is unavoidable:
1. It **MUST** be accompanied by a `# TODO:` comment explaining why and how to fix later
2. Ask operator approval before adding

```python
# BAD - unexplained ignore
result = dynamic_call()  # type: ignore

# GOOD - documented ignore with TODO
# TODO: Add proper typing when upstream library adds type stubs
result = dynamic_call()  # type: ignore[no-untyped-call]
```

We use the following tools for linting:

## Ruff (Linter + Formatter)

[Ruff](https://docs.astral.sh/ruff/) is our unified linter and formatter, replacing flake8, black, isort, pyupgrade, and autoflake. It is 10-100x faster than legacy tools.

**Configuration** (`pyproject.toml` or `ruff.toml`):

```toml
[tool.ruff]
line-length = 88
target-version = "py313"

[tool.ruff.lint]
select = ["E", "F", "I", "N", "W", "B", "UP"]  # B=bugbear, UP=pyupgrade
ignore = []

[tool.ruff.format]
docstring-code-format = true
```

**Commands**:
- `ruff check .` – lint all files
- `ruff check --fix .` – lint and auto-fix
- `ruff format .` – format all files
- `ruff format --check .` – check formatting without changes

**Run order**: `ruff check --fix . && ruff format .` (linter before formatter for import sorting)

## ty (Type Checker)

[ty](https://docs.astral.sh/ty/) is Astral's type checker written in Rust. As of December 2025, ty is pre-alpha (v0.0.x); use **mypy** for production type checking until ty reaches beta (expected late 2025).

**Quick check**: `uvx ty check`

## Interrogate

[Interrogate](https://interrogate.readthedocs.io/en/latest/) ensures docstring coverage. Configure minimum coverage thresholds in `pyproject.toml`.


