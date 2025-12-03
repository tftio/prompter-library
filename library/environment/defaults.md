# Environment Defaults

## Platform Assumptions

1. **Deploy to AWS by default** - Assume multi-region capability unless the operator specifies another provider.
2. **Support macOS and Linux workflows** - Scripts, Makefiles, and CLI flows must run on both platforms.
3. **Pin infrastructure dependencies to current stable releases** - Verify PostgreSQL, Docker, and platform services match approved versions.

## Required Tooling

1. **Use `just` as the command runner** - Add project recipes to root `justfile`; avoid makefiles and ad-hoc scripts.
2. **Manage Python dependencies with `uv`** - Use uv exclusively; pip, poetry, pdm require operator exception.
3. **Handle version bumps with `versioneer`** - Run for every release to keep VERSION and manifests in sync.
4. **Install required environment tools globally** - Ensures linting and validation succeed without manual setup.

## Required Global Tools

| Tool | Purpose |
|------|---------|
| `shellcheck` | Shell script linting |
| `shfmt` | Shell script formatting |
| `ruff` | Python linting and formatting |
| `mypy` | Python type checking (production) |
| `ty` | Python type checking (experimental, Astral) |
| `interrogate` | Python docstring coverage |
