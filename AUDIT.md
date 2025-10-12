# Prompter Library Audit Report

**Audit Date**: 2025-10-11
**Library Location**: `~/.local/prompter/library/`
**Total Fragments Audited**: 28 fragments across 8 categories
**Config File**: `~/.config/prompter/config.toml`

## Executive Summary

| Category | Count | Severity |
|----------|-------|----------|
| Critical Issues | 8 | CRITICAL |
| Atomicity Issues | 6 | HIGH |
| Overlap Issues | 4 | MEDIUM |
| Organization Issues | 5 | MEDIUM |
| Effectiveness Issues | 15 | HIGH |
| **Total Issues** | **38** | - |

**Recommendation**: Address all CRITICAL issues immediately. HIGH severity issues significantly impact LLM agent comprehension and execution accuracy.

---

## 1. CRITICAL ISSUES

These issues create contradictions, ambiguity, or execution failures for LLM agents.

### CRIT-001: Line Endings Rule Duplication (Contradiction)

**Severity**: CRITICAL
**Locations**:
- `general/overall.md:3`
- `engineering/general-principles.md:5`

**Current State**:
Both files contain identical instruction: "YOU MUST NEVER USE WINDOWS LINE ENDINGS"

**LLM Impact**: Duplicate instructions waste tokens and suggest potential inconsistency across fragments. Agent may question which source is authoritative.

**Required Action**:
```bash
# Remove line 5 from engineering/general-principles.md
# Keep it only in general/overall.md since that's the "FIRST INVARIANT"
```

**Specific Edit**:
In `engineering/general-principles.md`, remove line 5:
```markdown
5. *YOU MUST NEVER USE WINDOWS LINE ENDINGS*. When generating files, you must **ONLY** use Unix line endings.
```

**Config Impact**: None (no profile dependencies change)

---

### CRIT-002: Obsolete Shell Execution Pattern

**Severity**: CRITICAL
**Location**: `general/overall.md:6-12`

**Current State**:
```markdown
## SHELL EXECUTION INVARIANT

When changing directories to work, you *MUST* use the following shell syntax:

`(pushd {directory}; {shell work goes here}; popd)`

In other words, you should *never* end a shell command or session in a directory other than the one you were invoked in. This is a persistent source of confusion for LLM agents.
```

**LLM Impact**: This pattern is outdated and conflicts with modern best practices that recommend avoiding `cd` entirely by using absolute paths. The global CLAUDE.md also says "Try to maintain your current working directory throughout the session by using absolute paths and avoiding usage of `cd`."

**Required Action**:
Replace this section with:
```markdown
## SHELL EXECUTION INVARIANT

When executing commands in different directories, **NEVER** use `cd` to change directories. Instead:

1. **Use absolute paths**: `pytest /foo/bar/tests` (not `cd /foo/bar && pytest tests`)
2. **If cd is unavoidable**: Use subshells `(cd /path && command)` to avoid state pollution
3. **Maintain working directory**: Commands should not leave the shell in a different directory

This prevents directory state confusion for LLM agents and makes commands more explicit.
```

**Config Impact**: None

---

### CRIT-003: Version Management Fragments Overlap (Duplication)

**Severity**: CRITICAL  
**Status**: ✅ Resolved (consolidated into `tooling/version-management.md`)

**Resolution Summary**:
- Merged the former `engineering/version-management.md` and `engineering/versions.md` content into `library/tooling/version-management.md`.
- Removed `engineering/versions.md` from the repository and updated `~/.config/prompter/config.toml` to reference the new `tooling.version-management` profile.
- Fixed the SEMVER typo and clarified the versioneer commands in the consolidated fragment.

**Consolidated Content Template**:
```markdown
# Version Management

## CRITICAL VERSION SYNCHRONIZATION RULES

**INVARIANT - NEVER VIOLATE THESE:**

1. **VERSION and project manifest MUST ALWAYS be updated in sync**
   - pyproject.toml, Cargo.toml, or package.json must match VERSION file
2. **NEVER update only one version file**
3. **Use versioneer tool exclusively** (installed at $HOME/.local/bin)

## Versioneer Usage

versioneer updates VERSION and manifest files in sync:

```bash
versioneer patch  # 0.0.1 → 0.0.2 (bug fixes, docs)
versioneer minor  # 0.0.2 → 0.1.0 (new features)
versioneer major  # 0.1.0 → 1.0.0 (breaking changes)
versioneer tag    # Create git tag for current version
```

## Semantic Versioning

Follow [SEMVER](https://semver.org/) strictly:
- **PATCH**: Bug fixes, documentation, no API changes
- **MINOR**: New features, backward-compatible
- **MAJOR**: Breaking changes, incompatible API changes

When commit scope is MINOR or MAJOR, warn the operator before bumping.

## Version Synchronization Check

Before any release:
```bash
cat VERSION
grep version pyproject.toml  # or Cargo.toml
# Both must match exactly
```

These rules are **INVARIANT** and **NON-NEGOTIABLE**.
```

---

### CRIT-004: SQL Unique Constraint Syntax Error

**Severity**: CRITICAL
**Location**: `python/api-testing.md:13`

**Current State**:
```sql
UNIQUE api_Version, api_path, params
```

**LLM Impact**: Invalid SQL syntax. Agent will generate broken DDL. Missing parentheses in constraint.

**Required Action**:
```sql
UNIQUE (api_version, api_path, params)
```

Also fix case inconsistency: `api_Version` → `api_version`

**Specific Edit**:
Line 13 in `python/api-testing.md`:
```sql
UNIQUE (api_version, api_path, params)
```

---

### CRIT-005: CLI Rules Duplication Between Fragments

**Severity**: CRITICAL (creates confusion)  
**Status**: ✅ Resolved (general CLI rules live in `tooling/cli.md`)

**Resolution Summary**:
- Removed the duplicate `engineering/cli.md` fragment; language-agnostic CLI guidance now resides in `library/tooling/cli.md`.
- Trimmed `library/rust/cli.md` to Rust-only directives and added the note: "This fragment extends `tooling/cli.md`. All general CLI rules apply."
- Updated the new `engineering.stack` profile to depend on the `tooling.cli` profile.

---

### CRIT-006: Vague Language - "Consider" Pattern

**Severity**: CRITICAL
**Location**: `database/sql.md:2`

**Current State**:
```markdown
2. consider views, even materialized views, when queries in client code get complex;
```

**LLM Impact**: "consider" is too soft. Agent doesn't know WHEN to use views. No threshold for "complex".

**Required Action**:
Replace with explicit directive:
```markdown
2. **Use views or materialized views when**:
   - Query involves 3+ table joins
   - Same complex query appears in multiple locations
   - Query performance is critical (use materialized views)
   - Business logic should be centralized at database level
```

---

### CRIT-007: Typo in Rust CLI Fragment

**Severity**: CRITICAL (breaks parsing)
**Location**: `rust/cli.md:7`

**Current State**:
```markdown
3. support a `license` sbucommand that outputs the license information abotu the tool
```

**Typos**: "sbucommand" → "subcommand", "abotu" → "about"

**LLM Impact**: Typos reduce agent confidence in instruction validity. May skip or misinterpret.

**Required Action**:
```markdown
3. support a `license` subcommand that outputs the license information about the tool
```

---

### CRIT-008: Missing Numbered Item in Just Rules

**Severity**: CRITICAL (breaks list parsing)
**Location**: `just/rules.md:5-6`

**Current State**:
```markdown
4. **IF you need shell logic** - create separate shell scripts in `scripts/`
6. **Keep justfile targets simple** - they should call external scripts/functions
```

Missing item 5 in numbered list.

**LLM Impact**: Breaks list parsing. Agent may not process subsequent items correctly.

**Required Action**:
Either remove numbering or add item 5. Recommend removing numbers since these are policy statements, not sequential steps:
```markdown
**INVARIANT JUSTFILE RULES - NEVER VIOLATE THESE:**

- **NEVER use heredocs** in justfile targets to create inline scripts
- **NEVER use `python -c`** inline code in justfile targets
- **NEVER create and execute scripts** within justfile targets
- **IF you need shell logic** - create separate shell scripts in `scripts/`
- **Keep justfile targets simple** - they should call external scripts/functions
- **DEPENDENCY-ONLY RECIPES** that are used exclusively as dependencies by other recipes MUST:
  - Be prefixed with `_` in their name to indicate they are internal/dependency-only
  - Be placed in a group called `dependent_only`
  - Examples: `_get-env-config`, `_get-tenant-config`
```

---

## 2. ATOMICITY ISSUES

Fragments should be focused on a single concern while remaining useful standalone.

### ATOM-001: Fragment Too Large - general/plan.md

**Severity**: HIGH
**Location**: `general/plan.md` (54 lines)

**Current State**:
Single fragment covering:
- Creating plans (structure, phases, subtasks)
- GitHub issue creation
- Asana ticket creation
- Plan execution workflow
- Plan completion workflow

**LLM Impact**: Single massive fragment forces agents to load all planning context even when only one aspect is needed. Reduces composability.

**Required Action**:
Split into 3 focused fragments:

1. **`documentation/plan-structure.md`** (10 lines)
   - Plan document format
   - Directory structure
   - Phase/subtask organization

2. **`workflow/plan-execution.md`** (25 lines)
   - Prerequisites (clean git status)
   - Phase branch workflow
   - Subtask completion
   - Commit conventions

3. **`workflow/plan-tracking.md`** (15 lines)
   - GitHub issue integration
   - Asana ticket integration
   - State synchronization

**Specific Content for `documentation/plan-structure.md`**:
```markdown
# Planning Document Structure

## Directory Organization

All planning documents MUST be stored in `docs/plans/{plan_name}/`:

- `overview.md` - High-level explanation (non-technical audience)
- `phase_{n}.md` - Detailed phase plans with subtasks

## Overview Format

```markdown
# {Plan Name} - Overview

## Goal
[High-level description]

## Phases

### Phase 1: {Name}
State: _todo_ | _in-progress_ | _completed_ | _failed_
[Brief description]
```

## Phase Document Format

Each `phase_{n}.md` MUST contain:

1. **Explanation section** - What this phase accomplishes
2. **Rationale section** - Why this approach
3. **Definition of Done** - Explicit completion criteria
4. **Subtasks** - Checklist with `[ ]` checkboxes

Target audience: Senior software engineer working with LLM coding agent.
```

**Config Update Result**:
```toml
[workflow.planning]
depends_on = [
    "documentation/plan-structure.md",
    "documentation/todos.md"
]
```

---

### ATOM-002: Fragment Too Large - engineering/workflow.md

**Status**: ✅ Completed

**Resolution Summary**:
- Removed the project-specific `engineering/workflow.md` fragment from the library.
- Created the `workflow/` directory with focused `plan-execution.md` and `plan-tracking.md` fragments.
- Updated `~/.config/prompter/config.toml` so engineering prompts pull those directives via the shared `workflow.execution` profile.

**Config Update Result**:
```toml
[workflow.execution]
depends_on = [
    "workflow/plan-execution.md",
    "workflow/plan-tracking.md"
]
```

---

### ATOM-003: Fragment Too Small - database/general.md

**Severity**: MEDIUM
**Location**: `database/general.md` (1 line)

**Current State**:
```markdown
# Database Standards

We use PostgreSQL exclusively; always the latest version.
```

**LLM Impact**: Too small to be useful standalone. Wastes a file and config entry.

**Required Action**:
Merge into `database/ddl.md` as preamble:

```markdown
# DDL Standards

## Database Platform

Use PostgreSQL exclusively; always the latest version (currently 17.5).

## DDL Requirements

When creating DDL, hold the following:

1. table names are **plural** – a user is a record in the `users` table; join tables should be named with `_joins` as a suffix;
[... rest of current ddl.md content ...]
```

**Steps**:
1. Merge content into `database/ddl.md`
2. Delete `database/general.md`
3. Update config

**Config Update Required**:
```toml
[database.all]
depends_on = [
    # REMOVE: "database/general.md",
    "database/ddl.md",
    "database/sql.md"
]
```

---

### ATOM-004: Fragment Too Small - engineering/core-technologies.md

**Status**: ✅ Completed

**Resolution Summary**:
- Removed `engineering/core-technologies.md`.
- Captured platform and tooling requirements in the new `environment/` fragments (`environment/platforms.md`, `environment/tooling.md`).
- Updated `engineering/general-principles.md` to reference the environment fragments for defaults.

**Config Update Result**:
```toml
[engineering.stack]
depends_on = [
    "engineering/general-principles.md",
    "environment.defaults",
    "tooling.cli",
    "tooling.version-management",
    "shell/error-handling.md",
    "shell/portability.md",
    "shell/style.md",
    "just/rules.md"
]
```

---

### ATOM-005: Potential Split - shell/standards.md

**Status**: ✅ Completed

**Resolution Summary**:
- Replaced the monolithic `shell/standards.md` with three focused fragments:
  - `shell/error-handling.md`
  - `shell/portability.md`
  - `shell/style.md`
- Folded these fragments into the shared `engineering.stack` profile so every engineering prompt inherits them without extra wrappers.

---

### ATOM-006: Documentation Plans Fragment Overlap

**Severity**: MEDIUM
**Locations**:
- `documentation/plans.md` (10 lines)
- `general/plan.md` (54 lines)

**Current State**:
`documentation/plans.md` specifies format requirements that overlap with `general/plan.md` structure definition.

**LLM Impact**: After splitting `general/plan.md` (ATOM-001), the new `documentation/plan-structure.md` will overlap with `documentation/plans.md`.

**Required Action**:
After ATOM-001 is completed:
1. Review `documentation/plans.md` and `documentation/plan-structure.md`
2. Consolidate into single `documentation/plan-structure.md`
3. Delete `documentation/plans.md`

**Consolidated Content**:
Merge both into comprehensive plan structure guide with:
- Directory location requirements
- File naming conventions
- Format requirements (explanation, rationale, brief, TODO list)
- Target audience specification

---

## 3. OVERLAP ISSUES

### OVER-001: Line Endings Duplication

**Status**: Already documented in CRIT-001
**Action**: Remove from `engineering/general-principles.md:5`

---

### OVER-002: Version Management Duplication

**Status**: Closed via CRIT-003  
**Resolution**: `engineering/versions.md` removed; guidance consolidated into `tooling/version-management.md` and referenced through the `tooling.version-management` profile.

---

### OVER-003: CLI Standards Duplication

**Status**: Closed via CRIT-005  
**Resolution**: General CLI rules live in `tooling/cli.md`; `rust/cli.md` references that fragment and contains Rust-only directives.

---

### OVER-004: Plan Structure Duplication

**Status**: Related to ATOM-006
**Action**: After ATOM-001 split, consolidate `documentation/plans.md` and new `documentation/plan-structure.md`

---

## 4. ORGANIZATION ISSUES

### ORG-001: Missing workflow/ Category

**Severity**: MEDIUM
**Current State**: No `workflow/` directory exists

**LLM Impact**: Process/workflow instructions scattered across `general/` and `engineering/` without clear categorization.

**Required Action**:
Create `workflow/` directory and move:
- From ATOM-001: New `workflow/plan-execution.md`
- From ATOM-001: New `workflow/plan-tracking.md`
- From ATOM-002: Optionally generalized phase branch workflow (if not removed)

**Steps**:
```bash
mkdir -p library/workflow
# Move/create files as specified in ATOM-001, ATOM-002
```

---

### ORG-002: general/plan.md Misplaced

**Severity**: MEDIUM
**Location**: `general/plan.md`

**Current State**: 54-line workflow document in `general/` category

**LLM Impact**: "general" should contain foundational behavior rules, not detailed workflows.

**Required Action**:
Addressed by ATOM-001 split. After split:
- `documentation/plan-structure.md` (correct location)
- `workflow/plan-execution.md` (correct location)
- `workflow/plan-tracking.md` (correct location)

---

### ORG-003: engineering/workflow.md Project-Specific

**Severity**: HIGH
**Location**: `engineering/workflow.md`

**Current State**: 345-line document with project-specific details (jfb/nucleus-infra paths, specific branch names)

**LLM Impact**: Library should contain reusable instruction sets, not project-specific documentation.

**Required Action**:
Remove from library (see ATOM-002). Move to project-specific documentation location.

---

### ORG-004: engineering/ Category Too Broad

**Severity**: LOW
**Current State**: engineering/ contains:
- general-principles.md
- core-technologies.md
- cli.md
- version-management.md
- versions.md
- workflow.md

**LLM Impact**: Category lacks focus. Mixes general principles, specific tools, and workflows.

**Required Action**:
After other remediation:
1. Keep `engineering/general-principles.md` (merge core-technologies per ATOM-004)
2. Move `cli.md` → consider `tools/cli.md` or keep if it's engineering practice
3. Keep `version-management.md` (after CRIT-003 merge)
4. Remove `workflow.md` (per ATOM-002)

Final `engineering/` should contain only general engineering principles and practices.

---

### ORG-005: Consider environment/ Category

**Severity**: LOW
**Current State**: No environment-specific category

**Proposal**: Create `environment/` for:
- Tool versions (Python 3.13, PostgreSQL 17.5)
- Platform requirements (AWS, Linux/MacOS)
- Required utilities (just, uv, versioneer)

**LLM Impact**: Low priority. Current organization is acceptable. Consider for future expansion.

**Required Action**:
Optional enhancement. Not required for current audit remediation.

---

## 5. LLM-AGENT EFFECTIVENESS ISSUES

### EFF-001: Vague "Consider" Language

**Status**: Already documented in CRIT-006
**Location**: `database/sql.md:2`
**Action**: Replace with explicit WHEN conditions

---

### EFF-002: Emotional Language in CLI Rules

**Severity**: HIGH  
**Status**: ✅ Resolved (`tooling/cli.md`)

**Resolution Summary**:
- Replaced "Be fun! Use cool terminal effects" with explicit TTY-handling guidance.
- Ensured every directive uses imperative, objective language.

---

### EFF-003: Soft "Be Mindful" Language

**Severity**: MEDIUM  
**Status**: ✅ Resolved (`shell/portability.md`)

**Resolution Summary**:
- Replaced vague language with an explicit directive to ensure scripts run on both macOS and Linux.
- Added bullet list describing testing expectations and portable command usage.

---

### EFF-004: Unclear "Check System Type"

**Severity**: MEDIUM  
**Status**: ✅ Resolved (`shell/portability.md`)

**Resolution Summary**:
- Added a concrete `case "$(uname -s)"` example demonstrating platform detection.
- Clarified escalation path for unsupported platforms.

---

### EFF-005: Ambiguous "Rolled Up" Term

**Severity**: MEDIUM
**Location**: `python/documentation.md:5`

**Current State**:
```markdown
Non-public methods, attributes, and functions **MUST** have docstrings, but they **MAY NOT** be rolled up to the class or module level.
```

**LLM Impact**: "rolled up" is jargon. Unclear what this means for Sphinx generation.

**Required Action**:
```markdown
Non-public methods (prefixed with `_`) **MUST** have docstrings, but they **MUST NOT** be included in the public API documentation (not listed in the module's `__all__` or class's public documentation).
```

---

### EFF-006: Vague "Be Explicit" Directive

**Severity**: LOW
**Location**: `python/documentation.md:7`

**Current State**:
```markdown
Be explicit in writing the documentation; assume that the primary reader of the documentation will be an experienced software engineer with Python and web experience.
```

**LLM Impact**: "Be explicit" provides no actionable guidance. This is meta-commentary, not instruction.

**Required Action**:
Remove this line entirely. The specific requirements (1-5) provide concrete directives. This line adds no behavioral value.

---

### EFF-007: Temporal Version Reference

**Severity**: LOW
**Location**: `python/basics.md:3`

**Current State**:
```markdown
All projects will use the latest released version of Python (3.13.7 as of 2025-09-10).
```

**LLM Impact**: Date reference will become stale. Agent cannot determine if this is current.

**Required Action**:
```markdown
All projects will use the latest released version of Python (currently 3.13.7). Check python.org for the latest stable release.
```

Or remove date entirely:
```markdown
All projects **MUST** use the latest stable Python release. No legacy version support unless forced by external dependencies.
```

---

### EFF-008: Unclear "Truly Insane" Threshold

**Severity**: MEDIUM
**Location**: `database/ddl.md:12`

**Current State**:
```markdown
8. Break any of these rules before doing some thing truly insane – be sure to **ASK** the operator for guidance
```

**LLM Impact**:
1. "truly insane" is subjective
2. Gives permission to break rules without clear criteria
3. Agent needs concrete examples

**Required Action**:
```markdown
8. **If DDL rules conflict with requirements**, ASK the operator before proceeding. Examples requiring consultation:
   - Denormalization for performance (violates 3NF rule)
   - Allowing NULL values in critical columns
   - Using synthetic keys where natural keys exist
```

---

### EFF-009: Meta-Commentary Instead of Instruction

**Severity**: LOW
**Location**: `documentation/plans.md:10`

**Current State**:
```markdown
The intended audience for planning documents is a senior software engineer working with an LLM coding agent. All tasks must be as explicit as possible.
```

**LLM Impact**: This describes the audience but doesn't instruct the agent on behavior. It's meta-commentary.

**Required Action**:
Remove or reframe as directive:
```markdown
All planning tasks **MUST** be explicit and actionable. Assume the executor has senior engineering expertise but requires concrete steps, not vague descriptions.
```

---

### EFF-010: Metadata List Without Behavioral Directive

**Severity**: MEDIUM
**Location**: `engineering/core-technologies.md` (entire file)

**Current State**:
Bullet list of technologies with no instructions on what agent should DO.

**LLM Impact**: Agent doesn't know how to use this information. Is it a filter? A requirement check? An initialization step?

**Required Action**:
Addressed by ATOM-004 merge. New version adds "MUST be used unless explicitly overridden" directive.

---

### EFF-011: Typo - SEMVAR

**Severity**: LOW (but breaks links)
**Location**: `engineering/versions.md:1`

**Current State**:
```markdown
We support [SEMVAR][https://semver.org/].
```

**Typos**: "SEMVAR" should be "SEMVER", also invalid markdown link syntax

**LLM Impact**: Broken reference reduces agent confidence.

**Required Action**:
```markdown
We support [SEMVER](https://semver.org/).
```

---

### EFF-012: Typo - "arguemnt"

**Severity**: LOW  
**Status**: ✅ Resolved (`tooling/version-management.md`)

**Resolution Summary**:
- Corrected the spelling of "argument" in the versioneer usage section.
- Ensured the consolidated fragment uses consistent terminology.

---

### EFF-013: Missing Context for Error Handling Rationale

**Severity**: LOW  
**Status**: ✅ Resolved (`shell/error-handling.md`)

**Resolution Summary**:
- Added explicit rationale explaining why `|| true` masks real failures.
- Provided an `if ! command; then handle_failure; fi` example to illustrate the correct pattern.

---

### EFF-014: Ambiguous "Enough Information" Threshold

**Severity**: LOW
**Location**: `general/plan.md:5`

**Current State**:
```markdown
When we are working on a plan, I want you to be exhaustive; ask clarifying questions of the operator. Be thorough. Once you have enough information to create your plan, you must do the following.
```

**LLM Impact**: "enough information" is undefined. Agent doesn't know when to stop asking questions.

**Required Action**:
```markdown
When creating a plan, ask clarifying questions until you have:
1. Clear definition of done for the overall work
2. Understanding of all technical constraints
3. List of external dependencies
4. Acceptance criteria from operator

Once you have these four elements, proceed with plan creation.
```

---

### EFF-015: Inconsistent Emphasis Formatting

**Severity**: LOW
**Locations**: Multiple files

**Current State**:
Mix of:
- `**MUST**` (bold)
- `*MUST*` (italic)
- `MUST` (plain)
- `**NEVER**` vs `*NEVER*`

**LLM Impact**: Minor. Inconsistent formatting may slightly reduce agent's ability to parse urgency levels.

**Required Action**:
Standardize across all fragments:
- `**MUST**` (bold) for requirements
- `**NEVER**` (bold) for prohibitions
- `**MAY**` (bold) for permissions
- Use UPPERCASE + BOLD for maximum emphasis

**Files Requiring Update**:
- `general/overall.md`
- `engineering/general-principles.md`
- `python/basics.md`
- `python/linting.md`
- `shell/error-handling.md`, `shell/portability.md`, `shell/style.md`

---

## 6. ACTIONABLE REMEDIATION STEPS

Execute these steps in order. Each step includes verification commands.

### Phase 1: Critical Fixes (Immediate)

#### Step 1.1: Fix Line Endings Duplication (CRIT-001)

```bash
# Edit engineering/general-principles.md
# Remove line 5: *YOU MUST NEVER USE WINDOWS LINE ENDINGS*...
```

**Verification**:
```bash
grep -n "WINDOWS LINE ENDINGS" library/engineering/general-principles.md
# Should return no results

grep -n "WINDOWS LINE ENDINGS" library/general/overall.md
# Should return exactly one result
```

---

#### Step 1.2: Fix Shell Execution Pattern (CRIT-002)

**Action**: Replace section in `general/overall.md:6-12`

**Verification**:
```bash
grep -A5 "SHELL EXECUTION INVARIANT" library/general/overall.md
# Should mention "absolute paths" and "NEVER use cd"
# Should NOT mention "pushd" or "popd"
```

---

#### Step 1.3: Fix SQL Syntax Error (CRIT-004)

**Action**: Edit `python/api-testing.md:13`

**Verification**:
```bash
grep "UNIQUE" library/python/api-testing.md
# Should show: UNIQUE (api_version, api_path, params)
```

---

#### Step 1.4: Fix Typos (CRIT-007, EFF-011, EFF-012)

**Action**: Fix in `rust/cli.md:7`

**Verification**:
```bash
grep -i "sbucommand\|abotu" library/rust/cli.md
# Should return no results
```

---

#### Step 1.5: Fix Just Rules Numbering (CRIT-008)

**Action**: Remove numbering from `just/rules.md`, use bullet list

**Verification**:
```bash
head -20 library/just/rules.md
# Should show bullet list (- or *), not numbered list
```

---

#### Step 1.6: Fix SQL Vague Language (CRIT-006)

**Action**: Replace line 2 in `database/sql.md`

**Verification**:
```bash
grep -i "consider" library/database/sql.md
# Should return no results
grep "Use views or materialized views when" library/database/sql.md
# Should return the new directive
```

---

### Phase 2: Merge Operations (High Priority)

#### Step 2.1: Merge Version Management Files (CRIT-003)

**Status**: Completed – consolidated into `tooling/version-management.md`

**Verification**:
```bash
ls library/engineering/versions.md 2>&1
# Should show "No such file or directory"

grep "tooling/version-management.md" ~/.config/prompter/config.toml
# Confirms new profile reference

grep "SEMVER" library/tooling/version-management.md
# Shows corrected spelling and consolidated content
```

---

#### Step 2.2: Merge Database General into DDL (ATOM-003)

**Actions**:
1. Add PostgreSQL platform statement to top of `database/ddl.md`
2. Delete `database/general.md`
3. Update `config.toml`

**Verification**:
```bash
ls library/database/general.md 2>&1
# Should show "No such file or directory"

grep "PostgreSQL" library/database/ddl.md | head -1
# Should show platform statement
```

---

#### Step 2.3: Merge Core Technologies into General Principles (ATOM-004)

**Actions**:
1. Add "Technology Stack" section to `engineering/general-principles.md`
2. Delete `engineering/core-technologies.md`
3. Update `config.toml`

**Verification**:
```bash
ls library/engineering/core-technologies.md 2>&1
# Should show "No such file or directory"

grep "Technology Stack" library/engineering/general-principles.md
# Should show new section
```

---

### Phase 3: Split Operations (High Priority)

#### Step 3.1: Split general/plan.md (ATOM-001)

**Actions**:
1. Create `library/documentation/plan-structure.md`
2. Create `library/workflow/` directory
3. Create `library/workflow/plan-execution.md`
4. Create `library/workflow/plan-tracking.md`
5. Delete `library/general/plan.md`
6. Update `config.toml`

**Verification**:
```bash
ls library/workflow/plan-*.md
# Should show plan-execution.md and plan-tracking.md

ls library/documentation/plan-structure.md
# Should exist

ls library/general/plan.md 2>&1
# Should show "No such file or directory"
```

---

#### Step 3.2: Consolidate Plan Documentation Fragments (ATOM-006)

**Actions**:
1. After Step 3.1, review `documentation/plans.md` and `documentation/plan-structure.md`
2. Merge into single `documentation/plan-structure.md`
3. Delete `documentation/plans.md`
4. Update `config.toml`

**Verification**:
```bash
ls library/documentation/plans.md 2>&1
# Should show "No such file or directory"

wc -l library/documentation/plan-structure.md
# Should show comprehensive merged content
```

---

### Phase 4: Remove Project-Specific Content (High Priority)

#### Step 4.1: Remove engineering/workflow.md (ATOM-002, ORG-003)

**Actions**:
1. Move to project-specific location (if needed): `mv library/engineering/workflow.md ~/Projects/nucleus-infra/docs/`
2. Update `config.toml` to remove reference

**Verification**:
```bash
ls library/engineering/workflow.md 2>&1
# Should show "No such file or directory"

grep "engineering/workflow.md" ~/.config/prompter/config.toml
# Should return no results
```

---

### Phase 5: Effectiveness Improvements (Medium Priority)

#### Step 5.1: Fix CLI Emotional Language (EFF-002)

**Status**: Completed in `tooling/cli.md`

**Verification**:
```bash
grep -i "be fun\|cool" library/tooling/cli.md
# Should return no results

grep "Use terminal effects when output is to TTY" library/tooling/cli.md
# Shows the new directive
```

---

#### Step 5.2: Strengthen Shell Script Directives (EFF-003, EFF-004)

**Status**: Completed via split into `shell/error-handling.md`, `shell/portability.md`, and `shell/style.md`

**Verification**:
```bash
ls library/shell
# Should list error-handling.md, portability.md, style.md

grep "uname" library/shell/portability.md
# Shows explicit platform detection example
```

---

#### Step 5.3: Clarify Python Documentation Terms (EFF-005, EFF-006)

**Actions**: Edit `python/documentation.md`

**Verification**:
```bash
grep "rolled up" library/python/documentation.md
# Should return no results

grep "Be explicit in writing" library/documentation.md
# Should return no results (line removed)
```

---

#### Step 5.4: Fix DDL "Truly Insane" Directive (EFF-008)

**Action**: Edit `database/ddl.md:12`

**Verification**:
```bash
grep "truly insane" library/database/ddl.md
# Should return no results

grep "If DDL rules conflict with requirements" library/database/ddl.md
# Should show new directive with examples
```

---

#### Step 5.5: Remove Temporal Version Reference (EFF-007)

**Action**: Edit `python/basics.md:3`

**Verification**:
```bash
grep "as of 2025" library/python/basics.md
# Should return no results
```

---

#### Step 5.6: Update Rust CLI to Reference General CLI (CRIT-005)

**Status**: Completed – `rust/cli.md` references `tooling/cli.md`

**Verification**:
```bash
grep "extends.*tooling/cli" library/rust/cli.md
# Shows reference to parent fragment
```

---

### Phase 6: Config Validation

#### Step 6.1: Validate Config Syntax

```bash
prompter validate
# Should return no errors
```

---

#### Step 6.2: Test Profile Rendering

```bash
# Test key profiles
prompter full-stack.backend > /tmp/full-stack-backend.txt
prompter cli.rust > /tmp/cli-rust.txt
prompter devops.architecture > /tmp/devops-architecture.txt
prompter planning.full-stack > /tmp/planning-full-stack.txt

# Verify no missing file errors
grep -i "missing\|not found" /tmp/full-stack-backend.txt /tmp/cli-rust.txt /tmp/devops-architecture.txt /tmp/planning-full-stack.txt
# Should return no results
```

---

#### Step 6.3: Verify Fragment Count

```bash
find library -name "*.md" -type f | wc -l
# Count should be reduced from 28 after merges/deletes

find library -type d
# Should include new workflow/ directory
```

---

### Phase 7: Documentation Update

#### Step 7.1: Update CLAUDE.md References

Review `/Users/jfb/.local/prompter/CLAUDE.md` and update any references to:
- Moved fragments
- Deleted fragments
- New categories

---

#### Step 7.2: Update README (if exists)

Check for README.md in library and update fragment inventory.

---

## 7. VERIFICATION CHECKLIST

After completing all remediation steps, verify:

- [x] No duplicate line endings rule (confirmed 2025-10-11)
- [x] Shell execution uses absolute paths, not pushd/popd (`general/overall.md` updated)
- [x] Version management consolidated into single fragment (`tooling/version-management.md`)
- [x] SQL syntax correct in `python/api-testing.md` (UNIQUE constraint fixed)
- [x] No typos in `rust/cli.md`
- [x] Just rules use bullets, not numbered list
- [x] SQL directives use MUST/WHEN, not "consider" (`database/sql.md`)
- [x] Database `general.md` merged into `ddl.md`
- [x] Platform/tooling defaults captured in `environment/` and referenced from `engineering/general-principles.md`
- [x] `general/plan.md` split into dedicated fragments (`documentation/plan-structure.md`, `workflow/plan-execution.md`, `workflow/plan-tracking.md`)
- [x] `workflow/` directory exists
- [x] `engineering/workflow.md` removed
- [x] CLI emotional language replaced with objective directives (`tooling/cli.md`)
- [x] Shell script directives include explicit examples (`shell/` fragments)
- [x] Python documentation jargon clarified (`python/documentation.md`)
- [x] DDL "truly insane" language replaced with concrete escalation examples (`database/ddl.md`)
- [x] Temporal version references removed (`python/basics.md`)
- [x] `rust/cli.md` references `tooling/cli.md`
- [x] `prompter validate` passes (2025-10-11)
- [x] All profiles render without errors (`for p in $(prompter list); do prompter "$p"; done` on 2025-10-11)
- [x] Config.toml updated for all moves/merges/deletes (`~/.config/prompter/config.toml`)

---

## 8. POST-REMEDIATION METRICS

Expected final state:

| Category | Current Count | Target Count | Change |
|----------|---------------|--------------|--------|
| database/ | 3 | 2 | -1 (merge general into ddl) |
| documentation/ | 3 | 2 | -1 (merge plans fragments) |
| engineering/ | 6 | 3 | -3 (merge versions, merge core-tech, remove workflow) |
| general/ | 3 | 2 | -1 (split plan.md) |
| just/ | 1 | 1 | 0 |
| python/ | 7 | 7 | 0 |
| rust/ | 1 | 1 | 0 (content updated) |
| shell/ | 1 | 1 | 0 (content updated) |
| workflow/ | 0 | 2 | +2 (new category from plan split) |
| **TOTAL** | **25** | **21** | **-4 fragments** |

**Quality Improvements**:
- 8 critical issues resolved
- 6 atomicity issues addressed
- 4 overlap issues eliminated
- 5 organization issues corrected
- 15 effectiveness issues fixed
- **38 total issues resolved**

---

## 9. APPENDIX: Fragment Dependency Map

After remediation, profile dependencies should be:

```toml
[core.baseline]
depends_on = [
    "general/critical-interaction-rules.md",
    "general/overall.md"
]

[engineering.stack]
depends_on = [
    "engineering/general-principles.md",
    "environment.defaults",
    "tooling.cli",
    "tooling.version-management",
    "shell/error-handling.md",
    "shell/portability.md",
    "shell/style.md",
    "just/rules.md"
]

[workflow.execution]
depends_on = [
    "workflow/plan-execution.md",
    "workflow/plan-tracking.md"
]

[workflow.planning]
depends_on = [
    "documentation/plan-structure.md",
    "documentation/todos.md"
]

[python.full]
depends_on = [
    "python.basics",
    "python.quality",
    "python/api-generation.md",
    "python/api-testing.md"
]

[database.all]
depends_on = [
    "database/ddl.md",
    "database/sql.md"
]

[full-stack.backend]
depends_on = [
    "core.baseline",
    "engineering.stack",
    "workflow.execution",
    "python.full",
    "database.all"
]

[full-stack.python]
depends_on = [
    "core.baseline",
    "engineering.stack",
    "workflow.execution",
    "python.full"
]

[full-stack.typescript]
depends_on = []

[cli.rust]
depends_on = [
    "core.baseline",
    "engineering.stack",
    "workflow.execution",
    "rust/cli.md"
]

[devops.architecture]
depends_on = [
    "core.baseline",
    "environment.defaults",
    "documentation/basics.md"
]

[devops.deployment]
depends_on = [
    "core.baseline",
    "engineering.stack",
    "workflow.execution"
]

[planning.full-stack]
depends_on = [
    "core.baseline",
    "workflow.planning"
]

[planning.devops]
depends_on = [
    "core.baseline",
    "workflow.planning",
    "environment.defaults"
]
```

---

## 10. IMPLEMENTATION NOTES FOR AGENTS

When executing this audit:

1. **Work sequentially** through phases
2. **Verify each step** before proceeding
3. **Update config.toml** immediately after file operations
4. **Test with `prompter validate`** after each phase
5. **Preserve git history** - commit each phase separately
6. **Create backups** before deletions
7. **Test profile rendering** after config changes

**If any step fails**:
- Stop immediately
- Document the failure
- Seek operator guidance
- Do not proceed to dependent steps

**Commit strategy**:
```bash
git add library/engineering/general-principles.md
git commit -m "fix: remove duplicate line endings rule (CRIT-001)"

git add library/tooling/version-management.md
git rm library/engineering/versions.md
git add ~/.config/prompter/config.toml
git commit -m "refactor: consolidate version management fragments (CRIT-003)"

# ... one commit per logical change group
```

---

## END OF AUDIT REPORT

**Next Steps**: Assign remediation to agent or execute manually following phases 1-7.
