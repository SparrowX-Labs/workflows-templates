# Dev Quality Gate

Reusable workflow: `.github/workflows/dev-quality-gate.yaml`

## Purpose

Downloads the artifact from `dast.yaml`, validates the normalized and raw ZAP reports, applies repository policy thresholds to HIGH and MEDIUM DAST findings, and evaluates the deployed smoke and API-test statuses. API tests are blocking by default, but callers can mark them advisory when they depend on external microservices.

## Usage

```yaml
jobs:
  dev-quality-gate:
    if: always()
    needs: dast
    uses: SparrowX-ECS/workflows-templates/.github/workflows/dev-quality-gate.yaml@<version>
    with:
      dast-artifact: ${{ needs.dast.outputs.artifact-name }}
      security-policy-file: app-sec-policy.yaml
      smoke-test-status: ${{ needs.smoke-test.outputs.status }}
      api-test-status: ${{ needs.api-test.outputs.status }}
      api-test-blocking: false
```

## Inputs

| Input | Required | Description |
|---|---:|---|
| `dast-artifact` | yes | Artifact name returned by `dast.yaml` |
| `security-policy-file` | yes | Repository path to the DAST policy |
| `smoke-test-status` | yes | Status returned by `smoke-test.yaml` |
| `api-test-status` | yes | Status returned by `api-test.yaml` |
| `api-test-blocking` | no | Boolean; defaults to `true`. When `false`, a failed API test is reported as an advisory warning and does not fail the gate. |

## Outputs

| Output | Values | Description |
|---|---|---|
| `status` | `passed`, `failed` | Overall deployment and DAST quality-gate decision |

## Policy format

```yaml
dast:
  max_high: 0
  max_medium: 0
```

Both values must be non-negative integers. Findings at or below the configured thresholds pass; findings above a threshold fail the gate. Missing or invalid policy values fail the gate.

## Consumed reports

The DAST artifact must contain:

```text
combined-dast-results.json
zap-results.json
zap-report.html
```

The gate validates the two JSON files and recalculates total, HIGH, and MEDIUM counts from `zap-results.json` to detect mismatches in the normalized report. Smoke tests remain blocking. API tests are blocking when `api-test-blocking` is `true`; otherwise their status remains visible in the summary as an advisory warning.

## Summary

The workflow writes `## Dev Quality Gate` to `$GITHUB_STEP_SUMMARY`, including the gate result, blocking/advisory mode, finding counts, advisory warnings, failure reasons, and a link to the report artifact. It does not create a new report file.
