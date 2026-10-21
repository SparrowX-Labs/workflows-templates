# Publish Image Metadata

Reusable workflow: `.github/workflows/publish-image-metadata.yaml`

## Purpose

Publishes immutable deployment metadata for an image to AWS Systems Manager Parameter Store. The metadata is later consumed by image resolution and promotion/rollback workflows.

## Usage

```yaml
jobs:
  publish-image-metadata:
    uses: SparrowX-ECS/workflows-templates/.github/workflows/publish-image-metadata.yaml@<version>
    with:
      ssm-parameter-path: ${{ vars.DEV_DEPLOYED_PARAM_STORE_PATH }}
      env-params-file: ecs-parameters-dev.yaml
      image-tag: ${{ needs.build.outputs.image-tag }}
      image-uri: ${{ needs.sign-image.outputs.image-uri }}
```

## Inputs

| Input | Required | Expected values / description |
|---|---:|---|
| `ssm-parameter-path` | yes | SSM String parameter name to create or overwrite |
| `env-params-file` | yes | ECS parameters file used to resolve service and repository |
| `image-tag` | yes | Image tag being recorded |
| `image-uri` | yes | Immutable `registry/repository@sha256:digest` URI |

## Outputs

No reusable-workflow outputs are exposed.

## JSON parameter format

The workflow writes this JSON document as the SSM parameter value:

```json
{
  "service": "customer-api",
  "environment": "dev",
  "image": {
    "repository": "namespace/repository",
    "tag": "commit-sha",
    "digest": "sha256:...",
    "uri": "registry/namespace/repository@sha256:..."
  },
  "source": {
    "commit": "commit-sha",
    "repository": "owner/repository",
    "workflow": "CI Pipeline",
    "runId": "123456789"
  },
  "security": {
    "scanner": "trivy",
    "status": "passed"
  }
}
```

`service`, `environment`, and all `image` fields identify the deployed artifact. `source` records provenance. `security` records the scanner label and security status stored with the deployment metadata.
