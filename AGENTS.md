# Repository Guidelines

## Project Structure & Module Organization
This repository is the local fragment library consumed by the `prompter` CLI. All reusable instructions live in `library/`, grouped by topic: `general/` for core agent rules, `engineering/` for workflow and versioning, language-specific guidance under `python/`, `rust/`, and `shell/`, plus domain sets such as `database/` and `documentation/`. Keep `CLAUDE.md` and `AUDIT.md` up to date—they provide the canonical overview and audit history that downstream agents rely on.

## Build, Test & Development Commands
Content is pure Markdown, so there is no compile step; instead, lint and diff before you push. Run Markdown linting locally to catch format drift:
```bash
npx markdownlint "**/*.md"
```
Use ripgrep to confirm that new rules do not duplicate existing invariants:
```bash
rg "##" library/
```
Always review `git diff` to verify line endings remain LF-only.

## Coding Style & Naming Conventions
Fragments must stay instructional: use second-person imperatives, keep tone neutral, and bold critical words (`**MUST**`, `**NEVER**`). Follow the lowercase-hyphen naming pattern already in `library/` (for example, `critical-interaction-rules.md`). Structure each file with a single H1 title, concise sections, and fenced examples when illustrating commands or code.

## Testing Guidelines
There is no automated test suite; validation is editorial. Cross-check new guidance against existing fragments to avoid contradictions, and update `AUDIT.md` when you resolve issues it cites. If you introduce command sequences, execute them in a sandbox shell first to confirm they succeed without directory state changes.

## Commit & Pull Request Guidelines
Recent history favors short, lower-case subjects (`audit`, `baseline`); keep that style but expand in the body when the change is non-trivial. Group related fragment edits into a single commit, and describe the impacted categories in bullet form. Pull requests should link the motivating issue, note any new invariants, and call out files that downstream profiles must re-render.

## Security & Configuration
The GitLab pipeline only runs secret detection, so never add credentials or personal data—scan with `rg` for obvious keys before committing. If fragment changes require config adjustments, mirror them in `~/.config/prompter/config.toml` and document the update in the PR description.
