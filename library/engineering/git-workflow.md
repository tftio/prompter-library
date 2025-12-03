# Git Workflow

## Commit Strategy

### Atomic Commits vs. Squash

**Atomic commits** - Multiple logical commits in PR:
```text
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
```text
feat: add JWT authentication (#103)

Implements token generation, validation middleware,
integration tests, and documentation.
```

Use when:
- WIP commits clutter history ("fix typo", "oops", "trying again")
- Commits aren't logically independent
- Want clean linear history

### Rules for Atomic Commits

1. **Each commit must be functional** - Keep the build passing throughout PR
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

## Daily Development Flow

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

See `prompter rust.release-workflow` or `prompter tooling.version-management` for language-specific release processes.

## Git Best Practices

### Keep Secrets Out of Version Control
```text
Exclude from commits:
- .env files with real secrets
- API keys or tokens
- Passwords or credentials
- Private keys
```

Use `.gitignore` and secret management tools.

### Exclude Generated and Binary Files
```text
Keep out of repository:
- build/ or dist/ directories
- node_modules/ or venv/ directories
- Compiled binaries
- Large media files (use Git LFS instead)
```

### Write Descriptive Commit Messages
```text
Use specific messages:
- "fix: correct token validation" (clear)
- "feat: add user authentication" (specific)
- "refactor: extract auth service" (informative)
```

### Keep Commits Focused
```text
One logical change per commit:

Good:
- commit 1: feat: add authentication
- commit 2: fix: correct database bug
- commit 3: docs: update API guide
```

### Use force-with-lease for Feature Branches Only
```text
Protected branches (main):
- Push only via PR merge

Feature branches:
git push --force-with-lease origin feature/my-work
```

### Preserve Published History
```text
Rebase/amend only your own feature branch before merge.

Published commits shared with others: preserve as-is.
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
