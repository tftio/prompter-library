# Rust GitHub Actions CI Pipeline

## Three-Workflow Architecture

All Rust projects **MUST** implement three separate workflows:

1. **ci.yml** - Continuous integration (format, lint, test, build)
2. **release.yml** - Release automation with version validation
3. **deps.yml** - Dependency management and security scanning

## CI Pipeline (ci.yml)

### Trigger Configuration
```yaml
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

env:
  CARGO_TERM_COLOR: always
  CARGO_INCREMENTAL: 0
  RUSTFLAGS: "-D warnings"
```

### Job Structure

#### Format Job
```yaml
format:
  name: Format
  runs-on: ubuntu-latest
  steps:
    - name: Checkout
      uses: actions/checkout@v4

    - name: Install Rust toolchain
      uses: dtolnay/rust-toolchain@1.86.0
      with:
        components: rustfmt

    - name: Check formatting
      run: cargo fmt --all -- --check
```

#### Lint Job
```yaml
lint:
  name: Lint
  runs-on: ubuntu-latest
  steps:
    - name: Checkout
      uses: actions/checkout@v4

    - name: Install Rust toolchain
      uses: dtolnay/rust-toolchain@1.86.0
      with:
        components: clippy

    - name: Cache Cargo registry
      uses: actions/cache@v4
      with:
        path: |
          ~/.cargo/registry
          ~/.cargo/git
        key: ${{ runner.os }}-cargo-${{ hashFiles('**/Cargo.lock') }}
        restore-keys: |
          ${{ runner.os }}-cargo-

    - name: Run Clippy
      run: cargo clippy --all-targets -- -D warnings
```

#### Test Job with Coverage
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
  steps:
    - name: Checkout
      uses: actions/checkout@v4

    - name: Install Rust toolchain
      uses: dtolnay/rust-toolchain@master
      with:
        toolchain: ${{ matrix.rust }}

    - name: Cache Cargo registry
      uses: actions/cache@v4
      with:
        path: |
          ~/.cargo/registry
          ~/.cargo/git
          target/
        key: ${{ runner.os }}-${{ matrix.rust }}-cargo-${{ hashFiles('**/Cargo.lock') }}
        restore-keys: |
          ${{ runner.os }}-${{ matrix.rust }}-cargo-
          ${{ runner.os }}-cargo-

    - name: Install cargo-tarpaulin (Linux only)
      if: matrix.coverage
      run: cargo install cargo-tarpaulin

    - name: Run tests
      run: cargo test --all --verbose

    - name: Run tests with coverage (Linux only)
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
        name: codecov-umbrella
```

#### Security Audit Job
```yaml
audit:
  name: Security Audit
  runs-on: ubuntu-latest
  steps:
    - name: Checkout
      uses: actions/checkout@v4

    - name: Install Rust toolchain
      uses: dtolnay/rust-toolchain@1.86.0

    - name: Install cargo-audit
      run: cargo install cargo-audit

    - name: Run security audit
      run: cargo audit
```

#### Build Job
```yaml
build:
  name: Build
  needs: [format, lint, test, audit]
  strategy:
    matrix:
      include:
        - target: x86_64-unknown-linux-gnu
          os: ubuntu-latest
        - target: aarch64-unknown-linux-gnu
          os: ubuntu-latest
        - target: x86_64-apple-darwin
          os: macos-latest
        - target: aarch64-apple-darwin
          os: macos-14
        - target: x86_64-pc-windows-msvc
          os: windows-latest
  runs-on: ${{ matrix.os }}
  steps:
    - name: Checkout
      uses: actions/checkout@v4

    - name: Install Rust toolchain
      uses: dtolnay/rust-toolchain@1.86.0
      with:
        targets: ${{ matrix.target }}

    - name: Install cross-compilation tools (Linux ARM64)
      if: matrix.target == 'aarch64-unknown-linux-gnu'
      run: |
        sudo apt-get update
        sudo apt-get install -y gcc-aarch64-linux-gnu

    - name: Configure cross-compilation (Linux ARM64)
      if: matrix.target == 'aarch64-unknown-linux-gnu'
      run: |
        echo "[target.aarch64-unknown-linux-gnu]" >> ~/.cargo/config.toml
        echo "linker = \"aarch64-linux-gnu-gcc\"" >> ~/.cargo/config.toml

    - name: Build binary
      run: cargo build --release --target ${{ matrix.target }}

    - name: Upload binary artifact
      uses: actions/upload-artifact@v4
      with:
        name: projectname-${{ matrix.target }}
        path: |
          target/${{ matrix.target }}/release/projectname${{ matrix.os == 'windows-latest' && '.exe' || '' }}
```

#### Integration Test Job
```yaml
integration-test:
  name: Integration Tests
  needs: [build]
  runs-on: ubuntu-latest
  steps:
    - name: Checkout
      uses: actions/checkout@v4

    - name: Download Linux binary
      uses: actions/download-artifact@v4
      with:
        name: projectname-x86_64-unknown-linux-gnu
        path: ./bin

    - name: Make binary executable
      run: chmod +x ./bin/projectname

    - name: Basic smoke test
      run: ./bin/projectname version

    - name: End-to-end test
      run: |
        ./bin/projectname init
        ./bin/projectname list
        ./bin/projectname validate
```
