# Issue Tracking Overview

## Purpose

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
- When starting work (status -> In Progress)
- When creating GitHub issues (add comment with GH issue links)
- When opening PRs (add comment with PR links)
- When completing work (status -> Complete with summary)

## Why Two Systems?

**Asana** - Product perspective:
- Non-engineers need visibility without technical noise
- Focus on what is being built and when
- Business value and user impact

**GitHub Issues** - Engineering perspective:
- Developers need exhaustive context for implementation
- Technical decisions must be documented
- Future developers need historical rationale

## Why Exhaustive GitHub Issues?

- **Context preservation** - Future engineers understand why decisions were made
- **Onboarding** - New team members read issues to learn system design
- **Knowledge retention** - Prevents repeated discussions of settled decisions
- **Debugging** - Implementation notes help diagnose issues months later

## Why Multiple Checkpoints?

- **Transparency** - Stakeholders see progress without daily noise
- **Traceability** - Clear links between product requirements and implementation
- **Accountability** - Engineers demonstrate systematic approach to work
