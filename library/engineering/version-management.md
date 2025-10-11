# 🚨 CRITICAL VERSION MANAGEMENT RULES 🚨

**INVARIANT VERSION SYNCHRONIZATION RULES - NEVER VIOLATE THESE:**

1. **VERSION and the project manifest (pyproject.toml, Cargo.toml) MUST ALWAYS be updated in sync** - Both files contain version information and MUST match
2. **NEVER update only one version file** - Any version change requires updating both files simultaneously
3. **VERSION format** → Semantic versioning (e.g., "0.0.2")
4. **pyproject.toml version field** → Must match VERSION file exactly
5. **Infrastructure uses VERSION file** → For resource tagging and deployment versioning
6. **Python package uses pyproject.toml** → For package management and distribution

When possible, use the `versioneer` tool to set versions -- it should be installed in $HOME/.local/bin, which is on the path. It updates VERSION and the Cargo.toml or pyproject.toml files in sync, and optionaly adds git tags.

These rules are **INVARIANT** and **NON-NEGOTIABLE**. Version mismatches cause deployment inconsistencies.
