# Deploy

Reusable workflow: `.github/workflows/deploy.yaml`

## Purpose

Deploys a specified image tag to the shared ECS/CloudFormation service stack. It loads service and optional database configuration, applies the infrastructure template, waits for deployment stability, and records the deployment result.

## Usage

```yaml
jobs:
  deploy:
    uses: SparrowX-ECS/workflows-templates/.github/workflows/deploy.yaml@<version>
    with:
      environment: dev
      database-enabled: true
      env-params-file: ecs-parameters-dev.yaml
      image-tag: ${{ needs.build.outputs.image-tag }}
```

## Inputs

| Input | Required | Default | Expected values / description |
|---|---:|---|---|
| `environment` | yes | — | GitHub deployment environment, normally `dev` or `prod` |
| `database-enabled` | no | `false` | Boolean; enables database output discovery and configuration |
| `env-params-file` | yes | — | ECS service parameters file |
| `image-tag` | yes | — | Image tag to deploy |

## Outputs

No reusable-workflow outputs are exposed.

## Reports and summary

No JSON report is generated. Deployment configuration, CloudFormation/ECS progress, and the final deployment result are written to the job log and summary.
