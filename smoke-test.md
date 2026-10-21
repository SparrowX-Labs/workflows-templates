# Smoke Test

Reusable workflow: `.github/workflows/smoke-test.yaml`

## Purpose

Polls the configured deployed-service health endpoint until it returns a 2xx response or the timeout expires. A failed HTTP check is reported through outputs without failing the test job, allowing a later quality gate to decide the deployment result.

## Usage

```yaml
jobs:
  smoke-test:
    needs: deploy
    uses: SparrowX-ECS/workflows-templates/.github/workflows/smoke-test.yaml@<version>
    with:
      env-params-file: ecs-parameters-dev.yaml
      base-url: ${{ vars.DEV_BASE_URL }}
      timeout: '120'
```

## Inputs

| Input | Required | Default | Expected values / description |
|---|---:|---|---|
| `env-params-file` | yes | — | ECS parameters file containing `.loadBalancer.smokeTestPath` |
| `base-url` | yes | — | Service origin without the configured smoke-test path |
| `timeout` | no | `120` | Maximum wait in seconds |

## Outputs

| Output | Values | Description |
|---|---|---|
| `status` | `passed`, `failed` | Whether the endpoint eventually returned 2xx |
| `passed` | `1`, `0` | Number of smoke checks passed |
| `total` | `1` | Number of smoke checks |

## Reports and summary

No JSON report is generated. The summary records endpoint, timeout, method, status, and `passed/total`.
