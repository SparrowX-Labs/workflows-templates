# CI Quality Gate

Reusable workflow: `.github/workflows/ci-quality-gate.yaml`

## Purpose

Evaluates the image-security reports produced for an authoritative CI image. This gate covers image vulnerabilities and SBOM generation. Source controls such as secret scanning and SAST are evaluated by [`pr-quality-gate.yaml`](pr-quality-gate.md).

The gate fails for an invalid or missing policy, missing or invalid image reports, scanner errors, image findings above policy limits, report count mismatches, or a missing required SBOM.

## Usage

```yaml
jobs:
  ci-quality-gate:
    if: always()
    needs: image-security-scan
    uses: SparrowX-ECS/workflows-templates/.github/workflows/ci-quality-gate.yaml@<version>
    with:
      image-security-artifact: ${{ needs.image-security-scan.outputs.artifact-name }}
      security-policy-file: app-sec-policy.yaml
```

The policy file must exist in the application repository because this workflow checks out the repository before reading it.

## Inputs

| Input | Required | Description |
|---|---:|---|
| `image-security-artifact` | yes | Artifact name returned by `image-security-scan.yaml` |
| `security-policy-file` | yes | Repository path to the application security policy |

## Outputs

| Output | Values | Description |
|---|---|---|
| `status` | `passed`, `failed` | Overall image-security quality-gate decision |

## Policy values

```yaml
image_scan:
  max_critical: 0
  max_high: 0
sbom:
  required: true
```

The workflow validates all four values before evaluating reports. `max_critical` and `max_high` must be non-negative integers. `sbom.required` must be boolean.

## Consumed artifact and JSON schemas

The image-security artifact must contain:

```text
combined-image-security-scan-results.json
trivy-results.json
sbom.cdx.json
```

The combined report must provide this shape:

```json
{
  "schemaVersion": 1,
  "status": "passed",
  "imageScan": {
    "status": "passed",
    "total": 0,
    "critical": 0,
    "high": 0
  },
  "sbom": {
    "status": "generated",
    "components": 47
  }
}
```

The gate recalculates total, CRITICAL, and HIGH vulnerability counts from `trivy-results.json` and rejects mismatches. It also requires valid JSON in the raw Trivy report and CycloneDX SBOM.

## Summary

The workflow writes `## CI Security Quality Gate` to `$GITHUB_STEP_SUMMARY`, including:

1. Overall and image-gate results.
2. Failure reasons, when applicable.
3. Image scan status and vulnerability counts.
4. SBOM status and component count.
5. Links to the raw and combined reports in the uploaded artifact.

No new report file is created by the gate itself. Its internal `quality-gate-results` directory is temporary job state only.
