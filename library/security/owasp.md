# OWASP Security Standards

## SQL Injection Prevention

**Always use SQLAlchemy query language. Never use raw SQL strings.**

```python
# BAD - Raw SQL string (NEVER DO THIS)
query = f"SELECT * FROM users WHERE id = '{user_id}'"

# BAD - Even text() with parameters is discouraged
query = text("SELECT * FROM users WHERE id = :user_id")

# GOOD - SQLAlchemy ORM
user = session.query(User).filter(User.id == user_id).first()

# GOOD - SQLAlchemy Core
stmt = select(users_table).where(users_table.c.id == user_id)
result = conn.execute(stmt)

# GOOD - SQLModel
user = session.exec(select(User).where(User.id == user_id)).first()
```

SQLAlchemy and SQLModel handle parameterization automatically. See `prompter database.sql` for full SQL standards.

## Cross-Site Scripting (XSS) Prevention

### Output Encoding

All user-generated content **MUST** be escaped before rendering in HTML:

```python
from markupsafe import escape

# Always escape user content
safe_content = escape(user_input)
```

### Content Security Policy

API responses should include CSP headers:

```python
@app.middleware("http")
async def add_security_headers(request, call_next):
    response = await call_next(request)
    response.headers["Content-Security-Policy"] = "default-src 'self'"
    response.headers["X-Content-Type-Options"] = "nosniff"
    response.headers["X-Frame-Options"] = "DENY"
    return response
```

## Authentication

### Password Storage

**bcrypt is mandatory** for password hashing:

```python
import bcrypt

def hash_password(password: str) -> str:
    return bcrypt.hashpw(password.encode(), bcrypt.gensalt()).decode()

def verify_password(password: str, hashed: str) -> bool:
    return bcrypt.checkpw(password.encode(), hashed.encode())
```

### Session Management

- Use secure, httponly cookies for session tokens
- Implement session expiration
- Rotate session IDs after authentication
- Invalidate sessions on logout

### Token Handling

- JWTs should have short expiration times
- Store refresh tokens securely (not in localStorage)
- Validate token signatures on every request

## Authorization

### Principle of Least Privilege

- Users get minimum permissions needed
- Service accounts scoped to specific operations
- Database users limited to required tables/operations

### Access Control Checks

```python
def get_resource(resource_id: str, user: User) -> Resource:
    resource = db.get(resource_id)
    if resource is None:
        raise NotFoundError()

    # Always check authorization
    if not user.can_access(resource):
        raise ForbiddenError()

    return resource
```

## Code Review Checklist

Before merging, verify:

- [ ] No SQL string concatenation with user input
- [ ] All user content escaped before HTML rendering
- [ ] Passwords hashed with bcrypt
- [ ] Authorization checks on all protected endpoints
- [ ] No secrets in code or logs
- [ ] Input validation at API boundaries
- [ ] Security headers configured
- [ ] Rate limiting on authentication endpoints
