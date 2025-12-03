# Rust Project Structure Standards

## Required Project Files

All Rust projects **MUST** include these files:

### Core Files
- **Cargo.toml** - Package manifest with aggressive lints configuration
- **VERSION** - Single source of truth for version (synchronized with Cargo.toml via versioneer)
- **README.md** - Project documentation
- **LICENSE** - License file (prefer MIT, Apache-2.0, or dual licensing)
- **CLAUDE.md** - AI coding agent instructions

### Configuration Files
- **deny.toml** - cargo-deny configuration for license/security/dependency compliance
- **clippy.toml** - Clippy MSRV and complexity thresholds
- **rustfmt.toml** - Code formatting configuration
- **hooks.toml** - Git hooks configuration (peter-hook)
- **justfile** - Development workflow automation

### Directory Structure
```text
project/
├── src/
│   ├── main.rs           # Binary entry point
│   └── lib.rs            # Library code (optional)
├── tests/
│   └── cli.rs            # Integration tests
├── .github/
│   └── workflows/
│       ├── ci.yml        # Continuous integration
│       ├── release.yml   # Release automation
│       └── deps.yml      # Dependency management
├── Cargo.toml
├── VERSION
├── deny.toml
├── clippy.toml
├── rustfmt.toml
├── hooks.toml
└── justfile
```

## Cargo.toml Configuration

### Package Section
```toml
[package]
name = "project-name"
version = "1.0.0"              # Synchronized with VERSION file
edition = "2024"               # Use latest stable edition
rust-version = "1.85.0"        # MSRV for current tooling
license = "MIT"                # or "Apache-2.0" or "MIT OR Apache-2.0"
description = "Project description"
repository = "https://github.com/org/project"
keywords = ["cli", "tool"]
categories = ["command-line-utilities"]
```

### Aggressive Lints Configuration
```toml
[lints.rust]
unsafe_code = "warn"        # Allow unsafe only with explicit annotation
missing_docs = "deny"       # Require documentation for public APIs

[lints.clippy]
# Deny common issues
enum_glob_use = "deny"
wildcard_imports = "deny"

# Allow unavoidable Windows ecosystem duplicates
multiple_crate_versions = "allow"

# Maximum aggression - fail on any lint
all = { level = "deny", priority = -1 }
pedantic = { level = "deny", priority = -1 }
nursery = { level = "warn", priority = -1 }
cargo = { level = "warn", priority = -1 }
```

### Dependencies Best Practices
- Use rustls-tls instead of native-tls for better cross-platform compatibility
- Disable default features when not needed: `default-features = false`
- Pin specific features needed: `features = ["blocking", "json", "rustls-tls"]`

## Configuration File Templates

### clippy.toml
```toml
msrv = "1.85.0"
avoid-breaking-exported-api = false
cognitive-complexity-threshold = 30
```

### rustfmt.toml
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

### deny.toml Structure
- **[licenses]**: Whitelist allowed licenses (MIT, Apache-2.0, MPL-2.0, Unicode-3.0, CC0-1.0)
- **[bans]**: Control multiple versions, wildcard dependencies
- **[advisories]**: Security vulnerability handling
- **[sources]**: Restrict to crates.io registry only

## Standards Enforcement

1. **Use versioneer for Cargo.toml version** - Ensures atomic updates and synchronization
2. **Synchronize VERSION and Cargo.toml** - Enforced by git hooks and CI
3. **Require nightly for formatting** - rustfmt needs nightly for full feature set
4. **Use edition 2024** - Latest stable edition for modern patterns
5. **Set MSRV to 1.85.0** - Current stable with required features
6. **Single binary per project** - Keep functionality in one executable
7. **TOML for all configuration** - Standard format across the ecosystem
