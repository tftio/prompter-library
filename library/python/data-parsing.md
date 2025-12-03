# Data Parsing

Implements the parse-don't-validate principle (see `prompter engineering.parse-dont-validate`) for Python.

## Pydantic v2

**Pydantic is the default tool for parsing external data.**

```python
from pydantic import BaseModel, ValidationError
from datetime import datetime

class UserResponse(BaseModel):
    id: str
    email: str
    created_at: datetime

def fetch_user(user_id: str) -> UserResponse:
    """Fetch user from external API."""
    response = httpx.get(f"/users/{user_id}")
    response.raise_for_status()

    # Parse at the boundary - invalid data fails here
    return UserResponse.model_validate(response.json())
```

## Type Restrictions

### `Any` Type

`Any` is **only valid at system edges**, and must be immediately converted:

```python
# BAD - Any leaks into the system
def process_webhook(data: Any) -> None:
    do_something(data["field"])  # Runtime error waiting to happen

# GOOD - Parse immediately
def process_webhook(raw_data: Any) -> None:
    event = WebhookEvent.model_validate(raw_data)  # Typed from here on
    handle_event(event)
```

### Raw Collections

Convert these to typed containers at the boundary:
- `dict` → Pydantic models or TypedDict
- `list` with unknown contents → `list[SpecificType]`
- `tuple` without type parameters → typed tuples or dataclasses

## Error Handling

Parsing failures are domain errors - expected and recoverable:

```python
def parse_request(raw: dict) -> CreateUserRequest | ParseError:
    """Parse raw request into typed model."""
    try:
        return CreateUserRequest.model_validate(raw)
    except ValidationError as e:
        return ParseError(details=e.errors())
```

## Alternative Tooling

If Pydantic is too heavy:

| Library | Trade-off |
|---------|-----------|
| `msgspec` | Faster, stricter, less flexible |
| `attrs` + `cattrs` | Lightweight, explicit converters |
| `dataclasses` | Built-in, requires manual validation |

All must enforce: **invalid data raises, never silently passes**.

## FastAPI Integration

FastAPI uses Pydantic automatically for request parsing:

```python
@app.post("/users")
async def create_user(request: CreateUserRequest) -> UserResponse:
    # request is already parsed and validated
    user = await user_service.create(request)
    return UserResponse.model_validate(user)
```

For manual parsing (webhooks, background jobs), use `model_validate()` explicitly.
