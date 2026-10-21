# Copy Image

Reusable workflow: `.github/workflows/copy-image.yaml`

## Purpose

Copies an image by digest from a source ECR repository to a target ECR repository, copies its Cosign signature referrers, and verifies that the target tag resolves to the expected digest.

## Usage

```yaml
jobs:
  copy-image:
    needs: resolve-image
    uses: SparrowX-ECS/workflows-templates/.github/workflows/copy-image.yaml@<version>
    with:
      image-tag: ${{ needs.resolve-image.outputs.image-tag }}
      image-digest: ${{ needs.resolve-image.outputs.image-digest }}
      source-env-params-file: ecs-parameters-dev.yaml
      target-env-params-file: ecs-parameters-prod.yaml
```

## Inputs

| Input | Required | Expected values / description |
|---|---:|---|
| `image-tag` | yes | Image tag to copy |
| `image-digest` | yes | Expected immutable `sha256:<digest>` |
| `source-env-params-file` | yes | ECS parameters file for the source ECR repository |
| `target-env-params-file` | yes | ECS parameters file for the target ECR repository |

## Outputs

| Output | Description |
|---|---|
| `image-tag` | Copied image tag |
| `image-uri` | Target immutable URI in `registry/repository@sha256:digest` format |

## Reports and summary

No JSON report is generated. The summary records source and target repositories, tag, digest, whether the target tag already existed, and the job result.
