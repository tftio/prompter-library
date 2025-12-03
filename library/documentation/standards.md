# Documentation Standards

## Format Requirements

1. All documentation **MUST** be in Github Flavored Markdown format with `.md` extension.
2. All markdown documentation **MUST** pass `markdownlint-cli2` linting: `npx markdownlint-cli2 --fix $filename`
3. All documentation **MUST** have an up-to-date entry in the `docs/map.md` document map file.

## TODO Management

Maintain a top-level `TODO.md` with the current task list for development sessions:

- Sections corresponding to plans in `doc/plans/`
- `BUGFIX:` items for identified quick fixes
- Intended as a session checklist, not a long-lived backlog
