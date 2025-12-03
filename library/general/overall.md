# Core Invariants

## Line Ending Standard

**Use Unix line endings exclusively (LF).** Windows line endings (CRLF) are prohibited in all generated files.

## Shell Execution Standard

Maintain explicit directory context when executing commands:

1. **Use absolute paths**: `pytest /foo/bar/tests` (not `cd /foo/bar && pytest tests`)
2. **If `cd` is unavoidable**: Use subshells `(cd /path && command)` to avoid state pollution
3. **Maintain working directory**: Commands must not leave the shell in a different directory

This prevents directory state confusion for LLM agents and keeps command intent explicit.
