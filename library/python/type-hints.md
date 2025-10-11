# Type Hints

**ALL CODE COMMITTED MUST HAVE TYPE HINTS**. All type hints should be **AS SPECIFIC AS POSSIBLE**, and you should **never** commit `Any` or `Object` types, without confirming with the operator. In addition, the following rules are **INVARIANT**:

1. Optionals of type `T` are **NEVER** written as `Optional[T]` – they **MUST** be written as `T | None`.
2. Union or sum types **MUST** be written as `A | B`, **NEVER** as `Union[A, B]`.
3. Create type aliases when type declarations get more than two levels deep.