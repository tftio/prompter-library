# Rust CLI Structure

**NOTE**: This fragment extends `prompter tooling.cli`. All general CLI rules apply.

## Rust-Specific Requirements

All Rust CLI tools **MUST** additionally:

1. Support a `license` subcommand that outputs license information about the tool
2. Use `clap` with derive macros for argument parsing
3. Handle `--help` at every subcommand level (inherited from general CLI rules)
4. Support shell completion generation (`completions` subcommand)
5. Include `doctor` subcommand for health checks and diagnostics
6. Include `update` subcommand for self-update capability (optional but recommended)

## clap Configuration

```rust
use clap::{Parser, Subcommand};

#[derive(Parser)]
#[command(name = "projectname")]
#[command(version = env!("CARGO_PKG_VERSION"))]
#[command(about = "Project description", long_about = None)]
struct Cli {
    #[command(subcommand)]
    command: Commands,
}

#[derive(Subcommand)]
enum Commands {
    /// Run a specific profile
    Run {
        /// Profile name
        profile: String,

        /// Custom separator
        #[arg(short, long)]
        separator: Option<String>,
    },

    /// List available profiles
    List,

    /// Validate configuration
    Validate,

    /// Initialize default configuration
    Init,

    /// Show version information
    Version,

    /// Generate shell completions
    Completions {
        /// Shell type
        #[arg(value_enum)]
        shell: clap_complete::Shell,
    },

    /// Run health check and diagnostics
    Doctor,

    /// Update to latest version
    Update,

    /// Show license information
    License,
}
```

## Version Handling

```rust
// Pull version from Cargo.toml
#[command(version = env!("CARGO_PKG_VERSION"))]

// Or in version subcommand
fn show_version() {
    println!("{} {}", env!("CARGO_PKG_NAME"), env!("CARGO_PKG_VERSION"));
}
```

## Standard Subcommands

### Required Subcommands
- `version` - Show version information
- `help` - Show help (built-in with clap)
- `license` - Display license information

### Recommended Subcommands
- `init` - Initialize default configuration
- `validate` - Validate configuration
- `completions` - Generate shell completions
- `doctor` - Health checks and diagnostics
- `update` - Self-update to latest version

### Project-Specific Subcommands
Add as needed for your specific tool functionality.

## Dependencies

Required crates:

```toml
[dependencies]
clap = { version = "4.5", features = ["derive"] }
clap_complete = "4.5"
colored = "2.1"
dirs = "5.0"
indicatif = "0.17"
is-terminal = "0.4"
serde = { version = "1.0", features = ["derive"] }
toml = "0.8"

# For update functionality
reqwest = { version = "0.12", default-features = false, features = ["blocking", "json", "rustls-tls"] }
serde_json = "1.0"
```

## Best Practices

1. **Single binary** - Do not split functionality across multiple executables
2. **Version from Cargo.toml** - Use `env!("CARGO_PKG_VERSION")`
3. **Help everywhere** - clap provides this automatically
4. **Detect TTY** - Adjust output based on terminal detection
5. **Graceful degradation** - Work without colors/progress in non-TTY
6. **TOML configuration** - Store in `~/.config/{tool_name}/`
7. **Data separation** - Use `~/.local/{tool_name}/` for non-config data
8. **Shell completions** - Support bash, zsh, fish
9. **Health checks** - Implement doctor subcommand
10. **Self-update** - Implement update capability for production CLIs
