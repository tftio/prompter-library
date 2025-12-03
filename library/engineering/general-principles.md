# General Engineering Instructions

## Core Rules

1. **NEVER suppress errors** – fail fast so problems surface immediately instead of silently succeeding.
2. **NEVER HARDCODE VALUES** even if it simplifies a change; inject or configure everything.
3. **ASK THE OPERATOR when you are stuck** – do not apply ad-hoc fixes if you do not understand the issue.
4. **DO NOT USE MOCKS when writing tests** except at system boundaries; see `prompter engineering.testing-philosophy`.

For default platforms and toolchain requirements, follow the `environment` fragments.

## Error Taxonomy

There are two kinds of errors:

### Domain Errors (Recoverable)

Expected failure modes within the program's model:
- User not found
- Insufficient balance
- Invalid input
- Network timeout

These are part of normal operation. Handle them gracefully with proper error types.

### Invariant Violations (Non-Recoverable)

Impossible states that indicate bugs in the program:
- Null value where code guarantees non-null
- Invalid state after a sequence that should guarantee validity
- Broken assumptions about data relationships

These **MUST crash immediately** with diagnostics. Never attempt recovery from an invariant violation – the program state is corrupted.

### Assertions for Invariants

Use assertions with clear messages to enforce invariants:

```python
assert user is not None, "Invariant violated: user must exist after successful auth"
assert balance >= 0, f"Invariant violated: balance cannot be negative, got {balance}"
```

## Dependency Philosophy

**Dependencies solve generic problems; your code solves your specific problem.**

### Wrap Dependencies

Don't expose dependency APIs directly. Wrap them in domain-specific interfaces:

```python
# BAD - Exposing stripe directly
def charge_customer(stripe_customer_id: str, amount: int):
    return stripe.Charge.create(customer=stripe_customer_id, amount=amount)

# GOOD - Domain wrapper
def charge_customer(customer: Customer, amount: Money) -> ChargeResult:
    """Charge customer using payment provider."""
    response = stripe.Charge.create(
        customer=customer.payment_provider_id,
        amount=amount.cents,
    )
    return ChargeResult.from_stripe_response(response)
```

### Benefits of Wrapping

- Provides a seam for testing (swap implementations)
- Isolates dependency API changes
- Types match your domain, not the library's
- Single place to add logging, metrics, error handling
