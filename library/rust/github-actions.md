# Rust GitHub Actions CI/CD Patterns

## Three-Workflow Architecture

All Rust projects **MUST** implement three separate workflows:

1. **ci.yml** - Continuous integration (format, lint, test, build)
2. **release.yml** - Release automation with version validation
3. **deps.yml** - Dependency management and security scanning

## Workflow 1: CI Pipeline (ci.yml)

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

## Workflow 2: Release Automation (release.yml)

### Trigger Configuration
```yaml
on:
  push:
    tags:
      - 'projectname-v*'

permissions:
  contents: write
```

### Create Release Job
```yaml
create-release:
  runs-on: ubuntu-latest
  outputs:
    version: ${{ steps.version.outputs.version }}
  steps:
    - uses: actions/checkout@v4

    - name: Get version
      id: version
      run: |
        TAG_NAME=${GITHUB_REF#refs/tags/}
        VERSION=${TAG_NAME#projectname-}
        echo "version=$TAG_NAME" >> $GITHUB_OUTPUT
        echo "version_number=$VERSION" >> $GITHUB_OUTPUT

    - name: Validate version consistency
      shell: bash
      run: |
        TAG_VERSION=$(echo "${{ steps.version.outputs.version }}" | sed 's/projectname-v//')
        CARGO_VERSION=$(grep '^version = ' Cargo.toml | head -1 | sed 's/version = "\(.*\)"/\1/')
        echo "🏷️  Tag version: $TAG_VERSION"
        echo "📦 Cargo version: $CARGO_VERSION"
        if [ "$TAG_VERSION" != "$CARGO_VERSION" ]; then
          echo "❌ ERROR: Tag version ($TAG_VERSION) doesn't match Cargo.toml version ($CARGO_VERSION)"
          exit 1
        fi
        echo "✅ Version consistency validated"

    - name: Create Release
      run: |
        gh release create ${{ steps.version.outputs.version }} \
          --title "Project Name ${{ steps.version.outputs.version_number }}" \
          --generate-notes \
          --draft \
          --repo org/projectname
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### Build and Upload Job
```yaml
build-and-upload:
  needs: create-release
  strategy:
    matrix:
      include:
        - target: x86_64-unknown-linux-gnu
          os: ubuntu-latest
        - target: x86_64-apple-darwin
          os: macos-latest
        - target: aarch64-apple-darwin
          os: macos-latest
        - target: x86_64-pc-windows-msvc
          os: windows-latest
        - target: aarch64-unknown-linux-gnu
          os: ubuntu-latest
  runs-on: ${{ matrix.os }}
  steps:
    - uses: actions/checkout@v4

    - name: Verify version consistency
      shell: bash
      run: |
        EXPECTED_VERSION=$(echo "${{ needs.create-release.outputs.version }}" | sed 's/projectname-v//')
        CARGO_VERSION=$(grep '^version = ' Cargo.toml | head -1 | sed 's/version = "\(.*\)"/\1/')
        echo "🏷️  Tag version: $EXPECTED_VERSION"
        echo "📦 Cargo.toml version: $CARGO_VERSION"
        if [ "$EXPECTED_VERSION" != "$CARGO_VERSION" ]; then
          echo "❌ ERROR: Version mismatch"
          exit 1
        fi
        echo "✅ Version consistency verified for ${{ matrix.target }}"

    - uses: taiki-e/upload-rust-binary-action@v1
      with:
        bin: projectname
        target: ${{ matrix.target }}
        include: LICENSE,README.md,CLAUDE.md
        tar: unix
        zip: windows
        checksum: sha256
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### Publish Release Job
```yaml
publish-release:
  needs: [create-release, build-and-upload]
  runs-on: ubuntu-latest
  steps:
    - name: Publish Release
      run: |
        gh release edit ${{ needs.create-release.outputs.version }} \
          --draft=false \
          --repo org/projectname
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## Workflow 3: Dependency Management (deps.yml)

### Trigger Configuration
```yaml
on:
  schedule:
    - cron: '0 8 * * 1'  # Weekly Monday 8am
  workflow_dispatch:
  push:
    paths:
      - 'Cargo.toml'
      - 'Cargo.lock'

env:
  CARGO_TERM_COLOR: always
```

### Security Audit Job
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
      run: cargo audit --deny warnings

    - name: Create issue on security vulnerability
      if: failure()
      uses: actions/github-script@v7
      with:
        script: |
          const title = '🚨 Security vulnerability detected in dependencies';
          const body = `Security vulnerability detected.\n\nRun: \`cargo audit\``;
          const issues = await github.rest.issues.listForRepo({
            owner: context.repo.owner,
            repo: context.repo.repo,
            state: 'open',
            labels: ['security', 'dependencies']
          });
          const existing = issues.data.find(i => i.title.includes('Security vulnerability'));
          if (!existing) {
            await github.rest.issues.create({
              owner: context.repo.owner,
              repo: context.repo.repo,
              title: title,
              body: body,
              labels: ['security', 'dependencies', 'bug']
            });
          }
```

### Other Dependency Jobs
- **outdated**: Check for outdated dependencies with cargo-outdated
- **dependency-review**: PR-only dependency review
- **minimal-versions**: Test with minimal dependency versions
- **msrv-check**: Verify MSRV compatibility
- **licenses**: Check license compliance with cargo-deny
- **unused-deps**: Detect unused dependencies with cargo-udeps

## Key Patterns and Best Practices

### Caching Strategy
```yaml
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
```

### Cross-Platform Considerations
- Always use `shell: bash` for version validation scripts
- Handle Windows `.exe` extension conditionally
- Configure cross-compilation tools for ARM64 builds
- Test on macos-14 for Apple Silicon

### Version Validation
- Validate at release creation time
- Re-validate before each build
- Extract version from tag and compare with Cargo.toml
- Fail fast on version mismatches

### Binary Distribution
- Use taiki-e/upload-rust-binary-action for consistent packaging
- Include LICENSE, README, and CLAUDE.md
- Generate SHA256 checksums
- Create draft releases, publish after all builds complete

### Security and Compliance
- Run security audits on every dependency change
- Schedule weekly dependency scans
- Auto-create issues on security vulnerabilities
- Verify license compliance with cargo-deny
