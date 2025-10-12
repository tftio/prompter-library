# Rust Git Hooks Configuration with peter-hook

## Overview

All Rust projects **MUST** use peter-hook for git hooks with comprehensive quality checks matching CI pipeline.

## Required hooks.toml Configuration

Create `hooks.toml` in project root:

```toml
# Project hooks configuration
# Self-contained matching CI pipeline

# =============================================================================
# CODE QUALITY HOOKS (matches CI lint and format jobs)
# =============================================================================

[hooks.format-check]
command = ["cargo", "fmt", "--all", "--", "--check"]
modifies_repository = false
execution_type = "other"
run_always = true
description = "Check code formatting (matches CI format job)"

[hooks.clippy-check]
command = ["cargo", "clippy", "--all-targets", "--", "-D", "warnings"]
modifies_repository = false
execution_type = "other"
run_always = true
description = "Run clippy linting (matches CI lint job)"
env = { RUSTFLAGS = "-D warnings" }

# =============================================================================
# TEST HOOKS (matches CI test job)
# =============================================================================

[hooks.test-all]
command = ["cargo", "test", "--all", "--verbose"]
modifies_repository = false
execution_type = "other"
run_always = true
description = "Run all tests (matches CI test job)"
env = { RUSTFLAGS = "-D warnings" }

[hooks.security-audit]
command = ["cargo", "audit"]
modifies_repository = false
execution_type = "other"
run_always = true
description = "Run security audit (matches CI audit job)"

[hooks.secret-scan]
command = ["gitleaks", "detect", "--source=.", "--verbose", "--no-banner"]
modifies_repository = false
execution_type = "other"
run_always = true
description = "Scan for secrets and credentials in repository"

[hooks.venv-check]
command = ["unvenv"]
modifies_repository = false
execution_type = "in-place"
files = ["**/*.py", "**/pyvenv.cfg", "**/activate"]
description = "Detect uncommitted Python virtual environments"

# =============================================================================
# VERSION MANAGEMENT HOOKS
# =============================================================================

[hooks.version-sync-check]
command = ["versioneer", "verify"]
include = ["Cargo.toml", "VERSION"]
modifies_repository = false
execution_type = "other"
description = "Verify all version files are synchronized"

[hooks.tag-version-check]
command = ["sh", "-c", "if git describe --exact-match --tags HEAD 2>/dev/null | grep -q 'projectname-v'; then TAG_VERSION=$(git describe --exact-match --tags HEAD | sed 's/projectname-v//'); CARGO_VERSION=$(versioneer show); if [ \"$TAG_VERSION\" != \"$CARGO_VERSION\" ]; then echo '❌ ERROR: Git tag version ('$TAG_VERSION') does not match Cargo version ('$CARGO_VERSION')'; echo \"Use 'versioneer tag' to create tags that match the current version\"; exit 1; fi; echo '✅ Tag version matches Cargo version: '$TAG_VERSION; fi"]
include = ["Cargo.toml", "VERSION"]
modifies_repository = false
execution_type = "other"
description = "Verify git tag version matches Cargo.toml version"

# =============================================================================
# HOOK GROUPS
# =============================================================================

[groups.pre-commit]
includes = ["format-check", "clippy-check"]
execution = "sequential"
description = "Code quality checks before commit (matches CI format + lint jobs)"

[groups.pre-push]
includes = ["clippy-check", "test-all", "venv-check", "security-audit", "secret-scan", "version-sync-check", "tag-version-check"]
execution = "sequential"
description = "Full validation before push (matches CI lint + test + audit + security jobs)"
```

## Hook Customization for Your Project

### Update Tag Pattern
Replace `projectname-v` with your actual project tag format:

```toml
[hooks.tag-version-check]
command = ["sh", "-c", "if git describe --exact-match --tags HEAD 2>/dev/null | grep -q 'yourproject-v'; then TAG_VERSION=$(git describe --exact-match --tags HEAD | sed 's/yourproject-v//'); CARGO_VERSION=$(versioneer show); if [ \"$TAG_VERSION\" != \"$CARGO_VERSION\" ]; then echo '❌ ERROR: Git tag version ('$TAG_VERSION') does not match Cargo version ('$CARGO_VERSION')'; exit 1; fi; echo '✅ Tag version matches Cargo version: '$TAG_VERSION; fi"]
```

### Optional Hooks

Add these hooks if needed:

```toml
[hooks.dependency-check]
command = ["cargo", "deny", "check"]
modifies_repository = false
execution_type = "other"
run_always = true
description = "Check dependency compliance"

[hooks.unused-deps]
command = ["cargo", "machete"]
modifies_repository = false
execution_type = "other"
run_always = true
description = "Check for unused dependencies"
```

## Hook Installation

### Install peter-hook
```bash
# Install from crates.io
cargo install peter-hook

# Or build from source
cd peter-hook
cargo install --path .
```

### Install Hooks in Repository
```bash
# Navigate to project root
cd /path/to/project

# Install hooks
peter-hook install
```

### Verify Installation
```bash
# Check installed hooks
ls -la .git/hooks/

# Run hooks manually
peter-hook run pre-commit
peter-hook run pre-push
```

## Hook Execution Patterns

### Pre-commit Hooks
- **format-check**: Ensure code is formatted correctly
- **clippy-check**: Catch linting errors before commit
- **Execution**: Sequential (one after another)
- **Fast feedback**: Runs in seconds

### Pre-push Hooks
- **clippy-check**: Re-run linting
- **test-all**: Run full test suite
- **venv-check**: Detect Python virtual environments
- **security-audit**: Check for vulnerabilities
- **secret-scan**: Scan for committed secrets
- **version-sync-check**: Verify VERSION and Cargo.toml match
- **tag-version-check**: Verify git tag matches version
- **Execution**: Sequential (comprehensive validation)
- **May take minutes**: Full quality gate

## Integration with justfile

Add helper recipes to justfile:

```just
# Install git hooks using peter-hook
install-hooks:
    @echo "Installing git hooks with peter-hook..."
    @if command -v peter-hook >/dev/null 2>&1; then \
        peter-hook install; \
        echo "✅ Git hooks installed"; \
    else \
        echo "❌ peter-hook not found. Install with: cargo install peter-hook"; \
        exit 1; \
    fi

# Run pre-commit hooks manually
pre-commit:
    @if command -v peter-hook >/dev/null 2>&1; then \
        peter-hook run pre-commit; \
    else \
        echo "❌ peter-hook not found"; \
        exit 1; \
    fi

# Run pre-push hooks manually
pre-push:
    @if command -v peter-hook >/dev/null 2>&1; then \
        peter-hook run pre-push; \
    else \
        echo "❌ peter-hook not found"; \
        exit 1; \
    fi
```

## Version Validation Hook Details

### Version Sync Check
- Runs `versioneer verify`
- Ensures VERSION file matches Cargo.toml
- Triggers on changes to Cargo.toml or VERSION
- Prevents version drift

### Tag Version Check
- Runs only when HEAD is tagged
- Extracts version from git tag
- Compares with version from versioneer
- Prevents tag/version mismatches
- Critical for release automation

## Hook Execution Flow

### On Commit
```
1. format-check runs (cargo fmt --check)
   ↓ success
2. clippy-check runs (cargo clippy)
   ↓ success
3. Commit proceeds
```

### On Push
```
1. clippy-check runs
   ↓ success
2. test-all runs
   ↓ success
3. venv-check runs
   ↓ success
4. security-audit runs
   ↓ success
5. secret-scan runs
   ↓ success
6. version-sync-check runs
   ↓ success
7. tag-version-check runs (if tagged)
   ↓ success
8. Push proceeds
```

## Troubleshooting

### Hook Failures

**Format check fails:**
```bash
# Fix formatting
cargo +nightly fmt

# Re-attempt commit
git commit
```

**Clippy check fails:**
```bash
# See specific warnings
cargo clippy --all-targets -- -D warnings

# Fix issues and re-commit
```

**Version sync fails:**
```bash
# Synchronize versions
versioneer sync

# Add changes
git add Cargo.toml VERSION

# Re-commit
```

**Tag version mismatch:**
```bash
# Delete incorrect tag
git tag -d projectname-v1.0.0

# Use versioneer to create correct tag
versioneer tag --tag-format "projectname-v{version}"
```

### Bypass Hooks (Emergency Only)
```bash
# Skip pre-commit hooks (NOT RECOMMENDED)
git commit --no-verify

# Skip pre-push hooks (NOT RECOMMENDED)
git push --no-verify
```

**WARNING**: Bypassing hooks defeats quality gates and may cause CI failures.

## Hook Dependencies

### Required Tools
- **peter-hook**: Git hooks manager
- **versioneer**: Version management
- **cargo**: Rust toolchain
- **cargo-audit**: Security scanning
- **gitleaks** (optional): Secret scanning
- **unvenv** (optional): Virtual environment detection

### Installation Script
```bash
#!/usr/bin/env bash
# Install hook dependencies

cargo install peter-hook
cargo install versioneer
cargo install cargo-audit
cargo install unvenv

# Optional
brew install gitleaks  # or appropriate package manager
```

## CI/Hook Parity

Hooks **MUST** match CI jobs to provide local validation:

| Hook | CI Job | Purpose |
|------|--------|---------|
| format-check | format | Code formatting |
| clippy-check | lint | Linting |
| test-all | test | Testing |
| security-audit | audit | Security |
| version-sync-check | N/A | Version consistency |
| tag-version-check | release validation | Tag/version match |

This ensures developers catch issues locally before pushing.
