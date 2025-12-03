# Python Standards

All projects **MUST** use the latest stable Python release (3.14 as of December 2025); verify the version on python.org before creating virtual environments. Legacy versions are unsupported unless an external dependency forces an exception approved by the operator.

## Key Python 3.14 Features

- **Deferred evaluation of annotations** (PEP 649/749) – type hints are stored but not evaluated until accessed, improving startup performance
- **Template strings (t-strings)** (PEP 750) – `t""` prefix returns a `Template` object for custom string processing
- **Improved free-threaded mode** (PEP 703) – reduced GIL overhead, 5-10% single-threaded penalty

## Basics

Python modules **MUST** explicitly specify what symbols they export with a `__all__ = []` block at the bottom of the file.

All imports **MUST** be at the top of the file and sorted (standard library → third-party → local).

## Async vs Sync

**Prefer synchronous code by default.**

- Async adds complexity (colored functions, viral async/await)
- Sync code is easier to reason about, test, and debug
- Most applications don't need async for acceptable performance

**Use async only when:**
- Proven I/O-bound performance bottleneck exists
- High concurrency requirements (thousands of concurrent connections)
- Interfacing with async-only libraries

When using async, document the rationale clearly. Add async only when a specific bottleneck requires it.
