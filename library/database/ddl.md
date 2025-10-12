# DDL Standards

We target **PostgreSQL (latest stable release)** for every schema. Assume access to modern Postgres features (generated columns, JSONB, partial indexes) unless the operator states otherwise.

When creating DDL, follow these directives:

- **MUST pluralize table names** – a single user record lives in `users`; join tables end with `_joins`.
- **MUST default to UUID surrogate keys** unless the operator mandates another strategy.
- **WHEN enforcing natural keys**, add `ON UPDATE CASCADE` on every foreign key that references them.
- **MUST use constraints and custom functions** to enforce business rules in the database rather than in application code.
- **MUST create supporting indexes** for every foreign key and for predicates that appear in WHERE or JOIN clauses.
- **MUST avoid NULLs in business-critical columns**; request explicit approval before allowing nullable fields.
- **MUST normalize schemas to at least 3NF** unless the operator approves a denormalized design.
- **ASK the operator before deviating** from these rules, especially when:
  - Denormalizing tables to meet performance targets
  - Allowing NULL values in columns that drive business logic
  - Replacing reliable natural keys with synthetic keys
