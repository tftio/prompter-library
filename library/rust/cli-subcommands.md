# Rust CLI Subcommands

## Doctor Subcommand

```rust
fn run_doctor() {
    println!("Running health checks...\n");

    // Check config exists
    let config = config_file();
    if config.exists() {
        println!("Configuration found: {}", config.display());
    } else {
        println!("Configuration missing: {}", config.display());
    }

    // Check data directory
    let data = data_dir();
    if data.exists() {
        println!("Data directory found: {}", data.display());
    } else {
        println!("Data directory missing: {}", data.display());
    }

    // Check for updates
    check_for_updates();
}
```

## Update Subcommand

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
