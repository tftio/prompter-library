# Testing Philosophy

## Core Principle

**Test real systems, not simulations.**

The goal of testing is to verify your code works correctly against the actual systems it will run against in production. Simulations hide bugs.

## Database Testing

### PostgreSQL: Testcontainers Always

**Use real PostgreSQL via testcontainers.** SQLite behaves differently:
- Different SQL syntax
- Different constraint handling
- Different transaction semantics
- Different type coercion

Use testcontainers to spin up real PostgreSQL:

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

### Avoid In-Memory Fakes

In-memory database fakes hide:
- Constraint violations
- Transaction isolation issues
- Query performance problems
- Data type mismatches

## Test Doubles Hierarchy

### Fakes (In-Memory Implementations)

**AVOID.** Fakes hide consistency issues between your fake and the real system.

```python
# BAD - Fake hides real database behavior
class FakeUserRepository:
    def __init__(self):
        self.users = {}  # Doesn't enforce constraints!
```

### Stubs (Canned Responses)

**OK for external HTTP APIs.** You don't own the external system, so stubbing responses is acceptable.

```python
# OK - Stubbing external API response
def test_payment_processing(respx_mock):
    respx_mock.post("https://api.stripe.com/v1/charges").respond(
        json={"id": "ch_123", "status": "succeeded"}
    )
    result = process_payment(amount=1000)
    assert result.success
```

### Mocks (Behavior Verification)

**Only at system edges.** Mocks that verify call patterns couple tests to implementation details.

```python
# BAD - Verifying implementation details
mock_db.get_user.assert_called_once_with(user_id=123)

# BETTER - Verify outcomes
assert result.user.id == 123
```

## External Service Testing Priority

When testing code that calls external services, prefer in this order:

1. **Service's test/sandbox mode** - Use Stripe test mode, AWS LocalStack, etc.
2. **Stub at client level** - Replace `stripe_client.create_charge()` with test data
3. **Stub at HTTP level** - Intercept HTTP requests and return canned responses

## What to Test

### Test Outcomes, Not Implementation

```python
# BAD - Testing implementation
def test_user_creation():
    mock_repo.save.assert_called_once()

# GOOD - Testing outcome
def test_user_creation():
    user = create_user(email="test@example.com")
    assert user.id is not None
    assert db.get(user.id).email == "test@example.com"
```

### Test Error Paths

Every error condition should have a test:

```python
def test_user_not_found():
    with pytest.raises(NotFoundError):
        get_user(id="nonexistent")

def test_invalid_email():
    with pytest.raises(ValidationError):
        create_user(email="not-an-email")
```

## Test Organization

```text
tests/
  conftest.py           # Shared fixtures (postgres, etc.)
  test_models/
  test_services/
  test_routes/
  integration/          # Full system integration tests
```
