# Planning Document Structure

## Create the Plan

- **Interrogate requirements first** – continue asking questions until you have all of:
  - The overall definition of done
  - Every technical constraint and assumption
  - External dependencies, owners, and approvals
  - Acceptance criteria provided by the operator
- **Organize by phases** – every plan must be broken into ordered phases; each phase needs its own goal, explicit definition of done, and a checklist of subtasks sized for a single agent.
- **Document rationale** – capture assumptions, risks, and any code examples that clarify the approach inside each phase file.

## Persist the Plan

- Store plans in `docs/plans/{plan_name}/`.
- Author an `overview.md` pitched to a non-technical stakeholder and include a `State:` line for every phase (`_todo_`, `_in-progress_`, `_completed_`, `_failed_`).
- Create `phase_{n}.md` files per phase. Each file must include, in order: an explanation section, a rationale section, a concise brief, and a TODO checklist with `[ ] completed` items for every subtask.
- Whenever you reference external artifacts (issues, tickets, code reviews), link them directly from the relevant phase or overview section.

## Audience Expectations

- Write for a senior software engineer collaborating with an LLM agent—be explicit about goals, acceptance criteria, and hand-offs so execution never relies on implied context.
