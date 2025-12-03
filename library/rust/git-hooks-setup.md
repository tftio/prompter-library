# Rust Git Hooks Setup

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

## Integration with justfile

Add helper recipes to justfile:

```just
# Install git hooks using peter-hook
install-hooks:
    @echo "Installing git hooks with peter-hook..."
    @if command -v peter-hook >/dev/null 2>&1; then \
        peter-hook install; \
        echo "Git hooks installed"; \
    else \
        echo "peter-hook not found. Install with: cargo install peter-hook"; \
        exit 1; \
    fi

# Run pre-commit hooks manually
pre-commit:
    @if command -v peter-hook >/dev/null 2>&1; then \
        peter-hook run pre-commit; \
    else \
        echo "peter-hook not found"; \
        exit 1; \
    fi

# Run pre-push hooks manually
pre-push:
    @if command -v peter-hook >/dev/null 2>&1; then \
        peter-hook run pre-push; \
    else \
        echo "peter-hook not found"; \
        exit 1; \
    fi
```

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
