# Shell Scripting Standards

1. **UNLESS EXPLICITLY TOLD OTHERWISE**, *all* shell scripts **MUST** fail on *any* error.
2. **NEVER EVER EVER** use the `|| true` pattern to suppress errors.
3. All shell code to be written in `bash`.
4. Use the [Google style guide](https://google.github.io/styleguide/shellguide.html) for shell scripting.
5. All shell code **MUST** pass [shellcheck](https://www.shellcheck.net/) on its standard settings.
6. If calling out to other shell programs, be mindful that code **MUST** run on both MacOS **and** Linux.
7. When calling out to other shell tools, be sure to check the system type first.
8. Do not hardcode paths if possible; use `which` or `env` to find tools in the system
9. **Use `set -euo pipefail`** at the start of every shell script
10. **IF a command is expected to fail** - handle it with proper conditional logic like `if ! command; then handle_failure; fi`
11. **NEVER suppress errors with `2>/dev/null || true`** - always handle the actual condition
12. **NEVER use `((VAR++))` or `((VAR--))` arithmetic** - this is malformed bash syntax that causes script failures
13. **ALWAYS use `VAR=$((VAR + 1))` for arithmetic** - this is the correct POSIX-compliant syntax