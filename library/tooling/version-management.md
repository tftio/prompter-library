# Version Management

## CRITICAL VERSION SYNCHRONIZATION RULES

**INVARIANT – NEVER VIOLATE THESE:**

1. **Keep VERSION and the project manifest in sync** – `VERSION` and `pyproject.toml`/`Cargo.toml` MUST match exactly.
2. **Never touch one without the other** – Any version bump requires updating every file that stores the version string.
3. **Enforce SEMVER format** – Always use `MAJOR.MINOR.PATCH` (e.g., `0.0.2`).
4. **Protect downstream automation** – Infra tagging and release tooling read `VERSION`; package managers read the manifest. Drift breaks deployments.
5. **Use `versioneer` for all bumps** – Manual edits invite mistakes; the tool updates files atomically and can tag releases.

## Versioneer Usage

`versioneer` lives in `$HOME/.local/bin` and updates all version files together:

```bash
versioneer patch  # 0.0.1 → 0.0.2 (bug fixes, docs-only changes)
versioneer minor  # 0.0.2 → 0.1.0 (new features, backward compatible)
versioneer major  # 0.1.0 → 1.0.0 (breaking changes)
versioneer tag    # Create a git tag that matches VERSION
```

**Before running `minor` or `major`, warn the operator** so they can confirm release scope.

## Semantic Versioning

Follow [SEMVER](https://semver.org/) strictly:
- **PATCH** – Fixes, refactors, or documentation with no behavioral change.
- **MINOR** – New functionality that remains backward compatible.
- **MAJOR** – Breaking changes or contract updates.

## Version Synchronization Check

Verify files immediately after bumping:

```bash
cat VERSION
grep version pyproject.toml  # or Cargo.toml
```

Both outputs must match. If they diverge, rerun `versioneer` before committing.

## Rust-Specific Version Management

### VERSION File + Cargo.toml Synchronization

Rust projects **MUST** maintain synchronization between VERSION file and Cargo.toml:

```bash
# Verify synchronization
versioneer verify

# Synchronize if needed
versioneer sync
```

### Automated Release Workflow

Use `just release` for complete automated releases:

```bash
# Patch release (bug fixes)
just release patch

# Minor release (new features, backward compatible)
just release minor

# Major release (breaking changes)
just release major
```

### Release Workflow Stages

1. **Pre-flight checks**: Clean working directory, on main branch, synced with remote
2. **Quality gates**: Tests, security audit, dependency checks, linting
3. **Version bump**: Atomic update of VERSION and Cargo.toml via versioneer
4. **Commit creation**: Single commit with version changes
5. **Tag creation**: Git tag matching version via versioneer tag
6. **Push**: Push commit and tag to trigger GitHub Actions

### Git Hooks for Version Validation

Pre-push hooks **MUST** verify:

```bash
# Version synchronization
versioneer verify

# Tag/version consistency (when tagged)
if git describe --exact-match --tags HEAD 2>/dev/null | grep -q 'projectname-v'; then
    TAG_VERSION=$(git describe --exact-match --tags HEAD | sed 's/projectname-v//')
    CARGO_VERSION=$(versioneer show)
    if [ "$TAG_VERSION" != "$CARGO_VERSION" ]; then
        echo "ERROR: Tag version mismatch"
        exit 1
    fi
fi
```

### Tag Format for Rust Projects

Standard format: `{project-name}-v{version}`

Examples:
- `prompter-v1.5.5`
- `versioneer-v0.2.0`
- `peter-hook-v0.3.1`

Create tags with versioneer:

```bash
versioneer tag --tag-format "projectname-v{version}"
```

### GitHub Actions Version Validation

Release workflows **MUST** validate version consistency:

```yaml
- name: Validate version consistency
  shell: bash
  run: |
    TAG_VERSION=$(echo "${{ steps.version.outputs.version }}" | sed 's/projectname-v//')
    CARGO_VERSION=$(grep '^version = ' Cargo.toml | head -1 | sed 's/version = "\(.*\)"/\1/')
    if [ "$TAG_VERSION" != "$CARGO_VERSION" ]; then
      echo "ERROR: Version mismatch"
      exit 1
    fi
```

### Rust Version Management Rules

1. **NEVER manually edit Cargo.toml version** - Use versioneer only
2. **NEVER manually edit VERSION file** - Use versioneer only
3. **NEVER create git tags manually** - Use versioneer tag or just release
4. **ALWAYS use just release** - Ensures quality gates and atomic updates
5. **ALWAYS verify synchronization** - Run versioneer verify before commits
6. **ALWAYS use project-specific tag format** - Match your project name in tags

### Multi-Layer Validation

Rust projects enforce version consistency at three layers:

1. **Git Hooks (Local)**: Pre-push validation of version sync and tag consistency
2. **justfile (Automation)**: Release script validates and creates atomic commits
3. **GitHub Actions (CI)**: Release workflow validates before building binaries

This prevents tag/version mismatches that break releases.
