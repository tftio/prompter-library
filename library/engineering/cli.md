# CLI RULES

When writing CLI tools, you *MUST* adhere to the following standards:

1. Use the "Subcommand" pattern; where there is a base tool and extensions in the main binary
2. all cli tools *MUST* support a `version` subcommand; a `help` subcommand
3. all cli tools *MUST* check their stdout/stderr pty, and if they are *not* writing to a terminal, then all decoration, colors, emoji, &c are removed
4. Be fun! Use cool terminal effects wherever possible
5. I prefer a single binary, in Rust. 

Configuration files *must* be in `toml` format; they must live in `$HOME/.config/{tool_name}`; if they need non-config data, they should prefer to use `$HOME/.local/{tool_name}`

