# Rust Testing Standards

## Testing Requirements

All Rust projects **MUST** implement:

1. **Unit tests** embedded in source files with `#[cfg(test)]`
2. **Integration tests** in dedicated `tests/` directory
3. **80% minimum code coverage** via cargo-tarpaulin
4. **Cross-platform test verification** on Linux, macOS, and Windows

## Unit Tests Pattern

### Embedded in Source Files

```rust
// src/lib.rs

pub fn process_data(input: &str) -> Result<String, Error> {
    // Implementation
}

#[cfg(test)]
mod tests {
    use super::*;
    use tempfile::TempDir;

    #[test]
    fn test_process_data_valid() {
        let result = process_data("valid input");
        assert!(result.is_ok());
        assert_eq!(result.unwrap(), "expected output");
    }

    #[test]
    fn test_process_data_invalid() {
        let result = process_data("invalid");
        assert!(result.is_err());
    }

    #[test]
    fn test_with_temp_directory() {
        let temp_dir = TempDir::new().unwrap();
        let test_file = temp_dir.path().join("test.txt");
        // Test file operations
        std::fs::write(&test_file, "content").unwrap();
        // Temp dir automatically cleaned up on drop
    }
}
```

### Key Patterns
- Use `#[cfg(test)]` to exclude test code from production builds
- Use `use super::*;` to access parent module items
- Use `tempfile::TempDir` for file system tests (auto-cleanup)
- Test both success and error paths
- Test edge cases and boundary conditions

## Integration Tests

### CLI Testing Pattern

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

### Integration Test Best Practices
- Use `env!("CARGO_BIN_EXE_projectname")` to get binary path
- Create isolated temp directories for each test
- Use `Command::new()` to execute the binary
- Check both exit status and output
- Test with custom HOME directory for config isolation
- Clean up is automatic with TempDir

## Test Coverage with cargo-tarpaulin

### Configuration in CI

```yaml
# .github/workflows/ci.yml
- name: Install cargo-tarpaulin (Linux only)
  if: matrix.coverage
  run: cargo install cargo-tarpaulin

- name: Run tests with coverage
  if: matrix.coverage
  run: |
    cargo tarpaulin --all --out Xml --engine llvm --timeout 300 --fail-under 80

- name: Upload coverage to Codecov
  if: matrix.coverage
  uses: codecov/codecov-action@v4
  with:
    token: ${{ secrets.CODECOV_TOKEN }}
    files: ./tarpaulin-report.xml
    flags: unittests
```

### Local Coverage Check

```bash
# Install tarpaulin (Linux only)
cargo install cargo-tarpaulin

# Generate coverage report
cargo tarpaulin --all --out Html --engine llvm --timeout 300

# View HTML report
open tarpaulin-report.html

# Check coverage threshold
cargo tarpaulin --all --out Xml --engine llvm --timeout 300 --fail-under 80
```

### Coverage Requirements
- **Minimum 80% coverage** enforced in CI
- Linux only (llvm engine requirement)
- Include all crates with `--all`
- Timeout set to 300 seconds for complex tests
- XML output for CI integration, HTML for local viewing

## Cross-Platform Testing

### Test Matrix in CI

```yaml
test:
  name: Test
  strategy:
    matrix:
      os: [ubuntu-latest, macos-latest, macos-14, windows-latest]
      rust: [stable, "1.85.0"]
      include:
        - os: ubuntu-latest
          rust: stable
          coverage: true
  runs-on: ${{ matrix.os }}
```

### Platform-Specific Testing
- **ubuntu-latest**: Linux x64, coverage enabled
- **macos-latest**: macOS Intel
- **macos-14**: macOS Apple Silicon
- **windows-latest**: Windows x64
- Test both stable and MSRV Rust versions

### Platform-Specific Test Cases

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

## Running Tests

### Basic Commands
```bash
# Run all tests
cargo test --all

# Run with verbose output
cargo test --all --verbose

# Run specific test
cargo test test_name

# Run integration tests only
cargo test --test cli

# Run with RUSTFLAGS
RUSTFLAGS="-D warnings" cargo test --all
```

### justfile Integration
```just
# Run all tests
test:
    cargo test --all --verbose

# Run tests with coverage (Linux only)
test-coverage:
    cargo tarpaulin --all --out Html --engine llvm --timeout 300 --fail-under 80
```

## Test Quality Standards

### Coverage Focus Areas
- **Error paths**: Test all error conditions
- **Edge cases**: Boundary conditions, empty inputs, maximum sizes
- **File operations**: Use temporary directories
- **CLI interactions**: Test all subcommands and flag combinations
- **Configuration parsing**: Valid and invalid configurations

### What NOT to Test
- Third-party library internals
- Standard library behavior
- Trivial getters/setters without logic

### Test Naming Conventions
- Prefix with `test_`
- Use descriptive names: `test_parser_handles_empty_input`
- Group related tests with common prefixes

## Continuous Integration Testing

### Test Job Requirements
1. Run on multiple OS platforms
2. Test both stable and MSRV Rust versions
3. Generate coverage report (Linux only)
4. Upload coverage to Codecov
5. Fail on any test failure or coverage below threshold

### Expected Test Output
- Clear success/failure indicators
- Coverage percentage reported
- Failed test details with context
- Cross-platform compatibility confirmed
