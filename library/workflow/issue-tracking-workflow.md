# Issue Tracking PR Workflow

## Pull Request Workflow

### PR Creation

1. **Reference GitHub issue** in PR title and description:
   ```text
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
```text
Status: Not Started -> In Progress
Comment: Breaking this into GitHub issues for implementation tracking.
```

### Checkpoint 2: Issues Created
```text
Comment: Created engineering tasks:
- Authentication schema design: https://github.com/org/repo/issues/101
- JWT implementation: https://github.com/org/repo/issues/102
- API middleware: https://github.com/org/repo/issues/103
- Endpoints: https://github.com/org/repo/issues/104
- Tests: https://github.com/org/repo/issues/105
- Docs: https://github.com/org/repo/issues/106
```

### Checkpoint 3: PRs Opened
```text
Comment: Implementation PRs:
- https://github.com/org/repo/pull/234 (schema + JWT)
- https://github.com/org/repo/pull/235 (middleware + endpoints)
- https://github.com/org/repo/pull/236 (tests + docs)
```

### Checkpoint 4: Work Complete
```text
Status: In Progress -> Complete
Comment: Completed. All PRs merged, authentication fully implemented.

Summary:
- Implemented JWT-based authentication with 24hr token expiry
- Added middleware to all protected endpoints
- Full test coverage including edge cases
- API docs updated with authentication examples

Delivered in PRs: #234, #235, #236
```
