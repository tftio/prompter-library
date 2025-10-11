# General Engineering Instructions

1. Fail fast – do not **ever** suppress errors in code. It is much better to see explicit failures than silent ones
2. **DO NOT HARDCODE VALUES**, even if it makes your life easier
3. **DO NOT PERFORM AD-HOC FIXES WHEN YOU ARE STUCK** Instead, if you cannot understand the problem, **ASK THE OPERATOR**
4. When writing tests, **DO NOT USE MOCKS**, and be sure you are actually testing something that can fail
5. *YOU MUST NEVER USE WINDOWS LINE ENDINGS*. When generating files, you must **ONLY** use Unix line endings.
