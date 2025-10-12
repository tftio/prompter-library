# SQL Standards

1. Always use the explicit join syntax (`A LEFT OUTER JOIN B ON ..`) as opposed to the shorthand;
2. **Use views or materialized views when**:
   - A query involves 3 or more table joins
   - The same complex query appears in multiple locations
   - Query performance is critical and materialization keeps latency predictable
   - Business logic must be centralized at the database layer
