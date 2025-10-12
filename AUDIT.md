# Prompter Library Audit Report

**Audit Date**: 2025-10-11
**Library Location**: `~/.local/prompter/library/`
**Total Fragments Audited**: 28 fragments across 8 categories
**Config File**: `~/.config/prompter/config.toml`

## Executive Summary

| Category | Open | Severity |
|----------|-------|----------|
| Critical Issues | 0 | CRITICAL |
| Atomicity Issues | 0 | HIGH |
| Overlap Issues | 0 | MEDIUM |
| Organization Issues | 0 | MEDIUM |
| Effectiveness Issues | 0 | HIGH |
| **Total Open Issues** | **0** | - |

**Remediation Progress (2025-10-11)**:
- Resolved CRIT-001 through CRIT-008, ATOM-001 through ATOM-006, ORG-001 through ORG-005, EFF-001 through EFF-015
- Corrected typos logged as EFF-011 and EFF-012
- Remaining audit tasks: run validation (Phase 6) and keep documentation in sync as profiles evolve

**Recommendation**: Address all CRITICAL issues immediately. HIGH severity issues significantly impact LLM agent comprehension and execution accuracy.

---

## 1. CRITICAL ISSUES

These issues create contradictions, ambiguity, or execution failures for LLM agents.

### CRIT-001: Line Endings Rule Duplication (Contradiction)

**Severity**: CRITICAL  
**Status**: ✅ Resolved 2025-10-11 (duplicate directive removed from `engineering/general-principles.md`)  
**Locations**:
- `general/overall.md:3`
- `engineering/general-principles.md:5`

**Original State**: Both fragments repeated "YOU MUST NEVER USE WINDOWS LINE ENDINGS."

**LLM Impact**: Duplicate instructions waste tokens and create uncertainty about the authoritative source.

**Resolution Snapshot**: `engineering/general-principles.md` now stops at rule 4, leaving `general/overall.md` as the single source of truth.

**Config Impact**: None (no profile dependencies change)

---

### CRIT-002: Obsolete Shell Execution Pattern

**Severity**: CRITICAL  
**Status**: ✅ Resolved 2025-10-11 (shell invariant rewritten to ban `cd` usage)  
**Location**: `general/overall.md:6-12`

**Original State**:
```markdown
## SHELL EXECUTION INVARIANT

When changing directories to work, you *MUST* use the following shell syntax:

`(pushd {directory}; {shell work goes here}; popd)`

In other words, you should *never* end a shell command or session in a directory other than the one you were invoked in. This is a persistent source of confusion for LLM agents.
```

**LLM Impact**: The pushd/popd pattern conflicts with the global rule to avoid `cd`, forcing agents to reconcile contradictions.

**Resolution Snapshot**: Section now mandates absolute paths, allows subshells when unavoidable, and reiterates that commands must leave the working directory unchanged.

**Config Impact**: None

---

### CRIT-003: Version Management Fragments Overlap (Duplication)

**Severity**: CRITICAL  
**Status**: ✅ Resolved 2025-10-11 (merged into `tooling/version-management.md` and config updated)

**Original State**: Guidance was split between `engineering/version-management.md` (invariants) and `engineering/versions.md` (versioneer usage), forcing agents to reconcile duplicate content and typos.

**Resolution Snapshot**:
- `tooling/version-management.md` now contains the unified structure (invariants, versioneer commands, SEMVER rules, sync check).
- `engineering/versions.md` was deleted (see `git rm`).
- `/Users/jfb/.config/prompter/config.toml` no longer references the removed fragment.

**Verification**:
```bash
ls library/engineering/versions.md  # No such file or directory
rg "engineering/versions.md" /Users/jfb/.config/prompter/config.toml  # no matches
rg "SEMVER" library/tooling/version-management.md  # confirms corrected phrasing
```

---

### CRIT-004: SQL Unique Constraint Syntax Error

**Severity**: CRITICAL  
**Status**: ✅ Resolved 2025-10-11 (constraint rewritten with parentheses and consistent casing)  
**Location**: `python/api-testing.md:13`

**Original State**:
```sql
UNIQUE api_Version, api_path, params
```

**LLM Impact**: Invalid SQL syntax led to broken DDL and inconsistent column casing.

**Resolution Snapshot**: The `api_test_results` table now declares `UNIQUE (api_version, api_path, params)`, preserving valid syntax and consistent lowercase column names.

---

### CRIT-005: CLI Rules Duplication Between Fragments

**Severity**: CRITICAL  
**Status**: ✅ Resolved 2025-10-11 (`rust/cli.md` now extends the general CLI rules)

**Original State**: Both `engineering/cli.md` and `rust/cli.md` repeated the same `--help`/`version` requirements, giving conflicting emphasis.

**Resolution Snapshot**:
- `rust/cli.md` now opens with an explicit reference to `tooling/cli.md`.
- File only lists Rust-specific additions (license subcommand, enforced use of `clap`, subcommand help) plus implementation notes.

**Verification**:
```bash
sed -n '1,40p' library/rust/cli.md
```

---

### CRIT-006: Vague Language - "Consider" Pattern

**Severity**: CRITICAL  
**Status**: ✅ Resolved 2025-10-11 (directive now lists concrete triggers for views/materialized views)  
**Location**: `database/sql.md:2`

**Original State**:
```markdown
2. consider views, even materialized views, when queries in client code get complex;
```

**LLM Impact**: "Consider" provided no decision threshold, so agents could not determine when views were mandatory.

**Resolution Snapshot**: The directive now enumerates four concrete triggers requiring views or materialized views.

---

### CRIT-007: Typo in Rust CLI Fragment

**Severity**: CRITICAL (breaks parsing)
**Status**: ✅ Resolved 2025-10-11 (typos corrected in `library/rust/cli.md`)
**Location**: `rust/cli.md:7`

**Current State**:
```markdown
3. support a `license` sbucommand that outputs the license information abotu the tool
```

**Typos**: "sbucommand" → "subcommand", "abotu" → "about"

**LLM Impact**: Typos reduce agent confidence in instruction validity. May skip or misinterpret.

**Resolution Snapshot**:
Instruction now reads `support a \`license\` subcommand that outputs the license information about the tool`, restoring accurate spelling.

---

### CRIT-008: Missing Numbered Item in Just Rules

**Severity**: CRITICAL (breaks list parsing)
**Status**: ✅ Resolved 2025-10-11 (rules reformatted as a bullet list)
**Location**: `just/rules.md:5-6`

**Current State**:
```markdown
4. **IF you need shell logic** - create separate shell scripts in `scripts/`
6. **Keep justfile targets simple** - they should call external scripts/functions
```

Missing item 5 in numbered list.

**LLM Impact**: Breaks list parsing. Agent may not process subsequent items correctly.

**Resolution Snapshot**:
Numbered list replaced with bullets to maintain invariant ordering while preserving sub-points for dependency-only recipes.

---

## 2. ATOMICITY ISSUES

Fragments should be focused on a single concern while remaining useful standalone.

### ATOM-001: Fragment Too Large - general/plan.md

**Severity**: HIGH  
**Status**: ✅ Resolved 2025-10-11 (split into documentation + workflow fragments)

**Resolution Snapshot**:
- Created `documentation/plan-structure.md` for plan formatting requirements and deleted `general/plan.md`.
- Added `workflow/plan-execution.md` and `workflow/plan-tracking.md` to capture execution and tracker workflows.
- Updated `/Users/jfb/.config/prompter/config.toml` so `core.interaction` depends on the new files.

**Verification**:
```bash
ls library/workflow/plan-*.md  # shows plan-execution.md and plan-tracking.md
sed -n '1,40p' library/documentation/plan-structure.md
rg "general/plan.md" /Users/jfb/.config/prompter/config.toml  # no matches
```

---

### ATOM-002: Fragment Too Large - engineering/workflow.md

**Severity**: HIGH  
**Status**: ✅ Resolved 2025-10-11 (project-specific workflow removed)

**Resolution Snapshot**:
- Deleted `library/engineering/workflow.md`; no reusable fragment remains.
- Updated `/Users/jfb/.config/prompter/config.toml` so `core.engineering` excludes the workflow file.
- `CLAUDE.md` key fragment list now references the workflow replacements (`workflow/plan-*.md`).

**Verification**:
```bash
ls library/engineering/workflow.md 2>&1  # No such file or directory
rg "engineering/workflow.md" /Users/jfb/.config/prompter/config.toml  # no matches
rg "workflow/plan" CLAUDE.md  # confirms updated references
```

---

### ATOM-003: Fragment Too Small - database/general.md

**Severity**: MEDIUM  
**Status**: ✅ Resolved 2025-10-11 (folded into `database/ddl.md`)  
**Location**: `database/general.md` (1 line)

**Original State**:
```markdown
# Database Standards

We use PostgreSQL exclusively; always the latest version.
```

**LLM Impact**: Micro-fragment offered no actionable guidance yet consumed a profile slot.

**Resolution Snapshot**:
- `database/ddl.md` now starts with “We target PostgreSQL (latest stable release)” and notes availability of advanced Postgres features.
- `database/general.md` was deleted and removed from the `database.all` profile.

**Verification**:
```bash
ls library/database/general.md 2>&1  # No such file or directory
rg "PostgreSQL (latest stable release)" library/database/ddl.md
rg "database/general.md" /Users/jfb/.config/prompter/config.toml  # no matches
```

---

### ATOM-004: Fragment Too Small - engineering/core-technologies.md

**Severity**: MEDIUM
**Location**: `engineering/core-technologies.md` (7 lines)

**Current State**:
```markdown
# Core Technologies

- **AWS** multi-region cloud platform
- **Python** 3.13+ (latest release requirement)
- **PostgreSQL** 17.5 with pgAudit extension
- **just** command runner for development workflows
- **uv** for Python package management (ONLY tool to use, never pip/poetry/pdm)
```

**LLM Impact**:
1. This is metadata, not instructions
2. No behavioral directives for agent
3. Unclear what agent should DO with this info

**Status**: ✅ Resolved 2025-10-11 (integrated into `engineering/general-principles.md`)

**Resolution Snapshot**:
- Added “## Technology Stack (Default)” section to `engineering/general-principles.md` with explicit “Use these platform defaults unless overridden” language.
- Deleted `engineering/core-technologies.md` and removed it from the `core.engineering` profile.

**Verification**:
```bash
sed -n '1,30p' library/engineering/general-principles.md  # shows new Technology Stack section
ls library/engineering/core-technologies.md 2>&1  # No such file or directory
rg "core-technologies" /Users/jfb/.config/prompter/config.toml  # no matches
```

---

### ATOM-005: Potential Split - shell/standards.md

**Severity**: LOW  
**Status**: ✅ Resolved 2025-10-11 (shell guidance split into focused fragments)

**Resolution Snapshot**:
- Created `shell/error-handling.md` (fail-fast rules, `set -euo pipefail`, explicit branching) and `shell/portability.md` (cross-platform requirements, `uname` template, path resolution).
- Created `shell/style.md` for Bash tooling/style guide enforcement.
- Deleted `shell/standards.md` and updated profiles to depend on the new fragments individually.

**Verification**:
```bash
ls library/shell
rg "Shell Error Handling" library/shell/error-handling.md
rg "Shell Portability" library/shell/portability.md
rg "Shell Style" library/shell/style.md
```

---

### ATOM-006: Documentation Plans Fragment Overlap

**Severity**: MEDIUM  
**Status**: ✅ Resolved 2025-10-11 (consolidated into `documentation/plan-structure.md`)

**Resolution Snapshot**:
- Removed `documentation/plans.md` and folded the checklist/rationale requirements into `documentation/plan-structure.md`.
- New plan-structure fragment now covers directory organization, required sections, and audience guidance.

**Verification**:
```bash
ls library/documentation/plans.md 2>&1  # No such file or directory
rg "TODO" library/documentation/plan-structure.md  # shows required checklist language
```

---

## 3. OVERLAP ISSUES

### OVER-001: Line Endings Duplication

**Status**: ✅ Resolved via CRIT-001 on 2025-10-11
**Action**: Remove from `engineering/general-principles.md:5`

---

### OVER-002: Version Management Duplication

**Status**: ✅ Resolved via CRIT-003 on 2025-10-11
**Action**: Merge `engineering/versions.md` into `tooling/version-management.md`

---

### OVER-003: CLI Standards Duplication

**Status**: ✅ Resolved via CRIT-005 on 2025-10-11
**Action**: Update `rust/cli.md` to reference `tooling/cli.md` and contain only Rust-specific additions

---

### OVER-004: Plan Structure Duplication

**Status**: ✅ Resolved via ATOM-006 on 2025-10-11
**Action**: Consolidated into `documentation/plan-structure.md`

---

## 4. ORGANIZATION ISSUES

### ORG-001: Missing workflow/ Category

**Severity**: MEDIUM  
**Status**: ✅ Resolved 2025-10-11 (workflow directory created)

**Resolution Snapshot**:
- Added `library/workflow/` with `plan-execution.md` and `plan-tracking.md` from the ATOM-001 split.
- Updated config so `core.interaction` depends on these workflow fragments.

**Verification**:
```bash
ls library/workflow
rg "workflow/plan" /Users/jfb/.config/prompter/config.toml
```

---

### ORG-002: general/plan.md Misplaced

**Severity**: MEDIUM  
**Status**: ✅ Resolved via ATOM-001 on 2025-10-11

**Resolution Snapshot**:
- Removed `general/plan.md` and redistributed content to documentation/workflow directories.
- Verified new files live in categories that match their purpose.

---

### ORG-003: engineering/workflow.md Project-Specific

**Severity**: HIGH  
**Status**: ✅ Resolved via ATOM-002 on 2025-10-11

**Resolution Snapshot**: Project-specific workflow content removed from the library; config and documentation now point to reusable workflow fragments instead.

---

### ORG-004: engineering/ Category Too Broad

**Severity**: LOW  
**Status**: ✅ Resolved 2025-10-11 (tooling fragments moved under `tooling/`)

**Resolution Snapshot**: `engineering/` now contains only `general-principles.md`; CLI and version management guidance live in `tooling/`, and workflow material moved to `workflow/`.

**Verification**:
```bash
ls library/engineering
ls library/tooling
```

---

### ORG-005: Consider environment/ Category

**Severity**: LOW  
**Status**: ✅ Resolved 2025-10-11 (created environment/ category)

**Resolution Snapshot**:
- Added `environment/platforms.md` for AWS, OS, and infrastructure defaults.
- Added `environment/tooling.md` for required utilities (`just`, `uv`, `versioneer`, lint tools).
- Updated profiles so `core.engineering` depends on `environment.defaults`.

**Verification**:
```bash
ls library/environment
rg "environment.defaults" /Users/jfb/.config/prompter/config.toml
```

---

## 5. LLM-AGENT EFFECTIVENESS ISSUES

### EFF-001: Vague "Consider" Language

**Status**: ✅ Resolved via CRIT-006 on 2025-10-11  
**Location**: `database/sql.md:2`
**Action**: Replace with explicit WHEN conditions

---

### EFF-002: Emotional Language in CLI Rules

**Severity**: HIGH  
**Status**: ✅ Resolved 2025-10-11 (`tooling/cli.md` now uses imperative guidance)

**Resolution Snapshot**:
- Replaced "Be fun!" language with explicit requirements: detect TTY, limit styling to purposeful, accessible ANSI usage, and keep a single Rust binary.
- Added directives for providing `--help`, `--version`, and subcommand patterns while retaining TTY checks.

**Verification**:
```bash
rg "Be fun" -n library/tooling/cli.md  # no matches
sed -n '1,40p' library/tooling/cli.md  # shows new directive list
```

---

### EFF-003: Soft "Be Mindful" Language

**Severity**: MEDIUM  
**Status**: ✅ Resolved 2025-10-11 (shell portability rules made explicit)

**Resolution Snapshot**: `shell/portability.md` now mandates cross-platform testing, POSIX tooling, and operator approval for GNU-only flags.

**Verification**:
```bash
rg "Ensure every script runs on both" library/shell/portability.md
```

---

### EFF-004: Unclear "Check System Type"

**Severity**: MEDIUM  
**Status**: ✅ Resolved 2025-10-11 (added explicit `uname` switch-case example)

**Resolution Snapshot**: `shell/portability.md` now shows the full `case "$(uname -s)"` pattern, including an unsupported-platform guard.

**Verification**:
```bash
rg "Unsupported platform" library/shell/portability.md
```

---

### EFF-005: Ambiguous "Rolled Up" Term

**Severity**: MEDIUM  
**Status**: ✅ Resolved 2025-10-11 (clarified non-public doc guidance)

**Resolution Snapshot**: `python/documentation.md` now states that non-public members must be documented but excluded from public API surfaces (`__all__`, public docs).

**Verification**:
```bash
rg "\*\*MUST NOT\*\* be surfaced" library/python/documentation.md
```

---

### EFF-006: Vague "Be Explicit" Directive

**Severity**: LOW  
**Status**: ✅ Resolved 2025-10-11 (removed redundant commentary)

**Resolution Snapshot**: Deleted the non-actionable sentence from `python/documentation.md`; remaining numbered rules provide concrete expectations.

**Verification**:
```bash
rg "Be explicit in writing" library/python/documentation.md  # no matches
```

---

### EFF-007: Temporal Version Reference

**Severity**: LOW  
**Status**: ✅ Resolved 2025-10-11 (removed stale date reference)

**Resolution Snapshot**: `python/basics.md` now mandates using the latest stable Python release and instructs engineers to confirm the version on python.org before creating environments.

**Verification**:
```bash
rg "latest stable Python" library/python/basics.md
```

---

### EFF-008: Unclear "Truly Insane" Threshold

**Severity**: MEDIUM  
**Status**: ✅ Resolved 2025-10-11 (replaced with explicit escalation guidance)

**Resolution Snapshot**: `database/ddl.md` now lists specific scenarios that require operator approval before relaxing DDL invariants.

**Verification**:
```bash
rg "If any rule conflicts" -n library/database/ddl.md
```

---

### EFF-009: Meta-Commentary Instead of Instruction

**Severity**: LOW  
**Status**: ✅ Resolved via ATOM-006 on 2025-10-11

**Resolution Snapshot**:
- `documentation/plan-structure.md` now states: “Write for a senior software engineer collaborating with an LLM agent—be explicit about goals, acceptance criteria, and hand-offs.”
- Removed the prose-only sentence from the deleted `documentation/plans.md` fragment.

---

### EFF-010: Metadata List Without Behavioral Directive

**Severity**: MEDIUM  
**Status**: ✅ Resolved via ATOM-004 on 2025-10-11

`engineering/core-technologies.md` was removed; the technology requirements now live in `engineering/general-principles.md` as imperative defaults.

---

### EFF-011: Typo - SEMVAR

**Severity**: LOW (but breaks links)
**Status**: ✅ Resolved 2025-10-11 (`engineering/versions.md` link corrected)
**Location**: `engineering/versions.md:1`

**Current State**:
```markdown
We support [SEMVAR][https://semver.org/].
```

**Typos**: "SEMVAR" should be "SEMVER", also invalid markdown link syntax

**LLM Impact**: Broken reference reduces agent confidence.

**Resolution Snapshot**:
Fragment now states `We support [SEMVER](https://semver.org/)`, providing a valid link with proper spelling.

---

### EFF-012: Typo - "arguemnt"

**Severity**: LOW
**Status**: ✅ Resolved 2025-10-11 (`engineering/versions.md` wording corrected)
**Location**: `engineering/versions.md:7`

**Current State**:
```markdown
`versioneer` takes a single arguemnt: `patch`, `minor`, or `major`
```

**Typo**: "arguemnt" → "argument"

**Resolution Snapshot**:
Sentence now reads `` `versioneer` takes a single argument: `patch`, `minor`, or `major` ``.

---

### EFF-013: Missing Context for Error Handling Rationale

**Severity**: LOW  
**Status**: ✅ Resolved 2025-10-11 (documented rationale for error suppression rule)

**Resolution Snapshot**: `shell/error-handling.md` now explains why `|| true` is prohibited and demonstrates the correct conditional alternative.

**Verification**:
```bash
rg "handle_failure" library/shell/error-handling.md
```

---

### EFF-014: Ambiguous "Enough Information" Threshold

**Severity**: LOW  
**Status**: ✅ Resolved via ATOM-001 on 2025-10-11

**Resolution Snapshot**: `documentation/plan-structure.md` lists the four required inputs (definition of done, constraints, dependencies, acceptance criteria) before plan creation.

**Verification**:
```bash
rg "definition of done" library/documentation/plan-structure.md
```

---

### EFF-015: Inconsistent Emphasis Formatting

**Severity**: LOW  
**Status**: ✅ Resolved 2025-10-11 (standardized bold emphasis)

**Resolution Snapshot**: Updated `general/overall.md`, `engineering/general-principles.md`, `python/basics.md`, `python/linting.md`, and the new shell fragments (`shell/error-handling.md`, `shell/portability.md`, `shell/style.md`) to use bold uppercase for directives.

**Verification**:
```bash
rg '\*\*MUST\*\*' library/general/overall.md library/engineering/general-principles.md library/python/basics.md library/python/linting.md library/shell/error-handling.md library/shell/portability.md library/shell/style.md
rg '\*\*NEVER\*\*' library/general/overall.md library/engineering/general-principles.md library/python/basics.md library/python/linting.md library/shell/error-handling.md library/shell/portability.md library/shell/style.md
```

---

## 6. ACTIONABLE REMEDIATION STEPS

Execute these steps in order. Each step includes verification commands.

### Phase 1: Critical Fixes (Immediate)

#### Step 1.1: Fix Line Endings Duplication (CRIT-001)

**Status**: ✅ Completed 2025-10-11  
**Verification**:
```bash
grep -n "WINDOWS LINE ENDINGS" library/engineering/general-principles.md
grep -n "WINDOWS LINE ENDINGS" library/general/overall.md
```

---

#### Step 1.2: Fix Shell Execution Pattern (CRIT-002)

**Status**: ✅ Completed 2025-10-11  
**Verification**:
```bash
grep -A5 "SHELL EXECUTION INVARIANT" library/general/overall.md
```

---

#### Step 1.3: Fix SQL Syntax Error (CRIT-004)

**Status**: ✅ Completed 2025-10-11  
**Verification**:
```bash
grep "UNIQUE" library/python/api-testing.md
```

---

#### Step 1.4: Fix Typos (CRIT-007, EFF-011, EFF-012)

**Status**: ✅ Completed 2025-10-11  
**Verification**:
```bash
grep -i "sbucommand\\|abotu" library/rust/cli.md
grep -i "semvar\\|arguemnt" library/engineering/versions.md
```

---

#### Step 1.5: Fix Just Rules Numbering (CRIT-008)

**Status**: ✅ Completed 2025-10-11  
**Verification**:
```bash
head -20 library/just/rules.md
```

---

#### Step 1.6: Fix SQL Vague Language (CRIT-006)

**Status**: ✅ Completed 2025-10-11  
**Verification**:
```bash
grep -i "consider" library/database/sql.md
grep "Use views or materialized views when" library/database/sql.md
```

---

### Phase 2: Merge Operations (High Priority)

#### Step 2.1: Merge Version Management Files (CRIT-003)

**Status**: ✅ Completed 2025-10-11  
**Verification**:
```bash
ls library/engineering/versions.md 2>&1  # No such file or directory
rg "engineering/versions.md" /Users/jfb/.config/prompter/config.toml  # no matches
rg "SEMVER" library/tooling/version-management.md  # shows unified content
```

---

#### Step 2.2: Merge Database General into DDL (ATOM-003)

**Status**: ✅ Completed 2025-10-11  
**Verification**:
```bash
ls library/database/general.md 2>&1  # No such file or directory
rg "latest stable release" library/database/ddl.md  # shows new preamble
rg "database/general.md" /Users/jfb/.config/prompter/config.toml  # no matches
```

---

#### Step 2.3: Merge Core Technologies into General Principles (ATOM-004)

**Status**: ✅ Completed 2025-10-11  
**Verification**:
```bash
ls library/engineering/core-technologies.md 2>&1  # No such file or directory
rg "Technology Stack (Default)" library/engineering/general-principles.md
rg "core-technologies" /Users/jfb/.config/prompter/config.toml  # no matches
```

---

### Phase 3: Split Operations (High Priority)

#### Step 3.1: Split general/plan.md (ATOM-001)

**Status**: ✅ Completed 2025-10-11  
**Verification**:
```bash
ls library/workflow/plan-*.md
ls library/documentation/plan-structure.md
ls library/general/plan.md 2>&1
```

---

#### Step 3.2: Consolidate Plan Documentation Fragments (ATOM-006)

**Status**: ✅ Completed 2025-10-11  
**Verification**:
```bash
ls library/documentation/plans.md 2>&1
rg "State:" library/documentation/plan-structure.md
```

---

### Phase 4: Remove Project-Specific Content (High Priority)

#### Step 4.1: Remove engineering/workflow.md (ATOM-002, ORG-003)

**Status**: ✅ Completed 2025-10-11  
**Verification**:
```bash
ls library/engineering/workflow.md 2>&1
rg "engineering/workflow.md" /Users/jfb/.config/prompter/config.toml
```

---

### Phase 5: Effectiveness Improvements (Medium Priority)

#### Step 5.1: Fix CLI Emotional Language (EFF-002)

**Status**: ✅ Completed 2025-10-11  
**Verification**:
```bash
rg "Be fun" library/tooling/cli.md  # no matches
sed -n '1,40p' library/tooling/cli.md
```

---

#### Step 5.2: Strengthen Shell Script Directives (EFF-003, EFF-004)

**Status**: ✅ Completed 2025-10-11  
**Verification**:
```bash
rg "Ensure every script runs on both" library/shell/portability.md
rg "case \"\$(uname -s\)\"" library/shell/portability.md
```

---

#### Step 5.3: Clarify Python Documentation Terms (EFF-005, EFF-006)

**Status**: ✅ Completed 2025-10-11  
**Verification**:
```bash
rg "rolled up" library/python/documentation.md  # no matches
rg "\*\*MUST NOT\*\* be surfaced" library/python/documentation.md
```

---

#### Step 5.4: Fix DDL "Truly Insane" Directive (EFF-008)

**Status**: ✅ Completed 2025-10-11  
**Verification**:
```bash
rg "If any rule conflicts" library/database/ddl.md
```

---

#### Step 5.5: Remove Temporal Version Reference (EFF-007)

**Status**: ✅ Completed 2025-10-11  
**Verification**:
```bash
rg "latest stable Python" library/python/basics.md
```

---

#### Step 5.6: Update Rust CLI to Reference General CLI (CRIT-005)

**Action**: Rewrite `rust/cli.md` with reference to `tooling/cli.md`

**Verification**:
```bash
grep "extends.*tooling/cli" library/rust/cli.md
# Should show reference to parent fragment
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
prompter core.all > /tmp/core-test.txt
prompter python.full > /tmp/python-test.txt
prompter database.all > /tmp/database-test.txt

# Verify no missing file errors
grep -i "error\|missing\|not found" /tmp/*.txt
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

- [ ] No duplicate line endings rule
- [ ] Shell execution uses absolute paths, not pushd/popd
- [ ] Version management consolidated into single fragment
- [ ] SQL syntax correct in api-testing.md
- [ ] No typos in rust/cli.md
- [ ] Just rules use bullets, not numbered list
- [ ] SQL directives use MUST/WHEN, not "consider"
- [ ] Database general.md merged into ddl.md
- [ ] Core technologies merged into general-principles.md
- [ ] general/plan.md split into 3 fragments in correct directories
- [ ] workflow/ directory exists
- [ ] engineering/workflow.md removed or moved
- [ ] CLI emotional language replaced with objective directives
- [ ] Shell script directives include explicit examples
- [ ] Python documentation jargon clarified
- [ ] DDL "truly insane" replaced with concrete examples
- [ ] Temporal version references removed
- [ ] rust/cli.md references tooling/cli.md
- [ ] Environment defaults fragments included in core.engineering
- [ ] `prompter validate` passes
- [ ] All profiles render without errors
- [ ] Config.toml updated for all moves/merges/deletes

---

## 8. POST-REMEDIATION METRICS

Expected final state:

| Category | Current Count | Target Count | Change |
|----------|---------------|--------------|--------|
| database/ | 2 | 2 | -1 (general merged into ddl) |
| documentation/ | 3 | 3 | 0 (plan structure consolidated) |
| engineering/ | 1 | 1 | -5 (tooling/workflow content moved out) |
| general/ | 2 | 2 | -1 (plan split removed extra fragment) |
| just/ | 1 | 1 | 0 |
| python/ | 7 | 7 | 0 |
| rust/ | 1 | 1 | 0 (content updated) |
| shell/ | 3 | 3 | +2 (split into error-handling/portability/style) |
| tooling/ | 2 | 2 | +2 (new category for CLI/version management) |
| workflow/ | 2 | 2 | +2 (plan execution/tracking) |
| **TOTAL** | **24** | **24** | **0** |

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
[core.interaction]
depends_on = [
    "general/critical-interaction-rules.md",
    "general/overall.md"
]

[core.engineering]
depends_on = [
    "engineering/general-principles.md",  # includes core-technologies
    "tooling.cli",
    "tooling.version-management"  # consolidated
]

[workflow.planning]
depends_on = [
    "documentation/plan-structure.md",  # consolidated
    "workflow/plan-execution.md",       # from split
    "workflow/plan-tracking.md"         # from split
]

[python.basics]
depends_on = [
    "python/basics.md",
    "python/type-hints.md",
    "python/uv-package-manager.md"
]

[database.all]
depends_on = [
    "database/ddl.md",  # includes general
    "database/sql.md"
]

# ... other profiles updated similarly
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
