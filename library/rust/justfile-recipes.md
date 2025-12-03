# Rust justfile Recipes and Patterns

## Git Hooks Recipes

```just
# =============================================================================
# GIT HOOKS
# =============================================================================

# Run pre-commit hooks (format-check + clippy-check)
pre-commit:
    @if command -v peter-hook >/dev/null 2>&1; then \
        peter-hook run pre-commit; \
    else \
        echo "peter-hook not found. Install with: cargo install peter-hook"; \
        exit 1; \
    fi

# Run pre-push hooks (all quality checks)
pre-push:
    @if command -v peter-hook >/dev/null 2>&1; then \
        peter-hook run pre-push; \
    else \
        echo "peter-hook not found. Install with: cargo install peter-hook"; \
        exit 1; \
    fi

# =============================================================================
# WORKFLOW RECIPES
# =============================================================================

# Code quality checks
quality: pre-commit pre-push

# Full CI pipeline
ci: quality test build-release
    @echo "Full CI pipeline complete!"

# Development workflow - quick checks before commit
dev: format pre-commit test
    @echo "Development checks complete! Ready to commit."

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

## Key Patterns

### Tool Availability Checks

Always check if required tools are installed:

```just
audit:
    @if command -v cargo-audit >/dev/null 2>&1; then \
        cargo audit; \
    else \
        echo "cargo-audit not found. Install with: cargo install cargo-audit"; \
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
        echo "Nightly toolchain required"; \
        exit 1; \
    fi
```

### Recipe Dependencies

Use dependencies for composite workflows:

```just
# Full CI pipeline
ci: quality test build-release
    @echo "Full CI pipeline complete!"

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
