# CLI Standards

When writing CLI tools, you **MUST** adhere to the following standards:

1. Use the "subcommand" pattern—root command with dedicated subcommands for functionality.
2. Provide `--help` (and the `help` subcommand) at the root and every subcommand level.
3. Provide a `version` subcommand (or `--version` flag) that reports the semantic version.
4. Detect stdout/stderr TTY; when not interactive, disable all decorations, colors, emojis, or spinners.
5. When output is interactive, keep styling purposeful and accessible—use simple ANSI colors only when they clarify intent.
6. Ship a single Rust binary; do not split functionality across helper executables.

Configuration files **MUST** be in `toml` format; they must live in `$HOME/.config/{tool_name}`; if they need non-config data, they should prefer to use `$HOME/.local/{tool_name}`
