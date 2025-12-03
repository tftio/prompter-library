# Issue Tracking Workflow

## Overview

**Asana** tracks high-level product work for non-engineering stakeholders.
**GitHub Issues** tracks detailed engineering tasks with exhaustive context for implementation.

Engineers maintain both systems with clear separation of concerns.

## Work Context Discovery

**Always check for `.work-metadata.toml`** in the current worktree root before performing tracking operations.

This file contains:
- Asana task URL for current work
- GitHub Project URL for engineering breakdown
- Default assignee and labels for issues/PRs

**If `.work-metadata.toml` exists**: Parse it and use values for all Asana/GitHub operations.

**If missing**: Prompt user for Asana task and GitHub Project information, then offer to create the file.

See `prompter workflow.work-metadata` for complete schema and usage details.

## Asana Work Queue

### Purpose
- High-level product requirements and features
- Non-engineer visibility into progress
- Work assignment and prioritization

### Engineer Responsibilities
1. **Monitor regularly** - Check Asana board for new high-priority work
2. **Keep current** - Update status at key checkpoints (not every commit)
3. **Avoid noise** - Do not track implementation details in Asana
4. **Link artifacts** - Add GitHub issue links and PR URLs when created

### Status Updates
Update Asana at these checkpoints only:
- When starting work (status → In Progress)
- When creating GitHub issues (add comment with GH issue links)
- When opening PRs (add comment with PR links)
- When completing work (status → Complete with summary)

## GitHub Issues

### Purpose
- Detailed engineering task breakdown
- Exhaustive implementation context
- Technical decision documentation
- Historical record for future reference

### Creating Issues from Asana

**One Asana task → Multiple GitHub issues** (typically)

Break down product requirements into discrete engineering tasks:

```markdown
# Example Asana Task
Add user authentication to API

# Corresponding GitHub Issues
- #101: Design authentication schema and token strategy
- #102: Implement JWT token generation and validation
- #103: Add authentication middleware to FastAPI
- #104: Create user login/logout endpoints
- #105: Write integration tests for auth flow
- #106: Update API documentation with auth examples
```

### Issue Content Requirements

Each GitHub issue **MUST** contain:

1. **Context** - Why this task exists, what problem it solves
2. **Design notes** - Technical approach, alternatives considered, rationale
3. **Acceptance criteria** - Specific conditions that define "done"
4. **Implementation notes** - Discoveries, decisions, obstacles encountered during work
5. **Test plan** - How to verify the implementation works
6. **Links** - Reference to parent Asana task, related issues, relevant docs

**Template**:
```markdown
## Context
Brief explanation of why this task exists and what it accomplishes.

Related Asana task: [ASANA-123](https://app.asana.com/...)

## Design
Technical approach and rationale. Include alternatives considered.

## Acceptance Criteria
- [ ] Specific testable condition 1
- [ ] Specific testable condition 2
- [ ] Documentation updated

## Implementation Notes
(Add as work progresses - decisions made, obstacles, discoveries)

## Test Plan
How to verify this works correctly.

## Dependencies
Blocks: #102, #104
Blocked by: #98
```

### Issue Lifecycle

**States**: open → in_progress → (blocked?) → closed

**Update frequently** - Treat issues as living documents:
- Add design decisions as you make them
- Document why you chose approach X over Y
- Note unexpected obstacles or discoveries
- Link to related code, docs, or external resources
- Capture retrospective notes before closing

**Close with resolution** - When closing an issue:
- Summarize what was implemented
- Link to PR(s) that resolved it
- Note any deviations from original plan
- Document any follow-up work needed

## Pull Request Workflow

### PR Creation

1. **Reference GitHub issue** in PR title and description:
   ```
   Title: Fix authentication token expiry (#103)

   Description:
   Implements JWT token validation with configurable expiry.

   Resolves #103
   ```

2. **Include in PR description**:
   - Summary of changes
   - Testing performed
   - Breaking changes (if any)
   - Screenshots/output (if relevant)

3. **Link PR back**:
   - Add PR link to GitHub issue
   - Add PR link to Asana task as comment

### PR Standards

**Structure**:
```markdown
## Summary
Brief description of what this PR accomplishes.

## Changes
- Specific change 1
- Specific change 2

## Testing
- How these changes were tested
- Commands to reproduce tests

## Related
Resolves #123
Part of Asana task: [ASANA-456](link)
```

**Review requirements**:
- All CI checks must pass
- At least one approval (if team policy requires)
- No unresolved conversations
- Clean commit history or squash strategy

### PR Merge

When PR merges:
1. Close referenced GitHub issue with resolution notes
2. If all GH issues for an Asana task are closed:
   - Update Asana task status to Complete
   - Add completion comment with summary of work

## Multi-Checkpoint Asana Updates

### Checkpoint 1: Work Start
```
Status: Not Started → In Progress
Comment: Breaking this into GitHub issues for implementation tracking.
```

### Checkpoint 2: Issues Created
```
Comment: Created engineering tasks:
- Authentication schema design: https://github.com/org/repo/issues/101
- JWT implementation: https://github.com/org/repo/issues/102
- API middleware: https://github.com/org/repo/issues/103
- Endpoints: https://github.com/org/repo/issues/104
- Tests: https://github.com/org/repo/issues/105
- Docs: https://github.com/org/repo/issues/106
```

### Checkpoint 3: PRs Opened
```
Comment: Implementation PRs:
- https://github.com/org/repo/pull/234 (schema + JWT)
- https://github.com/org/repo/pull/235 (middleware + endpoints)
- https://github.com/org/repo/pull/236 (tests + docs)
```

### Checkpoint 4: Work Complete
```
Status: In Progress → Complete
Comment: Completed. All PRs merged, authentication fully implemented.

Summary:
- Implemented JWT-based authentication with 24hr token expiry
- Added middleware to all protected endpoints
- Full test coverage including edge cases
- API docs updated with authentication examples

Delivered in PRs: #234, #235, #236
```

## Rationale and Benefits

### Why Two Systems?

**Asana** - Product perspective:
- Non-engineers need visibility without technical noise
- Focus on what is being built and when
- Business value and user impact

**GitHub Issues** - Engineering perspective:
- Developers need exhaustive context for implementation
- Technical decisions must be documented
- Future developers need historical rationale

### Why Exhaustive GitHub Issues?

- **Context preservation** - Future engineers understand why decisions were made
- **Onboarding** - New team members read issues to learn system design
- **Knowledge retention** - Prevents repeated discussions of settled decisions
- **Debugging** - Implementation notes help diagnose issues months later

### Why Multiple Checkpoints?

- **Transparency** - Stakeholders see progress without daily noise
- **Traceability** - Clear links between product requirements and implementation
- **Accountability** - Engineers demonstrate systematic approach to work

## Anti-Patterns

### DON'T: Clutter Asana with Implementation Details
```
❌ Bad Asana comment:
"Fixed bug in token validation logic where expired tokens weren't
being rejected due to timezone offset calculation error in jwt.py:142"
```

Implementation details belong in GitHub issues, not Asana.

### DON'T: Skimp on GitHub Issue Context
```
❌ Bad GitHub issue:
Title: Fix auth bug
Description: The auth thing isn't working, need to fix it.
```

Issues must have full context - what's broken, why it matters, how to verify the fix.

### DON'T: Let Issues Go Stale
```
❌ Bad practice:
Open issue for 2 weeks, then close with "Done" and no other context.
```

Update issues as you work. Future readers need to understand what happened.

### DON'T: Create One-to-One Asana/GitHub Mapping
```
❌ Bad breakdown:
Asana: "Add authentication"
GitHub: "Add authentication" (single issue)
```

Break down into discrete tasks. Monolithic issues are hard to track and review.

## Examples

### Good Asana Task
```
Title: Add user authentication to API
Description: Users need secure authentication to access protected endpoints.

Priority: High
Assignee: engineer@example.com
Status: In Progress

Comments:
- [Created] Engineering breakdown into 6 GitHub issues: #101-106
- [PR Created] Implementation in PRs: #234, #235, #236
- [Complete] All PRs merged. JWT auth implemented with 24hr tokens.
```

### Good GitHub Issue
```
Title: Implement JWT token generation and validation (#102)

## Context
Need JWT-based authentication for API. This issue covers token
generation during login and validation in middleware.

Related Asana: [Add user authentication](asana-link)
Depends on: #101 (schema design)

## Design
Using pyjwt library with HS256 algorithm. Tokens expire after 24 hours.
Considered RS256 (rejected: unnecessary complexity for single-server setup).

Secret key stored in environment variable JWT_SECRET.

## Acceptance Criteria
- [x] Generate JWT with user_id claim on successful login
- [x] Set 24hr expiry on tokens
- [x] Validate signature and expiry in middleware
- [x] Return 401 for invalid/expired tokens
- [x] Unit tests for generation and validation
- [x] Integration test for full flow

## Implementation Notes
- Initially tried 1hr expiry but got feedback it was too short
- Added optional refresh token support for future extension
- Discovered pyjwt automatically validates expiry, simplified code

## Test Plan
```bash
# Generate token
curl -X POST /auth/login -d '{"email":"test@example.com"}'

# Use token (should succeed)
curl -H "Authorization: Bearer $TOKEN" /api/protected

# Wait 24hr or manipulate token (should fail with 401)
```

## Related
- PR: https://github.com/org/repo/pull/234
- Docs: Updated API authentication guide
```

## Tool Integration

### Work Metadata Integration

**All tools should read `.work-metadata.toml`** to determine correct tracking systems:

```bash
# Example: Tool reads metadata to get Asana task and GitHub project
if [ -f .work-metadata.toml ]; then
  ASANA_TASK=$(toml get .work-metadata.toml work.asana_task)
  GH_PROJECT=$(toml get .work-metadata.toml work.github_project)
  ASSIGNEE=$(toml get .work-metadata.toml tracking.default_assignee)
  LABELS=$(toml get .work-metadata.toml tracking.default_labels)
else
  echo "Error: No .work-metadata.toml found. Run work-start first."
  exit 1
fi
```

### Using GitHub CLI
```bash
# Read metadata for defaults
ASSIGNEE=$(toml get .work-metadata.toml tracking.default_assignee 2>/dev/null || echo "")
LABELS=$(toml get .work-metadata.toml tracking.default_labels 2>/dev/null || echo "")

# Create issue with metadata defaults
gh issue create --title "Implement JWT validation" \
  --body "$(cat issue-template.md)" \
  --assignee "$ASSIGNEE" \
  --label "$LABELS"

# Link PR to issue
gh pr create --title "Fix auth token expiry (#103)" \
  --body "Resolves #103" \
  --assignee "$ASSIGNEE"

# Add comment to issue
gh issue comment 103 --body "PR created: #234"
```

### Using Asana CLI
```bash
# Read Asana task from metadata
ASANA_TASK=$(toml get .work-metadata.toml work.asana_task)
TASK_ID=$(echo "$ASANA_TASK" | grep -oE '[0-9]+$')

# Update task status
asana-cli task update "$TASK_ID" --status "In Progress"

# Add comment with links
asana-cli task comment "$TASK_ID" \
  "Created GitHub issues: #101, #102, #103"
```

## Summary

**Asana** = What and when (product view, non-engineer friendly)
**GitHub Issues** = How and why (engineering view, exhaustive context)

Update both systems at key checkpoints. Keep Asana clean. Make GitHub issues information-rich.
