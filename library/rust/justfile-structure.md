# Rust justfile Structure

**NOTE**: This fragment extends `prompter just.rules`. All general justfile rules apply.

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
    @echo "Setup complete!"

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
        echo "Version bumped to: $(cat VERSION)"; \
    else \
        echo "versioneer not found. Install with: cargo install versioneer"; \
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
    @echo "Clean complete!"

# Build in debug mode
build:
    @echo "Building project..."
    cargo build
    @echo "Build complete!"

# Build in release mode
build-release:
    @echo "Building project in release mode..."
    cargo build --release
    @echo "Release build complete!"

# =============================================================================
# TEST COMMANDS
# =============================================================================

# Run all tests
test:
    @echo "Running tests..."
    cargo test --all --verbose
    @echo "Tests complete!"

# Run tests with coverage (Linux only)
test-coverage:
    @echo "Running tests with coverage..."
    @if command -v cargo-tarpaulin >/dev/null 2>&1; then \
        cargo tarpaulin --all --out Html --engine llvm --timeout 300 --fail-under 80; \
        echo "Coverage report generated"; \
    else \
        echo "cargo-tarpaulin not found. Install with: cargo install cargo-tarpaulin"; \
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
        echo "Code formatted"; \
    else \
        echo "Nightly toolchain required"; \
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
        echo "Security audit passed"; \
    else \
        echo "cargo-audit not found. Install with: cargo install cargo-audit"; \
        exit 1; \
    fi

# Dependency compliance check
deny:
    @echo "Checking dependency compliance..."
    @if command -v cargo-deny >/dev/null 2>&1; then \
        cargo deny check; \
        echo "Dependency compliance check passed"; \
    else \
        echo "cargo-deny not found. Install with: cargo install cargo-deny"; \
        exit 1; \
    fi

# Check for unused dependencies
unused-deps:
    @echo "Checking for unused dependencies..."
    @if command -v cargo-machete >/dev/null 2>&1; then \
        cargo machete; \
    else \
        echo "cargo-machete not found. Install with: cargo install cargo-machete"; \
        exit 1; \
    fi

# Check for outdated dependencies
outdated:
    @echo "Checking for outdated dependencies..."
    @if command -v cargo-outdated >/dev/null 2>&1; then \
        cargo outdated --root-deps-only; \
    else \
        echo "cargo-outdated not found. Install with: cargo install cargo-outdated"; \
        exit 1; \
    fi
```

## Required Recipe Groups

1. **Setup**: `default`, `setup`, `install-hooks`
2. **Version**: `version-show`, `bump-version`, `release`
3. **Build**: `clean`, `build`, `build-release`
4. **Test**: `test`, `test-coverage`
5. **Quality**: `format`, `format-check`, `lint`, `audit`, `deny`, `unused-deps`, `outdated`
6. **Hooks**: `pre-commit`, `pre-push`
7. **Workflows**: `quality`, `ci`, `dev`
8. **Run**: `run`, `run-release`
