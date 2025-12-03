# Git Commit Messages

## Format

**Structure**:
```text
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

## Rules

1. **First line = imperative mood** - "Add feature" not "Added feature" or "Adds feature"
2. **First line ≤ 72 characters** - Keep it concise
3. **Body explains why, not what** - Code shows what changed, commit explains rationale
4. **Reference issues** - Include GitHub issue numbers
5. **Omit AI attribution** - Exclude "Generated with Claude" or "Co-Authored-By: Claude"
6. **No hyperbole** - Avoid "amazing", "incredible", "awesome" and similar language
7. **Be specific** - Describe actual changes, not generic descriptions

## Good Examples

```text
feat: add JWT token validation middleware

Implements token signature verification and expiry checking for
all protected API endpoints. Tokens must be valid HS256 JWTs with
user_id claim and <24hr age.

Refs #102
```

```text
fix: correct timezone handling in token expiry calculation

Token expiry was incorrectly calculated due to naive datetime
comparison. Now uses UTC-aware datetimes throughout.

Fixes #145
```

```text
refactor: extract auth logic into separate service module

Moved JWT generation and validation from routes into dedicated
auth service for better separation of concerns and testability.

Refs #103
```

```text
test: add integration tests for authentication flow

Covers login, token generation, protected endpoint access,
and expired token rejection.

Refs #105
```

```text
chore: bump version to 1.2.0
```

## Bad Examples

```text
❌ Bad: Updated stuff
Why: Vague, no context, unclear what changed
```

```text
❌ Bad: fix: fixed the bug
Why: Redundant, uninformative
```

```text
❌ Bad: Amazing new authentication feature!
Why: Hyperbolic language, emoji unnecessary
```

```text
❌ Bad: feat: implement authentication

Generated with Claude Code

Co-Authored-By: Claude <noreply@anthropic.com>
Why: AI attribution not needed in commits
```

```text
❌ Bad: fixes stuff in auth.py
Why: No issue reference, vague description
```

## Issue References

**Use conventional references**:
- `Fixes #123` - Closes issue when commit is merged
- `Resolves #123` - Alternative to "Fixes"
- `Refs #123` - References issue without closing it
- `Part of #123` - Indicates partial work toward issue

**Reference GitHub issues only** - Not Asana IDs:
```text
✅ Good: Fixes #123
❌ Bad: Fixes ASANA-456
```

Asana references belong in PR descriptions, not commit messages.
