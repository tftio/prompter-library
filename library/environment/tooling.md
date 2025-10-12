# Environment Tooling

1. **Use `just` as the command runner** – add new project recipes to the root `justfile`; do not introduce makefiles or ad-hoc scripts.
2. **Manage Python dependencies with `uv`** – never fall back to pip, poetry, or pdm unless the operator grants an exception.
3. **Handle version bumps with `versioneer`** – run the tool for every release increment so `VERSION` and manifests stay in sync.
4. **Install required environment tools globally** (`shellcheck`, `ruff`, `ty`, `interrogate`) so linting and validation commands succeed without manual setup on each run.
