# Secrets Management

## Core Principle

**Store secrets outside version control.** This is non-negotiable.

## Environment Separation

| Environment | Storage | Access Pattern |
|-------------|---------|----------------|
| Local development | `.env` files | `python-dotenv` or similar |
| CI/CD | GitHub Secrets / CI variables | Environment injection |
| Production | AWS SSM Parameter Store | Runtime fetch with IAM |

## Local Development

### .env Files

Create `.env` in project root (must be in `.gitignore`):

```bash
# .env - Keep out of version control
DATABASE_URL=postgresql://localhost/myapp_dev
API_SECRET_KEY=dev-only-secret-key
STRIPE_SECRET_KEY=sk_test_xxx
```

Load with:

```python
from dotenv import load_dotenv
import os

load_dotenv()  # Load .env file

database_url = os.environ["DATABASE_URL"]
```

### .env.example

Commit a template without real values:

```bash
# .env.example - Safe to commit
DATABASE_URL=postgresql://localhost/myapp_dev
API_SECRET_KEY=replace-with-real-key
STRIPE_SECRET_KEY=sk_test_replace
```

## Production (AWS SSM)

### Parameter Store Pattern

Store secrets in SSM Parameter Store with SecureString type:

```python
import boto3

def get_secret(name: str) -> str:
    """Fetch secret from AWS SSM Parameter Store."""
    ssm = boto3.client("ssm")
    response = ssm.get_parameter(Name=name, WithDecryption=True)
    return response["Parameter"]["Value"]

# Usage
database_url = get_secret("/myapp/prod/database_url")
```

### Naming Convention

```text
/{app}/{environment}/{secret_name}

/myapp/prod/database_url
/myapp/prod/stripe_secret_key
/myapp/staging/database_url
```

### IAM Permissions

Service roles should have minimal SSM access:

```json
{
  "Effect": "Allow",
  "Action": ["ssm:GetParameter"],
  "Resource": "arn:aws:ssm:*:*:parameter/myapp/prod/*"
}
```

## Secret Rotation

### Database Credentials

- Rotate at least quarterly
- Use IAM database authentication where possible
- Maintain two active credentials during rotation

### API Keys

- Rotate immediately if exposed
- Use short-lived tokens where possible
- Implement key versioning for graceful rotation

## Audit Logging

- Log all secret access (who, when, which secret)
- Redact secret values from all log output
- Alert on unusual access patterns

```python
import structlog

logger = structlog.get_logger()

def get_secret(name: str, requester: str) -> str:
    logger.info("secret_accessed", secret_name=name, requester=requester)
    # ... fetch secret
```

## Secret Handling Rules

- **Store secrets in .env (local) or SSM (production)** – Git history is permanent and searchable.
- **Redact secrets from log output** – Logs are often aggregated and retained.
- **Use generic error messages for auth failures** – Specific messages leak implementation details.
- **Document secret locations in README, not code** – Comments are searchable in source.
- **Generate unique secrets per environment** – Shared secrets spread blast radius.
- **Share secrets via 1Password or SSM** – Unencrypted channels are logged and archived.
