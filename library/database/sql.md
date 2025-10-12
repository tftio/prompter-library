# SQL Standards

- **MUST use explicit JOIN clauses** (`FROM a LEFT OUTER JOIN b ON ...`) so predicates stay visible and never drift into implicit comma joins.
- **WHEN a query spans three or more tables**, refactor it into a view or materialized view to keep the logic reusable and reviewable.
- **WHEN the same complex query appears in multiple code paths**, centralize it in a shared view to prevent divergence.
- **WHEN latency targets demand precomputation**, ship a materialized view and document the refresh cadence alongside the consuming jobs.
- **WHEN business rules belong at the database layer**, encode them in views or functions so every client observes the same constraints.
