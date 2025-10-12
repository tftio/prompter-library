# Shell Error Handling

1. **Enable fail-fast mode** at the top of every script: `set -euo pipefail` stays in place unless the operator explicitly disables it.
2. **Never mask failures** — do not use constructs like `|| true` or `2>/dev/null || true`; they hide real problems.
3. **Handle expected failures explicitly**: 
   ```bash
   if ! command; then
       handle_failure
   fi
   ```
   Branching keeps intent clear and preserves exit statuses.
