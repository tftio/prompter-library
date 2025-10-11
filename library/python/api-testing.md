# API Testing

When creating a new database backed API, you must create a table, called `api_test_results` with the following structure:

```sql
CREATE TABLE api_test_results (
test_result_id UUID PRIMARY KEY,
api_version TEXT NOT NULL,
api_path TEXT NOT NULL,
params JSONB NOT NULL DEFAULT '{}':: JSONB,
result_code INT NOT NULL DEFAULT 200,
rv JSONB,
UNIQUE api_Version, api_path, params
);
```

This table will hold the results of running the API, for testing purposes. We will have a script that generates the data (exercises each API endpoint with specified parameters), and then a script that compares any two api versions.