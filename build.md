# Build Image

Reusable workflow: `.github/workflows/build.yaml`

## Purpose

Builds the application container with the commit SHA as its image tag. The caller selects whether the build is non-authoritative (pull request validation only) or authoritative (publish to ECR).

## Usage

Non-authoritative pull-request build:

```yaml
jobs:
  build:
    uses: SparrowX-ECS/workflows-templates/.github/workflows/build.yaml@<version>
    with:
      env-params-file: ecs-parameters-dev.yaml
      authoritative-build: false
```

Authoritative post-merge build:

```yaml
jobs:
  build:
    uses: SparrowX-ECS/workflows-templates/.github/workflows/build.yaml@<version>
    with:
      env-params-file: ecs-parameters-dev.yaml
      authoritative-build: true
```

## Inputs

| Input | Required | Default | Description |
|---|---:|---:|---|
| `env-params-file` | yes | — | ECS parameter file containing `.image.nameSpace` and `.image.repository`; used for authoritative ECR builds |
| `frontend` | no | `false` | Enables the frontend build argument from `.frontend.apiBaseUrl` |
| `authoritative-build` | no | `false` | When `true`, authenticates to ECR, skips an existing tag, and pushes a new image. When `false`, builds locally and does not use AWS or ECR. |

The caller must provide `AWS_REGION`, `AWS_ACCOUNT_ID`, and `AWS_ROLE_NAME` variables when `authoritative-build` is `true`. The OIDC role must be able to describe and push images in ECR.

## Outputs

| Output | Description |
|---|---|
| `image-tag` | The GitHub commit SHA used as the image tag |

## Build results and summary

The non-authoritative image is tagged locally as `<commit-sha>`. The authoritative image is tagged as `<account>.dkr.ecr.<region>.amazonaws.com/<namespace>/<repository>:<commit-sha>` and pushed to ECR unless that tag already exists.

The workflow creates no report or metadata file. It writes this summary shape to `$GITHUB_STEP_SUMMARY`:

```text
## Build
| ECR repository | repository or Unavailable |
| Image tag | commit SHA |
| Build mode | non-authoritative or authoritative |
| Image existed before build | true, false, or Unknown |
| Published to ECR | true or false |
| Result | success or failure |
```
