# Python Standards

All projects will use the latest released version of Python (3.13.7 as of 2025-09-10). We will **not** support older versions, unless forced to by external dependencies.

## Basics

Python modules *must* explicitly specify what symbols they export with a `__all__ = []` block at the bottom of the file.

All imports *must* be at the top of the file, and must be sorted.


