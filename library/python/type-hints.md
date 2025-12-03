# Type Hints

**ALL CODE COMMITTED MUST HAVE TYPE HINTS**. All type hints should be **AS SPECIFIC AS POSSIBLE**, and you should **never** commit `Any` or `object` types without confirming with the operator. In addition, the following rules are **INVARIANT**:

## Syntax Rules

1. Optionals of type `T` are **NEVER** written as `Optional[T]` – they **MUST** be written as `T | None`.
2. Union or sum types **MUST** be written as `A | B`, **NEVER** as `Union[A, B]`.
3. Use native generic syntax: `list[str]`, `dict[str, int]`, **NEVER** `List[str]` or `Dict[str, int]`.
4. Create type aliases when type declarations get more than two levels deep.

## Modern Features (Python 3.11+)

- Use `Self` for methods returning their own class type (from `typing` or native in 3.11+)
- Use `TypeIs` (3.13+) for type narrowing in type guards instead of `TypeGuard`
- Use `@override` decorator (3.12+) when overriding parent class methods
- Prefer new type parameter syntax (3.12+): `class Container[T]: ...` over `TypeVar`

## Deprecations to Avoid

- `AnyStr` – deprecated in 3.13, use constrained type parameters: `class A[T: (str, bytes)]: ...`
- `typing.List`, `typing.Dict`, etc. – use native `list`, `dict` directly
- `Union[A, B]` – use `A | B`
- `Optional[T]` – use `T | None`

## Type Aliases

**`NewType` for semantic distinction:**

```python
from typing import NewType

UserId = NewType('UserId', int)
AccountId = NewType('AccountId', int)

def get_user(user_id: UserId) -> User: ...  # Type checker catches UserId/AccountId mixups
```

**`TypeAlias` for readability of complex types:**

```python
from typing import TypeAlias

CoordinateSet: TypeAlias = set[tuple[int, float | None]]
HandlerMap: TypeAlias = dict[str, Callable[[Request], Response]]
```

Centralize type aliases in `models/types.py`.