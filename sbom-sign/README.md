# 🔒 SBOM Signing Action

A comprehensive GitHub Action for signing Software Bill of Materials (SBOM) files using CycloneDX CLI with secure key management, flexible installation methods, and automatic verification.

## ✨ Features

- 🔒 **RSA Digital Signatures** - Sign SBOM files with industry-standard RSA encryption
- 🐳 **Flexible Installation** - Install the CycloneDX CLI via Docker (any OS with Docker) or Homebrew (macOS/Linux), or skip installation if it's already available
- 🔐 **Secure Key Handling** - Temporary key files with restrictive permissions
- ✅ **Signature Verification** - Built-in signature validation after signing
- 🧹 **Automatic Cleanup** - Secure cleanup of temporary files
- 📋 **Comprehensive Validation** - Input validation with helpful error messages

## 🚀 Basic Usage

Sign an SBOM with default settings:

```yaml
- name: "🔒 Sign SBOM"
  uses: laerdal/github_actions/sbom-sign@main
  with:
    sbom-file-path: "./sbom.xml"
    signing-key: ${{ secrets.SIGNING_KEY }}
```

```yaml
- name: "🔒 Sign with custom output"
  uses: laerdal/github_actions/sbom-sign@main
  with:
    sbom-file-path: "./artifacts/sbom.xml"
    signing-key: ${{ secrets.PRIVATE_SIGNING_KEY }}
    output-path: "./signed-sbom.xml"
```

```yaml
- name: "🔒 Sign with specific CLI version"
  uses: laerdal/github_actions/sbom-sign@main
  with:
    sbom-file-path: "./sbom.xml"
    signing-key: ${{ secrets.SIGNING_KEY }}
    cyclonedx-cli-version: "0.27.2"
```

## 🔧 Advanced Usage

Full configuration with all available options:

```yaml
- name: "🔒 Advanced SBOM signing"
  uses: laerdal/github_actions/sbom-sign@main
  with:
    sbom-file-path: "./artifacts/sbom.xml"
    signing-key: ${{ secrets.DEPENDENCY_TRACKER_SIGNING_KEY }}
    installation-method: "docker"
    cyclonedx-cli-version: "0.29.1"
    output-path: "./artifacts/signed-sbom.xml"
    signing-key-file-path: "./temp/signing.pem"
    show-summary: "true"
```

## 🔐 Permissions Required

This action requires standard repository permissions:

```yaml
permissions:
  contents: read  # Required to checkout repository code
```

## 🏗️ CI/CD Example

Complete workflow for SBOM generation, signing, and publishing:

```yaml
name: "SBOM Security Pipeline"

on:
  push:
    branches: ["main"]
    tags: ["v*"]
  pull_request:
    branches: ["main"]

permissions:
  contents: read

jobs:
  secure-sbom:
    runs-on: ubuntu-latest

    steps:
      - name: "📥 Checkout repository"
        uses: actions/checkout@v6

      # ...

      - name: "📋 Generate SBOM"
        id: generate-sbom
        uses: laerdal/github_actions/dotnet-cyclonedx@main
        with:
          path: "./src/MyProject.csproj"
          output: "./artifacts"
          output-format: "xml"

      - name: "🔒 Sign SBOM"
        id: sign-sbom
        uses: laerdal/github_actions/sbom-sign@main
        with:
          sbom-file-path: ${{ steps.generate-sbom.outputs.sbom-file }}
          signing-key: ${{ secrets.SBOM_SIGNING_KEY }}
          output-path: "./signed-sbom.xml"
          cyclonedx-cli-version: "0.29.1"
          show-summary: "true"

      - name: "📤 Upload signed SBOM"
        uses: laerdal/github_actions/sbom-publish@main
        with:
          sbom-file-path: ${{ steps.sign-sbom.outputs.signed-sbom-path }}
          project-name: "MyProject"
          project-version: ${{ github.ref_name }}
          dependency-tracker-url: ${{ vars.DEPENDENCY_TRACKER_URL }}
          dependency-tracker-api-key: ${{ secrets.DEPENDENCY_TRACKER_API_KEY }}

      - name: "📁 Archive signed SBOM"
        uses: actions/upload-artifact@v7
        with:
          name: "signed-sbom-${{ github.sha }}"
          path: ${{ steps.sign-sbom.outputs.signed-sbom-path }}
          retention-days: 90
```

## 📋 Inputs

| Input                   | Description                                                | Required | Default             | Example                              |
|-------------------------|------------------------------------------------------------|----------|---------------------|--------------------------------------|
| `sbom-file-path`        | Path to the SBOM file to sign                              | ✅ Yes   | -                   | `./sbom.xml`, `./artifacts/sbom.xml` |
| `signing-key`           | Private signing key content (PEM format)                   | ✅ Yes   | -                   | `${{ secrets.SIGNING_KEY }}`         |
| `installation-method`   | Installation method for the CycloneDX CLI                  | ❌ No    | `docker`            | `docker`, `brew`, `skip`             |
| `cyclonedx-cli-version` | CycloneDX CLI tool version (used with the `docker` method) | ❌ No    | `0.29.1`            | `0.29.1`, `0.27.2`                   |
| `output-path`           | Path for signed SBOM output                                | ❌ No    | Same as input       | `./signed-sbom.xml`                  |
| `signing-key-file-path` | Path for the temporary signing key file                    | ❌ No    | `./signing-key.pem` | `./temp/signing.pem`                 |
| `show-summary`          | Display action summary                                     | ❌ No    | `false`             | `true`, `false`                      |

## 📤 Outputs

| Output                | Description                                      | Type     | Example                |
|-----------------------|--------------------------------------------------|----------|------------------------|
| `signed-sbom-path`    | Path to the signed SBOM file                     | `string` | `./signed-sbom.xml`    |
| `signature-algorithm` | Signature algorithm used                         | `string` | `RS256`                |
| `signing-timestamp`   | ISO timestamp when SBOM was signed               | `string` | `2024-10-04T12:30:45Z` |
| `cli-version`         | CycloneDX CLI version that performed the signing | `string` | `0.29.1`               |
| `file-size`           | Size of signed SBOM file in bytes                | `string` | `15728`                |

## 🔗 Related Actions

| Action                  | Purpose                | Repository                                |
|-------------------------|------------------------|-------------------------------------------|
| 📋 **dotnet-cyclonedx** | Generate SBOM files    | `laerdal/github_actions/dotnet-cyclonedx` |
| 📤 **sbom-publish**     | Publish signed SBOMs   | `laerdal/github_actions/sbom-publish`     |
| 📊 **gh-sbom**          | GitHub SBOM operations | `laerdal/github_actions/gh-sbom`          |
| 🚀 **dotnet**           | .NET build operations  | `laerdal/github_actions/dotnet`           |

## 💡 Examples

### Basic SBOM Signing

```yaml
- name: "Sign SBOM with default settings"
  uses: laerdal/github_actions/sbom-sign@main
  with:
    sbom-file-path: "./sbom.xml"
    signing-key: ${{ secrets.PRIVATE_SIGNING_KEY }}
```

### Custom CLI Version

```yaml
- name: "Sign with specific CLI version"
  uses: laerdal/github_actions/sbom-sign@main
  with:
    sbom-file-path: "./build/sbom.xml"
    signing-key: ${{ secrets.SIGNING_KEY }}
    cyclonedx-cli-version: "0.26.0"
    output-path: "./artifacts/signed-sbom.xml"
```

### Pre-installed CLI

```yaml
- name: "Install CycloneDX CLI"
  run: |
    curl -L -o cyclonedx https://github.com/CycloneDX/cyclonedx-cli/releases/download/v0.29.1/cyclonedx-linux-x64
    chmod +x cyclonedx
    sudo mv cyclonedx /usr/local/bin/

- name: "Sign SBOM with pre-installed CLI"
  uses: laerdal/github_actions/sbom-sign@main
  with:
    sbom-file-path: "./sbom.xml"
    signing-key: ${{ secrets.SIGNING_KEY }}
    installation-method: "skip"
```

### Chain with SBOM Generation

```yaml
- name: "Generate SBOM"
  id: generate
  uses: laerdal/github_actions/dotnet-cyclonedx@main
  with:
    path: "./MyProject.csproj"
    output-format: "xml"

- name: "Sign Generated SBOM"
  uses: laerdal/github_actions/sbom-sign@main
  with:
    sbom-file-path: ${{ steps.generate.outputs.sbom-file }}
    signing-key: ${{ secrets.SIGNING_KEY }}
    show-summary: "true"
```

### Multi-Key Signing Strategy

```yaml
- name: "Sign with primary key"
  id: primary-sign
  uses: laerdal/github_actions/sbom-sign@main
  with:
    sbom-file-path: "./sbom.xml"
    signing-key: ${{ secrets.PRIMARY_SIGNING_KEY }}
    output-path: "./sbom-signed-primary.xml"

- name: "Sign with backup key"
  uses: laerdal/github_actions/sbom-sign@main
  with:
    sbom-file-path: ${{ steps.primary-sign.outputs.signed-sbom-path }}
    signing-key: ${{ secrets.BACKUP_SIGNING_KEY }}
    output-path: "./sbom-dual-signed.xml"
```

## 🔧 Installation Methods

The action supports three ways of getting the CycloneDX CLI onto the runner, selected via `installation-method`:

| Method   | Description                                                                                               | Requirements                                   |
|----------|-----------------------------------------------------------------------------------------------------------|------------------------------------------------|
| `docker` | *(default)* Pulls `cyclonedx/cyclonedx-cli:<cyclonedx-cli-version>` and runs signing inside the container | Docker available on the runner                 |
| `brew`   | Installs the CLI via Homebrew (`cyclonedx/cyclonedx/cyclonedx-cli`)                                       | Homebrew available on the runner (macOS/Linux) |
| `skip`   | Assumes `cyclonedx` is already installed and available on `PATH`                                          | CLI pre-installed by an earlier step           |

> ⚠️ **Docker mount constraint**: the `docker` method mounts the current working directory (`$PWD`)
> into the container as `/workspace`. Always pass `sbom-file-path`/`output-path` as paths **relative
> to the working directory** (e.g. `./sbom.xml`); absolute paths outside the working directory are
> not visible inside the container and will cause signing to fail.

## 🔐 Security Features

The action implements multiple security layers:

- **Restrictive File Permissions**: Temporary key files created with 600 permissions
- **Automatic Cleanup**: Secure deletion of temporary files even on failure
- **Input Validation**: Comprehensive validation of required parameters
- **Signature Verification**: Post-signing validation of signature integrity
- **Secure Key Handling**: The key is written only to a temporary, restricted-permission file that is securely removed immediately after signing, and is masked from workflow logs

## 🔑 Key Format Requirements

The signing key must be in PEM format:

```text
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEA...
-----END RSA PRIVATE KEY-----
```

### Key Generation

Generate RSA private keys for SBOM signing:

```bash
# Generate 2048-bit RSA private key
openssl genrsa -out private-key.pem 2048

# Extract public key
openssl rsa -in private-key.pem -pubout -out public-key.pem

# Convert to PKCS#8 format (if needed)
openssl pkcs8 -topk8 -inform PEM -outform PEM -nocrypt \
  -in private-key.pem -out private-key-pkcs8.pem
```

### Key Storage Best Practices

1. **GitHub Secrets**: Store keys in GitHub repository secrets
2. **Base64 Encoding**: Encode keys for secret storage if needed
3. **Key Rotation**: Regularly rotate signing keys
4. **Access Control**: Limit access to signing keys
5. **Backup Strategy**: Maintain secure backups of keys

## 🐛 Troubleshooting

### Common Issues

#### CycloneDX CLI Not Found

**Problem**: Docker or Homebrew is not available on the runner, so `installation-method: docker`
(the default) or `installation-method: brew` failed to install the CycloneDX CLI

**Solution**: Switch installation method, or pre-install the CLI and use `installation-method: skip`:

```yaml
- name: "Install CycloneDX CLI manually"
  run: |
    curl -L -o cyclonedx https://github.com/CycloneDX/cyclonedx-cli/releases/download/v0.29.1/cyclonedx-linux-x64
    chmod +x cyclonedx
    sudo mv cyclonedx /usr/local/bin/

- name: "Sign SBOM with pre-installed CLI"
  uses: laerdal/github_actions/sbom-sign@main
  with:
    sbom-file-path: "./sbom.xml"
    signing-key: ${{ secrets.SIGNING_KEY }}
    installation-method: "skip"
```

#### Invalid Key Format

**Problem**: Signing key is not in proper PEM format

**Solution**: Validate key format before use:

```yaml
- name: "Validate signing key"
  run: |
    echo "${{ secrets.SIGNING_KEY }}" > temp-key.pem
    if ! openssl rsa -in temp-key.pem -check -noout; then
      echo "❌ Invalid signing key format"
      exit 1
    fi
    rm temp-key.pem
    echo "✅ Key validation passed"
```

#### Permission Denied on Key File

**Problem**: Cannot create or access temporary key file

**Solution**: Check filesystem permissions and working directory:

```yaml
- name: "Debug file permissions"
  run: |
    pwd
    ls -la .
    touch test-file && rm test-file
    echo "✅ Filesystem permissions OK"
```

#### Signature Verification Failed

**Problem**: Generated signature cannot be verified

**Solution**: Check CycloneDX CLI version compatibility and key validity:

```yaml
- name: "Debug signing process"
  run: |
    echo "Input SBOM size: $(stat -c%s sbom.xml)"
    echo "CycloneDX CLI version: $(cyclonedx --version)"
    echo "Platform: ${{ runner.os }}-${{ runner.arch }}"
```

### Debug Tips

1. **Enable Detailed Logging**: Set `show-summary: "true"`
2. **Validate Key Locally**: Test key format with OpenSSL
3. **Check File Sizes**: Ensure SBOM files are not empty
4. **Verify CLI Installation**: Check CycloneDX CLI availability

## 📝 Requirements

- GitHub Actions runner (Windows, Linux, or macOS)
- Valid SBOM file in CycloneDX format
- RSA private key in PEM format
- Docker (for the default `installation-method: docker`) or Homebrew (for `installation-method: brew`), unless using `installation-method: skip` with a pre-installed CLI

## 🔧 Advanced Features

### Conditional Signing

```yaml
- name: "Check if signing required"
  id: check-signing
  run: |
    if [[ "${{ github.ref }}" == "refs/heads/main" ]] || [[ "${{ github.ref }}" == refs/tags/* ]]; then
      echo "requires-signing=true" >> $GITHUB_OUTPUT
    else
      echo "requires-signing=false" >> $GITHUB_OUTPUT
    fi

- name: "Sign SBOM"
  if: steps.check-signing.outputs.requires-signing == 'true'
  uses: laerdal/github_actions/sbom-sign@main
  with:
    sbom-file-path: "./sbom.xml"
    signing-key: ${{ secrets.SIGNING_KEY }}
```

### Signature Verification Workflow

```yaml
- name: "Sign and verify SBOM"
  id: sign
  uses: laerdal/github_actions/sbom-sign@main
  with:
    sbom-file-path: "./sbom.xml"
    signing-key: ${{ secrets.SIGNING_KEY }}

- name: "Verify signature integrity"
  run: |
    # Check for signature elements in XML
    if ! grep -q "signature" "${{ steps.sign.outputs.signed-sbom-path }}"; then
      echo "❌ No signature found in SBOM"
      exit 1
    fi

    echo "✅ SBOM signature verification passed"
```

## 📄 License

This action is part of the Laerdal Medical GitHub Actions collection and follows the same license terms.

---

> 💡 **Tip**: Combine this action with our SBOM generation and publishing actions for complete supply chain security workflows.
