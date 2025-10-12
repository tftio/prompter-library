# 🚨 CRITICAL JUSTFILE RULES 🚨

**INVARIANT JUSTFILE RULES - NEVER VIOLATE THESE:**

- **NEVER use heredocs** in justfile targets to create inline scripts
- **NEVER use `python -c`** inline code in justfile targets
- **NEVER create and execute scripts** within justfile targets
- **IF you need shell logic** - create separate shell scripts in `scripts/`
- **Keep justfile targets simple** - they should call external scripts/functions
- **DEPENDENCY-ONLY RECIPES** that are used exclusively as dependencies by other recipes MUST:
   - Be prefixed with `_` in their name to indicate they are internal/dependency-only
   - Be placed in a group called `dependent_only`
   - Examples: `_get-env-config`, `_get-tenant-config`

These rules are **INVARIANT** and **NON-NEGOTIABLE**. Any violation is unacceptable.
