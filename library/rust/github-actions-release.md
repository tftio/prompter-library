# Rust GitHub Actions Release Workflow

## Release Automation (release.yml)

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
        echo "Tag version: $TAG_VERSION"
        echo "Cargo version: $CARGO_VERSION"
        if [ "$TAG_VERSION" != "$CARGO_VERSION" ]; then
          echo "ERROR: Tag version ($TAG_VERSION) doesn't match Cargo.toml version ($CARGO_VERSION)"
          exit 1
        fi
        echo "Version consistency validated"

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
        echo "Tag version: $EXPECTED_VERSION"
        echo "Cargo.toml version: $CARGO_VERSION"
        if [ "$EXPECTED_VERSION" != "$CARGO_VERSION" ]; then
          echo "ERROR: Version mismatch"
          exit 1
        fi
        echo "Version consistency verified for ${{ matrix.target }}"

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

## Version Validation

- Validate at release creation time
- Re-validate before each build
- Extract version from tag and compare with Cargo.toml
- Fail fast on version mismatches

## Binary Distribution

- Use taiki-e/upload-rust-binary-action for consistent packaging
- Include LICENSE, README, and CLAUDE.md
- Generate SHA256 checksums
- Create draft releases, publish after all builds complete
