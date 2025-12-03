# Rust Release Reference

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
