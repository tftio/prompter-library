# Rust Release Automation

## Overview

All Rust projects **MUST** use automated release workflows with multi-layer version validation to prevent tag/version mismatches.

## Release Workflow Layers

### Layer 1: Git Hooks (Local Validation)
- Pre-push hooks verify version synchronization
- Tag/version consistency checked before push
- Implemented via peter-hook (see rust/git-hooks.md)

### Layer 2: GitHub Actions (CI Validation)
- Release workflow validates tag version matches Cargo.toml
- Build jobs re-validate before binary creation
- Implemented in .github/workflows/release.yml

### Layer 3: Automated Script (Process Automation)
- justfile release recipe orchestrates entire workflow
- Runs quality gates before version bumps
- Creates atomic commits with version + tag

## Release Recipe Pattern

```just
# Release workflow with comprehensive validation
release level:
    #!/usr/bin/env bash
    set -euo pipefail

    # Validate bump type
    case "{{ level }}" in
        patch|minor|major) ;;
        *)
            echo "Invalid bump type: {{ level }}"
            echo "Usage: just release [patch|minor|major]"
            exit 1
            ;;
    esac

    echo "Starting release workflow for projectname..."

    # Step 1: Prerequisites validation
    echo "Step 1: Validating prerequisites..."

    if ! command -v versioneer >/dev/null 2>&1; then
        echo "versioneer is required but not installed"
        exit 1
    fi

    if ! git rev-parse --git-dir >/dev/null 2>&1; then
        echo "Not in a git repository"
        exit 1
    fi

    # Check working directory is clean
    if ! git diff-index --quiet HEAD --; then
        echo "Working directory is not clean"
        git status --short
        exit 1
    fi

    # Check we're on main branch
    CURRENT_BRANCH=$(git branch --show-current)
    if [ "$CURRENT_BRANCH" != "main" ]; then
        echo "Must be on main branch (currently on: $CURRENT_BRANCH)"
        exit 1
    fi

    # Check we're up to date with remote
    git fetch origin main
    LOCAL=$(git rev-parse HEAD)
    REMOTE=$(git rev-parse origin/main)
    if [ "$LOCAL" != "$REMOTE" ]; then
        echo "Local main not up-to-date with origin/main"
        exit 1
    fi

    # Verify version files are synchronized
    if ! versioneer verify >/dev/null 2>&1; then
        echo "Version files are not synchronized"
        exit 1
    fi

    CURRENT_VERSION=$(cat VERSION)
    echo "Prerequisites validated"
    echo "   Current version: $CURRENT_VERSION"

    # Step 2: Quality gates
    echo "Step 2: Running quality gates..."
    just test
    just audit
    just deny
    just pre-commit
    echo "All quality gates passed"

    # Step 3: Version management
    echo "Step 3: Bumping {{ level }} version..."
    versioneer {{ level }}
    NEW_VERSION=$(cat VERSION)

    if ! versioneer verify >/dev/null 2>&1; then
        echo "Version synchronization failed after bump"
        exit 1
    fi

    echo "Version bumped: $CURRENT_VERSION -> $NEW_VERSION"

    # Step 4: Create commit
    echo "Step 4: Committing changes..."
    git add Cargo.toml Cargo.lock VERSION
    git commit -m "chore: bump version to $NEW_VERSION"
    echo "Changes committed"

    # Step 5: Create tag
    echo "Step 5: Creating git tag..."
    versioneer tag --tag-format "projectname-v{version}"

    TAG_VERSION=$(git describe --exact-match --tags HEAD | sed "s/projectname-v//")
    if [ "$TAG_VERSION" != "$NEW_VERSION" ]; then
        echo "Tag version mismatch"
        exit 1
    fi
    echo "Created tag: projectname-v$NEW_VERSION"

    # Step 6: Interactive confirmation
    echo ""
    echo "Release Summary:"
    echo "   Version: $NEW_VERSION"
    echo "   Tag: projectname-v$NEW_VERSION"
    echo ""

    if [ -t 0 ]; then
        echo -n "Push release to GitHub? [y/N]: "
        read -r response
        case "$response" in
            [yY]|[yY][eE][sS]) ;;
            *)
                echo "Release prepared but not pushed"
                exit 0
                ;;
        esac
    fi

    # Step 7: Push
    echo "Step 6: Pushing to GitHub..."
    git push origin main
    git push --tags
    echo "Release $NEW_VERSION pushed successfully!"
```

## Usage

```bash
# Patch release (1.0.0 -> 1.0.1)
just release patch

# Minor release (1.0.0 -> 1.1.0)
just release minor

# Major release (1.0.0 -> 2.0.0)
just release major
```
