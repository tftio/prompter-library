# Rust justfile Patterns

**NOTE**: This fragment extends `just/rules.md`. All general justfile rules apply.

## Standard Rust justfile Structure

All Rust projects **MUST** include a justfile with these standard recipes:

```just
# Project - Development Workflow
# Requires: just, peter-hook, versioneer

# Default recipe to display available commands
default:
    @just --list

# Setup development environment
setup:
    @echo "Setting up project development environment..."
    @just install-hooks
    @echo "✅ Setup complete!"

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

# =============================================================================
# VERSION MANAGEMENT
# =============================================================================

# Show current version
version-show:
    @echo "Current version: $(cat VERSION)"
    @echo "Cargo.toml version: $(grep '^version' Cargo.toml | cut -d'"' -f2)"

# Bump version (patch|minor|major)
bump-version level:
    @echo "Bumping {{ level }} version..."
    @if command -v versioneer >/dev/null 2>&1; then \
        versioneer {{ level }}; \
        echo "✅ Version bumped to: $(cat VERSION)"; \
    else \
        echo "❌ versioneer not found. Install with: cargo install versioneer"; \
        exit 1; \
    fi

# Release workflow with comprehensive validation (see rust/release-workflow.md)
release level:
    #!/usr/bin/env bash
    set -euo pipefail
    # [Full release script from rust/release-workflow.md]

# =============================================================================
# BUILD COMMANDS
# =============================================================================

# Clean build artifacts
clean:
    @echo "Cleaning build artifacts..."
    cargo clean
    @rm -rf target/
    @echo "✅ Clean complete!"

# Build in debug mode
build:
    @echo "Building project..."
    cargo build
    @echo "✅ Build complete!"

# Build in release mode
build-release:
    @echo "Building project in release mode..."
    cargo build --release
    @echo "✅ Release build complete!"

# =============================================================================
# TEST COMMANDS
# =============================================================================

# Run all tests
test:
    @echo "Running tests..."
    cargo test --all --verbose
    @echo "✅ Tests complete!"

# Run tests with coverage (Linux only)
test-coverage:
    @echo "Running tests with coverage..."
    @if command -v cargo-tarpaulin >/dev/null 2>&1; then \
        cargo tarpaulin --all --out Html --engine llvm --timeout 300 --fail-under 80; \
        echo "✅ Coverage report generated"; \
    else \
        echo "❌ cargo-tarpaulin not found. Install with: cargo install cargo-tarpaulin"; \
        exit 1; \
    fi

# =============================================================================
# QUALITY COMMANDS
# =============================================================================

# Format code (requires nightly)
format:
    @echo "Formatting code..."
    @if rustup toolchain list | grep -q nightly; then \
        cargo +nightly fmt; \
        echo "✅ Code formatted"; \
    else \
        echo "❌ Nightly toolchain required"; \
        echo "Install with: rustup install nightly"; \
        exit 1; \
    fi

# Check code formatting
format-check:
    @just pre-commit

# Lint code with clippy
lint:
    @just pre-commit

# Security audit
audit:
    @echo "Running security audit..."
    @if command -v cargo-audit >/dev/null 2>&1; then \
        cargo audit; \
        echo "✅ Security audit passed"; \
    else \
        echo "❌ cargo-audit not found. Install with: cargo install cargo-audit"; \
        exit 1; \
    fi

# Dependency compliance check
deny:
    @echo "Checking dependency compliance..."
    @if command -v cargo-deny >/dev/null 2>&1; then \
        cargo deny check; \
        echo "✅ Dependency compliance check passed"; \
    else \
        echo "❌ cargo-deny not found. Install with: cargo install cargo-deny"; \
        exit 1; \
    fi

# Check for unused dependencies
unused-deps:
    @echo "Checking for unused dependencies..."
    @if command -v cargo-machete >/dev/null 2>&1; then \
        cargo machete; \
    else \
        echo "❌ cargo-machete not found. Install with: cargo install cargo-machete"; \
        exit 1; \
    fi

# Check for outdated dependencies
outdated:
    @echo "Checking for outdated dependencies..."
    @if command -v cargo-outdated >/dev/null 2>&1; then \
        cargo outdated --root-deps-only; \
    else \
        echo "❌ cargo-outdated not found. Install with: cargo install cargo-outdated"; \
        exit 1; \
    fi

# =============================================================================
# GIT HOOKS
# =============================================================================

# Run pre-commit hooks (format-check + clippy-check)
pre-commit:
    @if command -v peter-hook >/dev/null 2>&1; then \
        peter-hook run pre-commit; \
    else \
        echo "❌ peter-hook not found. Install with: cargo install peter-hook"; \
        exit 1; \
    fi

# Run pre-push hooks (all quality checks)
pre-push:
    @if command -v peter-hook >/dev/null 2>&1; then \
        peter-hook run pre-push; \
    else \
        echo "❌ peter-hook not found. Install with: cargo install peter-hook"; \
        exit 1; \
    fi

# =============================================================================
# WORKFLOW RECIPES
# =============================================================================

# Code quality checks
quality: pre-commit pre-push

# Full CI pipeline
ci: quality test build-release
    @echo "✅ Full CI pipeline complete!"

# Development workflow - quick checks before commit
dev: format pre-commit test
    @echo "✅ Development checks complete! Ready to commit."

# =============================================================================
# RUN COMMANDS
# =============================================================================

# Run the built binary
run *args:
    cargo run -- {{ args }}

# Run the binary with release optimizations
run-release *args:
    cargo run --release -- {{ args }}
```

## Recipe Organization

### Required Recipe Groups

1. **Setup**: `default`, `setup`, `install-hooks`
2. **Version**: `version-show`, `bump-version`, `release`
3. **Build**: `clean`, `build`, `build-release`
4. **Test**: `test`, `test-coverage`
5. **Quality**: `format`, `format-check`, `lint`, `audit`, `deny`, `unused-deps`, `outdated`
6. **Hooks**: `pre-commit`, `pre-push`
7. **Workflows**: `quality`, `ci`, `dev`
8. **Run**: `run`, `run-release`

## Key Patterns

### Tool Availability Checks

Always check if required tools are installed:

```just
audit:
    @if command -v cargo-audit >/dev/null 2>&1; then \
        cargo audit; \
    else \
        echo "❌ cargo-audit not found. Install with: cargo install cargo-audit"; \
        exit 1; \
    fi
```

### Conditional Execution

Use conditionals for platform-specific or optional features:

```just
format:
    @if rustup toolchain list | grep -q nightly; then \
        cargo +nightly fmt; \
    else \
        echo "❌ Nightly toolchain required"; \
        exit 1; \
    fi
```

### Recipe Dependencies

Use dependencies for composite workflows:

```just
# Full CI pipeline
ci: quality test build-release
    @echo "✅ Full CI pipeline complete!"

# Quality checks depend on pre-commit and pre-push
quality: pre-commit pre-push
```

### Shebang Recipes for Complex Logic

Use shebang for multi-step recipes:

```just
release level:
    #!/usr/bin/env bash
    set -euo pipefail

    # Validate input
    case "{{ level }}" in
        patch|minor|major) ;;
        *) echo "Invalid: {{ level }}"; exit 1 ;;
    esac

    # Multi-step release process
    # ...
```

## Standard Workflows

### Developer Workflow
```bash
# Quick pre-commit check
just dev

# Full local validation
just quality

# Run tests
just test
```

### Release Workflow
```bash
# Patch release
just release patch

# Minor release
just release minor

# Major release
just release major
```

### CI Simulation
```bash
# Run complete CI pipeline locally
just ci
```

## Integration Points

### With peter-hook
```just
pre-commit:
    peter-hook run pre-commit

pre-push:
    peter-hook run pre-push
```

### With versioneer
```just
version-show:
    @echo "Current version: $(cat VERSION)"

bump-version level:
    versioneer {{ level }}

release level:
    # Uses versioneer for version bumps and tagging
```

### With GitHub Actions
- `just ci` mirrors GitHub Actions CI workflow
- `just pre-commit` mirrors format and lint jobs
- `just pre-push` mirrors comprehensive validation
- `just test` mirrors test job
- `just audit` mirrors security audit job

## Best Practices

### Error Handling
- Use `set -euo pipefail` in shebang recipes
- Check tool availability before use
- Provide clear error messages with installation instructions
- Exit with non-zero status on errors

### Output Clarity
- Use emojis for status (✅ ❌ ⚠️)
- Echo step descriptions before actions
- Suppress noise with `@` prefix
- Show progress for long-running operations

### Consistency
- Keep recipe names consistent across projects
- Match CI job names where applicable
- Use same parameters and flags as CI
- Maintain alphabetical or logical grouping

### Documentation
- Add comments for recipe groups
- Include usage examples in comments
- Document required tools in header
- Reference related documentation fragments

## Project-Specific Customization

### Binary Name
Replace `projectname` with actual binary name:
```just
run *args:
    cargo run -- {{ args }}
```

### Tag Format
Update tag format in release recipe:
```just
versioneer tag --tag-format "yourproject-v{version}"
```

### Additional Recipes
Add project-specific recipes as needed:
```just
# Install project locally
install:
    cargo install --path .

# Generate documentation
docs:
    cargo doc --no-deps --open
```

## Maintenance

### Keep Synchronized With
- GitHub Actions workflows (ci.yml)
- Git hooks configuration (hooks.toml)
- Project documentation (CLAUDE.md)
- Quality tool configurations

### Update When
- Adding new quality tools
- Changing CI pipeline
- Updating version management
- Adding new project commands
