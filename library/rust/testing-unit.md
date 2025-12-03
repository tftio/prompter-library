# Rust Unit Testing

## Testing Requirements

All Rust projects **MUST** implement:

1. **Unit tests** embedded in source files with `#[cfg(test)]`
2. **Integration tests** in dedicated `tests/` directory
3. **80% minimum code coverage** via cargo-tarpaulin
4. **Cross-platform test verification** on Linux, macOS, and Windows

## Unit Tests Pattern

### Embedded in Source Files

```rust
// src/lib.rs

pub fn process_data(input: &str) -> Result<String, Error> {
    // Implementation
}

#[cfg(test)]
mod tests {
    use super::*;
    use tempfile::TempDir;

    #[test]
    fn test_process_data_valid() {
        let result = process_data("valid input");
        assert!(result.is_ok());
        assert_eq!(result.unwrap(), "expected output");
    }

    #[test]
    fn test_process_data_invalid() {
        let result = process_data("invalid");
        assert!(result.is_err());
    }

    #[test]
    fn test_with_temp_directory() {
        let temp_dir = TempDir::new().unwrap();
        let test_file = temp_dir.path().join("test.txt");
        // Test file operations
        std::fs::write(&test_file, "content").unwrap();
        // Temp dir automatically cleaned up on drop
    }
}
```

### Key Patterns
- Use `#[cfg(test)]` to exclude test code from production builds
- Use `use super::*;` to access parent module items
- Use `tempfile::TempDir` for file system tests (auto-cleanup)
- Test both success and error paths
- Test edge cases and boundary conditions

## Test Quality Standards

### Coverage Focus Areas
- **Error paths**: Test all error conditions
- **Edge cases**: Boundary conditions, empty inputs, maximum sizes
- **File operations**: Use temporary directories
- **CLI interactions**: Test all subcommands and flag combinations
- **Configuration parsing**: Valid and invalid configurations

### What NOT to Test
- Third-party library internals
- Standard library behavior
- Trivial getters/setters without logic

### Test Naming Conventions
- Prefix with `test_`
- Use descriptive names: `test_parser_handles_empty_input`
- Group related tests with common prefixes
