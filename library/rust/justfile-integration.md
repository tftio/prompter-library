# Rust justfile Integration

## Integration Points

### With peter-hook
```just
pre-commit:
    peter-hook run pre-commit

pre-push:
    peter-hook run pre-push
```

### With versioneer
```just
version-show:
    @echo "Current version: $(cat VERSION)"

bump-version level:
    versioneer {{ level }}

release level:
    # Uses versioneer for version bumps and tagging
```

### With GitHub Actions
- `just ci` mirrors GitHub Actions CI workflow
- `just pre-commit` mirrors format and lint jobs
- `just pre-push` mirrors comprehensive validation
- `just test` mirrors test job
- `just audit` mirrors security audit job

## Best Practices

### Error Handling
- Use `set -euo pipefail` in shebang recipes
- Check tool availability before use
- Provide clear error messages with installation instructions
- Exit with non-zero status on errors

### Output Clarity
- Use emojis for status (checkmarks, x marks, warning signs)
- Echo step descriptions before actions
- Suppress noise with `@` prefix
- Show progress for long-running operations

### Consistency
- Keep recipe names consistent across projects
- Match CI job names where applicable
- Use same parameters and flags as CI
- Maintain alphabetical or logical grouping

### Documentation
- Add comments for recipe groups
- Include usage examples in comments
- Document required tools in header
- Reference related documentation fragments

## Project-Specific Customization

### Binary Name
Replace `projectname` with actual binary name:
```just
run *args:
    cargo run -- {{ args }}
```

### Tag Format
Update tag format in release recipe:
```just
versioneer tag --tag-format "yourproject-v{version}"
```

### Additional Recipes
Add project-specific recipes as needed:
```just
# Install project locally
install:
    cargo install --path .

# Generate documentation
docs:
    cargo doc --no-deps --open
```

## Maintenance

### Keep Synchronized With
- GitHub Actions workflows (ci.yml)
- Git hooks configuration (hooks.toml)
- Project documentation (CLAUDE.md)
- Quality tool configurations

### Update When
- Adding new quality tools
- Changing CI pipeline
- Updating version management
- Adding new project commands
