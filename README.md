# Prompter Library Overview

This repository is the home-grown knowledge base that the `prompter` CLI assembles into working instructions for coding agents. Every fragment under `library/` is a terse, imperative rule set; profiles in `~/.config/prompter/config.toml` compose those fragments for different workflows (full-stack development, CLI tooling, architecture planning, etc.). You run `prompter <profile>` to render the full instruction pack for an agent, so keeping these fragments consistent is critical.

## Fragment Layout

```
library/
├── environment/      # Platform and tooling defaults (AWS, macOS/Linux, just, uv, versioneer)
├── engineering/      # Cross-cutting engineering principles
├── tooling/          # Language-agnostic CLI and version-management standards
├── workflow/         # Plan execution/tracking processes
├── shell/            # Shell error handling, portability, and style guides
├── documentation/    # Planning and documentation structure
├── database/, python/, rust/, general/, just/ ... (domain-specific rules)
```

Fragments are Markdown files with level‑1 titles and bold directives (e.g., `**MUST**`, `**NEVER**`). Split them when a topic grows beyond a single concern; keep examples short and executable.

## Daily Operations

- **Render instructions**: `prompter full-stack.backend`, `prompter cli.rust`, `prompter devops.architecture`, or any other profile prints the composed guidance for that session.
- **Validate profiles**: `prompter validate` ensures the config references existing fragments.
- **Add or move fragments**: update the `depends_on` list in `~/.config/prompter/config.toml`, refresh `CLAUDE.md` for operator context, and log the change in `AUDIT.md`.
- **Shell command convention**: always run commands with explicit paths (`{"command": ["bash","-lc", "..."], "workdir": "/Users/jfb/.local/prompter"}`) so we never mutate directory state.

## Contribution Checklist

1. Write or edit fragments using imperative language; standardize emphasis with `**MUST**`, `**NEVER**`, etc.
2. Keep fragments self-contained—no cross-references that assume prior context.
3. Update `AUDIT.md` when you close or add an item; record verification commands.
4. Run `prompter validate` and spot-check key profiles (`full-stack.backend`, `cli.rust`, `devops.deployment`, `planning.full-stack`).
5. Coordinate with the operator before introducing new categories or removing existing ones.

The goal is predictable, composable instruction sets that agents can consume without guesswork. Treat every fragment as production documentation.
