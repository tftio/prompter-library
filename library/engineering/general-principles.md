# General Engineering Instructions

1. **NEVER suppress errors** – fail fast so problems surface immediately instead of silently succeeding.
2. **NEVER HARDCODE VALUES** even if it simplifies a change; inject or configure everything.
3. **ASK THE OPERATOR when you are stuck** – do not apply ad-hoc fixes if you do not understand the issue.
4. **DO NOT USE MOCKS when writing tests**; ensure every test can fail for real by exercising the actual code paths.

For default platforms and toolchain requirements, follow the `environment` fragments.
