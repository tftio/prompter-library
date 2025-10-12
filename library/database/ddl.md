# DDL Standards

We target **PostgreSQL (latest stable release)** for every schema. Assume access to modern Postgres features (generated columns, JSONB, partial indexes) unless the operator states otherwise.

When creating DDL, hold the following:

1. table names are **plural** – a user is a record in the `users` table; join tables should be named with `_joins` as a suffix;
2. when we need synthetic keys, use a `UUID` unless otherwise instructed;
3. when using natural keys in constraints, the foreign key constraint **MUST** have `on update cascade` to maintain referential integrity
4. feel free to use constraints, including custom functions
5. feel free to add indices
6. I strongly prefer not to have null values, unless otherwise explicitly instructed;
7. You **MUST** normalize tables to at least the 3NF, unless otherwise explicitly instructed;
8. **If any rule conflicts with requirements, ASK the operator before proceeding**. Consult them before:
   - Denormalizing tables for performance (relaxes 3NF)
   - Allowing NULL values in columns that drive business logic
   - Replacing natural keys with synthetic keys when natural keys are viable
