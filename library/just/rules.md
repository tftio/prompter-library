# Justfile Rules

**Invariant rules for justfile targets:**

- **Write shell scripts as separate files** – Heredocs in justfiles create maintenance burden.
- **Call Python modules, not inline code** – `python -c` hides logic and breaks linting.
- **Reference external scripts from targets** – Inline script generation bypasses version control.
- **Keep justfile targets simple** – Targets should call external scripts/functions.
- **Prefix internal recipes with `_`** – Recipes used only as dependencies belong in `dependent_only` group.
  - Examples: `_get-env-config`, `_get-tenant-config`

**These rules take primacy over convenience.**
