# Rust Testing CI Integration

## Test Coverage with cargo-tarpaulin

### Configuration in CI

```yaml
# .github/workflows/ci.yml
- name: Install cargo-tarpaulin (Linux only)
  if: matrix.coverage
  run: cargo install cargo-tarpaulin

- name: Run tests with coverage
  if: matrix.coverage
  run: |
    cargo tarpaulin --all --out Xml --engine llvm --timeout 300 --fail-under 80

- name: Upload coverage to Codecov
  if: matrix.coverage
  uses: codecov/codecov-action@v4
  with:
    token: ${{ secrets.CODECOV_TOKEN }}
    files: ./tarpaulin-report.xml
    flags: unittests
```

### Local Coverage Check

```bash
# Install tarpaulin (Linux only)
cargo install cargo-tarpaulin

# Generate coverage report
cargo tarpaulin --all --out Html --engine llvm --timeout 300

# View HTML report
open tarpaulin-report.html

# Check coverage threshold
cargo tarpaulin --all --out Xml --engine llvm --timeout 300 --fail-under 80
```

### Coverage Requirements
- **Minimum 80% coverage** enforced in CI
- Linux only (llvm engine requirement)
- Include all crates with `--all`
- Timeout set to 300 seconds for complex tests
- XML output for CI integration, HTML for local viewing

## Cross-Platform Testing

### Test Matrix in CI

```yaml
test:
  name: Test
  strategy:
    matrix:
      os: [ubuntu-latest, macos-latest, macos-14, windows-latest]
      rust: [stable, "1.85.0"]
      include:
        - os: ubuntu-latest
          rust: stable
          coverage: true
  runs-on: ${{ matrix.os }}
```

### Platform-Specific Testing
- **ubuntu-latest**: Linux x64, coverage enabled
- **macos-latest**: macOS Intel
- **macos-14**: macOS Apple Silicon
- **windows-latest**: Windows x64
- Test both stable and MSRV Rust versions

## Running Tests

### Basic Commands
```bash
# Run all tests
cargo test --all

# Run with verbose output
cargo test --all --verbose

# Run specific test
cargo test test_name

# Run integration tests only
cargo test --test cli

# Run with RUSTFLAGS
RUSTFLAGS="-D warnings" cargo test --all
```

### justfile Integration
```just
# Run all tests
test:
    cargo test --all --verbose

# Run tests with coverage (Linux only)
test-coverage:
    cargo tarpaulin --all --out Html --engine llvm --timeout 300 --fail-under 80
```

## Continuous Integration Testing

### Test Job Requirements
1. Run on multiple OS platforms
2. Test both stable and MSRV Rust versions
3. Generate coverage report (Linux only)
4. Upload coverage to Codecov
5. Fail on any test failure or coverage below threshold

### Expected Test Output
- Clear success/failure indicators
- Coverage percentage reported
- Failed test details with context
- Cross-platform compatibility confirmed
