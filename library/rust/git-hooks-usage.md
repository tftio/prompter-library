# Rust Git Hooks Usage

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
```text
1. format-check runs (cargo fmt --check)
   | success
2. clippy-check runs (cargo clippy)
   | success
3. Commit proceeds
```

### On Push
```text
1. clippy-check runs
   | success
2. test-all runs
   | success
3. venv-check runs
   | success
4. security-audit runs
   | success
5. secret-scan runs
   | success
6. version-sync-check runs
   | success
7. tag-version-check runs (if tagged)
   | success
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
