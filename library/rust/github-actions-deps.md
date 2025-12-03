# Rust GitHub Actions Dependency Management

## Dependency Workflow (deps.yml)

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
          const title = 'Security vulnerability detected in dependencies';
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

### Security and Compliance
- Run security audits on every dependency change
- Schedule weekly dependency scans
- Auto-create issues on security vulnerabilities
- Verify license compliance with cargo-deny
