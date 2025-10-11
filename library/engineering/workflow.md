# Infrastructure Directory Workflow

This document defines the development workflow for the nucleus-infra infrastructure directory.

## INVARIANTS

These rules are **NON-NEGOTIABLE** and must be followed for all work:

1. **Every push to origin MUST have a version bump**
2. **Intermediate commits MUST be squashed before push**
3. **Remote commits are complete units of work**
4. **Work happens on `phase_{n}` branches, merged to the base branch before push**
5. **Continuous state synchronization across 4 tracking systems**

## The 4 Tracking Systems

All work must be continuously synchronized across:

1. **GitHub Issues** - Acceptance criteria checkboxes, progress comments
2. **TODO.md** - Top-level checklist for current work session
3. **CHANGELOG.md** - Record of what was completed and when
4. **docs/plans/** - Planning documents reflecting actual implementation

**Update frequency**: After each meaningful piece of work, not in batches.

## Workflow Steps

### Step 1: Start Work Session

Assuming the base branch is `username/branch`:

```bash
# Ensure on $BASE_BRANCH branch
git checkout $BASE_BRANCH
git status

# Note current version
cat VERSION  # e.g., "0.0.1"

# Create phase branch for this work
# Phase 1 = Week 1-4, Phase 2 = Weeks 5-12, Phase 3 = Weeks 13+
git checkout -b phase_1
```

### Step 2: Do Work + Track Continuously

For each small piece of work completed:

1. **Make changes** (AWS console work, write docs, create Terraform modules, etc.)

2. **Immediately update tracking systems**:
   - Update `TODO.md` - mark completed items
   - Update GitHub issue acceptance criteria via GitHub tools
   - Update `CHANGELOG.md` - record what was done under `## Unreleased`
   - Update `docs/plans/*.md` if approach changed from original plan

3. **Commit locally** on `phase_N` branch:
   ```bash
   git add .
   git commit -m "WIP: describe what was just completed"
   ```

**Important**: Never batch tracking updates. Update all 4 systems immediately after completing each piece of work.

### Step 3: Complete Unit of Work

When a logical unit is complete (e.g., finished issue #1411):

1. Verify all 4 tracking systems are synchronized
2. Squash all WIP commits on `phase_N` branch:
   ```bash
   git rebase -i HEAD~N  # N = number of WIP commits
   # In editor: mark all but first commit as 'squash'
   # Rewrite commit message to describe complete unit
   ```

### Step 4: Version Bump on Phase Branch

```bash
# Still on phase_N branch
# Determine version bump type:
# - patch: bug fixes, small changes, documentation
# - minor: new features, new issues completed
# - major: breaking changes, phase completions

versioneer patch  # or minor, or major
# This updates VERSION and pyproject.toml in sync
```

### Step 5: Verify Version Sync

```bash
cat VERSION
grep version pyproject.toml
# Both should show new version (e.g., "0.0.2")
```

### Step 6: Final Commit on Phase Branch

```bash
git add VERSION pyproject.toml
git commit --amend  # Add version bump to squashed commit

# Final commit message format:
# "feat: [complete description of unit]
#
# - Detail 1
# - Detail 2
# - Detail 3
#
# Closes #1411
# Version: 0.0.2"
```

### Step 7: Merge to base branch

```bash
# Switch to $BASE_BRANCH
git checkout $BASE_BRANCH

# Merge phase branch
git merge phase_1

# Verify merge
git log -1
cat VERSION  # Should show new version
```

### Step 8: Tag (Optional but Recommended)

```bash
# Tag with version for easy reference
git tag v0.0.2
```

### Step 9: Push to Origin

```bash
# Push branch and tags
git push origin jfb/nucleus-infra
git push origin v0.0.2  # If tagged
```

### Step 10: Clean Up Phase Branch

```bash
# Delete local phase branch
git branch -d phase_1

# Delete remote phase branch if it was pushed
git push origin --delete phase_1  # Only if phase_1 was pushed to remote
```

## Example Complete Cycle

```bash
# Starting: on jfb/nucleus-infra, VERSION = 0.0.1

# Create phase branch
git checkout -b phase_1

# === Work on issue #1411 ===

# Create OUs in AWS console
git commit -m "WIP: create OUs in AWS console"

# Update tracking systems
vim TODO.md CHANGELOG.md
# Use GitHub tools to update issue #1411
git add TODO.md CHANGELOG.md
git commit -m "WIP: update tracking systems"

# Write documentation
vim docs/setup/aws-organization.rst
git add docs/setup/aws-organization.rst
git commit -m "WIP: write docs/setup/aws-organization.rst"

# Update tracking again
vim TODO.md CHANGELOG.md
git add TODO.md CHANGELOG.md
git commit -m "WIP: update tracking for docs"

# === Unit complete - prepare for push ===

# Squash all WIP commits on phase_1
git rebase -i HEAD~4
# In editor: squash all to one commit
# New message: "Configure AWS Organization with Control Tower"

# Version bump on phase_1
versioneer minor  # VERSION: 0.0.1 → 0.1.0

# Amend version into commit
git add VERSION pyproject.toml
git commit --amend
# Final message:
# "feat: configure AWS Organization with Control Tower
#
# - Created 4 OUs: Management, ControlPlane, Tenants, Sandbox
# - Configured SCPs for region enforcement
# - Enabled consolidated billing
# - Documented in docs/setup/aws-organization.rst
# - Updated GitHub issue #1411 acceptance criteria
# - Updated TODO.md and CHANGELOG.md
#
# Closes #1411
# Version: 0.1.0"

# Merge back to jfb/nucleus-infra
git checkout jfb/nucleus-infra
git merge phase_1

# Tag
git tag v0.1.0

# Push
git push origin jfb/nucleus-infra
git push origin v0.1.0

# Clean up
git branch -d phase_1
```


## Version Bump Guidelines

### Patch (0.0.1 → 0.0.2)

- Documentation updates
- Minor fixes
- Single acceptance criteria completed
- Small configuration changes

### Minor (0.0.2 → 0.1.0)

- Complete issue/task
- New feature added
- AWS resource created
- Terraform module completed
- Significant documentation added

### Major (0.1.0 → 1.0.0)

- Phase completion
- Breaking changes
- Major milestone reached
- Architecture changes

## Commit Message Format

```
<type>: <short summary>

<detailed description with bullet points>
- Detail 1
- Detail 2
- Detail 3

<optional issue references>
Closes #1411
Relates to #1412

Version: <version number>
```

**Types**: `feat`, `fix`, `docs`, `refactor`, `test`, `chore`

## Pre-Push Checklist

Before pushing to origin, verify:

- [ ] All WIP commits squashed on `phase_N` branch
- [ ] TODO.md reflects current state
- [ ] GitHub issues updated with acceptance criteria checked
- [ ] CHANGELOG.md has entries under `## Unreleased`
- [ ] Plans in `docs/plans/` updated if approach deviated
- [ ] `versioneer` run (VERSION and pyproject.toml/Cargo.toml/package.json match)
- [ ] Clean commit message describing complete unit
- [ ] Version number in commit message
- [ ] Merged to $BASE_BRANCH
- [ ] Tagged with version (recommended)
- [ ] Phase branch deleted after merge

## Common Mistakes to Avoid

1. **Batching tracking updates** - Update all 4 systems continuously, not at the end
2. **Forgetting version bump** - Every push must have a version change
3. **Pushing phase branch** - Always merge to `$BASE_BRANCH` first
4. **Not squashing commits** - All WIP commits must be squashed before merge
5. **Incomplete commit messages** - Must include version number and issue references
6. **Desynchronized VERSION/pyproject.toml** - Always use `versioneer` tool

## Emergency Procedures

### Forgot to Version Bump

```bash
# If already pushed without version bump
git checkout $BASE_BRANCH
versioneer patch  # Bump version
git add VERSION pyproject.toml
git commit -m "chore: version bump missed in previous commit

Version: 0.0.2"
git push origin $BASE_BRANCH
```

### Need to Fix Tracking After Push

```bash
# Create quick fix on phase branch
git checkout -b phase_1
# Update TODO.md, CHANGELOG.md, etc.
git add TODO.md CHANGELOG.md
git commit -m "docs: synchronize tracking systems"
versioneer patch
git add VERSION pyproject.toml
git commit --amend
git checkout $BASE_BRANCH
git merge phase_1
git push origin $BASE_BRANCH
git branch -d phase_1
```

## Tools

- **versioneer**: Version management tool (bumps VERSION and pyproject.toml)
  - `versioneer patch` - Increment patch version
  - `versioneer minor` - Increment minor version
  - `versioneer major` - Increment major version
  - `versioneer tag` - Add git tag for current version

- **GitHub CLI tools**: Update issues programmatically
  - Use MCP GitHub tools to update issues
  - Check/uncheck acceptance criteria
  - Add comments with progress updates

## Questions?

If uncertain about workflow:
1. Check this document first
2. Verify all 4 tracking systems are synchronized
3. Ensure version bump before push
4. When in doubt, create smaller units of work with more frequent pushes
