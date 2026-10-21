# Production Quality Gate

Reusable workflow: `.github/workflows/prod-quality-gate.yaml`

## Purpose

Evaluates production smoke-test and deployed API-test statuses. The gate itself does not fail its job; it reports a decision so the caller can publish successful deployment metadata or start a rollback. API tests are blocking by default, but callers can mark them advisory when they depend on external microservices.

## Usage

```yaml
jobs:
  prod-quality-gate:
    needs: [smoke-test, api-test]
    if: always()
    uses: SparrowX-ECS/workflows-templates/.github/workflows/prod-quality-gate.yaml@<version>
    with:
      smoke-test-status: ${{ needs.smoke-test.outputs.status }}
      api-test-status: ${{ needs.api-test.outputs.status }}
      api-test-blocking: false
```

## Inputs

| Input | Required | Expected values / description |
|---|---:|---|
| `smoke-test-status` | yes | `passed` or `failed` from `smoke-test.yaml` |
| `api-test-status` | yes | `passed` or `failed` from `api-test.yaml` |
| `api-test-blocking` | no | Boolean; defaults to `true`. When `false`, a failed API test is reported as an advisory warning and does not make the gate `failed`. |

## Outputs

| Output | Values | Description |
|---|---|---|
| `status` | `passed`, `failed` | `passed` only when both input statuses equal `passed` |

## Reports and summary

No JSON report is generated. The summary lists both input statuses, whether API tests are blocking or advisory, and the resulting gate decision. The workflow intentionally exits successfully for either decision; downstream jobs must branch on `needs.prod-quality-gate.outputs.status`. Smoke tests always remain blocking.
