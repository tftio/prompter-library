# Rust CLI Features

## PTY Detection

```rust
use is_terminal::IsTerminal;

fn main() {
    let is_interactive = std::io::stdout().is_terminal();

    if is_interactive {
        // Use colors, emojis, spinners
        println!("Success!");
    } else {
        // Plain output for pipes
        println!("Success!");
    }
}
```

## Terminal Effects

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

## Shell Completions

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

## Configuration Management

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
