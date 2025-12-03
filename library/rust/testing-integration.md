# Rust Integration Testing

## CLI Testing Pattern

```rust
// tests/cli.rs

use std::env;
use std::fs;
use std::path::PathBuf;
use std::process::Command;

/// Get the path to the compiled binary using cargo environment variable
const fn bin_path() -> &'static str {
    env!("CARGO_BIN_EXE_projectname")
}

/// Create unique temporary directory for test isolation
fn tmp_home(prefix: &str) -> PathBuf {
    let mut p = env::temp_dir();
    let unique = format!(
        "{}_{}_{}",
        prefix,
        std::process::id(),
        std::time::SystemTime::now()
            .duration_since(std::time::UNIX_EPOCH)
            .unwrap()
            .as_nanos()
    );
    p.push(unique);
    p
}

#[test]
fn test_version_command() {
    let output = Command::new(bin_path())
        .arg("version")
        .output()
        .expect("Failed to execute command");

    assert!(output.status.success());
    let stdout = String::from_utf8_lossy(&output.stdout);
    assert!(stdout.contains(env!("CARGO_PKG_VERSION")));
}

#[test]
fn test_help_command() {
    let output = Command::new(bin_path())
        .arg("help")
        .output()
        .expect("Failed to execute command");

    assert!(output.status.success());
}

#[test]
fn test_with_config() {
    let home = tmp_home("test_config");
    fs::create_dir_all(&home).unwrap();

    let output = Command::new(bin_path())
        .env("HOME", &home)
        .arg("init")
        .output()
        .unwrap();

    assert!(
        output.status.success(),
        "Command failed: {}",
        String::from_utf8_lossy(&output.stderr)
    );

    // Verify config was created
    let config_path = home.join(".config/projectname/config.toml");
    assert!(config_path.exists());
}
```

## Integration Test Best Practices
- Use `env!("CARGO_BIN_EXE_projectname")` to get binary path
- Create isolated temp directories for each test
- Use `Command::new()` to execute the binary
- Check both exit status and output
- Test with custom HOME directory for config isolation
- Clean up is automatic with TempDir

## Test Organization

### Directory Structure
```text
project/
├── src/
│   ├── main.rs      # Contains #[cfg(test)] unit tests
│   └── lib.rs       # Contains #[cfg(test)] unit tests
└── tests/
    ├── cli.rs       # CLI integration tests
    ├── config.rs    # Configuration tests
    └── common/
        └── mod.rs   # Shared test utilities
```

### Common Test Utilities

```rust
// tests/common/mod.rs

use std::path::PathBuf;

pub fn fixture_path(name: &str) -> PathBuf {
    let mut path = PathBuf::from(env!("CARGO_MANIFEST_DIR"));
    path.push("tests/fixtures");
    path.push(name);
    path
}

pub fn read_fixture(name: &str) -> String {
    std::fs::read_to_string(fixture_path(name))
        .expect("Failed to read fixture")
}
```

## Platform-Specific Test Cases

```rust
#[test]
#[cfg(unix)]
fn test_unix_permissions() {
    use std::os::unix::fs::PermissionsExt;
    // Unix-specific test
}

#[test]
#[cfg(windows)]
fn test_windows_paths() {
    // Windows-specific test
}

#[test]
fn test_cross_platform() {
    // Works on all platforms
}
```
