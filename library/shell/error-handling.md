# Shell Error Handling

1. **Enable fail-fast mode** at the top of every script:
   ```bash
   set -o errexit   # Exit on error (same as set -e)
   set -o nounset   # Exit on undefined variable (same as set -u)
   set -o pipefail  # Pipeline fails if any command fails
   ```
   These settings stay in place unless the operator explicitly disables them.

2. **Never mask failures** — do not use constructs like `|| true` or `2>/dev/null || true`; they hide real problems.

3. **Handle expected failures explicitly**:
   ```bash
   if ! command; then
       handle_failure
   fi
   ```
   Branching keeps intent clear and preserves exit statuses.

4. **Always use explicit exit codes** — do not rely on the last command's return value:
   ```bash
   exit 0  # Success
   exit 1  # General error
   ```

5. **Direct errors to stderr**:
   ```bash
   echo 'Error: something failed' >&2
   ```
