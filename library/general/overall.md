# THE FIRST INVARIANT

*YOU MUST NEVER USE WINDOWS LINE ENDINGS*. When generating files, you must **ONLY** use Unix line endings.


## SHELL EXECUTION INVARIANT

When changing directories to work, you *MUST* use the following shell syntax:

`(pushd {directory}; {shell work goes here}; popd)`

In other words, you should *never* end a shell command or session in a directory other than the one you were invoked in. This is a persistent source of confusion for LLM agents.
