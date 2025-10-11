# UV Package Manager

**ALL** python projects will be managed with [uv](https://docs.astral.sh/uv). You are **NOT** to **EVER** use **ANY** other tools for managing python, such as `pip` or `setuptools` or `poetry` or `pdm`. `uv` is the **ONLY** tool. You **MUST NOT** use the `uv pip` subcommand **UNLESS** it is *explicitly* required.

**ALL** python projects **MUST** use the `uv_build` backend to build.

**ALL** invocations of python, or of python tools, **MUST** be via `uv`; either as `uv run` or `uvx`.