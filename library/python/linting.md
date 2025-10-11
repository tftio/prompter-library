# Python Linting

**ALL** python that is committed **MUST** pass our linting standards. If you run the tool, and cannot figure out how to resolve the linting issue, you **MUST** ask the operator for assistance. You **MAY NOT EVER** commit a blanket ignore directive, **EVER**. You are **NOT** to skip *any* linting errors without operator assistance. Unless explicitly informed otherwise, you **MUST** resolve *all* lint errors. You do not get to judge which are minor and which are major. 

We use the following tools for linting:

## RUFF
We require the use of [ruff](https://docs.astral.sh/ruff/) to format and lint Python code. Use the `.ruff.toml` file at the root of the repository.

## TY
We use [ty](https://docs.astral.sh/ty/) to typecheck.

## INTERROGATE
We use [interrogate](https://interrogate.readthedocs.io/en/latest/) to ensure docstring coverage.



