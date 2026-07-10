# Actions Documentation

Welcome to the comprehensive documentation for **Actions** - a collection of enterprise-grade GitHub Actions for modern CI/CD workflows.

## 🚀 What are Actions?

Actions is a curated collection of powerful, reliable, and well-documented GitHub Actions designed to streamline your development workflows. Each action follows enterprise best practices and provides comprehensive functionality for common CI/CD tasks.

## 📚 Action Categories

### 🔧 .NET Development Actions

- **[.NET Tool Install](dotnet-tool-install/README.md)** - Install and manage .NET global tools
- **[.NET Build](dotnet/README.md)** - Build .NET projects with advanced configuration
- **[NuGet Feed Setup](dotnet-nuget-feed-setup/README.md)** - Configure NuGet package sources
- **[NuGet Upload](dotnet-nuget-upload/README.md)** - Publish packages to NuGet repositories

### 📖 Documentation Actions

- **[DocFX Build](dotnet-docfx-build/README.md)** - Build static documentation sites
- **[DocFX Metadata](dotnet-docfx-metadata/README.md)** - Extract API metadata from .NET code
- **[DocFX PDF](dotnet-docfx-pdf/README.md)** - Generate PDF documentation

### 🔒 Security & Supply Chain

- **[Install Apple Certificate](install-apple-certificate/README.md)** - Install Apple certificates for iOS/macOS development
- **[CycloneDX SBOM](dotnet-cyclonedx/README.md)** - Generate Software Bill of Materials
- **[gh-sbom](gh-sbom/README.md)** - Generate repository SBOMs (SPDX/CycloneDX) via the GitHub CLI
- **[SBOM Sign](sbom-sign/README.md)** - Sign SBOM files with RSA signatures using the CycloneDX CLI
- **[SBOM Publish](sbom-publish/README.md)** - Upload SBOM files to Dependency-Track and compatible systems

### 🚀 Release & Versioning

- **[Generate Version](generate-version/README.md)** - Intelligent semantic versioning
- **[Git Tag](git-tag/README.md)** - Create and manage Git tags
- **[GitHub Release](github-release/README.md)** - Create GitHub releases with assets

### 🎨 Utilities

- **[Generate Badge](generate-badge/README.md)** - Create dynamic badges for repositories

## ✨ Key Features

### 🛡️ **Enterprise-Ready**

- Comprehensive input validation
- Detailed error handling and logging
- Cross-platform compatibility (Linux, macOS, Windows)
- Secure secret handling

### 📊 **Rich Outputs**

- Detailed action summaries
- Comprehensive output variables
- Integration-friendly results
- Audit trail capabilities

### 🔧 **Developer Experience**

- Clear, comprehensive documentation
- Real-world usage examples
- Troubleshooting guides
- Best practices included

### 🎯 **Standards Compliance**

All actions follow the Actions Guidelines:

- **Validation Step** - Comprehensive input validation
- **Main Action Step** - Core functionality execution
- **Summary Step** - Detailed reporting and results

## 🚀 Quick Start

### Basic Usage

```yaml
# Example: Version generation and release
- name: Generate Version
  id: version
  uses: ./generate-version
  with:
    major: '1'
    minor: '0'

- name: Create Git Tag
  uses: ./git-tag
  with:
    tag: ${{ steps.version.outputs.VERSION_CORE }}
    prefix: 'v'

- name: Create GitHub Release
  uses: ./github-release
  with:
    tag: v${{ steps.version.outputs.VERSION_CORE }}
    generate-notes: 'true'
```

### Advanced Workflow

```yaml
name: Complete CI/CD Pipeline

on:
  push:
    branches: [ main ]

jobs:
  build-and-release:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
      with:
        fetch-depth: 0

    # Generate semantic version
    - name: Generate Version
      id: version
      uses: ./generate-version
      with:
        major: '1'
        minor: '0'
        output-props: 'Directory.Build.props'

    # Build .NET project
    - name: Build Application
      uses: ./dotnet
      with:
        command: 'build'
        configuration: 'Release'

    # Generate SBOM
    - name: Generate SBOM
      uses: ./dotnet-cyclonedx
      with:
        path: '.'
        output: 'sbom'

    # Create version tag
    - name: Create Tag
      uses: ./git-tag
      with:
        tag: ${{ steps.version.outputs.VERSION_CORE }}
        prefix: 'v'

    # Create GitHub release
    - name: Create Release
      uses: ./github-release
      with:
        tag: v${{ steps.version.outputs.VERSION_CORE }}
        generate-notes: 'true'
        assets: |
          dist/*.zip
          sbom/*.xml
```

## 📖 Documentation Structure

- **Action Reference** - Detailed documentation for each action
- **Examples** - Real-world usage scenarios
- **Best Practices** - Recommended patterns and configurations
- **Troubleshooting** - Common issues and solutions

## 🤝 Contributing

We welcome contributions! Please see our [Contributing Guidelines](.github/copilot-instructions.md) for details on:

- Action development standards
- Documentation requirements
- Testing procedures
- Code review process

## 📄 License

This project is licensed under the terms specified in [LICENSE.md](LICENSE.md).

## 🆘 Support

- **Documentation**: Browse the action-specific documentation
- **Issues**: Report bugs and request features on GitHub
- **Community**: Join discussions and share experiences

---

*Built with ❤️ for the GitHub Actions community*
