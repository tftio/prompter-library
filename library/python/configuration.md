# Python Configuration Standards

## Configuration Format

**TOML is the only acceptable configuration format.**

- JSON is **forbidden** for configuration (no comments, verbose syntax)
- YAML is **forbidden** for configuration (whitespace-sensitive, security concerns with arbitrary code execution)

## Configuration vs Secrets

| Type | Storage | Example |
|------|---------|---------|
| Configuration | TOML files | Feature flags, timeouts, URLs, thresholds |
| Secrets | Environment / SSM | API keys, passwords, tokens |

**Environment variables are for secrets only, not configuration.**

## File Structure

```
project/
  config/
    default.toml      # Base configuration
    development.toml  # Development overrides
    production.toml   # Production overrides
```

## Loading Pattern

```python
import tomllib
from pathlib import Path
from pydantic import BaseModel

class AppConfig(BaseModel):
    database_url: str
    cache_ttl: int
    feature_flags: dict[str, bool]

def load_config(env: str = "development") -> AppConfig:
    """Load configuration from TOML files."""
    config_dir = Path("config")

    # Load base config
    with open(config_dir / "default.toml", "rb") as f:
        config = tomllib.load(f)

    # Overlay environment-specific config
    env_file = config_dir / f"{env}.toml"
    if env_file.exists():
        with open(env_file, "rb") as f:
            env_config = tomllib.load(f)
        config = _deep_merge(config, env_config)

    return AppConfig(**config)
```

## Secrets Management

See `prompter security.secrets` for secrets handling:
- Local: `.env` files (gitignored)
- Production: AWS SSM Parameter Store
