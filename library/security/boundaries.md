# Security Boundaries

## Parse, Don't Validate

**All external data MUST be parsed into typed containers at the system boundary.**

The principle: Convert unstructured data into structured types immediately upon entry. Then enjoy type safety throughout the interior of your system.

## The Boundary Rule

```
External World          |  Your System
                        |
  JSON response    ---> |  parse() ---> TypedModel
  User input       ---> |  parse() ---> TypedModel
  File contents    ---> |  parse() ---> TypedModel
  Config files     ---> |  parse() ---> TypedModel
                        |
```

**Never pass raw data through the system:**
- No raw `dict` from JSON responses
- No raw `str` from user input
- No untyped data structures

## Implementation Pattern

```python
from pydantic import BaseModel, ValidationError

class UserResponse(BaseModel):
    id: str
    email: str
    created_at: datetime

def fetch_user(user_id: str) -> UserResponse:
    """Fetch user from external API.

    Returns typed UserResponse or raises on invalid data.
    """
    response = httpx.get(f"/users/{user_id}")
    response.raise_for_status()

    # Parse at the boundary - invalid data fails here, not deep in business logic
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

Never pass these through your system:
- `dict` (use Pydantic models or TypedDict)
- `list` with unknown contents (use `list[SpecificType]`)
- `tuple` without type parameters

## Error Handling at Boundaries

Parsing failures are **domain errors** (see `prompter engineering.general-principles`):

```python
def parse_request(raw: dict) -> CreateUserRequest | ParseError:
    """Parse raw request into typed model.

    Returns ParseError for invalid data - this is expected and recoverable.
    """
    try:
        return CreateUserRequest.model_validate(raw)
    except ValidationError as e:
        return ParseError(details=e.errors())
```

## Tooling

Use Pydantic v2 for parsing. Alternatives (if Pydantic is too heavy):
- `msgspec` - faster, stricter
- `attrs` + `cattrs` - lightweight
- `dataclasses` with manual validation

All must enforce: **invalid data raises, never silently passes**.
