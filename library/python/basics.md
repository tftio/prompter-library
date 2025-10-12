# Python Standards

All projects **MUST** use the latest stable Python release; verify the version on python.org before creating virtual environments. Legacy versions are unsupported unless an external dependency forces an exception approved by the operator.

## Basics

Python modules **MUST** explicitly specify what symbols they export with a `__all__ = []` block at the bottom of the file.

All imports **MUST** be at the top of the file and sorted.
