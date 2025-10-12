# Plan Tracking Requirements

## GitHub Issues

- Check `.agent-metadata.toml` for GitHub project configuration (`tracking.github_project`, `tracking.default_assignee`, `tracking.default_tags`); if missing, request the relevant GitHub Project and assignee from the operator before creating issues.
- Open one issue per phase; include the phase subtasks as a checklist and apply all tags from metadata or operator (plus domain tags such as `database`, `testing`, etc.).
- Link each issue back into the corresponding section of `overview.md` and `phase_{n}.md` so the plan remains the single source of truth.

## Asana Visibility

- Check `.agent-metadata.toml` for Asana project configuration (`tracking.asana_project`); if missing, ask which Asana project should mirror the plan.
- Create an Asana task that mirrors the plan overview, including links to every phase issue.
- Add the Asana link to `overview.md` so stakeholders can jump directly to the tracker.

## Ongoing Updates

- Whenever progress is made, update GitHub issues, Asana, and the plan documents immediately—never batch these changes.
- On plan completion, mark the Asana task complete with a concise summary of the delivered work and deviations from the original assumptions.
