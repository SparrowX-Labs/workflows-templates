# Image Security Scan

Reusable workflow: `.github/workflows/image-security-scan.yaml`

## Purpose

Scans an immutable commit-tagged ECR image with Trivy and generates a CycloneDX JSON SBOM. Vulnerability findings are reported; `ci-quality-gate.yaml` decides whether they block CI.

## Usage

```yaml
jobs:
  image-security-scan:
    needs: build
    uses: SparrowX-ECS/workflows-templates/.github/workflows/image-security-scan.yaml@<version>
    with:
      env-params-file: ecs-parameters-dev.yaml
      image-tag: ${{ needs.build.outputs.image-tag }}
```

## Inputs

| Input | Required | Description |
|---|---:|---|
| `env-params-file` | yes | ECS parameter file containing `.image.nameSpace` and `.image.repository` |
| `image-tag` | yes | Immutable commit-based ECR tag |

The caller must provide `AWS_REGION`, `AWS_ACCOUNT_ID`, and `AWS_ROLE_NAME` repository or organization variables. The configured OIDC role needs access to pull the image from ECR.

## Outputs

| Output | Description |
|---|---|
| `artifact-name` | Uploaded artifact containing the three image-security JSON reports |

Artifact contents:

```text
combined-image-security-scan-results.json
trivy-results.json
sbom.cdx.json
```

## Combined JSON format

```json
{
  "schemaVersion": 1,
  "status": "findings",
  "image": {"tag": "abc1234"},
  "imageScan": {
    "status": "findings",
    "total": 3,
    "critical": 1,
    "high": 2,
    "report": "trivy-results.json"
  },
  "sbom": {
    "status": "generated",
    "format": "cyclonedx-json",
    "components": 47,
    "report": "sbom.cdx.json"
  }
}
```

The vulnerability scan includes OS and library vulnerabilities with `CRITICAL,HIGH` severity and unfixed findings. The image and overall statuses are `passed`, `findings`, or `error`; SBOM status is `generated` or `error`.

Example raw Trivy vulnerability result:

```json
{
  "Results": [
    {
      "Target": "alpine:3.20",
      "Type": "alpine",
      "Vulnerabilities": [
        {
          "VulnerabilityID": "CVE-2025-0001",
          "PkgName": "example-package",
          "InstalledVersion": "1.0.0",
          "FixedVersion": "1.0.1",
          "Severity": "HIGH"
        }
      ]
    }
  ]
}
```

Example CycloneDX result:

```json
{
  "bomFormat": "CycloneDX",
  "specVersion": "1.5",
  "components": [
    {
      "type": "library",
      "name": "example-package",
      "version": "1.0.0",
      "purl": "pkg:pypi/example-package@1.0.0"
    }
  ]
}
```
