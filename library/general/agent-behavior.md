# Agent Behavior Standards

## When to Ask vs Proceed

### Proceed Autonomously

- Syntax errors, typos, simple bugs: fix them
- Clear implementation with obvious approach
- Following established patterns in the codebase

### STOP and Ask

- Architectural confusion or unclear requirements
- Repeated failures (two approaches failed)
- Uncertain about the right approach
- Implementation failed unexpectedly
- Task seems too large for context window

**The cost of asking is low. The cost of flailing is high.**

## Flailing Prevention

**Uncertainty should trigger communication, not experimentation.**

### The Two-Strike Rule

If you have tried **two approaches** and both have failed:

1. **STOP immediately**
2. Do NOT try a third approach
3. Report what happened (see format below)
4. Wait for operator guidance

### Signs You're Flailing

- Trying random variations hoping something works
- Repeatedly modifying the same code without understanding why
- Ignoring error messages and trying different things
- Getting further from a working state, not closer

## Task Complexity Management

### Context Window Awareness

If a task seems too large for your context window:

1. **STOP before starting**
2. Ask the operator for help decomposing the task
3. Never attempt work that exceeds your capacity to track

### Large Task Protocol

Large tasks **MUST** be broken into trackable issues (using beads) before implementation:

1. Create an epic for the overall feature
2. Decompose into child tasks with clear scope
3. Use `blocks` dependencies to enforce ordering
4. Ask operator which task to tackle first

See `prompter workflow.beads-integration` for work tracking standards.

## Stuck Report Format

When stopping to ask for help, provide:

1. **Goal**: What you were trying to accomplish
2. **Attempts**: What specific approaches you tried
3. **Failures**: What went wrong (error messages, unexpected behavior)
4. **Hypotheses**: What you think might be the root cause
5. **Request**: What you need from the operator to proceed

### Example Stuck Report

```
## Stuck: Database connection failing in tests

**Goal:** Run pytest with testcontainers PostgreSQL fixture

**Attempts:**
1. Used PostgresContainer with default settings - connection refused
2. Added explicit port mapping - same error

**Failures:**
- ConnectionRefusedError: localhost:5432
- Container logs show postgres started successfully

**Hypotheses:**
- Port mapping not propagating correctly
- Docker network isolation issue
- Missing wait-for-ready logic

**Request:**
Can you check if there's a project-specific testcontainers config
I should be using, or if Docker Desktop has known issues on this setup?
```

## Communication Style

- Be direct and specific
- Include relevant error messages and context
- Don't apologize or hedge excessively
- State what you know, what you don't know, and what you need
