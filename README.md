# Reusable Workflows

Reusable GitHub Actions workflows. Reference them from any repository via `uses:`.

## Workflows

### Build and Push Docker Image

[`.github/workflows/build-push-image.yml`](.github/workflows/build-push-image.yml) — Builds a Docker image and pushes it to Amazon ECR.

**Inputs**

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `image-name` | **yes** | — | Docker image name, e.g. `account-service` |
| `image-tag` | no | `github.ref_name` | Image tag |
| `push` | no | `true` | Push image to registry |
| `context` | no | `.` | Docker build context path |
| `dockerfile` | no | `Dockerfile` | Dockerfile path relative to context |
| `branch-or-tag` | no | `github.ref_name` | Git ref to checkout |
| `platforms` | no | `linux/amd64` | Target platforms (e.g. `linux/amd64,linux/arm64`) |
| `build-args` | no | `""` | Newline-separated build args |
| `cache` | no | `true` | Enable Docker layer cache via registry |
| `cache-mode` | no | `min` | Cache mode: `min` or `max` |
| `runner` | no | `ubuntu-latest` | GitHub Actions runner label |
| `aws-region` | no | `ap-southeast-1` | AWS region for ECR |

**Secrets**

| Secret | Required | Description |
|--------|----------|-------------|
| `aws-access-key-id` | **yes** | AWS access key ID |
| `aws-secret-access-key` | **yes** | AWS secret access key |
| `build-secrets` | no | Newline-separated secret build args (e.g. `GH_TOKEN=xxx`) |

**Outputs**

| Output | Description |
|--------|-------------|
| `image-uri` | Full image URI with tag |
| `image-digest` | Image digest (sha256) |

## Usage

### Minimal

```yaml
name: Build
on:
  push:
    branches: [main, staging, develop]

jobs:
  build:
    uses: <org>/github-workflows/.github/workflows/build-push-image.yml@main
    with:
      image-name: my-service
    secrets:
      aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
      aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

### Multi-platform with build secrets

```yaml
name: Build
on:
  push:
    branches: [main]

jobs:
  build:
    uses: <org>/github-workflows/.github/workflows/build-push-image.yml@main
    with:
      image-name: my-service
      platforms: linux/amd64,linux/arm64
      cache-mode: max
      build-args: |
        NODE_ENV=production
        API_URL=https://api.example.com
    secrets:
      aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
      aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      build-secrets: |
        GH_TOKEN=${{ secrets.GH_TOKEN }}
```

### Build only (no push, e.g. PR validation)

```yaml
name: PR Check
on:
  pull_request:

jobs:
  build:
    uses: <org>/github-workflows/.github/workflows/build-push-image.yml@main
    with:
      image-name: my-service
      push: false
    secrets:
      aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
      aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```
