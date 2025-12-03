=========================================
Prompter Library Style Guide Compliance
=========================================

:Date: 2025-12-03
:Status: **COMPLETE**
:Reference: docs/claude-style-guide.rst

This plan addresses style guide compliance issues found during audit of the
prompter library fragments.

-----------------
Progress Summary
-----------------

**Last Updated:** 2025-12-03

.. list-table::
   :header-rows: 1
   :widths: 10 30 15 45

   * - Phase
     - Description
     - Status
     - Notes
   * - 1
     - Add language tags to code blocks
     - **DONE**
     - ~120 code blocks tagged with appropriate languages
   * - 2
     - Rewrite negative framing to positive
     - **DONE**
     - ~60 instances converted across all fragments
   * - 3
     - Split oversized fragments
     - **DONE**
     - All 9 oversized files split into 31 focused fragments
   * - 4
     - Consolidate small fragments
     - **DONE**
     - todos+basics → standards.md; platforms+tooling → defaults.md
   * - 5
     - Standardize emphasis (ALL CAPS → bold)
     - **DONE**
     - Remaining ALL CAPS are SQL keywords (acceptable)

**Completed Splits:**

- engineering/git-standards.md → git-commits.md, git-branches.md, git-pull-requests.md, git-workflow.md
- workflow/work-metadata.md → work-metadata-schema.md, work-metadata-lifecycle.md, work-metadata-tools.md, work-metadata-reference.md
- workflow/issue-tracking.md → issue-tracking-overview.md, issue-tracking-github.md, issue-tracking-workflow.md, issue-tracking-reference.md
- rust/github-actions.md → github-actions-ci.md, github-actions-release.md, github-actions-deps.md
- rust/justfile-patterns.md → justfile-structure.md, justfile-recipes.md, justfile-integration.md
- rust/cli.md → cli-structure.md, cli-features.md, cli-subcommands.md
- rust/release-workflow.md → release-automation.md, release-validation.md, release-reference.md
- rust/git-hooks.md → git-hooks-config.md, git-hooks-setup.md, git-hooks-usage.md
- rust/testing.md → testing-unit.md, testing-integration.md, testing-ci.md

**Completed Merges:**

- documentation/todos.md + documentation/basics.md → documentation/standards.md
- environment/platforms.md + environment/tooling.md → environment/defaults.md

**Final Statistics:**

- Largest fragment: 299 lines (rust/quality-tooling.md)
- All fragments now under 300-line maximum
- Library validates successfully
- All key profiles render correctly

.. contents:: Table of Contents
   :depth: 2
   :local:

-----------------
Executive Summary
-----------------

**48 fragments audited** totaling 6,342 lines.

Key findings:

- **9 fragments exceed 300-line maximum** (need splitting)
- **9 fragments under 20-line minimum** (consider merging)
- **~120+ code blocks missing language tags**
- **~60 instances of negative framing** requiring positive rewrites
- **~10 instances of ALL CAPS emphasis** instead of bold

-----------------
Priority 1: Critical
-----------------

Fragments Exceeding 300-Line Maximum
====================================

These fragments must be split into focused sub-fragments per style guide
section "Fragment Length".

.. list-table::
   :header-rows: 1
   :widths: 40 15 45

   * - Fragment
     - Lines
     - Recommended Split
   * - workflow/work-metadata.md
     - 519
     - Split into: work-metadata-schema.md, work-metadata-usage.md, work-metadata-automation.md
   * - engineering/git-standards.md
     - 498
     - **DONE** — Split into: git-commits.md, git-branches.md, git-pull-requests.md, git-workflow.md
   * - rust/github-actions.md
     - 452
     - Split into: github-actions-ci.md, github-actions-release.md, github-actions-deps.md
   * - rust/justfile-patterns.md
     - 422
     - Split into: justfile-basics.md, justfile-recipes.md, justfile-patterns.md
   * - workflow/issue-tracking.md
     - 411
     - Split into: issue-tracking-github.md, issue-tracking-asana.md, issue-tracking-sync.md
   * - rust/cli.md
     - 393
     - Split into: cli-structure.md, cli-arguments.md, cli-output.md, cli-testing.md
   * - rust/release-workflow.md
     - 358
     - Split into: release-process.md, release-automation.md, release-rollback.md
   * - rust/git-hooks.md
     - 349
     - Split into: git-hooks-config.md, git-hooks-implementation.md
   * - rust/testing.md
     - 337
     - Split into: testing-unit.md, testing-integration.md, testing-ci.md

-----------------
Priority 2: High
-----------------

Missing Language Tags on Code Blocks
====================================

Per style guide: "Always specify language for syntax highlighting."

The following fragments have multiple code blocks without language tags:

**Rust fragments (highest count):**

- rust/github-actions.md — 25+ YAML blocks
- rust/quality-tooling.md — 12+ blocks (toml, bash, just)
- rust/justfile-patterns.md — 9+ just blocks
- rust/cli.md — 12+ blocks (rust, toml, bash)
- rust/testing.md — 7 blocks (rust, yaml, bash, just)
- rust/git-hooks.md — 6 blocks (toml, bash, just)
- rust/release-workflow.md — 5 blocks (bash, yaml)
- rust/project-structure.md — 3 blocks (text, toml)

**Python fragments:**

- python/project-structure.md — 9 blocks (text, python)
- python/data-parsing.md — 4 blocks (python)
- python/migrations.md — 3 blocks (python)
- python/api-testing.md — 3 blocks (python, sql)
- python/logging.md — 2 blocks (python)
- python/linting.md — 2 blocks (python, toml)
- python/documentation.md — 2 blocks (python)
- python/type-hints.md — 2 blocks (python)
- python/configuration.md — 1 block (python)
- python/api-generation.md — 1 block (python)
- python/basics.md — 1 block (python)

**Other fragments:**

- engineering/git-standards.md — 12 blocks (text, markdown, bash)
- security/secrets.md — 5 blocks (bash, python, json)
- workflow/beads-integration.md — 7 blocks (bash)
- workflow/issue-tracking.md — 6 blocks (markdown, bash)
- security/owasp.md — 3 blocks (python)
- tooling/version-management.md — 4 blocks (bash, yaml)
- shell/error-handling.md — 3 blocks (bash)
- shell/style.md — 2 blocks (bash)
- shell/portability.md — 1 block (bash)
- database/sql.md — 2 blocks (python, text)

**Remediation approach:**

For each file, add appropriate language tags:

- Python code: ``python``
- Bash/shell: ``bash``
- YAML: ``yaml``
- TOML: ``toml``
- Rust: ``rust``
- Just recipes: ``just`` or ``makefile``
- SQL: ``sql``
- JSON: ``json``
- Directory structures: ``text`` or no tag
- Markdown templates: ``markdown``

-----------------
Priority 3: Medium
-----------------

Negative Framing Requiring Positive Rewrites
============================================

Per style guide: "Write instructions as **actions to take**, not **actions to
avoid**."

The following instances require transformation to positive framing:

**general/critical-interaction-rules.md** (5 instances)

Current::

    2. **NEVER EXPRESS EMOTION** The agent does not *have* emotions...
    3. **NEVER PRAISE THE USER** The user is not looking for affirmation.
    4. **NEVER SAY YOU UNDERSTAND THE USER'S MENTAL STATE**...
    5. **NEVER OFFER FEEDBACK WITHOUT BEING ASKED**...

Proposed::

    2. **Maintain factual, professional tone** - The agent communicates without emotional expression.
    3. **State facts without affirmation** - The user seeks technical assistance, not praise.
    4. **Report observations, not interpretations** - The agent has no theory of mind.
    5. **Provide feedback only when requested** - The operator determines when review is needed.

**engineering/general-principles.md** (4 instances)

Current::

    1. **NEVER suppress errors** – fail fast...
    2. **NEVER HARDCODE VALUES** even if it simplifies...
    4. **DO NOT USE MOCKS when writing tests**...
    Never attempt recovery from an invariant violation...

Proposed::

    1. **Surface all errors immediately** (fail-fast pattern) – Problems reveal themselves at the source.
    2. **Inject or configure all values** – Hardcoded values create hidden dependencies.
    4. **Test against real systems** – Mocks hide consistency issues between fake and real behavior.
    Crash immediately on invariant violations – Recovery is impossible from corrupted state.

**just/rules.md** (3 instances)

Current::

    - **NEVER use heredocs** in justfile targets...
    - **NEVER use `python -c`** inline code...
    - **NEVER create and execute scripts** within justfile targets

Proposed::

    - **Write shell scripts as separate files** – Heredocs in justfiles create maintenance burden.
    - **Call Python modules, not inline code** – ``python -c`` hides logic and breaks linting.
    - **Reference external scripts from targets** – Inline script generation bypasses version control.

**security/secrets.md** (6 instances)

Current::

    - **Never** commit secrets to git...
    - **Never** log secret values
    - **Never** include secrets in error messages
    - **Never** store secrets in code comments
    - **Never** use the same secrets across environments
    - **Never** share secrets via Slack, email...

Proposed::

    - **Store secrets in .env (local) or SSM (production)** – Git history is permanent and public.
    - **Redact secrets from log output** – Logs are often aggregated and retained.
    - **Use generic error messages for auth failures** – Specific messages leak implementation details.
    - **Document secret locations in README, not code** – Comments are searchable in source.
    - **Generate unique secrets per environment** – Shared secrets spread blast radius.
    - **Share secrets via 1Password or SSM** – Unencrypted channels are logged and archived.

**Additional files with negative framing:**

- engineering/parse-dont-validate.md — 4 instances
- engineering/testing-philosophy.md — 2 instances
- engineering/git-standards.md — 10 instances (DON'T section headers)
- workflow/work-metadata.md — 4 instances (DON'T section headers)
- workflow/issue-tracking.md — 4 instances (DON'T section headers)
- rust/release-workflow.md — 3 instances
- rust/project-structure.md — 1 instance
- shell/error-handling.md — 1 instance
- shell/style.md — 2 instances
- security/owasp.md — 2 instances
- tooling/version-management.md — 3 instances
- python/type-hints.md — 3 instances
- python/api-testing.md — 2 instances
- python/basics.md — 1 instance
- python/linting.md — 1 instance
- python/documentation.md — 2 instances
- python/uv-package-manager.md — 2 instances
- python/migrations.md — 1 instance
- python/project-structure.md — 2 instances
- python/logging.md — 1 instance
- general/overall.md — 2 instances
- general/agent-behavior.md — 2 instances

ALL CAPS Emphasis Requiring Bold Conversion
===========================================

Per style guide: "Use **bold** for key terms... Avoid ALL CAPS for emphasis."

**general/critical-interaction-rules.md**

Current::

    🚨 CRITICAL INTERACTION RULES 🚨
    THESE ARE INVARIANT. DO NOT VIOLATE THESE, WHATEVER YOU DO.
    THESE RULES MUST NEVER BE VIOLATED.

Proposed::

    **Critical Interaction Rules**
    **These rules are invariant. Violation is unacceptable.**
    **These rules take primacy over all other instructions.**

**just/rules.md**

Current::

    🚨 CRITICAL JUSTFILE RULES 🚨
    INVARIANT JUSTFILE RULES - NEVER VIOLATE THESE:

Proposed::

    **Critical Justfile Rules**
    **Invariant rules for justfile targets:**

**tooling/version-management.md**

Current::

    **INVARIANT – NEVER VIOLATE THESE:**
    1. **NEVER manually edit Cargo.toml version**...
    2. **NEVER manually edit VERSION file**...
    3. **NEVER create git tags manually**...

Proposed::

    **Version Management Invariants**
    1. **Use versioneer exclusively for Cargo.toml versions** – Manual edits create sync failures.
    2. **Use versioneer exclusively for VERSION files** – Direct edits bypass validation.
    3. **Use versioneer tag or release scripts for git tags** – Manual tags lack metadata.

**python/uv-package-manager.md**

Current::

    You are **NOT** to **EVER** use **ANY** other tools...

Proposed::

    **Use uv exclusively for Python package management** – pip, poetry, pdm are superseded.

-----------------
Priority 4: Low
-----------------

Fragments Under 20-Line Minimum
===============================

Per style guide: "Minimum: 20 lines (if shorter, consider merging with related
fragment)."

.. list-table::
   :header-rows: 1
   :widths: 40 15 45

   * - Fragment
     - Lines
     - Recommendation
   * - documentation/todos.md
     - 2
     - Merge into documentation/basics.md
   * - documentation/basics.md
     - 4
     - Expand with rationale or merge with plan-structure.md
   * - environment/platforms.md
     - 5
     - Merge into environment/tooling.md
   * - general/critical-interaction-rules.md
     - 10
     - Keep separate (intentionally terse for emphasis)
   * - tooling/cli.md
     - 12
     - Expand with examples or merge into relevant language fragments
   * - general/overall.md
     - 14
     - Keep separate (foundational rules)
   * - just/rules.md
     - 15
     - Keep separate or expand with examples
   * - environment/tooling.md
     - 17
     - Expand with rationale
   * - workflow/plan-tracking.md
     - 18
     - Keep separate (distinct concern from plan-execution.md)

**Recommended merges:**

1. documentation/todos.md + documentation/basics.md → documentation/standards.md — **DONE**
2. environment/platforms.md + environment/tooling.md → environment/defaults.md — **DONE**

Weak Language Instances
=======================

Per style guide: Avoid "try to", "consider", "you might", "good practice",
"when possible".

Only 2 instances found:

1. **rust/cli.md:393** — "Consider implementing update capability"

   Change to: "Implement self-update capability for production CLIs"

2. **engineering/git-standards.md:229** — "strongly consider splitting"

   Change to: "split PRs exceeding 1000 lines"

-----------------
Implementation Plan
-----------------

Phase 1: Code Block Language Tags — **DONE**
=============================================

Mechanical fixes, low risk.

1. Process all rust/* fragments (7 files, ~80 blocks)
2. Process all python/* fragments (13 files, ~25 blocks)
3. Process remaining fragments (~15 blocks)

**Validation:** Each file should have no untagged code blocks after
``grep -E '^\`\`\`$'`` returns zero results.

Phase 2: Negative Framing Rewrites — **DONE**
==============================================

Requires careful attention to preserve meaning while improving clarity.

1. Start with general/* fragments (foundational rules)
2. Process engineering/* fragments
3. Process security/* fragments
4. Process python/* fragments
5. Process rust/* fragments
6. Process workflow/* fragments

**Validation:** ``grep -E '\b(NEVER|DON'\''T|MUST NOT|AVOID|DO NOT)\b'`` should
return only instances that are:

- Paired with positive guidance immediately following
- Safety-critical prohibitions with obvious alternatives

Phase 3: Fragment Splitting — **DONE**
======================================

Highest risk, requires careful dependency analysis.

For each oversized fragment:

1. Identify logical boundaries
2. Create new fragment files
3. Update all profile references in ``~/.config/prompter/config.toml``
4. Validate with ``prompter validate``
5. Test affected profiles render correctly

**Order of operations:**

1. engineering/git-standards.md (highest impact, most references)
2. workflow/work-metadata.md
3. workflow/issue-tracking.md
4. rust/github-actions.md
5. rust/justfile-patterns.md
6. rust/cli.md
7. rust/release-workflow.md
8. rust/git-hooks.md
9. rust/testing.md

Phase 4: Fragment Consolidation — **DONE**
==========================================

Lower priority, optional.

1. Merge documentation/todos.md into documentation/basics.md
2. Merge environment/platforms.md into environment/tooling.md
3. Expand remaining short fragments with rationale

Phase 5: Emphasis Standardization — **DONE**
=============================================

Convert ALL CAPS to **bold** in:

1. general/critical-interaction-rules.md
2. just/rules.md
3. tooling/version-management.md
4. python/uv-package-manager.md

Remove emoji from headers where present.

-----------------
Validation Checklist
-----------------

After all changes:

.. code-block:: bash

   # Validate library integrity
   prompter validate

   # Check for remaining untagged code blocks
   find ~/.local/prompter/library -name "*.md" -exec grep -l '^\`\`\`$' {} \;

   # Check for remaining ALL CAPS emphasis
   grep -rE '\b[A-Z]{5,}\b' ~/.local/prompter/library/*.md

   # Check for remaining weak language
   grep -riE '(try to|consider|you might|good practice|when possible)' ~/.local/prompter/library/

   # Check line counts
   wc -l ~/.local/prompter/library/**/*.md | sort -n | tail -20

   # Test key profiles
   prompter python.full > /dev/null && echo "python.full OK"
   prompter rust.full > /dev/null && echo "rust.full OK"

-----------------
Risk Assessment
-----------------

**Phase 1 (Code blocks):** Low risk — Purely additive changes.

**Phase 2 (Negative framing):** Medium risk — Semantic changes require review
to ensure meaning is preserved.

**Phase 3 (Splitting):** High risk — Profile dependencies may break. Requires
config.toml updates and comprehensive testing.

**Phase 4 (Consolidation):** Low risk — Simple file operations with config
updates.

**Phase 5 (Emphasis):** Low risk — Cosmetic changes only.

-----------------
Success Criteria
-----------------

1. ``prompter validate`` passes
2. All profiles render without error
3. No code blocks without language tags
4. No ALL CAPS emphasis (except in code examples)
5. Negative framing reduced by >80%
6. No fragments over 300 lines
7. No fragments under 10 lines (unless intentionally terse)
