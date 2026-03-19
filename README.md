# Reusable Workflows

Reusable GitHub Actions workflows. Reference them from any repository via `uses:`.

## Build and Push Docker Image

[`.github/workflows/build-push-image.yml`](.github/workflows/build-push-image.yml) — Builds a Docker image and pushes to any container registry.

Supported registries: **Amazon ECR**, **Google Artifact Registry**, **GitHub Container Registry**, **Docker Hub**, and any **custom** registry.

### Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `registry` | no | `ecr` | Registry type: `ecr` \| `gar` \| `ghcr` \| `dockerhub` \| `custom` |
| `registry-url` | for gar/custom | `""` | Registry hostname (e.g. `asia-southeast1-docker.pkg.dev`) |
| `image-name` | **yes** | — | Image path within registry (see examples below) |
| `image-tag` | no | `github.ref_name` | Image tag |
| `push` | no | `true` | Push image to registry |
| `context` | no | `.` | Docker build context path |
| `dockerfile` | no | `Dockerfile` | Dockerfile path |
| `branch-or-tag` | no | `github.ref_name` | Git ref to checkout |
| `platforms` | no | `linux/amd64` | Target platforms |
| `build-args` | no | `""` | Newline-separated build args |
| `cache` | no | `true` | Enable Docker layer cache via registry |
| `cache-mode` | no | `min` | Cache mode: `min` or `max` |
| `runner` | no | `ubuntu-latest` | GitHub Actions runner label |
| `aws-region` | no | `ap-southeast-1` | AWS region (ECR only) |

### Secrets (pass only what your registry needs)

| Secret | Registries | Description |
|--------|-----------|-------------|
| `aws-access-key-id` | ECR | AWS access key ID |
| `aws-secret-access-key` | ECR | AWS secret access key |
| `gcp-credentials` | GAR | GCP service account JSON key |
| `registry-username` | GHCR, DockerHub, Custom | Registry username |
| `registry-password` | GHCR, DockerHub, Custom | Registry password or token |
| `build-secrets` | all | Newline-separated secret build args |

### Outputs

| Output | Description |
|--------|-------------|
| `image-uri` | Full image URI with tag (e.g. `ghcr.io/org/app:v1.0`) |
| `image-digest` | Image digest (sha256) |

### Image naming by registry

| Registry | `registry-url` | `image-name` | Resulting image |
|----------|---------------|--------------|-----------------|
| ECR | *(auto)* | `my-service` | `123456.dkr.ecr.region.amazonaws.com/my-service:tag` |
| GAR | `asia-southeast1-docker.pkg.dev` | `project/repo/image` | `asia-southeast1-docker.pkg.dev/project/repo/image:tag` |
| GHCR | *(auto)* | `org/image` | `ghcr.io/org/image:tag` |
| DockerHub | *(auto)* | `user/image` | `docker.io/user/image:tag` |
| Custom | `registry.example.com` | `team/image` | `registry.example.com/team/image:tag` |

---

## Usage Examples

### Amazon ECR (default)

```yaml
jobs:
  build:
    uses: <org>/github-workflows/.github/workflows/build-push-image.yml@main
    with:
      image-name: my-service
    secrets:
      aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
      aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

### Google Artifact Registry

```yaml
jobs:
  build:
    uses: <org>/github-workflows/.github/workflows/build-push-image.yml@main
    with:
      registry: gar
      registry-url: asia-southeast1-docker.pkg.dev
      image-name: my-project/my-repo/my-service
      image-tag: ${{ github.ref_name }}-${{ github.sha }}
    secrets:
      gcp-credentials: ${{ secrets.GCP_CREDENTIALS }}
```

### GitHub Container Registry

```yaml
jobs:
  build:
    uses: <org>/github-workflows/.github/workflows/build-push-image.yml@main
    with:
      registry: ghcr
      image-name: ${{ github.repository }}
    secrets:
      registry-username: ${{ github.actor }}
      registry-password: ${{ secrets.GITHUB_TOKEN }}
```

### Docker Hub

```yaml
jobs:
  build:
    uses: <org>/github-workflows/.github/workflows/build-push-image.yml@main
    with:
      registry: dockerhub
      image-name: myuser/my-service
    secrets:
      registry-username: ${{ secrets.DOCKERHUB_USERNAME }}
      registry-password: ${{ secrets.DOCKERHUB_TOKEN }}
```

### Custom / Private Registry

```yaml
jobs:
  build:
    uses: <org>/github-workflows/.github/workflows/build-push-image.yml@main
    with:
      registry: custom
      registry-url: harbor.internal.company.com
      image-name: team/my-service
    secrets:
      registry-username: ${{ secrets.REGISTRY_USER }}
      registry-password: ${{ secrets.REGISTRY_PASS }}
```

### Multi-platform with build secrets

```yaml
jobs:
  build:
    uses: <org>/github-workflows/.github/workflows/build-push-image.yml@main
    with:
      image-name: my-service
      platforms: linux/amd64,linux/arm64
      cache-mode: max
      build-args: |
        NODE_ENV=production
    secrets:
      aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
      aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      build-secrets: |
        GH_TOKEN=${{ secrets.GH_TOKEN }}
```

### Build only (PR validation, no push)

```yaml
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

---

## Monorepo CI/CD

[`.github/workflows/monorepo-cicd.yml`](.github/workflows/monorepo-cicd.yml) — Detects changes in a monorepo and builds only the affected services. Image name is auto-derived: `{repo-name}-{service}`.

Supported registries: **Amazon ECR**, **Google Artifact Registry**, **GitHub Container Registry**, **Docker Hub**, and any **custom** registry.

Uses `secrets: inherit` — no need to declare secrets in the caller. The workflow picks secrets by convention.

### Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `services` | **yes** | — | JSON array of service directory names |
| `registry` | no | `dockerhub` | Registry type: `ecr` \| `gar` \| `ghcr` \| `dockerhub` \| `custom` |
| `registry-url` | for gar/custom | `""` | Registry hostname (e.g. `asia-southeast1-docker.pkg.dev`) |
| `registry-namespace` | no | `github.repository_owner` | Image namespace/owner |
| `image-tag` | no | `{branch}-{short-sha}` | Image tag |
| `push` | no | `true` | Push image to registry |
| `branch-or-tag` | no | `github.ref_name` | Git ref to checkout |
| `platforms` | no | `linux/amd64` | Target platforms |
| `build-args` | no | `""` | Newline-separated build args |
| `cache` | no | `true` | Enable Docker layer cache via registry |
| `cache-mode` | no | `min` | Cache mode: `min` or `max` |
| `runner` | no | `ubuntu-latest` | GitHub Actions runner label |
| `aws-region` | no | `ap-southeast-1` | AWS region (ECR only) |

### Secrets (via `secrets: inherit`)

No explicit secret mapping needed in caller — just use `secrets: inherit`. The workflow reads secrets by well-known names:

| Registry | Required secrets in caller repo |
|----------|---------------------------------|
| ECR | `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY` |
| GAR | `GCP_CREDENTIALS` |
| DockerHub / Custom | `REGISTRY_USERNAME`, `REGISTRY_PASSWORD` |
| GHCR | `GITHUB_TOKEN` *(automatic)* |
| *(optional)* | `BUILD_SECRETS` — newline-separated secret build args |

### How it works

1. Detects which service directories have changes (via `dorny/paths-filter`)
2. Runs a matrix build **in parallel** for each changed service
3. Image name follows the convention: `{namespace}/{repo-name}-{service}:{tag}`

For example, in a repo named `clawfriend` with services `["backend","frontend"]`:
- `docker.io/myuser/clawfriend-backend:main-a1b2c3d4`
- `docker.io/myuser/clawfriend-frontend:main-a1b2c3d4`

### Usage — DockerHub (minimal)

```yaml
jobs:
  cicd:
    uses: <user>/github-workflows/.github/workflows/monorepo-cicd.yml@main
    with:
      services: '["backend","frontend","sync-transaction"]'
    secrets: inherit
```

### Usage — GHCR

```yaml
jobs:
  cicd:
    uses: <user>/github-workflows/.github/workflows/monorepo-cicd.yml@main
    with:
      services: '["backend","frontend"]'
      registry: ghcr
    secrets: inherit
```

### Usage — Amazon ECR

```yaml
jobs:
  cicd:
    uses: <user>/github-workflows/.github/workflows/monorepo-cicd.yml@main
    with:
      services: '["api","worker"]'
      registry: ecr
    secrets: inherit
```

### Usage — Google Artifact Registry

```yaml
jobs:
  cicd:
    uses: <user>/github-workflows/.github/workflows/monorepo-cicd.yml@main
    with:
      services: '["api","worker"]'
      registry: gar
      registry-url: asia-southeast1-docker.pkg.dev
    secrets: inherit
```

### Usage — Custom tag & namespace

```yaml
jobs:
  cicd:
    uses: <user>/github-workflows/.github/workflows/monorepo-cicd.yml@main
    with:
      services: '["api","worker"]'
      registry-namespace: my-dockerhub-user
      image-tag: v1.2.3
    secrets: inherit
```
