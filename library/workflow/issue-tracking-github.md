# GitHub Issues Workflow

## Purpose
- Detailed engineering task breakdown
- Exhaustive implementation context
- Technical decision documentation
- Historical record for future reference

## Creating Issues from Asana

**One Asana task -> Multiple GitHub issues** (typically)

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

## Issue Content Requirements

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

## Issue Lifecycle

**States**: open -> in_progress -> (blocked?) -> closed

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
