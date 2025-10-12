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
