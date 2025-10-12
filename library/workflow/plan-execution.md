# Plan Execution Workflow

## Preconditions

- Start from a **clean git status**; refuse to continue if uncommitted changes exist.
- Read `overview.md` and the active `phase_{n}.md` to restate the goal, scope, and current checkboxes before writing code.
- Compare plan state with GitHub/Asana trackers; stop and escalate if anything is inconsistent.

## Phase Delivery

1. Check out a new branch named `{plan_name}_phase_{n}` for the first incomplete phase. If it already exists, warn the operator and wait for instructions.
2. Work the subtasks sequentially. After each subtask:
   - Commit with a Conventional Commit message.
   - Run `versioneer patch` (or `minor`/`major` if the operator explicitly approves a larger bump).
   - Update the plan files, GitHub issue, and Asana ticket with results and SHAs.
3. When a phase is finished, squash the phase commits into a single commit that summarizes the delivered work.
4. For the next phase, branch from the previous phase branch (`{plan_name}_phase_{n+1}`) so the history stays chained.

## Completion

- After the final phase, consolidate the history to one comprehensive commit, merge back to the base branch, and ensure every tracker reflects the final state.
- If resuming an existing phase branch, re-read the plan, issues, and tickets to confirm nothing diverged before proceeding; escalate any mismatch immediately.
