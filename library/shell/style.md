# Shell Style & Tooling

1. **Write scripts in Bash** (`#!/usr/bin/env bash`) and keep syntax compatible with modern Bash releases.
2. **Follow the [Google shell style guide](https://google.github.io/styleguide/shellguide.html)** for naming, quoting, and structure.
3. **Run [shellcheck](https://www.shellcheck.net/) with default settings** before committing; fix every reported warning.
4. **Use POSIX arithmetic syntax** — avoid `((VAR++))` or `((VAR--))`; instead write `VAR=$((VAR + 1))` (and subtract similarly).
