===================================
Prompter Fragment Style Guide
===================================

:Version: 1.0.0
:Date: December 2025
:Audience: Fragment authors and maintainers
:Purpose: Standards for writing effective LLM coding agent instructions

This guide defines how to write prompter fragments that are optimized for
Claude 4.x models (Opus 4.5, Sonnet 4.5, and successors). These models respond
differently than earlier generations, requiring specific patterns for maximum
effectiveness.

.. contents:: Table of Contents
   :depth: 3
   :local:

-----------------
Core Principles
-----------------

Claude 4.x models are characterized by:

1. **Precise instruction following** - They pay close attention to exact wording
2. **Direct action orientation** - They prefer to act rather than discuss
3. **Example sensitivity** - They weight examples heavily in their behavior
4. **Contextual reasoning** - They respond better when they understand WHY

These characteristics inform every guideline in this document.


The Prime Directive
===================

Every instruction in a fragment must answer two questions:

1. **WHAT** should the agent do?
2. **WHY** does this matter?

Instructions without rationale are weaker than instructions with rationale.
Claude 4.x models are more likely to follow rules when they understand the
reasoning behind them.

.. code-block:: text

   WEAK (what only):
   "Use Unix line endings."

   STRONG (what + why):
   "Use Unix line endings. Windows line endings (CRLF) cause diff noise,
   break shell scripts, and create inconsistent behavior across platforms."


-----------------------
Framing: Positive First
-----------------------

Claude 4.x documentation explicitly states: *"Frame positively: 'Write in
flowing prose paragraphs' rather than 'Don't use markdown'"*

Negative framing tells the model what to avoid but leaves ambiguity about
what TO do. Positive framing provides clear direction.


The Positive Framing Rule
=========================

Write instructions as **actions to take**, not **actions to avoid**.

.. list-table:: Negative to Positive Transformations
   :header-rows: 1
   :widths: 40 60

   * - Negative (Avoid)
     - Positive (Preferred)
   * - "NEVER use raw SQL strings"
     - "Use SQLAlchemy query language for all database operations"
   * - "Don't hardcode values"
     - "Inject or configure all values"
   * - "Never suppress errors"
     - "Surface all errors immediately (fail-fast pattern)"
   * - "Don't use mocks in tests"
     - "Test against real systems; use stubs only at external boundaries"
   * - "NEVER express emotion"
     - "Maintain factual, professional tone"
   * - "Don't use cd to change directories"
     - "Use absolute paths for all commands"
   * - "Avoid correlated subqueries"
     - "Use JOINs or CTEs instead of correlated subqueries"
   * - "Don't commit secrets"
     - "Store secrets in .env (local) or SSM (production)"


When Negative Framing Is Acceptable
===================================

Negative framing is acceptable in two cases:

1. **Safety-critical prohibitions** where the positive alternative is obvious:

   .. code-block:: text

      "Never commit credentials to version control."

   The positive alternative (use secret management) is well-understood.

2. **Paired with positive guidance** immediately following:

   .. code-block:: text

      "Avoid in-memory fakes for database testing. Use testcontainers to spin
      up real PostgreSQL instances instead."


Structure for Prohibitions
==========================

When you must state a prohibition, follow this structure:

.. code-block:: text

   [Positive instruction first]
   [Negative clarification if needed]
   [Rationale]

Example:

.. code-block:: text

   Use SQLAlchemy query language for all database operations. Raw SQL strings
   are prohibited. SQLAlchemy handles parameterization automatically,
   preventing SQL injection vulnerabilities.


----------------------------
Contextual Framing: The WHY
----------------------------

Claude 4.x documentation states: *"Explain why your instruction matters.
'Never use ellipses - a text-to-speech engine won't pronounce them correctly'
is more effective than simply stating 'NEVER use ellipses.'"*


The Rationale Requirement
=========================

Every rule must include rationale. Place the WHY immediately after the WHAT.

.. list-table:: Rules With and Without Rationale
   :header-rows: 1
   :widths: 50 50

   * - Without Rationale (Weak)
     - With Rationale (Strong)
   * - "Use the src/ layout for Python projects."
     - "Use the src/ layout for Python projects. This prevents accidental imports from the source directory during testing, ensuring tests run against installed code."
   * - "Enable fail-fast mode in shell scripts."
     - "Enable fail-fast mode (set -euo pipefail) in shell scripts. Silent failures hide bugs and create inconsistent state that is difficult to debug."
   * - "Wrap external dependencies."
     - "Wrap external dependencies in domain-specific interfaces. This provides a seam for testing, isolates API changes, and lets your types match your domain."


Rationale Patterns
==================

Use these patterns to explain WHY:

**Consequence pattern**: What happens if ignored

.. code-block:: text

   "Parse external data into typed containers at system boundaries. Untyped
   data propagating through the system causes runtime errors deep in business
   logic where they are difficult to diagnose."

**Benefit pattern**: What is gained by compliance

.. code-block:: text

   "Use testcontainers for database testing. Real PostgreSQL instances catch
   constraint violations, transaction isolation issues, and SQL dialect
   differences that in-memory fakes hide."

**Context pattern**: When/where this applies

.. code-block:: text

   "Use relative imports within packages, absolute imports for external
   dependencies. This makes refactoring easier and clearly distinguishes
   internal from external code."

**Trade-off pattern**: Why this choice over alternatives

.. code-block:: text

   "Prefer synchronous code by default. Async adds complexity (colored
   functions, viral async/await) while most applications do not have the
   I/O-bound bottlenecks that justify this overhead."


------------------------
Example Precision
------------------------

Claude 4.x documentation states: *"Claude 4.x models pay close attention to
details and examples as part of their precise instruction following
capabilities."*


Example Requirements
====================

1. **Every rule with implementation implications needs an example**
2. **Examples must be syntactically correct and runnable**
3. **Examples must demonstrate exactly the behavior you want**
4. **Bad examples must be clearly labeled and paired with good examples**


Example Structure
=================

Use this structure for code examples:

.. code-block:: text

   [Instruction]
   [Rationale]

   [Good example with label]

   [Bad example with label - optional, only if clarifying]


Good Example Format
-------------------

.. code-block:: markdown

   Use SQLAlchemy ORM for all database queries. This handles parameterization
   automatically and prevents SQL injection.

   ```python
   # Query using SQLAlchemy ORM
   user = session.query(User).filter(User.id == user_id).first()

   # Query using SQLModel
   user = session.exec(select(User).where(User.id == user_id)).first()
   ```


Bad Example Format
------------------

When showing what to avoid, always:

1. Label it clearly as BAD
2. Explain WHY it is bad
3. Follow immediately with the GOOD alternative

.. code-block:: markdown

   Use SQLAlchemy ORM for all database queries. This handles parameterization
   automatically and prevents SQL injection.

   ```python
   # BAD - Raw SQL string allows injection attacks
   query = f"SELECT * FROM users WHERE id = '{user_id}'"

   # GOOD - SQLAlchemy ORM handles parameterization
   user = session.query(User).filter(User.id == user_id).first()
   ```


Example Anti-Patterns
=====================

**Anti-pattern: Bad example without good alternative**

.. code-block:: markdown

   # WRONG - leaves reader without guidance

   ```python
   # BAD - don't do this
   cursor.execute(f"SELECT * FROM users WHERE id = {user_id}")
   ```

**Anti-pattern: Example that doesn't match the instruction**

.. code-block:: markdown

   # WRONG - instruction says ORM, example shows Core

   Use SQLAlchemy ORM for queries.

   ```python
   stmt = select(users_table).where(users_table.c.id == user_id)  # This is Core, not ORM
   ```

**Anti-pattern: Syntactically incorrect example**

.. code-block:: markdown

   # WRONG - missing imports, won't run

   ```python
   user = session.query(User).filter(User.id == user_id).first()
   # Reader doesn't know where session, User come from
   ```

For complex examples, include necessary imports and context.


--------------------------------
Claude 4.x Specific Guidance
--------------------------------

These sections address behaviors specific to Claude 4.x models that should
be reflected in fragments.


Overengineering Prevention
==========================

Claude 4.x models tend to overengineer solutions. Include explicit guidance
against this:

.. code-block:: markdown

   ## Minimal Solutions

   Implement the simplest solution that satisfies the requirement:

   - Address current needs, not hypothetical future requirements
   - Avoid premature abstractions
   - Skip configuration options that aren't immediately needed
   - Don't add features "just in case"

   Three similar lines of code is better than a premature abstraction.


Read Before Edit
================

Claude 4.x can speculate about code without reading it. Include explicit
read-first guidance:

.. code-block:: markdown

   ## Code Understanding First

   Read and understand code before proposing modifications:

   1. Read the file(s) you intend to modify
   2. Understand existing patterns and conventions
   3. Only then propose specific, targeted changes

   Speculation about unread code leads to incorrect modifications.


Parallel Execution
==================

Claude 4.x supports parallel tool calling. Include guidance when relevant:

.. code-block:: markdown

   ## Parallel Operations

   Execute independent operations in parallel:

   - Reading multiple files: parallel tool calls
   - Running independent tests: parallel execution
   - Searching different patterns: parallel searches

   Only sequence operations with dependencies between them.


Incremental Progress
====================

Claude 4.x handles long tasks better with incremental progress patterns:

.. code-block:: markdown

   ## Incremental Progress

   For multi-step tasks:

   1. Complete one step fully before starting the next
   2. Verify each step succeeded before proceeding
   3. Track progress in structured format (issues, todos)
   4. Commit working increments rather than batching all changes


Action-Oriented Language
========================

Claude 4.x prefers direct action requests over questions or suggestions:

.. list-table:: Question vs Action Framing
   :header-rows: 1
   :widths: 50 50

   * - Suggestive (Weaker)
     - Action-Oriented (Stronger)
   * - "Consider using type hints"
     - "Add type hints to all function signatures"
   * - "You might want to add tests"
     - "Write tests for all new functionality"
   * - "It would be good to validate input"
     - "Validate all input at API boundaries"


-----------------------
Fragment Structure
-----------------------


Standard Fragment Template
==========================

.. code-block:: markdown

   # [Topic Title]

   [One-paragraph summary of what this fragment covers and why it matters]

   ## [First Major Section]

   [Positive instruction]

   [Rationale explaining why]

   ```[language]
   # Example demonstrating the instruction
   [code]
   ```

   ## [Second Major Section]

   [Continue pattern...]


Section Ordering
================

Order sections by:

1. **Most important first** - Core rules and requirements
2. **General before specific** - Principles before implementation details
3. **Positive before negative** - What to do before what to avoid
4. **Common before rare** - Frequent cases before edge cases


Heading Levels
==============

- ``#`` - Fragment title (one per file)
- ``##`` - Major sections
- ``###`` - Subsections within major sections
- ``####`` - Avoid; indicates fragment should be split


Fragment Length
===============

- **Target**: 50-150 lines
- **Maximum**: 300 lines
- **Minimum**: 20 lines (if shorter, consider merging with related fragment)

Fragments exceeding 300 lines should be split into focused sub-fragments.


---------------------
Formatting Standards
---------------------


Emphasis
========

Use consistent emphasis throughout:

- **Bold** for key terms, requirements, and important concepts
- *Italics* for introducing new terms or light emphasis
- ``code`` for inline code, commands, file names, and technical identifiers

Avoid:

- ALL CAPS for emphasis (use bold instead)
- Underlining (not supported in markdown)
- Emoji (inconsistent rendering, unprofessional)


Lists
=====

Use bullets for unordered items:

.. code-block:: markdown

   Benefits of this approach:

   - Provides type safety
   - Enables better IDE support
   - Catches errors at compile time

Use numbers for sequential steps or ranked items:

.. code-block:: markdown

   Migration process:

   1. Create backup of existing data
   2. Run schema migration
   3. Verify data integrity
   4. Update application code


Tables
======

Use tables for structured comparisons:

.. code-block:: markdown

   | Type | Use Case | Example |
   |------|----------|---------|
   | `bug` | Something broken | Login fails silently |
   | `feature` | New functionality | Add OAuth support |
   | `task` | Implementation work | Refactor auth module |


Code Blocks
===========

Always specify language for syntax highlighting:

.. code-block:: markdown

   ```python
   def example():
       pass
   ```

   ```bash
   echo "shell commands"
   ```

   ```toml
   [section]
   key = "value"
   ```

Use plain code blocks for pseudo-code or mixed content:

.. code-block:: markdown

   ```
   project/
     src/
       myapp/
     tests/
   ```


-------------------
Language Standards
-------------------


Voice and Tone
==============

**Use imperative mood** - Direct commands, not suggestions:

.. code-block:: text

   GOOD: "Use absolute paths for all commands."
   BAD:  "You should use absolute paths."
   BAD:  "It's recommended to use absolute paths."
   BAD:  "Consider using absolute paths."

**Be direct** - State requirements clearly:

.. code-block:: text

   GOOD: "All functions require type hints."
   BAD:  "Type hints are generally a good practice."

**Be specific** - Avoid vague qualifiers:

.. code-block:: text

   GOOD: "Split files exceeding 300 lines."
   BAD:  "Split files that are too long."


Prohibited Language
===================

Avoid these patterns that create weak instructions:

.. list-table::
   :header-rows: 1
   :widths: 30 70

   * - Pattern
     - Problem
   * - "Try to..."
     - Implies optional compliance
   * - "Consider..."
     - Suggests rather than instructs
   * - "You might..."
     - Weakens the requirement
   * - "It's good practice to..."
     - Vague, no clear requirement
   * - "When possible..."
     - Creates ambiguous exceptions
   * - "Should" without qualifier
     - Unclear if required or suggested


Quantifiers
===========

Be specific about requirements:

.. code-block:: text

   VAGUE:  "Keep functions small."
   SPECIFIC: "Functions exceeding 50 lines should be reviewed for splitting."

   VAGUE:  "Write tests."
   SPECIFIC: "Every public function requires at least one test."

   VAGUE:  "Document important code."
   SPECIFIC: "All public modules and classes require docstrings."


Modal Verbs
===========

Use modal verbs consistently:

- **MUST** / **REQUIRED** - Absolute requirement, no exceptions
- **SHOULD** / **RECOMMENDED** - Strong preference, exceptions need justification
- **MAY** / **OPTIONAL** - Acceptable but not required

.. code-block:: markdown

   All functions **MUST** have type hints.
   Functions **SHOULD** be under 50 lines.
   Functions **MAY** include inline comments for complex logic.


---------------------------------
Cross-References
---------------------------------

When referencing other fragments, use the prompter profile syntax:

.. code-block:: markdown

   See `prompter engineering.testing-philosophy` for testing guidelines.
   See `prompter python.type-hints` for type annotation requirements.

This allows readers to load the referenced content.


------------------------
Quality Checklist
------------------------

Before committing a fragment, verify:


Content Quality
===============

.. code-block:: text

   [ ] Every rule has rationale (WHY)
   [ ] Instructions use positive framing (DO, not DON'T)
   [ ] Examples are syntactically correct
   [ ] Bad examples are paired with good examples
   [ ] No vague qualifiers (try, consider, might)
   [ ] Specific quantities where applicable


Claude 4.x Optimization
=======================

.. code-block:: text

   [ ] Includes overengineering prevention guidance (if applicable)
   [ ] Uses action-oriented language
   [ ] Examples reinforce desired behavior precisely
   [ ] Complex sections use clear structure


Formatting
==========

.. code-block:: text

   [ ] Bold for emphasis (not CAPS)
   [ ] Code blocks have language specified
   [ ] Tables for structured comparisons
   [ ] Consistent heading hierarchy
   [ ] No emoji


Technical
=========

.. code-block:: text

   [ ] Unix line endings (LF only)
   [ ] No trailing whitespace
   [ ] Fragment under 300 lines
   [ ] Cross-references use prompter syntax


------------------------
Refactoring Existing Fragments
------------------------

When updating fragments to meet this guide:


Step 1: Identify Negative Framing
=================================

Search for these patterns and convert to positive:

- "NEVER"
- "DON'T"
- "MUST NOT"
- "AVOID"
- "DO NOT"


Step 2: Add Missing Rationale
=============================

For each rule without WHY:

1. Ask: What bad thing happens if this rule is ignored?
2. Ask: What good thing happens if this rule is followed?
3. Add one sentence capturing the answer


Step 3: Verify Examples
=======================

For each code example:

1. Confirm it compiles/runs
2. Confirm it demonstrates exactly the desired pattern
3. Add good example if only bad example exists


Step 4: Apply Formatting Standards
==================================

1. Convert CAPS emphasis to **bold**
2. Remove emoji
3. Ensure code blocks have language tags
4. Check heading hierarchy


-----------------------
Examples of Well-Written Fragments
-----------------------


Example: Error Handling (Good)
==============================

.. code-block:: markdown

   # Error Handling

   Surface all errors immediately using the fail-fast pattern. Silent failures
   hide bugs and create inconsistent state that is difficult to diagnose.

   ## Error Categories

   Errors fall into two categories with different handling requirements:

   **Domain errors** are expected failure modes within the program's model:
   user not found, insufficient balance, invalid input, network timeout.
   Handle these gracefully with proper error types.

   **Invariant violations** indicate bugs in the program: null values where
   code guarantees non-null, invalid state after operations that should
   guarantee validity. These crash immediately with diagnostics. Recovery
   from an invariant violation is not possible because program state is
   corrupted.

   ## Assertions for Invariants

   Enforce invariants with assertions that include diagnostic information:

   ```python
   assert user is not None, f"Invariant violated: user must exist after auth"
   assert balance >= 0, f"Invariant violated: balance cannot be negative, got {balance}"
   ```

   Assertions with clear messages make debugging faster when invariants fail.


Example: Database Testing (Good)
================================

.. code-block:: markdown

   # Database Testing

   Test against real PostgreSQL using testcontainers. Real database instances
   catch constraint violations, transaction isolation issues, and SQL dialect
   differences that in-memory fakes hide.

   ## Testcontainers Setup

   ```python
   import pytest
   from testcontainers.postgres import PostgresContainer

   @pytest.fixture(scope="session")
   def postgres():
       with PostgresContainer("postgres:17") as pg:
           yield pg.get_connection_url()

   @pytest.fixture
   def db_session(postgres):
       engine = create_engine(postgres)
       with Session(engine) as session:
           yield session
           session.rollback()
   ```

   Session-scoped container startup amortizes the ~2 second startup cost
   across all tests. Per-test rollback ensures test isolation.

   ## Why Not SQLite?

   SQLite differs from PostgreSQL in ways that hide production bugs:

   - Different SQL syntax for common operations
   - Different constraint enforcement behavior
   - Different transaction isolation semantics
   - Different type coercion rules

   Tests passing against SQLite provide false confidence about production
   behavior.


-----------------------
Appendix: Transformation Reference
-----------------------

Quick reference for common negative-to-positive transformations:

.. list-table::
   :header-rows: 1
   :widths: 50 50

   * - Negative Form
     - Positive Form
   * - "Never use X"
     - "Use Y instead of X"
   * - "Don't do X"
     - "Do Y"
   * - "Avoid X"
     - "Prefer Y"
   * - "X is forbidden"
     - "Use Y for this purpose"
   * - "Must not X"
     - "Must Y"
   * - "X is not allowed"
     - "Y is required"
   * - "Don't forget to X"
     - "X is required"
   * - "Never commit without X"
     - "Include X in every commit"
   * - "Don't skip X"
     - "Complete X before proceeding"
   * - "X is prohibited"
     - "Y is the required approach"
