# Issue Tracking Reference

## Best Practices

### Keep Asana High-Level
```text
Good Asana comment:
"Engineering breakdown complete. GitHub issues #101-106 created for implementation tracking."

Implementation details belong in GitHub issues; Asana stays product-focused.
```

### Provide Full GitHub Issue Context
```text
Good GitHub issue:
Title: Fix JWT token expiry validation (#103)
Description: Tokens with timezone-offset expiry are incorrectly accepted.
Expected: 401 for expired tokens. Actual: 200 accepted.
Verify: Run auth_test.py::test_expired_token_rejection
```

Issues must have full context - what's broken, why it matters, how to verify the fix.

### Keep Issues Current
```text
Good practice:
Update issues as you work with design decisions and implementation notes.
Close with resolution summary and links to PRs.
```

Future readers need to understand what happened.

### Break Down Work into Discrete Tasks
```text
Good breakdown:
Asana: "Add authentication"
GitHub: 6 issues - schema design, JWT implementation, middleware, endpoints, tests, docs
```

Discrete tasks are easier to track and review.

## Examples

### Good Asana Task
```text
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
```markdown
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
