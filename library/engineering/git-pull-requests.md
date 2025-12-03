# Git Pull Requests

## PR Title

**Format**: `<type>: <brief summary> (#issue-number)`

**Examples**:
```text
feat: add JWT authentication middleware (#103)
fix: correct token expiry calculation (#145)
refactor: extract auth service (#167)
docs: add authentication examples to API guide (#112)
```

## PR Description

**Required sections**:

```markdown
## Summary
1-3 sentence overview of what this PR accomplishes.

## Changes
- Specific change 1
- Specific change 2
- Specific change 3

## Testing
How these changes were verified:
- Unit tests added/updated
- Integration tests added/updated
- Manual testing performed

Commands to reproduce:
```bash
pytest tests/test_auth.py
```

## Breaking Changes
[If none, say "None"]
- Breaking change description
- Migration path

## Related
Resolves #123
Part of Asana task: [Task title](asana-link)
```

## PR Best Practices

1. **Keep PRs focused** - One logical change per PR
2. **Reference issues** - Use "Resolves #123" to auto-close
3. **Describe testing** - How you verified the changes work
4. **Note breaking changes** - Call out API/behavior changes explicitly
5. **Link to Asana** - Include Asana task link for product context
6. **Update docs** - Include doc changes in same PR as code changes
7. **Clean commits** - Either atomic commits or squash on merge

## PR Size Guidelines

**Ideal PR**: 200-500 lines changed
**Maximum PR**: 1000 lines (beyond this, split the PR)

**When to split large PRs**:
- Refactoring + new feature → Two PRs (refactor first, feature second)
- Multiple independent features → One PR per feature
- Large file renames + logic changes → Two PRs (rename first, logic second)

## PR Review Checklist

Before requesting review:
- [ ] All CI checks passing
- [ ] Tests added/updated
- [ ] Documentation updated
- [ ] No debug code or commented-out sections
- [ ] Clean commit history or ready to squash
- [ ] Self-reviewed the diff
