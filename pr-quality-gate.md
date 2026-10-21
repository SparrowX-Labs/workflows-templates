# PR Quality Gate

Reusable workflow: `.github/workflows/pr-quality-gate.yaml`

## Purpose

Evaluates source-security reports for pull requests. This gate covers secret scanning and SAST. Image vulnerabilities and SBOM requirements are evaluated later by [`ci-quality-gate.yaml`](ci-quality-gate.md) for the authoritative image build.

The gate fails for an invalid or missing policy, missing or invalid source reports, scanner errors, secret findings when configured to block them, or SAST findings above the configured maximum.

## Usage

```yaml
jobs:
  pr-quality-gate:
    if: always()
    needs: source-security-scan
    uses: SparrowX-ECS/workflows-templates/.github/workflows/pr-quality-gate.yaml@<version>
    with:
      source-security-artifact: ${{ needs.source-security-scan.outputs.artifact-name }}
      security-policy-file: app-sec-policy.yaml
```

The policy file must exist in the application repository because this workflow checks out the repository before reading it.

## Inputs

| Input | Required | Description |
|---|---:|---|
| `source-security-artifact` | yes | Artifact name returned by `source-security-scan.yaml` |
| `security-policy-file` | yes | Repository path to the application security policy |

## Outputs

| Output | Values | Description |
|---|---|---|
| `status` | `passed`, `failed` | Overall PR source-security quality-gate decision |

## Policy values

```yaml
secret_scanning:
  block_on_findings: true
sast:
  max_findings: 0
```

The workflow validates both values before evaluating reports. `block_on_findings` must be boolean and `max_findings` must be a non-negative integer.

## Consumed artifact and JSON schemas

The source-security artifact must contain:

```text
combined-source-security-scan-results.json
trivy-secret-results.json
semgrep-results.json
```

The combined report must provide this shape:

```json
{
  "schemaVersion": 1,
  "status": "passed",
  "secretScanning": {
    "status": "passed",
    "findings": 0
  },
  "sast": {
    "status": "passed",
    "findings": 0
  }
}
```

The gate evaluates the combined report. Secret findings block the gate only when `secret_scanning.block_on_findings` is `true`; SAST findings block it when their count exceeds `sast.max_findings`. The raw Trivy and Semgrep files are included in the artifact and linked in the summary for investigation.

## Summary

The workflow writes `## PR Security Quality Gate` to `$GITHUB_STEP_SUMMARY`, including:

1. Overall PR-gate result.
2. Failure reasons, when applicable.
3. Secret scanning and SAST statuses and finding counts.
4. Links to the raw and combined reports in the uploaded artifact.

No new report file is created by the gate itself. Its internal `quality-gate-results` directory is temporary job state only.
