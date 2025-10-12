# Rust Quality Tooling Standards

## Required Quality Tools

All Rust projects **MUST** use these quality tools:

### Core Linting & Formatting
1. **cargo-clippy** - Lint with pedantic and nursery checks
2. **rustfmt (nightly)** - Code formatting with stable features
3. **cargo-audit** - Security vulnerability scanning
4. **cargo-deny** - License and dependency compliance

### Advanced Analysis
5. **cargo-machete** - Detect unused dependencies
6. **cargo-udeps** - Find unused dependencies (requires nightly)
7. **cargo-outdated** - Check for outdated dependencies
8. **cargo-msrv** - Verify minimum supported Rust version

## Clippy Configuration

### Aggressive Linting in Cargo.toml
```toml
[lints.clippy]
# Deny common issues
enum_glob_use = "deny"
wildcard_imports = "deny"

# Allow only unavoidable ecosystem duplicates
multiple_crate_versions = "allow"

# Maximum aggression
all = { level = "deny", priority = -1 }
pedantic = { level = "deny", priority = -1 }
nursery = { level = "warn", priority = -1 }
cargo = { level = "warn", priority = -1 }
```

### clippy.toml Configuration
```toml
msrv = "1.85.0"
avoid-breaking-exported-api = false
cognitive-complexity-threshold = 30
```

### Running Clippy
```bash
# Standard check (fails on any warning)
cargo clippy --all-targets -- -D warnings

# With RUSTFLAGS for CI
RUSTFLAGS="-D warnings" cargo clippy --all-targets -- -D warnings
```

## Formatting Standards

### rustfmt.toml Configuration
```toml
edition = "2024"
msrv = "1.70.0"
max_width = 100
hard_tabs = false
tab_spaces = 4

# Stable features only
force_explicit_abi = true
use_field_init_shorthand = true
use_try_shorthand = true
```

### Running Format Checks
```bash
# Check formatting (CI mode)
cargo fmt --all -- --check

# Apply formatting (requires nightly)
cargo +nightly fmt

# Install nightly if needed
rustup install nightly
```

## Security Auditing with cargo-audit

### Installation
```bash
cargo install cargo-audit
```

### Usage
```bash
# Check for security vulnerabilities
cargo audit

# Fail on any warnings (CI mode)
cargo audit --deny warnings
```

### Integration Points
- Run in pre-push git hooks
- Run in CI security audit job
- Schedule weekly scans via GitHub Actions

## Dependency Compliance with cargo-deny

### deny.toml Configuration Structure

```toml
[graph]
all-features = true

[advisories]
git-fetch-with-cli = true

[licenses]
# Whitelist acceptable licenses
allow = ["MIT", "Apache-2.0", "MPL-2.0", "Unicode-3.0", "CC0-1.0"]
confidence-threshold = 0.9

[bans]
multiple-versions = "warn"
wildcards = "deny"

# Skip Windows ecosystem duplicates
skip = [
  { crate = "windows-sys" },
  { crate = "windows-targets" },
  # ... other Windows crates
]

[sources]
unknown-registry = "deny"
unknown-git = "deny"
allow-registry = ["https://github.com/rust-lang/crates.io-index"]
```

### Running cargo-deny
```bash
# Install
cargo install cargo-deny

# Check all policies
cargo deny check

# Check specific policy
cargo deny check licenses
cargo deny check advisories
cargo deny check bans
```

## Unused Dependencies Detection

### cargo-machete (stable)
```bash
# Install
cargo install cargo-machete

# Detect unused dependencies
cargo machete
```

### cargo-udeps (nightly)
```bash
# Install
cargo install cargo-udeps

# Check for unused dependencies
cargo +nightly udeps --all-targets
```

## Dependency Update Management

### cargo-outdated
```bash
# Install
cargo install cargo-outdated

# Check for outdated dependencies
cargo outdated --root-deps-only

# Generate report
cargo outdated --root-deps-only > outdated-report.txt
```

## MSRV Verification

### cargo-msrv
```bash
# Install
cargo install cargo-msrv

# Verify current MSRV
cargo msrv --output-format json

# Check compatibility
cargo check --all
```

## Minimal Versions Testing

Test with minimum allowed dependency versions:

```bash
# Switch to nightly
rustup default nightly

# Generate minimal lockfile
cargo update -Z minimal-versions

# Switch back to stable
rustup default stable

# Test with minimal versions
cargo test --all

# Restore original lockfile
git checkout -- Cargo.lock
```

## Quality Standards Enforcement

### Pre-commit Requirements
- Code formatting verification (cargo fmt --check)
- Clippy linting passes (no warnings)

### Pre-push Requirements
- All tests pass
- Security audit passes
- No unused dependencies detected

### CI Requirements
- All quality checks pass on multiple platforms
- 80% minimum test coverage
- No security vulnerabilities
- License compliance verified
- MSRV compatibility confirmed

### Release Requirements
- All quality gates passed
- Version synchronization verified
- No failing tests
- Security audit clean
- Dependency compliance verified

## Tool Installation Script

```bash
#!/usr/bin/env bash
# Install all required quality tools

# Nightly toolchain for formatting and udeps
rustup install nightly

# Core quality tools
cargo install cargo-audit
cargo install cargo-deny
cargo install cargo-machete
cargo install cargo-udeps
cargo install cargo-outdated
cargo install cargo-msrv

# Optional but recommended
cargo install cargo-tarpaulin  # Linux only
```

## Integration with justfile

```just
# Format code (requires nightly)
format:
    cargo +nightly fmt

# Check formatting
format-check:
    cargo fmt --all -- --check

# Lint with clippy
lint:
    cargo clippy --all-targets -- -D warnings

# Security audit
audit:
    cargo audit

# Dependency compliance
deny:
    cargo deny check

# Check for unused dependencies
unused-deps:
    cargo machete
    cargo +nightly udeps --all-targets

# Check for outdated dependencies
outdated:
    cargo outdated --root-deps-only

# Run all quality checks
quality: format-check lint audit deny
```
