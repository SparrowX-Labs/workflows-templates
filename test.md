# Test

Reusable workflow: `.github/workflows/test.yaml`

## Purpose

Runs the application’s repository test suite for Python or Node projects. Python services can optionally run against a temporary PostgreSQL container.

## Usage

```yaml
jobs:
  test:
    uses: SparrowX-ECS/workflows-templates/.github/workflows/test.yaml@<version>
    with:
      env-params-file: ecs-parameters-dev.yaml
      runtime: python
      database: true
```

## Inputs

| Input | Required | Default | Expected values / description |
|---|---:|---|---|
| `env-params-file` | yes | — | ECS parameters file; Python database tests read `.database.name` |
| `runtime` | yes | — | `python` or `node` |
| `python-version` | no | `3.12` | Python runtime version |
| `node-version` | no | `22` | Node.js runtime version |
| `database` | no | `false` | Boolean; starts PostgreSQL 16 for Python tests |

## Outputs

No reusable-workflow outputs are exposed.

## Reports and summary

No JSON report is generated. Test results are emitted by the selected test runner in the job log. Python runs `pytest tests`; Node runs `npm test -- --run`.
