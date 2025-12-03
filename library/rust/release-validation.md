# Rust Release Validation

## Version Management Rules

### Critical Rules
1. **Use versioneer for Cargo.toml version** - Ensures atomic updates
2. **Use versioneer tag or release script for git tags** - Ensures consistency
3. **Use automated release workflow** - Prevents version/tag mismatches
4. **Run quality gates before version bump** - Ensures release quality

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
      echo "ERROR: Version mismatch"
      exit 1
    fi
```

## Tag Format Pattern

### Standard Format
```text
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
# Example: VERSION contains "1.0.0" -> creates "projectname-v1.0.0"
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
