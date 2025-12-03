# Parse, Don't Validate

## Core Principle

**All external data MUST be parsed into typed containers at the system boundary.**

Convert unstructured data into structured types immediately upon entry. Type safety throughout the interior of your system follows automatically.

## The Boundary Rule

```text
External World          |  Your System
                        |
  JSON response    ---> |  parse() ---> TypedModel
  User input       ---> |  parse() ---> TypedModel
  File contents    ---> |  parse() ---> TypedModel
  Config files     ---> |  parse() ---> TypedModel
                        |
```

**Parse all data at the boundary:**
- Convert raw dictionaries from JSON responses into typed models
- Parse raw strings from user input into structured types
- Use typed containers for all data structures

## Why This Matters

1. **Single point of validation** - Parse once at the boundary, trust types internally
2. **Fail-fast** - Invalid data surfaces immediately, not deep in business logic
3. **Type safety** - Compiler/type checker enforces correctness after parsing
4. **Eliminates redundant checks** - No defensive `if field is not None` scattered throughout

## Type Restrictions

### Untyped Values

Language-equivalent of `Any` or `Object` is **only valid at system edges**, and must be immediately converted. Convert untyped values to typed containers before they propagate through the system.

### Raw Collections

Convert these to typed containers at the boundary:
- Untyped dictionaries/maps → dataclasses, structs, models
- Lists/arrays with unknown element types → typed lists with known element types
- Tuples without type parameters → named tuples or dataclasses

Use data classes, structs, models, or language-specific typed collections.

## Error Handling at Boundaries

Parsing failures are **domain errors** (recoverable), not invariant violations:

- Invalid input is expected - users make mistakes, APIs change, files get corrupted
- Return error types or raise recoverable exceptions
- Provide clear error messages about what was wrong
- Reject malformed data explicitly with clear error messages

See `prompter engineering.general-principles` for the error taxonomy.

## Language-Specific Implementation

- **Python**: See `prompter python.data-parsing` for Pydantic patterns
- **Rust**: Use `serde` with strict deserialization
- **TypeScript**: Use `zod`, `io-ts`, or similar runtime validators
- **Go**: Define structs and use `json.Unmarshal` with strict mode
