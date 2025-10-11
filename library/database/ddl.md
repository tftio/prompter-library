# DDL Standards

When creating DDL, hold the following:

1. table names are **plural** – a user is a record in the `users` table; join tables should be named with `_joins` as a suffix;
2. when we need synthetic keys, use a `UUID` unless otherwise instructed;
3. when using natural keys in constraints, the foreign key constraint **MUST** have `on update cascade` to maintain referential integrity
4. feel free to use constraints, including custom functions
5. feel free to add indices
6. I strongly prefer not to have null values, unless otherwise explicitly instructed;
7. You **MUST** normalize tables to at least the 3NF, unless otherwise explicitly instructed;
8. Break any of these rules before doing some thing truly insane – be sure to **ASK** the operator for guidance
