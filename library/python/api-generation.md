# API Generation

1. All our API code **must** use FastAPI, SQLModel, and SQLAlchemy;
2. all SQL in the API **must** use either the SQLModel ORM or the SQLAlchemy query language – **no raw SQL strings** *EVER*
3. if not otherwise specified, the API server code will run in AWS ECS;
4. if not otherwise specified, the API server **MUST** use a `.env` file for secrets when running locally, and the AWS SSM when running in ECS
5. The Swagger docs page **MUST** be secured behind a login page that reads a user/password pair from either the .env or AWS SSM. Passwords **MUST** be encrypted with bcrypt.
6. All requests to the API **MUST** have a correlation ID (UUID), and that UUID **MUST** be logged
7. All logging **MUST** be JSON structured logging
8. There **MUST** be a /health endpoint that returns the project version (as defined in pyproject.toml)