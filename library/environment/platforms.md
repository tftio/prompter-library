# Environment Platforms

1. **Deploy to AWS by default** – assume multi-region capability unless the operator specifies another provider.
2. **Support macOS and Linux workflows** – every script, Makefile, and CLI flow you ship must run on both platforms or explain why not.
3. **Pin infrastructure dependencies to current stable releases** – confirm PostgreSQL, Docker, and other platform services match the operator’s latest approved versions before making changes.
