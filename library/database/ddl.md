# DDL Standards

We target **PostgreSQL 17** (latest stable as of December 2025) for every schema. Assume access to modern Postgres features (generated columns, JSONB, partial indexes, JSON_TABLE) unless the operator states otherwise.

## Naming Conventions

- **MUST use lowercase with underscores** for all identifiers (tables, columns, functions)
- **MUST pluralize table names** – a single user record lives in `users`; join tables end with `_joins`

## Keys and Constraints

- **MUST default to UUID surrogate keys** unless the operator mandates another strategy
- **WHEN enforcing natural keys**, add `ON UPDATE CASCADE` on every foreign key that references them
- **MUST use constraints and custom functions** to enforce business rules in the database rather than in application code

## Data Types

- **Use specific types over TEXT**: ENUM for fixed value sets, UUID for identifiers, TIMESTAMPTZ for times
- **MUST use TIMESTAMPTZ** with UTC for all timestamp columns, never naive timestamps
- **Use JSONB** for semi-structured data (with GIN indexes for query performance)
- Columns with fixed and small value space (under a dozen values) **SHOULD** use ENUM types

## Indexing

- **MUST create supporting indexes** for every foreign key and for predicates that appear in WHERE or JOIN clauses
- Match index type to use case:
  - **B-Tree** (default): equality and range queries
  - **GIN**: full-text search, JSONB queries
  - **BRIN**: large, sequentially stored datasets (time-series)
- **Do not over-index** – excessive indexes slow INSERT/UPDATE/DELETE operations

## Normalization and NULLs

- **MUST avoid NULLs in business-critical columns**; request explicit approval before allowing nullable fields
- **MUST normalize schemas to at least 3NF** unless the operator approves a denormalized design

## Deviations

**ASK the operator before deviating** from these rules, especially when:
- Denormalizing tables to meet performance targets
- Allowing NULL values in columns that drive business logic
- Replacing reliable natural keys with synthetic keys
