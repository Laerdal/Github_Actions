# 📤 SBOM Publishing Action

A comprehensive GitHub Action for uploading Software Bill of Materials (SBOM) files to Dependency Tracker systems with advanced retry logic, validation, and monitoring capabilities.

## ✨ Features

- 📤 **Dependency Tracker Upload** - Upload SBOM files to Dependency-Track and compatible systems
- 🔄 **Intelligent Retry Logic** - Configurable retry attempts with exponential backoff
- ⏱️ **Timeout Management** - Configurable request timeouts for large files
- 📊 **Detailed Metrics** - Upload status, timing, and response information
- 🔗 **Project URL Generation** - Direct links to projects in dependency tracker
- ✅ **Response Validation** - Comprehensive upload verification
- 👨‍👩‍👧‍👦 **Parent Project Support** - Hierarchical project organization
- 🛡️ **Secure Authentication** - API key-based authentication with secure handling

## 🚀 Basic Usage

Upload an SBOM with default settings:

```yaml
- name: "Upload SBOM"
  uses: laerdal/github_actions/sbom-publish@main
  with:
    sbom-file-path: "./sbom.xml"
    project-name: "MyProject"
    project-version: "1.0.0"
    dependency-tracker-url: "https://tracker.example.com/api/v1/bom"
    dependency-tracker-api-key: ${{ secrets.DEPENDENCY_TRACKER_API_KEY }}
```

```yaml
- name: "Upload with parent project"
  uses: laerdal/github_actions/sbom-publish@main
  with:
    sbom-file-path: "./component-sbom.xml"
    project-name: "MyComponent"
    project-tags: "my-component,release"
    project-version: "2.1.0"
    parent-project-name: "MainApplication"
    parent-project-version: "1.5.0"
    dependency-tracker-url: ${{ vars.DEPENDENCY_TRACKER_URL }}
    dependency-tracker-api-key: ${{ secrets.API_KEY }}
```

```yaml
- name: "Upload with retry configuration"
  uses: laerdal/github_actions/sbom-publish@main
  with:
    sbom-file-path: "./large-sbom.xml"
    project-name: "LargeProject"
    project-version: "3.0.0"
    dependency-tracker-url: ${{ vars.TRACKER_URL }}
    dependency-tracker-api-key: ${{ secrets.API_KEY }}
    timeout-seconds: "600"
    retry-attempts: "5"
```

## 🔧 Advanced Usage

Full configuration with all available options:

```yaml
- name: "Advanced SBOM upload"
  uses: laerdal/github_actions/sbom-publish@main
  with:
    sbom-file-path: "./artifacts/signed-sbom.xml"
    project-name: "MyProject"
    project-tags: "my-project,release"
    project-version: "1.2.3"
    dependency-tracker-url: "https://tracker.example.com/api/v1/bom"
    dependency-tracker-api-key: ${{ secrets.API_KEY }}
    parent-project-name: "[Group::ParentProject]"
    parent-project-version: "2.0.0"
    auto-create: "true"
    timeout-seconds: "600"
    retry-attempts: "5"
    retry-delay-seconds: "15"
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
name: "📋 SBOM Publishing Pipeline"

on:
  push:
    branches: ["main"]
    tags: ["v*"]
  release:
    types: [published]

permissions:
  contents: read

jobs:
  publish-sbom:
    runs-on: ubuntu-latest
    environment: production

    steps:
      - name: "📥 Checkout repository"
        uses: actions/checkout@v6

      # ...

      - name: "📋 Generate SBOM"
        id: generate-sbom
        uses: laerdal/github_actions/dotnet-cyclonedx@main
        with:
          path: "./src/MyProject.csproj"
          output-format: "xml"

      - name: "🔒 Sign SBOM"
        id: sign-sbom
        uses: laerdal/github_actions/sbom-sign@main
        with:
          sbom-file-path: ${{ steps.generate-sbom.outputs.sbom-file }}
          signing-key: ${{ secrets.SBOM_SIGNING_KEY }}

      - name: "📤 Upload to Dependency Tracker"
        id: upload-sbom
        uses: laerdal/github_actions/sbom-publish@main
        with:
          sbom-file-path: ${{ steps.sign-sbom.outputs.signed-sbom-path }}
          project-name: "MyProject"
          project-version: ${{ steps.version.outputs.version }}
          dependency-tracker-url: ${{ vars.DEPENDENCY_TRACKER_URL }}
          dependency-tracker-api-key: ${{ secrets.DEPENDENCY_TRACKER_API_KEY }}
          timeout-seconds: "300"
          retry-attempts: "3"
          show-summary: "true"

      - name: "📊 Report results"
        run: |
          echo "Upload status: ${{ steps.upload-sbom.outputs.upload-status }}"
          echo "Project URL: ${{ steps.upload-sbom.outputs.project-url }}"
          echo "HTTP status: ${{ steps.upload-sbom.outputs.http-status-code }}"

      - name: "📁 Archive SBOM"
        uses: actions/upload-artifact@v7
        with:
          name: "sbom-${{ steps.version.outputs.version }}"
          path: ${{ steps.sign-sbom.outputs.signed-sbom-path }}
          retention-days: 365
```

## 📋 Inputs

| Input                        | Description                             | Required | Default | Example                                     |
|------------------------------|-----------------------------------------|----------|---------|---------------------------------------------|
| `sbom-file-path`             | Path to the SBOM file to upload         | ✅ Yes   | -       | `./sbom.xml`, `./artifacts/signed-sbom.xml` |
| `project-name`               | Name of the project                     | ✅ Yes   | -       | `MyProject`, `web-api-service`              |
| `project-version`            | Version of the project                  | ✅ Yes   | -       | `1.0.0`, `2.1.3-beta`                       |
| `dependency-tracker-url`     | Dependency Tracker API endpoint URL     | ✅ Yes   | -       | `https://tracker.example.com/api/v1/bom`    |
| `dependency-tracker-api-key` | API key for authentication              | ✅ Yes   | -       | `${{ secrets.DEPENDENCY_TRACKER_API_KEY }}` |
| `project-tags`               | Project tags (optional)                 | ❌ No    | -       | `nugets,release`                            |
| `parent-project-name`        | Name of parent project (optional)       | ❌ No    | -       | `ParentProject`, `main-application`         |
| `parent-project-version`     | Version of parent project (optional)    | ❌ No    | -       | `2.0.0`, `1.5.0-stable`                     |
| `auto-create`                | Auto-create project if it doesn't exist | ❌ No    | `true`  | `true`, `false`                             |
| `timeout-seconds`            | Request timeout in seconds              | ❌ No    | `300`   | `60`, `600`, `1200`                         |
| `retry-attempts`             | Number of retry attempts                | ❌ No    | `3`     | `1`, `5`, `10`                              |
| `retry-delay-seconds`        | Delay between retries in seconds        | ❌ No    | `10`    | `5`, `30`, `60`                             |
| `show-summary`               | Display action summary                  | ❌ No    | `false` | `true`, `false`                             |

## 📤 Outputs

| Output             | Description                               | Type     | Example                                    |
|--------------------|-------------------------------------------|----------|--------------------------------------------|
| `upload-status`    | Upload status result                      | `string` | `success`, `failed`                        |
| `http-status-code` | HTTP status code from request             | `string` | `200`, `400`, `500`                        |
| `response-body`    | Response body (first 1000 chars)          | `string` | `{"status":"success"}`                     |
| `upload-timestamp` | ISO timestamp of upload completion        | `string` | `2024-10-04T12:30:45Z`                     |
| `project-url`      | URL to view project in dependency tracker | `string` | `https://tracker.example.com/projects/123` |
| `upload-duration`  | Upload duration in seconds                | `string` | `12.5`                                     |
| `file-size`        | Size of uploaded SBOM file                | `string` | `15728`                                    |

## 🔗 Related Actions

| Action                  | Purpose                | Repository                                |
|-------------------------|------------------------|-------------------------------------------|
| 📋 **dotnet-cyclonedx** | Generate SBOM files    | `laerdal/github_actions/dotnet-cyclonedx` |
| 🔒 **sbom-sign**        | Sign SBOM files        | `laerdal/github_actions/sbom-sign`        |
| 📊 **gh-sbom**          | GitHub SBOM operations | `laerdal/github_actions/gh-sbom`          |
| 🔢 **generate-version** | Version generation     | `laerdal/github_actions/generate-version` |

## 💡 Examples

### Basic Upload

```yaml
- name: "Upload SBOM to dependency tracker"
  uses: laerdal/github_actions/sbom-publish@main
  with:
    sbom-file-path: "./sbom.xml"
    project-name: "my-application"
    project-version: "1.0.0"
    dependency-tracker-url: "https://dtrack.company.com/api/v1/bom"
    dependency-tracker-api-key: ${{ secrets.DTRACK_API_KEY }}
```

### Upload with Parent Project

```yaml
- name: "Upload component SBOM"
  uses: laerdal/github_actions/sbom-publish@main
  with:
    sbom-file-path: "./component-sbom.xml"
    project-name: "authentication-service"
    project-version: "2.1.0"
    parent-project-name: "microservices-platform"
    parent-project-version: "1.5.0"
    dependency-tracker-url: ${{ vars.DEPENDENCY_TRACKER_URL }}
    dependency-tracker-api-key: ${{ secrets.API_KEY }}
```

### High-Reliability Upload

```yaml
- name: "Upload critical service SBOM"
  uses: laerdal/github_actions/sbom-publish@main
  with:
    sbom-file-path: "./critical-sbom.xml"
    project-name: "payment-service"
    project-version: "3.0.0"
    dependency-tracker-url: "https://tracker.example.com/api/v1/bom"
    dependency-tracker-api-key: ${{ secrets.API_KEY }}
    auto-create: "true"
    retry-attempts: "5"
    timeout-seconds: "600"
    retry-delay-seconds: "30"
```

### Complete Pipeline Chain

```yaml
- name: "Generate SBOM"
  id: generate
  uses: laerdal/github_actions/dotnet-cyclonedx@main
  with:
    path: "./MyProject.csproj"
    output-format: "xml"

- name: "Sign SBOM"
  id: sign
  uses: laerdal/github_actions/sbom-sign@main
  with:
    sbom-file-path: ${{ steps.generate.outputs.sbom-file }}
    signing-key: ${{ secrets.SIGNING_KEY }}

- name: "Upload signed SBOM"
  uses: laerdal/github_actions/sbom-publish@main
  with:
    sbom-file-path: ${{ steps.sign.outputs.signed-sbom-path }}
    project-name: "MyProject"
    project-version: "1.0.0"
    dependency-tracker-url: ${{ vars.DEPENDENCY_TRACKER_URL }}
    dependency-tracker-api-key: ${{ secrets.DEPENDENCY_TRACKER_API_KEY }}
```

### Conditional Upload Based on Environment

```yaml
- name: "Upload to production tracker"
  if: github.ref == 'refs/heads/main'
  uses: laerdal/github_actions/sbom-publish@main
  with:
    sbom-file-path: "./sbom.xml"
    project-name: "MyProject-prod"
    project-version: ${{ github.ref_name }}
    dependency-tracker-url: ${{ vars.PROD_TRACKER_URL }}
    dependency-tracker-api-key: ${{ secrets.PROD_API_KEY }}

- name: "Upload to staging tracker"
  if: github.ref == 'refs/heads/develop'
  uses: laerdal/github_actions/sbom-publish@main
  with:
    sbom-file-path: "./sbom.xml"
    project-name: "MyProject-staging"
    project-version: ${{ github.ref_name }}
    dependency-tracker-url: ${{ vars.STAGING_TRACKER_URL }}
    dependency-tracker-api-key: ${{ secrets.STAGING_API_KEY }}
```

## 🔌 API Compatibility

This action is designed to work with Dependency-Track and compatible APIs:

### Request Format

- **Method**: POST
- **Content-Type**: `multipart/form-data`
- **Authentication**: `X-API-Key` header

### Form Data

| Field            | Description              | Type              |
|------------------|--------------------------|-------------------|
| `bom`            | SBOM file content        | File              |
| `autoCreate`     | Auto-create project flag | Boolean           |
| `projectName`    | Project name             | String            |
| `projectVersion` | Project version          | String            |
| `parentName`     | Parent project name      | String (optional) |
| `parentVersion`  | Parent project version   | String (optional) |

### Response Handling

The action validates responses and handles various HTTP status codes:

- **200-299**: Success responses
- **400-499**: Client errors (logged and failed)
- **500-599**: Server errors (retried based on configuration)

## 🔄 Retry Logic

The action implements intelligent retry logic with configurable parameters:

1. **Configurable Attempts**: Set `retry-attempts` (default: 3)
2. **Exponential Backoff**: Delay increases between retries
3. **Timeout Handling**: Per-request timeout with `timeout-seconds`
4. **Status Code Validation**: Only retries on network/timeout errors
5. **Permanent Failure Detection**: Stops retrying on authentication/validation errors

## 🐛 Troubleshooting

### Common Issues

#### 401 Unauthorized

**Problem**: API key authentication failed

**Solution**: Verify API key and permissions:

```yaml
- name: "Test API connectivity"
  run: |
    curl -I -H "X-API-Key: ${{ secrets.DEPENDENCY_TRACKER_API_KEY }}" \
      "${{ vars.DEPENDENCY_TRACKER_URL }}"
```

#### 404 Not Found

**Problem**: Dependency tracker URL is incorrect

**Solution**: Verify the API endpoint URL:

```yaml
- name: "Validate API endpoint"
  run: |
    echo "Testing URL: ${{ vars.DEPENDENCY_TRACKER_URL }}"
    curl -I "${{ vars.DEPENDENCY_TRACKER_URL }}"
```

#### 413 Payload Too Large

**Problem**: SBOM file exceeds server limits

**Solution**: Check file size and server configuration:

```yaml
- name: "Check SBOM file size"
  run: |
    size=$(stat -c%s "sbom.xml" 2>/dev/null || stat -f%z "sbom.xml")
    echo "SBOM file size: $size bytes"
    if [ $size -gt 10485760 ]; then
      echo "⚠️ File larger than 10MB, consider compression"
    fi
```

#### Timeout Errors

**Problem**: Upload times out for large files

**Solution**: Increase timeout and retry settings:

```yaml
- name: "Upload large SBOM"
  uses: laerdal/github_actions/sbom-publish@main
  with:
    sbom-file-path: "./large-sbom.xml"
    project-name: "MyProject"
    project-version: "1.0.0"
    dependency-tracker-url: ${{ vars.DEPENDENCY_TRACKER_URL }}
    dependency-tracker-api-key: ${{ secrets.DEPENDENCY_TRACKER_API_KEY }}
    timeout-seconds: "1200"  # 20 minutes
    retry-attempts: "5"
    retry-delay-seconds: "60"
```

### Debug Tips

1. **Enable Summary**: Set `show-summary: "true"` for detailed output
2. **Check File Existence**: Verify SBOM file exists and is readable
3. **Test API Manually**: Use curl to test API connectivity
4. **Validate Credentials**: Ensure API key has required permissions

## 📝 Requirements

- GitHub Actions runner (Windows, Linux, or macOS)
- Valid SBOM file in CycloneDX format
- Dependency-Track or compatible system
- Valid API key with upload permissions
- Network access to dependency tracker

## 🔧 Advanced Features

### Multi-Environment Deployment

```yaml
strategy:
  matrix:
    environment: [dev, staging, prod]
    include:
      - environment: dev
        tracker_url: DEV_TRACKER_URL
        api_key: DEV_API_KEY
      - environment: staging
        tracker_url: STAGING_TRACKER_URL
        api_key: STAGING_API_KEY
      - environment: prod
        tracker_url: PROD_TRACKER_URL
        api_key: PROD_API_KEY

steps:
  - name: "Upload to ${{ matrix.environment }}"
    uses: laerdal/github_actions/sbom-publish@main
    with:
      sbom-file-path: "./sbom.xml"
      project-name: "MyProject-${{ matrix.environment }}"
      project-version: "1.0.0"
      dependency-tracker-url: ${{ vars[matrix.tracker_url] }}
      dependency-tracker-api-key: ${{ secrets[matrix.api_key] }}
```

### Batch Upload Multiple SBOMs

```yaml
- name: "Upload multiple SBOMs"
  run: |
    for sbom_file in ./sboms/*.xml; do
      project_name=$(basename "$sbom_file" .xml)

      echo "Uploading SBOM for $project_name..."

      # Use the action for each SBOM
      # Note: This would need to be implemented as a reusable workflow
      # for actual batch processing
    done
```

### Monitoring and Alerting

```yaml
- name: "Upload SBOM with monitoring"
  id: upload
  uses: laerdal/github_actions/sbom-publish@main
  with:
    sbom-file-path: "./sbom.xml"
    project-name: "MyProject"
    project-version: "1.0.0"
    dependency-tracker-url: ${{ vars.DEPENDENCY_TRACKER_URL }}
    dependency-tracker-api-key: ${{ secrets.DEPENDENCY_TRACKER_API_KEY }}

- name: "Send success notification"
  if: steps.upload.outputs.upload-status == 'success'
  run: |
    curl -X POST \
      -H "Content-Type: application/json" \
      -d '{"text":"✅ SBOM uploaded successfully for ${{ github.repository }}"}' \
      "${{ secrets.SLACK_WEBHOOK_URL }}"

- name: "Send failure alert"
  if: failure()
  run: |
    curl -X POST \
      -H "Content-Type: application/json" \
      -d '{"text":"❌ SBOM upload failed for ${{ github.repository }}"}' \
      "${{ secrets.SLACK_WEBHOOK_URL }}"
```

## 📄 License

This action is part of the Laerdal Medical GitHub Actions collection and follows the same license terms.

---

> 💡 **Tip**: Combine this action with our SBOM generation and signing actions for complete supply chain security workflows.
