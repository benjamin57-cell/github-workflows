# Reusable Workflows

This repository contains a collection of reusable workflows for GitHub Actions. The workflows are written in YAML and can be used in your own workflows by referencing the raw file on the default branch.

# Usage

Here is the sample usage of this reusable workflows

## The CI workflow

```yaml
name: Continuous Integration
on:
  push:
    branches:
      - main
      - staging
      - develop
  pull_request:

jobs:
  install-deps:
    uses: Salary-Hero/github-workflows/.github/workflows/ci.yaml@main
    with:
      node-version: 18

  lint:
    needs: install-deps
    uses: Salary-Hero/github-workflows/.github/workflows/ci.yaml@main
    with:
      target: lint
      node-version: 18

  test:
    needs: install-deps
    uses: Salary-Hero/github-workflows/.github/workflows/ci.yaml@main
    with:
      target: test
      node-version: 18

  build:
    needs: install-deps
    uses: Salary-Hero/github-workflows/.github/workflows/ci.yaml@main
    with:
      target: build
      node-version: 18
```

The workflow above will run the following steps:

1. Install dependencies (and save them to the GitHub Cache)

And the following steps in parallel:

- Restore dependencies cache and lint the code
- Restore dependencies cache and run the tests
- Restore dependencies cache and build the code

## The Deploy workflow

```yaml
name: Publish Docker image and deploy to EKS
on:
  push:
    branches:
      - main
      - staging
      - develop

jobs:
  deploy:
    uses: Salary-Hero/github-workflows/.github/workflows/deploy.yaml@main
    with:
      service-name: my-service # Just change this
      environment: ${{ github.ref_name }}
      prefix-image-tag: salary-hero
      kustomize-repository: Salary-Hero/infra
      kustomize-path: kustomize
      kustomize-branch: main
      eks-cluster: nonprod
      update-mode: open_pull_request
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      AWS_REGION: ${{ secrets.AWS_REGION }}
      GH_TOKEN: ${{ secrets.GH_TOKEN }} # This token should have an read/write access to the Salary-Hero/infra repository
```

If you have multiple EKS clusters which you needs to add conditional logic to your workflow, you can use the following example:

```yaml
name: Publish Docker image and deploy to EKS
on:
  push:
    branches:
      - main
      - staging
      - develop

jobs:
  prepare-env:
    runs-on: ubuntu-latest
    outputs:
      cluster: ${{ steps.set-cluster.outputs.cluster }}
      update-mode: ${{ steps.set-cluster.outputs.update-mode }}
    steps:
      - name: Set cluster
        id: set-cluster
        run: |
          if [[ ${{ github.ref_name  }} == "main" ]]; then
            echo "cluster=prod" >> $GITHUB_OUTPUT
            echo "update-mode=open_pull_request" >> $GITHUB_OUTPUT
          elif [[ ${{ github.ref_name  }} == "staging" ]]; then
            echo "cluster=nonprod" >> $GITHUB_OUTPUT
            echo "update-mode=commit" >> $GITHUB_OUTPUT
          elif [[ ${{ github.ref_name  }} == "develop" ]]; then
            echo "cluster=nonprod" >> $GITHUB_OUTPUT
            echo "update-mode=commit" >> $GITHUB_OUTPUT
          fi
  deploy:
    needs: prepare-env
    uses: Salary-Hero/github-workflows/.github/workflows/deploy.yaml@main
    with:
      service-name: my-service # Just change this
      environment: ${{ github.ref_name }}
      prefix-image-tag: salary-hero
      kustomize-repository: Salary-Hero/infra
      kustomize-path: kustomize
      kustomize-branch: main
      eks-cluster: ${{ needs.prepare-env.outputs.cluster }}
      update-mode: ${{ needs.prepare-env.outputs.update-mode }}
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      AWS_REGION: ${{ secrets.AWS_REGION }}
      GH_TOKEN: ${{ secrets.GH_TOKEN }} # This token should have an read/write access to the Salary-Hero/infra repository
```
