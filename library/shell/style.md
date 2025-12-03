# Shell Style & Tooling

1. **Write scripts in Bash** (`#!/usr/bin/env bash`) and keep syntax compatible with modern Bash releases.
2. **Follow the [Google shell style guide](https://google.github.io/styleguide/shellguide.html)** for naming, quoting, and structure.
3. **Use POSIX arithmetic syntax** — avoid `((VAR++))` or `((VAR--))`; instead write `VAR=$((VAR + 1))` (and subtract similarly).

## Quality Tools

4. **Run [ShellCheck](https://www.shellcheck.net/) with default settings** before committing; fix every reported warning. ShellCheck is non-negotiable in CI pipelines.

   Common codes to know:
   | Code | Issue |
   |------|-------|
   | SC2086 | Double-quote to prevent globbing and word splitting |
   | SC2044 | Don't loop over `find` output; use `-exec` or `while read` |
   | SC2045 | Don't iterate over `ls` output; use globs |
   | SC2181 | Check exit code directly (`if cmd;`), not via `$?` |

5. **Run [shfmt](https://github.com/mvdan/sh)** for consistent formatting:
   ```bash
   shfmt -w -i 4 script.sh  # Format with 4-space indentation
   shfmt -d script.sh       # Check formatting (diff mode)
   ```

## When to Use Shell vs Python

### Shell is Appropriate For:
- **One-shot scripts** - Single-purpose, won't be reused or generalized
- **Simple orchestration** - Calling a sequence of commands
- **Glue scripts** - Connecting existing tools

### Python is Required For:
- **Reusable automation** - Anything that needs to generalize across a domain
- **Complex logic** - Conditionals, loops over data structures, error handling
- **Scripts over 100 lines** - Complexity threshold

### Never Do This:
Chain Python scripts via subprocess. This gives you the worst of both worlds: shell's lack of testability combined with Python's clunky pipeline creation.

Instead: Put logic in library code, create thin Typer CLI wrappers. See `prompter python.project-structure`.
