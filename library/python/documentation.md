# Python Documentation

Without further instructions you **MUST NOT** mention dunder methods or attributes in the public documentation.

Non-public methods, attributes, and functions **MUST** have docstrings, but they **MUST NOT** be surfaced in the public API documentation (exclude them from `__all__`, public doc tables, and high-level overviews).

1. All functions, methods, classes, and modules **MUST** have comprehensive documentation in reST/Sphinx format.

   Adhere to reST conventions – we are going to use Sphinx to generate documentation.

2. **ALWAYS** adhere to the [pydocstyle](https://www.pydocstyle.org/en/stable/) standards

3. Python modules

   The module **MUST** have a docstring, and the docstring **MUST** enumerate the contents of the module, including all public symbols – (those exposed in the `__all__` variable)

4. Python functions

   All functions **MUST** have a docstring, and the docstring **MUST** enumerate the arguments and return values, as well as types, and also any exceptions explicitly mentioned in the code;

5. Python classes

   The class **MUST** have a docstring, and the docstring **MUST** enumerate the methods and attributes of the class.
