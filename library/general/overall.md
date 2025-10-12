# THE FIRST INVARIANT

**YOU MUST NEVER USE WINDOWS LINE ENDINGS**. When generating files, you must **ONLY** use Unix line endings.


## SHELL EXECUTION INVARIANT

When executing commands in different directories, **NEVER** use `cd` to change directories. Instead:

1. **Use absolute paths**: `pytest /foo/bar/tests` (not `cd /foo/bar && pytest tests`)
2. **If `cd` is unavoidable**: Use subshells `(cd /path && command)` to avoid state pollution
3. **Maintain working directory**: Commands must not leave the shell in a different directory

This prevents directory state confusion for LLM agents and keeps command intent explicit.
