# VERSIONING

We support [SEMVAR][https://semver.org/]. If in your judgement the scope of a commit is either minor or major, you are licensed to use `versioneer` to update the minor or major version. Be explicit! WArn the operator!

Use the `versioneer` tool for both version changes *AND* git tagging for releases. Versioneer understands pyproject.toml and Cargo.toml files, and the `tag` subcommand adds a tag.

`versioneer` takes a single arguemnt: `patch`, `minor`, or `major`; these bump the respective version correctly. 
