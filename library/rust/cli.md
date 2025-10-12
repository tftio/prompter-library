# Rust CLI Standards

**NOTE**: This fragment extends `tooling/cli.md`. All general CLI rules apply.

## Rust-Specific Requirements

All Rust CLI tools **MUST** additionally:

1. Support a `license` subcommand that outputs license information about the tool.
2. Use `clap` (or `structopt`) for argument parsing.
3. Handle `--help` at every subcommand level (inherits the general requirement).

## Implementation Notes

- Prefer `clap::derive` for ergonomics and consistent help output.
- Pull the version string from `Cargo.toml` via `env!("CARGO_PKG_VERSION")`.
