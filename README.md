# Build and Push Image GitHub Action

A comprehensive GitHub Action for building and pushing Docker/OCI images to registries with built-in security scanning and multi-platform support.

## Features

- 🐳 **Multi-Registry Support**: Supports both Docker Hub (public images) and Artifactory (private images)
- 🏗️ **Multi-Platform Builds**: Build for multiple architectures simultaneously (default: linux/amd64, linux/arm64)
- 🔒 **Security Scanning**: Integrated Trivy vulnerability scanning before pushing
- 📦 **Build Attestations**: Generates SLSA build provenance attestations
- ⚡ **Smart Caching**: GitHub Actions cache integration for faster builds
- 🔑 **Secrets Management**: Support for build secrets, environment variables, and files
- 🎯 **Flexible Configuration**: Extensive customization options for build context, args, and more

## Usage

### Public Images (Docker Hub)

For public images that should be available to everyone, use Docker Hub:

```yaml
- name: Build and push public image
  uses: NethermindEth/github-action-image-build-and-push@v1
  with:
    registry: "dockerhub"
    image_name: "nethermindeth/myapp"
    image_tags: "v1.0.0,latest"
```

### Private Images (Artifactory)

For private/internal images, use Artifactory:

```yaml
- name: Build and push private image
  uses: NethermindEth/github-action-image-build-and-push@v1
  with:
    registry: "artifactory"
    image_name: "kyoto-oci-local-dev/myapp"
    image_tags: "v1.0.0,latest"
```

## Complete Workflow Examples

> 💡 **Quick Start**: Check out the ready-to-use example workflows in the [`examples/`](examples/) directory:
> - [`docker-hub-workflow.yml`](examples/docker-hub-workflow.yml) - For public image publishing
> - [`artifactory-workflow.yml`](examples/artifactory-workflow.yml) - For private image publishing

### Basic Example

```yaml
name: Build and Deploy

on:
  push:
    branches: [main]
    tags: ['v*']
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Login to Docker Hub
        if: github.event_name != 'pull_request'
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build and push
        uses: NethermindEth/github-action-image-build-and-push@v1
        with:
          registry: "dockerhub"
          image_name: "NethermindEth/myapp"
          image_tags: "latest"
```

### Advanced Example with Custom Configuration

```yaml
name: Advanced Build

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Login to Artifactory
        uses: docker/login-action@v3
        with:
          registry: nethermind.jfrog.io
          username: ${{ secrets.ARTIFACTORY_USERNAME }}
          password: ${{ secrets.ARTIFACTORY_PASSWORD }}

      - name: Build and push with custom config
        uses: NethermindEth/github-action-image-build-and-push@v1
        with:
          registry: "artifactory"
          image_name: "nubia-oci-local-dev/myapp"
          image_tags: "v1.2.3,latest"
          platforms: "linux/amd64,linux/arm64,linux/arm/v7"
          context: "./app"
          dockerfile_path: "./app/Dockerfile.prod"
          setup-qemu: "true"
          build-args: |
            VERSION=1.2.3
            BUILD_DATE=${{ github.event.head_commit.timestamp }}
          secrets: |
            GIT_AUTH_TOKEN=${{ secrets.GITHUB_TOKEN }}
          cache_from: |
            type=gha
            type=registry,ref=nethermind.jfrog.io/nubia-oci-local-dev/myapp:buildcache
          cache_to: |
            type=gha,mode=max
            type=registry,ref=nethermind.jfrog.io/nubia-oci-local-dev/myapp:buildcache,mode=max
```

### Multi-Stage Build with Secrets

```yaml
- name: Build with secrets
  uses: NethermindEth/github-action-image-build-and-push@v1
  with:
    registry: "artifactory"
    image_name: "nubia-oci-local-dev/secure-app"
    image_tags: "latest"
    secrets: |
      NPM_TOKEN=${{ secrets.NPM_TOKEN }}
      API_KEY=${{ secrets.API_KEY }}
    secret-files: |
      ssh_key=~/.ssh/id_rsa
    build-args: |
      NODE_ENV=production
      API_URL=https://api.example.com
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `registry` | Registry to use: `"dockerhub"` or `"artifactory"` | ✅ | - |
| `image_name` | Name of the image | ✅ | - |
| `image_tags` | Comma-separated list of tags | ✅ | - |
| `platforms` | Comma-separated list of target platforms | ❌ | `linux/amd64,linux/arm64` |
| `context` | Build context path | ❌ | `.` |
| `dockerfile_path` | Path to Dockerfile | ❌ | `Dockerfile` |
| `setup-qemu` | Set up QEMU for cross-platform builds | ❌ | `false` |
| `cache_from` | External cache sources | ❌ | `type=gha` |
| `cache_to` | Cache export destinations | ❌ | `type=gha,mode=max` |
| `secrets` | Build secrets (key=value format) | ❌ | `""` |
| `secret-envs` | Secret environment variables | ❌ | `""` |
| `secret-files` | Secret files | ❌ | `""` |
| `ulimit` | Ulimit options | ❌ | `""` |
| `build-args` | Build-time variables | ❌ | `""` |
| `run_trivy` | Run Trivy scan | ❌ | `true` |
| `ignore_trivy` | Ignore Trivy scan errors | ❌ | `false` |


## Security Features

### Vulnerability Scanning

The action automatically scans images for vulnerabilities using Trivy before pushing:

- Scans for **CRITICAL** and **HIGH** severity vulnerabilities
- Fails the build if vulnerabilities are found
- Supports `.trivyignore` file for suppressing specific vulnerabilities
- Only scans for fixed vulnerabilities by default

### Build Attestations

Automatically generates SLSA build provenance attestations that:

- Prove the image was built by your GitHub Actions workflow
- Provide transparency about the build process
- Enable supply chain security verification

## Platform Support

The action supports building for multiple platforms simultaneously:

- `linux/amd64` - Standard x86_64 Linux
- `linux/arm64` - ARM 64-bit (Apple Silicon, ARM servers)
- `linux/arm/v7` - ARM 32-bit v7
- `linux/arm/v6` - ARM 32-bit v6
- `linux/386` - x86 32-bit
- `windows/amd64` - Windows x86_64

## Caching

The action uses GitHub Actions cache by default, but supports various cache backends:

```yaml
# GitHub Actions cache (default)
cache_from: type=gha
cache_to: type=gha,mode=max

# Registry cache
cache_from: type=registry,ref=myregistry.com/myapp:buildcache
cache_to: type=registry,ref=myregistry.com/myapp:buildcache,mode=max

# Local cache
cache_from: type=local,src=/tmp/.buildx-cache
cache_to: type=local,dest=/tmp/.buildx-cache-new,mode=max
```

## Troubleshooting

### Common Issues

1. **Authentication Failures**
   - Ensure you're logged in to the registry before using this action
   - Verify your credentials are correct and have push permissions

2. **Platform Build Failures**
   - Set `setup-qemu: "true"` for cross-platform builds
   - Some base images may not support all platforms

3. **Trivy Scan Failures**
   - Review the security scan results
   - Add entries to `.trivyignore` if needed
   - Update base images to fix vulnerabilities


## Contributing

> **Note**: This action uses `action.yaml` (not `action.yml`) as the main action definition file, following GitHub's recommended naming convention.

Issues and pull requests are welcome! Please ensure you:

1. Test your changes thoroughly
2. Update documentation as needed
3. Follow the existing code style
4. Add appropriate tests
5. Post link to the tests (which will be a workflow running this action)
