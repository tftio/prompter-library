# Python Documentation

## Documentation Philosophy

Since we mandate full type hints, docstrings should focus on **what types cannot express**:

- **WHY** – Rationale, context, design decisions
- **WHEN** – Preconditions, valid states, timing requirements
- **WHO** – Intended callers, audience, use cases

Do **NOT** repeat type information (types are in hints). Do **NOT** explain HOW (code shows implementation).

## Format

Use reST/Sphinx format. Adhere to [pydocstyle](https://www.pydocstyle.org/en/stable/) standards.

## Visibility Rules

- **MUST NOT** mention dunder methods in public documentation
- Non-public methods need docstrings but **MUST NOT** appear in public API docs
- Exclude non-public symbols from `__all__`

## Docstring Requirements

### Modules

The module **MUST** have a docstring describing:
- Purpose and responsibility
- Public symbols (those in `__all__`)

### Functions

Focus on behavior, not types:

```python
def process_payment(customer: Customer, amount: Money) -> PaymentResult:
    """Process a payment for a customer.

    Charges the customer's default payment method. If the charge fails,
    sends a notification email and logs the failure for retry.

    Raises:
        PaymentDeclined: If the payment provider rejects the charge.
        CustomerNotFound: If customer has no payment method configured.
    """
```

Note: No `:param:` or `:returns:` for types – they're in the signature.

### Classes

Describe purpose and usage patterns:

```python
class OrderProcessor:
    """Processes orders through the fulfillment pipeline.

    Use this class when orders need validation, inventory checks,
    and shipping label generation. For simple refunds, use
    RefundProcessor instead.

    Thread-safe for concurrent order processing.
    """
```
