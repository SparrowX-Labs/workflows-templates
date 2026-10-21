# Resolve Image

Reusable workflow: `.github/workflows/resolve-image.yaml`

## Purpose

Reads published image metadata from SSM Parameter Store, validates that the tag exists in the ECR repository and resolves to the recorded digest, and exposes immutable image values for deployment or promotion.

## Usage

```yaml
jobs:
  resolve-image:
    uses: SparrowX-ECS/workflows-templates/.github/workflows/resolve-image.yaml@<version>
    with:
      ssm-parameter-path: ${{ vars.PROD_CANDIDATE_PARAM_STORE_PATH }}
      env-params-file: ecs-parameters-dev.yaml
```

## Inputs

| Input | Required | Expected values / description |
|---|---:|---|
| `ssm-parameter-path` | yes | SSM parameter containing the published image metadata JSON |
| `env-params-file` | yes | ECS parameters file containing the ECR namespace and repository |

## Outputs

| Output | Description |
|---|---|
| `image-tag` | Validated image tag |
| `image-digest` | Validated `sha256:<digest>` |
| `image-uri` | Immutable `registry/repository@sha256:digest` URI from the metadata |

## Consumed JSON format

The SSM value must be valid JSON containing:

```json
{
  "image": {
    "tag": "commit-sha",
    "digest": "sha256:...",
    "uri": "registry/repository@sha256:..."
  }
}
```

The workflow requires `image.tag` and `image.digest`, then verifies the digest against ECR before producing outputs.
