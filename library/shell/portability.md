# Shell Portability

1. **Ensure every script runs on both macOS and Linux**:
   - Test critical scripts locally or through CI runners for each platform before committing.
   - Prefer POSIX-compatible syntax for tools such as `sed`, `awk`, and `find`.
   - Avoid GNU-only flags unless the operator explicitly approves them.
2. **Detect the platform before running platform-specific commands**:
   ```bash
   case "$(uname -s)" in
       Darwin*)
           # macOS-specific logic
           ;;
       Linux*)
           # Linux-specific logic
           ;;
       *)
           echo "Unsupported platform: $(uname -s)" >&2
           exit 1
           ;;
   esac
   ```
3. **Do not hardcode paths**; resolve executables with `which` or `env` so scripts respect the operator’s environment.
