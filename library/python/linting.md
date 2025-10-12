# Python Linting

- **MUST** run the approved lint tools and resolve every reported error before committing; if you are blocked, ask the operator for guidance instead of shipping a warning.
- **MUST NOT** add blanket ignore directives or skip lint errors without explicit operator approval.
- **MUST** treat every lint issue as blocking unless the operator designates a specific exception.

We use the following tools for linting:

## RUFF
We require the use of [ruff](https://docs.astral.sh/ruff/) to format and lint Python code. Use the `.ruff.toml` file at the root of the repository.

## TY
We use [ty](https://docs.astral.sh/ty/) to typecheck.

## INTERROGATE
We use [interrogate](https://interrogate.readthedocs.io/en/latest/) to ensure docstring coverage.


