# Rust Automated Release Workflow

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

## Release Automation with justfile

### Release Recipe Pattern

```just
# Release workflow with comprehensive validation
release level:
    #!/usr/bin/env bash
    set -euo pipefail

    # Validate bump type
    case "{{ level }}" in
        patch|minor|major) ;;
        *)
            echo "❌ Invalid bump type: {{ level }}"
            echo "Usage: just release [patch|minor|major]"
            exit 1
            ;;
    esac

    echo "🚀 Starting release workflow for projectname..."

    # Step 1: Prerequisites validation
    echo "Step 1: Validating prerequisites..."

    if ! command -v versioneer >/dev/null 2>&1; then
        echo "❌ versioneer is required but not installed"
        exit 1
    fi

    if ! git rev-parse --git-dir >/dev/null 2>&1; then
        echo "❌ Not in a git repository"
        exit 1
    fi

    # Check working directory is clean
    if ! git diff-index --quiet HEAD --; then
        echo "❌ Working directory is not clean"
        git status --short
        exit 1
    fi

    # Check we're on main branch
    CURRENT_BRANCH=$(git branch --show-current)
    if [ "$CURRENT_BRANCH" != "main" ]; then
        echo "❌ Must be on main branch (currently on: $CURRENT_BRANCH)"
        exit 1
    fi

    # Check we're up to date with remote
    git fetch origin main
    LOCAL=$(git rev-parse HEAD)
    REMOTE=$(git rev-parse origin/main)
    if [ "$LOCAL" != "$REMOTE" ]; then
        echo "❌ Local main not up-to-date with origin/main"
        exit 1
    fi

    # Verify version files are synchronized
    if ! versioneer verify >/dev/null 2>&1; then
        echo "❌ Version files are not synchronized"
        exit 1
    fi

    CURRENT_VERSION=$(cat VERSION)
    echo "✅ Prerequisites validated"
    echo "   Current version: $CURRENT_VERSION"

    # Step 2: Quality gates
    echo "Step 2: Running quality gates..."
    just test
    just audit
    just deny
    just pre-commit
    echo "✅ All quality gates passed"

    # Step 3: Version management
    echo "Step 3: Bumping {{ level }} version..."
    versioneer {{ level }}
    NEW_VERSION=$(cat VERSION)

    if ! versioneer verify >/dev/null 2>&1; then
        echo "❌ Version synchronization failed after bump"
        exit 1
    fi

    echo "✅ Version bumped: $CURRENT_VERSION → $NEW_VERSION"

    # Step 4: Create commit
    echo "Step 4: Committing changes..."
    git add Cargo.toml Cargo.lock VERSION
    git commit -m "chore: bump version to $NEW_VERSION"
    echo "✅ Changes committed"

    # Step 5: Create tag
    echo "Step 5: Creating git tag..."
    versioneer tag --tag-format "projectname-v{version}"

    TAG_VERSION=$(git describe --exact-match --tags HEAD | sed "s/projectname-v//")
    if [ "$TAG_VERSION" != "$NEW_VERSION" ]; then
        echo "❌ Tag version mismatch"
        exit 1
    fi
    echo "✅ Created tag: projectname-v$NEW_VERSION"

    # Step 6: Interactive confirmation
    echo ""
    echo "📋 Release Summary:"
    echo "   Version: $NEW_VERSION"
    echo "   Tag: projectname-v$NEW_VERSION"
    echo ""

    if [ -t 0 ]; then
        echo -n "Push release to GitHub? [y/N]: "
        read -r response
        case "$response" in
            [yY]|[yY][eE][sS]) ;;
            *)
                echo "ℹ️  Release prepared but not pushed"
                exit 0
                ;;
        esac
    fi

    # Step 7: Push
    echo "Step 6: Pushing to GitHub..."
    git push origin main
    git push --tags
    echo "🎉 Release $NEW_VERSION pushed successfully!"
```

### Usage

```bash
# Patch release (1.0.0 -> 1.0.1)
just release patch

# Minor release (1.0.0 -> 1.1.0)
just release minor

# Major release (1.0.0 -> 2.0.0)
just release major
```

## Version Management Rules

### Critical Rules (NEVER VIOLATE)
1. **Never manually edit Cargo.toml version** - Use versioneer
2. **Never create git tags manually** - Use versioneer tag or release script
3. **Always use automated release workflow** - Prevents version/tag mismatches
4. **Always run quality gates before version bump** - Ensures release quality

### Version Synchronization
- VERSION file is single source of truth
- Cargo.toml synchronized via versioneer
- Git tags created by versioneer with project-specific format
- All three must match exactly

## Quality Gates

### Pre-Release Validation
All of these **MUST** pass before version bump:

```bash
# Run tests
just test

# Security audit
just audit

# Dependency compliance
just deny

# Format and lint checks
just pre-commit
```

### Post-Bump Validation
```bash
# Verify synchronization
versioneer verify

# Verify tag matches version
TAG_VERSION=$(git describe --exact-match --tags HEAD | sed "s/projectname-v//")
CARGO_VERSION=$(versioneer show)
[ "$TAG_VERSION" = "$CARGO_VERSION" ]
```

## GitHub Actions Integration

### Tag-Triggered Release

When tag is pushed, GitHub Actions:
1. Validates tag version matches Cargo.toml
2. Creates draft release with version in title
3. Builds binaries for all platforms
4. Re-validates version before each build
5. Uploads binaries with SHA256 checksums
6. Publishes release

### Version Validation in CI

```yaml
- name: Validate version consistency
  shell: bash
  run: |
    TAG_VERSION=$(echo "${{ steps.version.outputs.version }}" | sed 's/projectname-v//')
    CARGO_VERSION=$(grep '^version = ' Cargo.toml | head -1 | sed 's/version = "\(.*\)"/\1/')
    if [ "$TAG_VERSION" != "$CARGO_VERSION" ]; then
      echo "❌ ERROR: Version mismatch"
      exit 1
    fi
```

## Tag Format Pattern

### Standard Format
```
{project-name}-v{version}
```

### Examples
- prompter: `prompter-v1.5.5`
- versioneer: `versioneer-v0.2.0`
- peter-hook: `peter-hook-v0.3.1`

### versioneer Tag Creation
```bash
# Automatic tag with format
versioneer tag --tag-format "projectname-v{version}"

# Reads VERSION file and creates matching tag
# Example: VERSION contains "1.0.0" → creates "projectname-v1.0.0"
```

## Release Checklist

Before initiating release:
- [ ] All changes committed
- [ ] Working directory clean
- [ ] On main branch
- [ ] Synced with origin/main
- [ ] VERSION and Cargo.toml synchronized
- [ ] All tests passing
- [ ] No security vulnerabilities
- [ ] Dependency compliance verified
- [ ] Format and lint checks passing

After release:
- [ ] GitHub Actions triggered
- [ ] All platform builds succeeded
- [ ] Draft release created
- [ ] Binaries uploaded with checksums
- [ ] Release published (manually or automatically)

## Troubleshooting

### Version Mismatch Errors
```bash
# Check current state
versioneer show
cat VERSION
grep '^version' Cargo.toml

# Fix synchronization
versioneer sync

# Verify
versioneer verify
```

### Tag Conflicts
```bash
# Delete incorrect local tag
git tag -d projectname-v1.0.0

# Delete incorrect remote tag (use carefully!)
git push origin :refs/tags/projectname-v1.0.0

# Recreate with versioneer
versioneer tag --tag-format "projectname-v{version}"
```

### Failed Release
```bash
# If push failed after tag creation, can retry
git push origin main
git push --tags

# GitHub Actions will trigger on tag push
```

### Rollback Release
```bash
# If release hasn't been pushed yet
git reset --hard origin/main

# If release was pushed
# 1. Delete the GitHub release via web UI
# 2. Delete remote tag: git push origin :refs/tags/projectname-v1.0.0
# 3. Reset to previous version: git reset --hard <previous-commit>
# 4. Force push: git push origin main --force
```

## Integration with CI/CD

### Workflow Dependencies
- **versioneer**: Version management tool
- **peter-hook**: Git hooks for local validation
- **GitHub Actions**: CI/CD platform
- **taiki-e/upload-rust-binary-action**: Binary distribution

### Environment Variables
- `GITHUB_TOKEN`: For creating releases and uploading assets
- `CODECOV_TOKEN`: For test coverage reporting (optional)

### Secrets Management
- Store `GITHUB_TOKEN` as repository secret
- Store `CODECOV_TOKEN` if using Codecov

## Benefits

### Multi-Layer Validation Prevents:
- Tag/version mismatches in releases
- Manual version management errors
- Releases with failing tests
- Releases with security vulnerabilities
- Incomplete or broken releases

### Automation Provides:
- Consistent release process
- Atomic version updates
- Comprehensive quality gates
- Cross-platform binary distribution
- Automatic changelog generation
