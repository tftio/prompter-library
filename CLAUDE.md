# CLAUDE.md - Prompter Library Instructions

## Project Overview

This directory contains a curated library of reusable prompt fragments for LLM coding agents, managed by the `prompter` CLI tool (Rust implementation at `/Users/jfb/Projects/rust/prompter`).

**Purpose**: Composable, modular instruction sets for AI coding agents that enforce coding standards, development practices, and technical constraints.

**Architecture**:
- **Library**: `~/.local/prompter/library/` - Markdown fragments organized by topic
- **Config**: `~/.config/prompter/config.toml` - TOML profiles defining fragment compositions
- **CLI Tool**: Rust binary that renders profiles by concatenating fragments in dependency order

## Critical Rules

### 1. Fragment Format Standards

**MANDATORY**:
- Unix line endings only (LF, never CRLF)
- Markdown format (.md extension)
- Clear, imperative instructions
- No emotional language or praise
- Technical precision over verbosity

**Structure**:
```markdown
# Title (Brief, Descriptive)

[Optional context paragraph]

## Section Heading

1. **Rule or principle** - Explanation
2. **Next rule** - Explanation

[Optional examples in code blocks]
```

### 2. Fragment Organization

**Directory Structure**:
```
library/
├── general/          # Core interaction rules, overall principles
├── engineering/      # General software engineering practices
├── python/          # Python-specific standards
├── rust/            # Rust-specific standards
├── database/        # Database design and SQL standards
├── shell/           # Shell scripting standards
├── documentation/   # Documentation requirements
└── just/            # Justfile standards
```

**Naming Convention**:
- Use lowercase with hyphens: `critical-interaction-rules.md`
- Be specific: `api-testing.md` not `testing.md`
- Group related content: `database/ddl.md`, `database/sql.md`

### 3. Writing Fragment Content

**DO**:
- Use imperative mood ("Use", "Avoid", "Never")
- Provide concrete examples
- State constraints explicitly
- Include rationale when non-obvious
- Use bold/italics for emphasis: **MUST**, *should*, **NEVER**

**DON'T**:
- Include user-specific details (paths, names, etc.)
- Write prose explanations (fragments are instructions, not documentation)
- Contradict other fragments
- Use ambiguous language ("try to", "consider", "maybe")

**Example - Good**:
```markdown
## Error Handling

**Fail fast** - Never suppress errors in code. Explicit failures reveal problems immediately.

```python
# Good
result = risky_operation()

# Bad - suppresses failure
try:
    result = risky_operation()
except Exception:
    result = None  # Silent failure
```

**Example - Bad**:
```markdown
## Error Handling

I think it's generally a good practice to handle errors properly.
You might want to consider letting errors bubble up sometimes,
as this could potentially help with debugging.
```

### 4. Fragment Dependencies

When creating fragments, consider:
- **Granularity**: One fragment per logical concern
- **Reusability**: Fragments compose into multiple profiles
- **Independence**: Each fragment stands alone (minimal cross-references)
- **Hierarchy**: General → Specific (general/overall.md before python/api-testing.md)

### 5. Profile Management

**Config Location**: `~/.config/prompter/config.toml`

**Profile Definition**:
```toml
[profile.name]
depends_on = [
    "general/critical-interaction-rules.md",
    "engineering/general-principles.md",
    "python/basics.md"
]
```

**Rules**:
- Profiles can reference other profiles
- File paths are relative to library root
- First occurrence wins (deduplication)
- Depth-first resolution respects `depends_on` order

## Working with the Rust CLI

### Development Workflow

**Location**: `/Users/jfb/Projects/rust/prompter`

**Key Files**:
- `src/main.rs` - CLI implementation
- `Cargo.toml` - Rust dependencies and metadata
- `VERSION` - Version file (managed by versioneer)
- `docs/CONFIG.md` - Configuration documentation

**Commands**:
```bash
# Validate library references
prompter validate

# List available profiles
prompter list

# Render a profile
prompter python.api

# Test during development
cargo test
cargo test --test cli  # Integration tests
```

### Version Management

**CRITICAL**: Use versioneer exclusively for version changes

```bash
versioneer patch    # 1.0.0 → 1.0.1
versioneer minor    # 1.0.1 → 1.1.0
versioneer major    # 1.1.0 → 2.0.0
versioneer sync     # Synchronize VERSION and Cargo.toml
versioneer verify   # Check version consistency
```

**NEVER**:
- Manually edit Cargo.toml version
- Create git tags manually (use `versioneer tag`)
- Skip version bumps when releasing

## Maintenance Tasks

### Adding a New Fragment

1. **Determine placement**:
   ```bash
   # List current structure
   find ~/.local/prompter/library -name "*.md" | sort
   ```

2. **Create fragment**:
   ```bash
   # Example: new TypeScript linting standards
   vim ~/.local/prompter/library/typescript/linting.md
   ```

3. **Follow format**:
   - Clear title
   - Imperative instructions
   - Concrete examples
   - Unix line endings

4. **Update profiles** (if needed):
   ```bash
   vim ~/.config/prompter/config.toml
   # Add to relevant profiles
   ```

5. **Validate**:
   ```bash
   prompter validate
   prompter typescript.standards  # Test render
   ```

### Updating Existing Fragments

**Process**:
1. Read current fragment completely
2. Identify specific changes needed
3. Maintain consistent voice and structure
4. Test affected profiles
5. Verify no conflicts with other fragments

**Example**:
```bash
# Before editing
prompter validate
prompter python.api > /tmp/before.txt

# Make changes
vim ~/.local/prompter/library/python/linting.md

# After editing
prompter validate
prompter python.api > /tmp/after.txt

# Review changes
diff /tmp/before.txt /tmp/after.txt
```

### Creating New Profiles

1. **Identify use case**: What coding scenario does this support?
2. **Select fragments**: Which existing fragments apply?
3. **Define profile**:
   ```toml
   [rust.cli]
   depends_on = [
       "general/critical-interaction-rules.md",
       "engineering/general-principles.md",
       "rust/cli.md"
   ]
   ```
4. **Test composition**:
   ```bash
   prompter rust.cli | wc -l  # Check output size
   prompter rust.cli | grep "CRITICAL"  # Verify key content
   ```

### Removing Fragments

**Before removal**:
1. Search all config files for references:
   ```bash
   grep -r "fragment-name.md" ~/.config/prompter/
   ```
2. Check all profiles that might depend on it:
   ```bash
   prompter list
   # Test each relevant profile
   ```
3. Remove from all profile definitions
4. Validate:
   ```bash
   prompter validate
   ```
5. Only then delete the file

## Quality Standards

### Fragment Review Checklist

- [ ] Unix line endings (LF only)
- [ ] Markdown formatting valid
- [ ] Clear, imperative language
- [ ] No emotional or praising language
- [ ] No user-specific details
- [ ] Concrete examples where helpful
- [ ] No contradictions with other fragments
- [ ] Tested in at least one profile
- [ ] Validated with `prompter validate`

### Profile Testing

**Render test**:
```bash
prompter profile.name > /tmp/test.txt
echo $?  # Should be 0
wc -l /tmp/test.txt  # Check reasonable size
```

**Content verification**:
```bash
# Check for key instructions
prompter profile.name | grep -i "critical"
prompter profile.name | grep -i "never"

# Verify structure
prompter profile.name | head -20  # Check pre-prompt
prompter profile.name | tail -20  # Check post-prompt
```

## Integration with Development

### Using Prompter in Workflows

**With Claude Code**:
```bash
# Generate context for a Python API project
prompter python.api | pbcopy
# Paste into Claude Code session
```

**With Other LLMs**:
```bash
# Generate and pipe to LLM CLI
prompter rust.cli | llm "Implement a CLI arg parser"
```

**In Scripts**:
```bash
#!/bin/bash
CONTEXT=$(prompter python.api)
echo "$CONTEXT" | your-ai-tool --stdin
```

### Project-Specific Configs

For project-specific instruction sets:
```bash
# Project directory structure
my-project/
├── .prompter/
│   ├── config.toml
│   └── library/
│       └── project-specific.md

# Use project config
prompter --config my-project/.prompter/config.toml project.profile
```

## Anti-Patterns

### Fragment Anti-Patterns

**BAD - Emotional/praising**:
```markdown
Great job! You're doing amazing work. Keep it up!
```

**BAD - Vague**:
```markdown
Try to write good tests when you can.
```

**BAD - User-specific**:
```markdown
Use /home/john/projects/myapp as the working directory.
```

**BAD - Conversational**:
```markdown
So, like, when you're writing code, you probably want to think about...
```

**GOOD - Direct, technical**:
```markdown
**Write tests with real assertions** - Test code must verify specific conditions that can fail.
```

### Profile Anti-Patterns

**BAD - Circular dependencies**:
```toml
[profile.a]
depends_on = ["profile.b"]

[profile.b]
depends_on = ["profile.a"]
```

**BAD - Overly granular**:
```toml
[python.api.testing.unit.mocking]
depends_on = [...]
```

**GOOD - Logical grouping**:
```toml
[python.api]
depends_on = [
    "python/basics.md",
    "python/api-generation.md",
    "python/api-testing.md"
]
```

## Troubleshooting

### Common Issues

**"Missing file" error**:
```bash
# Check file exists
ls -la ~/.local/prompter/library/path/to/file.md

# Check path in config (must be relative to library root)
grep -n "file.md" ~/.config/prompter/config.toml
```

**"Cycle detected" error**:
```bash
# Review profile dependencies
prompter list
# Check for circular references in config.toml
```

**"Unknown profile" error**:
```bash
# List available profiles
prompter list
# Check profile name spelling in config.toml
```

**Validation fails**:
```bash
prompter validate
# Follow error messages
# Common causes: missing files, typos in profile names, circular deps
```

## Development Standards

### Rust CLI Development

When working on the Rust CLI tool:

1. **Read comprehensively** - Don't assume, read the actual code
2. **Test thoroughly** - Unit tests AND integration tests
3. **Verify changes**:
   ```bash
   cargo test
   cargo clippy
   cargo build --release
   ./target/release/prompter validate
   ```
4. **Version management** - Use versioneer for all version changes
5. **Documentation** - Update docs/CONFIG.md if behavior changes

### Pre-Release Checklist

Before pushing changes:

- [ ] All tests pass: `cargo test`
- [ ] No clippy warnings: `cargo clippy`
- [ ] Integration tests pass: `cargo test --test cli`
- [ ] Version bumped with versioneer
- [ ] VERSION and Cargo.toml synchronized
- [ ] Changes documented in relevant docs
- [ ] Library validated: `prompter validate`
- [ ] Example profiles render correctly

## Reference

### Key Fragments

- `general/critical-interaction-rules.md` - Core behavioral constraints
- `general/overall.md` - Overall project approach
- `engineering/general-principles.md` - Universal engineering rules (see environment for platforms/tooling)
- `environment/platforms.md` - Default infrastructure and OS assumptions
- `environment/tooling.md` - Required dev tooling and package management defaults
- `tooling/cli.md` - Language-agnostic CLI standards
- `tooling/version-management.md` - Versioneer usage and invariants
- `documentation/plan-structure.md` - Planning document format and audience
- `documentation/todos.md` - TODO management
- `workflow/plan-execution.md` - Phase workflow for executing plans
- `workflow/plan-tracking.md` - Rules for GitHub/Asana synchronization
- `shell/error-handling.md` - Fail-fast expectations for scripts
- `shell/portability.md` - Cross-platform requirements and platform detection
- `shell/style.md` - Style, tooling, and arithmetic conventions for Bash

### Key Commands

```bash
# Library management
prompter init                    # Initialize config and library
prompter list                    # Show all profiles
prompter validate                # Validate config and references

# Profile rendering
prompter profile.name            # Render profile
prompter -s "\n---\n" profile    # With separator
prompter -p "Custom" profile     # Custom pre-prompt

# Development
cargo test                       # Run tests
cargo clippy                     # Lint
versioneer patch                 # Bump version
```

### Documentation

- `/Users/jfb/Projects/rust/prompter/README.md` - CLI tool overview
- `/Users/jfb/Projects/rust/prompter/docs/CONFIG.md` - Configuration guide
- `~/.config/prompter/config.toml` - Profile definitions
- `~/.local/prompter/library/` - Fragment library

## Summary

This library exists to provide consistent, composable instruction sets for AI coding agents. Fragments are technical, imperative, and focused. Profiles compose fragments for specific development contexts. The Rust CLI tool validates and renders these compositions.

**Core principles**:
1. Technical precision
2. Composability through dependencies
3. Explicit over implicit
4. Validated consistency
5. Unix philosophy (do one thing well)
