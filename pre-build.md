# Pre-Build

Reusable workflow: `.github/workflows/pre-build.yaml`

## Purpose

Determines whether a new image is required before the build stage. Source changes require a build; when code is unchanged, the workflow checks whether the currently deployed ECS image still exists in ECR.

## Usage

```yaml
jobs:
  pre-build:
    uses: SparrowX-ECS/workflows-templates/.github/workflows/pre-build.yaml@<version>
    with:
      env-params-file: ecs-parameters-dev.yaml
```

## Inputs

| Input | Required | Expected values / description |
|---|---:|---|
| `env-params-file` | yes | ECS parameters file containing service and image configuration |

## Outputs

| Output | Values | Description |
|---|---|---|
| `build-needed` | `true`, `false` | Whether the caller should run a new image build |

The job also calculates an internal deployment filter value, but it is not exposed as a reusable-workflow output.

## Reports and summary

No JSON report is generated. The summary records whether code changed, whether a build is needed, and the reason.
