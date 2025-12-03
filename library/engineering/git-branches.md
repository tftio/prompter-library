# Git Branches

## Naming Convention

**Format**: `<type>/<brief-description>`

**Examples**:
```text
feature/jwt-authentication
fix/token-expiry-bug
refactor/auth-service-extraction
docs/api-authentication-guide
chore/update-dependencies
```

## Rules

1. **Use hyphens** - Not underscores or camelCase
2. **Lowercase only** - Avoid uppercase characters
3. **Descriptive** - Someone should understand the work from branch name
4. **No issue numbers** - Branch name describes work, not ticket system
5. **Short-lived** - Merge within days, not weeks

## Good Branch Names

```text
✅ feature/user-authentication
✅ fix/database-connection-timeout
✅ refactor/split-monolithic-handler
✅ test/integration-auth-flow
```

## Bad Branch Names

```text
❌ my-branch
❌ fix_bug
❌ issue-123
❌ WIP-TEMP-NEW-STUFF
❌ johns-branch
```
