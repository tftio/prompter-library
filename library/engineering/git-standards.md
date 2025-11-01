# Git Standards

## Commit Messages

### Format

**Structure**:
```
<type>: <brief summary>

[optional detailed explanation]

[optional references]
```

**Types**:
- `feat`: New feature
- `fix`: Bug fix
- `refactor`: Code restructuring without behavior change
- `test`: Adding or updating tests
- `docs`: Documentation changes
- `chore`: Build, tooling, dependencies, version bumps
- `perf`: Performance improvements
- `style`: Code style/formatting (not CSS styling)

### Rules

1. **First line = imperative mood** - "Add feature" not "Added feature" or "Adds feature"
2. **First line ≤ 72 characters** - Keep it concise
3. **Body explains why, not what** - Code shows what changed, commit explains rationale
4. **Reference issues** - Include GitHub issue numbers
5. **No AI attribution** - Never include "Generated with Claude" or "Co-Authored-By: Claude"
6. **No hyperbole** - Avoid "amazing", "incredible", "awesome" and similar language
7. **Be specific** - Describe actual changes, not generic descriptions

### Good Examples

```
feat: add JWT token validation middleware

Implements token signature verification and expiry checking for
all protected API endpoints. Tokens must be valid HS256 JWTs with
user_id claim and <24hr age.

Refs #102
```

```
fix: correct timezone handling in token expiry calculation

Token expiry was incorrectly calculated due to naive datetime
comparison. Now uses UTC-aware datetimes throughout.

Fixes #145
```

```
refactor: extract auth logic into separate service module

Moved JWT generation and validation from routes into dedicated
auth service for better separation of concerns and testability.

Refs #103
```

```
test: add integration tests for authentication flow

Covers login, token generation, protected endpoint access,
and expired token rejection.

Refs #105
```

```
chore: bump version to 1.2.0
```

### Bad Examples

```
❌ Bad: Updated stuff
Why: Vague, no context, unclear what changed
```

```
❌ Bad: fix: fixed the bug
Why: Redundant, uninformative
```

```
❌ Bad: Amazing new authentication feature! 🎉
Why: Hyperbolic language, emoji unnecessary
```

```
❌ Bad: feat: implement authentication

Generated with Claude Code

Co-Authored-By: Claude <noreply@anthropic.com>
Why: AI attribution not needed in commits
```

```
❌ Bad: fixes stuff in auth.py
Why: No issue reference, vague description
```

### Issue References

**Use conventional references**:
- `Fixes #123` - Closes issue when commit is merged
- `Resolves #123` - Alternative to "Fixes"
- `Refs #123` - References issue without closing it
- `Part of #123` - Indicates partial work toward issue

**Reference GitHub issues only** - Not Asana IDs:
```
✅ Good: Fixes #123
❌ Bad: Fixes ASANA-456
```

Asana references belong in PR descriptions, not commit messages.

## Branches

### Naming Convention

**Format**: `<type>/<brief-description>`

**Examples**:
```
feature/jwt-authentication
fix/token-expiry-bug
refactor/auth-service-extraction
docs/api-authentication-guide
chore/update-dependencies
```

### Rules

1. **Use hyphens** - Not underscores or camelCase
2. **Lowercase only** - Avoid uppercase characters
3. **Descriptive** - Someone should understand the work from branch name
4. **No issue numbers** - Branch name describes work, not ticket system
5. **Short-lived** - Merge within days, not weeks

### Good Branch Names

```
✅ feature/user-authentication
✅ fix/database-connection-timeout
✅ refactor/split-monolithic-handler
✅ test/integration-auth-flow
```

### Bad Branch Names

```
❌ my-branch
❌ fix_bug
❌ issue-123
❌ WIP-TEMP-NEW-STUFF
❌ johns-branch
```

## Pull Requests

### PR Title

**Format**: `<type>: <brief summary> (#issue-number)`

**Examples**:
```
feat: add JWT authentication middleware (#103)
fix: correct token expiry calculation (#145)
refactor: extract auth service (#167)
docs: add authentication examples to API guide (#112)
```

### PR Description

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

### PR Best Practices

1. **Keep PRs focused** - One logical change per PR
2. **Reference issues** - Use "Resolves #123" to auto-close
3. **Describe testing** - How you verified the changes work
4. **Note breaking changes** - Call out API/behavior changes explicitly
5. **Link to Asana** - Include Asana task link for product context
6. **Update docs** - Include doc changes in same PR as code changes
7. **Clean commits** - Either atomic commits or squash on merge

### PR Size Guidelines

**Ideal PR**: 200-500 lines changed
**Maximum PR**: 1000 lines (beyond this, strongly consider splitting)

**When to split large PRs**:
- Refactoring + new feature → Two PRs (refactor first, feature second)
- Multiple independent features → One PR per feature
- Large file renames + logic changes → Two PRs (rename first, logic second)

### PR Review Checklist

Before requesting review:
- [ ] All CI checks passing
- [ ] Tests added/updated
- [ ] Documentation updated
- [ ] No debug code or commented-out sections
- [ ] Clean commit history or ready to squash
- [ ] Self-reviewed the diff

## Commit Strategy

### Atomic Commits vs. Squash

**Atomic commits** - Multiple logical commits in PR:
```
feat: add JWT generation function
feat: add JWT validation middleware
test: add JWT integration tests
docs: document JWT authentication
```

Use when:
- Each commit is independently valuable
- History benefits from granular steps
- Working on complex feature with logical progression

**Squash merge** - Combine all commits into one:
```
feat: add JWT authentication (#103)

Implements token generation, validation middleware,
integration tests, and documentation.
```

Use when:
- WIP commits clutter history ("fix typo", "oops", "trying again")
- Commits aren't logically independent
- Want clean linear history

### Rules for Atomic Commits

1. **Each commit must be functional** - Don't break the build mid-PR
2. **Each commit has clear purpose** - Not "WIP" or "fixes"
3. **Tests pass at each commit** - Bisecting history should work
4. **Logical progression** - Tell a story through commits

## Branch Management

### Main Branch Protection

**main branch rules**:
- No direct commits (use PRs)
- Require passing CI checks
- Require review approval (if team policy)
- No force pushes
- Always deployable

### Working on Branches

**Create branch**:
```bash
# From up-to-date main
git checkout main
git pull origin main
git checkout -b feature/jwt-authentication
```

**Keep branch updated**:
```bash
# Rebase on main regularly
git fetch origin
git rebase origin/main

# Or merge if collaboration on branch
git merge origin/main
```

**Push branch**:
```bash
# First push
git push -u origin feature/jwt-authentication

# Subsequent pushes
git push
```

### Merge Conflicts

**Resolution process**:
1. Fetch latest main: `git fetch origin`
2. Rebase onto main: `git rebase origin/main`
3. Resolve conflicts in editor
4. Stage resolved files: `git add <files>`
5. Continue rebase: `git rebase --continue`
6. Force push (rebase rewrites history): `git push --force-with-lease`

**Prevent conflicts**:
- Rebase on main frequently (daily for long-lived branches)
- Keep PRs small and short-lived
- Communicate with team about overlapping work

## Git Workflow Summary

### Daily Development Flow

1. **Start work**:
   ```bash
   git checkout main
   git pull origin main
   git checkout -b feature/new-work
   ```

2. **Make changes**:
   ```bash
   # Edit files
   git add <files>
   git commit -m "feat: descriptive message"
   ```

3. **Push and create PR**:
   ```bash
   git push -u origin feature/new-work
   gh pr create --title "feat: descriptive title (#123)" --body "$(cat pr-template.md)"
   ```

4. **Address review feedback**:
   ```bash
   # Make changes
   git add <files>
   git commit -m "fix: address review feedback"
   git push
   ```

5. **Merge PR**:
   - Via GitHub UI (click Merge button)
   - Or via CLI: `gh pr merge --squash`

6. **Clean up**:
   ```bash
   git checkout main
   git pull origin main
   git branch -d feature/new-work
   ```

### Release Flow

See `rust/release-workflow.md` or `tooling/version-management.md` for language-specific release processes.

## Git Anti-Patterns

### DON'T: Commit Secrets or Credentials
```
❌ Never commit:
- .env files with real secrets
- API keys or tokens
- Passwords or credentials
- Private keys
```

Use `.gitignore` and secret management tools.

### DON'T: Commit Generated or Binary Files
```
❌ Avoid committing:
- build/ or dist/ directories
- node_modules/ or venv/ directories
- Compiled binaries
- Large media files (use Git LFS)
```

### DON'T: Use Vague Commit Messages
```
❌ Bad messages:
- "fix"
- "update"
- "changes"
- "WIP"
- "asdfasdf"
```

### DON'T: Mix Unrelated Changes
```
❌ Bad commit:
feat: add authentication and fix database bug and update docs

Should be three separate commits or PRs.
```

### DON'T: Force Push to Shared Branches
```
❌ NEVER:
git push --force origin main

✅ OK for your feature branch:
git push --force-with-lease origin feature/my-work
```

### DON'T: Rewrite Published History
```
❌ Don't rebase/amend commits that others have based work on

Exception: Your own feature branch before merge.
```

## Tools and Configuration

### Recommended Git Config

```bash
# Your identity
git config --global user.name "Your Name"
git config --global user.email "you@example.com"

# Better diffs
git config --global diff.algorithm histogram

# Rebase by default on pull
git config --global pull.rebase true

# Better merge conflict markers
git config --global merge.conflictstyle diff3

# Prune deleted remote branches on fetch
git config --global fetch.prune true
```

### Git Aliases

```bash
# Shortcuts for common operations
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.cm commit
git config --global alias.lg "log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
```

### Integration with GitHub CLI

```bash
# Create PR from current branch
gh pr create

# View PR status
gh pr status

# Merge PR
gh pr merge --squash

# View PR diff
gh pr diff

# Check out PR locally
gh pr checkout 123
```

## Summary

**Commits**: Clean, descriptive, reference issues, no AI attribution
**Branches**: Short-lived, descriptive names, type prefixes
**PRs**: Focused changes, comprehensive descriptions, link to issues and Asana
**History**: Keep it clean - either atomic commits or squash merge
