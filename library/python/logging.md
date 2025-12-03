# Python Logging Standards

## Library Choice

Use **structlog** for all logging. It provides structured logging internally while supporting human-readable output for CLI tools.

## Output Modes

CLI tools **MUST** support both human-readable and structured output:

- **Default**: Human-readable, minimal output (errors only)
- `--structured-logs`: JSON structured output for machine parsing

```python
import structlog

def configure_logging(structured: bool = False, verbosity: int = 0) -> None:
    """Configure logging based on output mode and verbosity."""
    if structured:
        structlog.configure(
            processors=[
                structlog.processors.JSONRenderer()
            ]
        )
    else:
        structlog.configure(
            processors=[
                structlog.dev.ConsoleRenderer()
            ]
        )
```

## Verbosity Convention

CLI tools **MUST** follow this verbosity flag pattern:

| Flag | Level | Output |
|------|-------|--------|
| (none) | ERROR | Errors only |
| `-v` | INFO | Informational messages |
| `-vv` | DEBUG | Debug details |
| `--quiet` | NONE | Suppress all output (exit code only) |

```python
import typer

app = typer.Typer()

@app.command()
def main(
    verbose: int = typer.Option(0, "-v", "--verbose", count=True),
    quiet: bool = typer.Option(False, "--quiet", "-q"),
    structured_logs: bool = typer.Option(False, "--structured-logs"),
) -> None:
    configure_logging(structured=structured_logs, verbosity=verbose, quiet=quiet)
```

## Logging Content

- **Always include correlation IDs** in request-scoped logs
- **Log at appropriate levels**:
  - ERROR: Failures requiring attention
  - WARNING: Unexpected but handled conditions
  - INFO: Significant state changes, request completion
  - DEBUG: Detailed diagnostic information
- **Redact secrets, passwords, and PII from all log output**
