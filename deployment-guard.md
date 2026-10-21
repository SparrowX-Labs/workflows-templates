# Deployment Guard

Reusable workflow: `.github/workflows/deployment-guard.yaml`

## Purpose

Reads application and infrastructure deployment-state flags and determines whether deployment is allowed for the selected environment.

## Usage

```yaml
jobs:
  deployment-guard:
    uses: SparrowX-ECS/workflows-templates/.github/workflows/deployment-guard.yaml@<version>
    with:
      env-params-file: ecs-parameters-dev.yaml
      environment: dev
```

## Inputs

| Input | Required | Expected values / description |
|---|---:|---|
| `env-params-file` | yes | Application ECS parameters file containing `.appStack.state` |
| `environment` | yes | Environment name used to locate the infrastructure parameters file, normally `dev` or `prod` |

## Outputs

| Output | Values | Description |
|---|---|---|
| `deployment-enabled` | `true`, `false` | `true` only when both application and root stack states are `enabled` |

The accepted state values are `enabled` and `disabled`. Invalid values fail the guard.

## Reports and summary

No JSON report is generated. The summary records the environment, application state, root-stack state, and deployment decision.
