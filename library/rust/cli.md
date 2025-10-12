# Rust CLI Standards

**NOTE**: This fragment extends `tooling/cli.md`. All general CLI rules apply.

## Rust-Specific Requirements

All Rust CLI tools **MUST** additionally:

1. Support a `license` subcommand that outputs license information about the tool
2. Use `clap` with derive macros for argument parsing
3. Handle `--help` at every subcommand level (inherited from general CLI rules)
4. Support shell completion generation (`completions` subcommand)
5. Include `doctor` subcommand for health checks and diagnostics
6. Include `update` subcommand for self-update capability (optional but recommended)

## Implementation Patterns

### clap Configuration

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

### Version Handling

```rust
// Pull version from Cargo.toml
#[command(version = env!("CARGO_PKG_VERSION"))]

// Or in version subcommand
fn show_version() {
    println!("{} {}", env!("CARGO_PKG_NAME"), env!("CARGO_PKG_VERSION"));
}
```

### PTY Detection

```rust
use is_terminal::IsTerminal;

fn main() {
    let is_interactive = std::io::stdout().is_terminal();

    if is_interactive {
        // Use colors, emojis, spinners
        println!("✅ Success!");
    } else {
        // Plain output for pipes
        println!("Success!");
    }
}
```

### Terminal Effects

```rust
use colored::*;
use indicatif::{ProgressBar, ProgressStyle};

fn show_progress() {
    let pb = ProgressBar::new(100);
    pb.set_style(
        ProgressStyle::default_bar()
            .template("{spinner:.green} [{bar:40.cyan/blue}] {pos}/{len} {msg}")
            .unwrap()
    );

    for i in 0..100 {
        pb.set_position(i);
        // Work...
    }
    pb.finish_with_message("Done!");
}

fn show_status() {
    println!("{}", "Success!".green().bold());
    println!("{}", "Warning!".yellow());
    println!("{}", "Error!".red());
}

// Conditional formatting
fn format_output(msg: &str, is_tty: bool) -> String {
    if is_tty {
        msg.green().to_string()
    } else {
        msg.to_string()
    }
}
```

### Shell Completions

```rust
use clap::CommandFactory;
use clap_complete::{generate, Shell};

fn generate_completions(shell: Shell) {
    let mut cmd = Cli::command();
    generate(shell, &mut cmd, "projectname", &mut std::io::stdout());
}

// Usage in subcommand
Commands::Completions { shell } => {
    generate_completions(shell);
}
```

### Configuration Management

```rust
use std::path::PathBuf;
use dirs;

fn config_dir() -> PathBuf {
    dirs::config_dir()
        .expect("Failed to find config directory")
        .join("projectname")
}

fn data_dir() -> PathBuf {
    dirs::data_local_dir()
        .expect("Failed to find data directory")
        .join("projectname")
}

fn config_file() -> PathBuf {
    config_dir().join("config.toml")
}

fn library_dir() -> PathBuf {
    data_dir().join("library")
}
```

### Doctor Subcommand

```rust
fn run_doctor() {
    println!("Running health checks...\n");

    // Check config exists
    let config = config_file();
    if config.exists() {
        println!("✅ Configuration found: {}", config.display());
    } else {
        println!("❌ Configuration missing: {}", config.display());
    }

    // Check data directory
    let data = data_dir();
    if data.exists() {
        println!("✅ Data directory found: {}", data.display());
    } else {
        println!("❌ Data directory missing: {}", data.display());
    }

    // Check for updates
    check_for_updates();
}
```

### Update Subcommand

```rust
use reqwest::blocking::Client;
use serde::Deserialize;

#[derive(Deserialize)]
struct Release {
    tag_name: String,
    assets: Vec<Asset>,
}

#[derive(Deserialize)]
struct Asset {
    name: String,
    browser_download_url: String,
}

fn check_for_updates() -> Result<(), Box<dyn std::error::Error>> {
    let current_version = env!("CARGO_PKG_VERSION");
    let client = Client::new();

    let release: Release = client
        .get("https://api.github.com/repos/org/project/releases/latest")
        .header("User-Agent", "projectname")
        .send()?
        .json()?;

    let latest_version = release.tag_name.trim_start_matches("projectname-v");

    if latest_version > current_version {
        println!("Update available: {} -> {}", current_version, latest_version);
        println!("Run: projectname update");
    } else {
        println!("You're up to date!");
    }

    Ok(())
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

## Error Handling

```rust
use std::process;

// Exit with error
fn exit_with_error(msg: &str) -> ! {
    eprintln!("Error: {}", msg);
    process::exit(1);
}

// Result-based error handling
fn main() {
    if let Err(e) = run() {
        eprintln!("Error: {}", e);
        process::exit(1);
    }
}

fn run() -> Result<(), Box<dyn std::error::Error>> {
    // Main logic
    Ok(())
}
```

## Configuration File Format

Use TOML for configuration:

```toml
# ~/.config/projectname/config.toml

[settings]
default_value = "something"

[profile.example]
depends_on = ["file1.md", "file2.md"]
```

Parse with:
```rust
use toml;
use serde::Deserialize;

#[derive(Deserialize)]
struct Config {
    settings: Settings,
    profile: HashMap<String, Profile>,
}

fn load_config() -> Result<Config, Box<dyn std::error::Error>> {
    let content = std::fs::read_to_string(config_file())?;
    let config: Config = toml::from_str(&content)?;
    Ok(config)
}
```

## Testing CLI

Integration tests for CLI:

```rust
// tests/cli.rs
use std::process::Command;

const fn bin_path() -> &'static str {
    env!("CARGO_BIN_EXE_projectname")
}

#[test]
fn test_version() {
    let output = Command::new(bin_path())
        .arg("version")
        .output()
        .expect("Failed to execute");

    assert!(output.status.success());
    let stdout = String::from_utf8_lossy(&output.stdout);
    assert!(stdout.contains(env!("CARGO_PKG_VERSION")));
}

#[test]
fn test_help() {
    let output = Command::new(bin_path())
        .arg("help")
        .output()
        .expect("Failed to execute");

    assert!(output.status.success());
}
```

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
10. **Self-update** - Consider implementing update capability
