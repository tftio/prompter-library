# API Testing

## Database Testing Requirements

### Use Real PostgreSQL

**Use testcontainers for real PostgreSQL in tests.** SQLite behaves differently:

```python
import pytest
from testcontainers.postgres import PostgresContainer

@pytest.fixture(scope="session")
def postgres():
    with PostgresContainer("postgres:17") as pg:
        yield pg.get_connection_url()
```

See `prompter engineering.testing-philosophy` for full testing standards.

### Avoid In-Memory Fakes

**Use real database connections via testcontainers.** In-memory fakes hide:
- Constraint violations
- Transaction isolation issues
- SQL dialect differences
- Query performance problems

## API Regression Testing

When creating a new database-backed API, create a table for regression testing:

```sql
CREATE TABLE api_test_results (
    test_result_id UUID PRIMARY KEY,
    api_version TEXT NOT NULL,
    api_path TEXT NOT NULL,
    params JSONB NOT NULL DEFAULT '{}'::JSONB,
    result_code INT NOT NULL DEFAULT 200,
    rv JSONB,
    UNIQUE (api_version, api_path, params)
);
```

This holds results for comparison across API versions.

## External Service Mocking

For external services (Stripe, Twilio, etc.), prefer in this order:

1. **Service's test/sandbox mode** - Real behavior, safe environment
2. **Stub at client level** - Replace client methods with test data
3. **Stub at HTTP level** - Intercept requests with respx/responses

```python
# Example: Stubbing at client level
def test_payment_with_stub(monkeypatch):
    def mock_charge(*args, **kwargs):
        return ChargeResult(id="ch_test", status="succeeded")

    monkeypatch.setattr(payment_client, "create_charge", mock_charge)
    result = process_payment(amount=1000)
    assert result.success
```
