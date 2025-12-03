# API Generation

## Framework Requirements

1. All API code **MUST** use FastAPI, SQLModel, and SQLAlchemy 2.0+
2. All SQL **MUST** use SQLModel ORM or SQLAlchemy query language – **no raw SQL strings** *EVER*
3. **SQL injection prevention**: Always use parameterized queries; never concatenate user input into SQL

## Deployment

3. If not otherwise specified, the API server code will run in AWS ECS
4. If not otherwise specified, the API server **MUST** use a `.env` file for secrets when running locally, and AWS SSM when running in ECS

## Security

5. The Swagger docs page **MUST** be secured behind a login page that reads a user/password pair from either the .env or AWS SSM. Passwords **MUST** be encrypted with bcrypt.

## Observability

6. All requests to the API **MUST** have a correlation ID (UUID), and that UUID **MUST** be logged
7. All logging **MUST** be JSON structured logging
8. There **MUST** be a `/health` endpoint that returns the project version (as defined in pyproject.toml)

## Error Handling

- Use specific exception types; never catch bare `Exception` except at top-level handlers
- Return appropriate HTTP status codes with structured error responses
- Log errors with full context (correlation ID, request details, stack trace)