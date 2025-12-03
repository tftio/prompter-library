# SQL Standards

## Query Structure

- **MUST use explicit JOIN clauses** (`FROM a LEFT OUTER JOIN b ON ...`) so predicates stay visible and never drift into implicit comma joins.
- **Avoid correlated subqueries** that run once per row; replace with JOINs or CTEs when appropriate.
- Use **range comparisons** for date filtering, not functions like `EXTRACT()` which prevent index usage.

## Security

**ALL SQL in Python code MUST use SQLAlchemy query language or SQLModel. No raw SQL strings. EVER.**

```python
# FORBIDDEN - Raw SQL strings
cursor.execute(f"SELECT * FROM users WHERE id = {user_id}")
conn.execute(text("SELECT * FROM users WHERE id = :id"), {"id": user_id})

# REQUIRED - SQLAlchemy ORM
user = session.query(User).filter(User.id == user_id).first()

# REQUIRED - SQLAlchemy Core
stmt = select(users_table).where(users_table.c.id == user_id)

# REQUIRED - SQLModel
user = session.exec(select(User).where(User.id == user_id)).first()
```

This applies to all database operations including:
- Queries (SELECT)
- Mutations (INSERT, UPDATE, DELETE)
- Schema operations (use Alembic, see `prompter python.migrations`)

See `prompter security.owasp` for additional SQL injection prevention guidelines.

## Views and Reusability

- **WHEN a query spans three or more tables**, refactor it into a view or materialized view to keep the logic reusable and reviewable.
- **WHEN the same complex query appears in multiple code paths**, centralize it in a shared view to prevent divergence.
- **WHEN latency targets demand precomputation**, ship a materialized view and document the refresh cadence alongside the consuming jobs.
- **WHEN business rules belong at the database layer**, encode them in views or functions so every client observes the same constraints.

## PostgreSQL 17 Features

- **JSON_TABLE()** – convert JSON data to table format for easier querying
- **JSON_EXISTS, JSON_QUERY, JSON_VALUE** – structured JSON access
- **MERGE with RETURNING** – conditional updates with result sets
- **Improved CTE optimization** – PostgreSQL 17 better optimizes CTEs in complex queries

## Performance

- Enable `pg_stat_statements` to identify slow queries by total execution time.
- Ensure **autovacuum** is enabled and properly tuned; without it, table bloat degrades performance.
- Use **connection pooling** (PgBouncer) for high-concurrency applications.
- Kill IDLE IN TRANSACTION connections over 10 minutes.
