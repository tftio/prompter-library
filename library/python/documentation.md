# Python Documentation

Without further instructions you **MUST NOT** mention dunder methods or attributes in the public documentation.

Non-public methods, attributes, and functions **MUST** have docstrings, but they **MAY NOT** be rolled up to the class or module level.

Be explicit in writing the documentation; assume that the primary reader of the documentation will be an experienced software engineer with Python and web experience.

1. all functions, methods, classes, and modules **MUST** have comprehensive documentation, in reST/Sphinx format.

   Adhere to reST conventions – we are going to use Sphinx to generate documentation.

2. **ALWAYS** adhere to the [pydocstyle](https://www.pydocstyle.org/en/stable/) standards

3. python modules

   The module **MUST** have a docstring, and the docstring **MUST** enumerate the contents of the module, including all public symbols – (those exposed in the `__all__` variable)

4. python functions

   All functions **MUST** have a docstring, and the docstring **MUST** enumerate the arguments and return values, as well as types, and also any exceptions explicitly mentioned in the code;

5. python classes

   the class **MUST** have a docstring, and the docstring **MUST** enumerate the methods and attributes of the class.