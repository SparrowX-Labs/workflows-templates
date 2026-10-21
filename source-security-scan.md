# Source Security Scan

Reusable workflow: `.github/workflows/source-security-scan.yaml`

## Purpose

Combines Trivy secret scanning and runtime-aware Semgrep SAST into one reusable workflow. Findings are reported and uploaded; blocking decisions are made by `pr-quality-gate.yaml`.

## Usage

```yaml
jobs:
  source-security-scan:
    uses: SparrowX-ECS/workflows-templates/.github/workflows/source-security-scan.yaml@<version>
    with:
      runtime: python
```

## Inputs

| Input | Required | Values / description |
|---|---:|---|
| `runtime` | yes | `python` or `node`; selects the Semgrep rules |

Python rules: `p/owasp-top-ten`, `p/security-audit`, `p/python`, `p/fastapi`, `p/sql-injection`.

Node rules: `p/owasp-top-ten`, `p/react`, `p/typescript`, `p/javascript`.

## Outputs

| Output | Description |
|---|---|
| `artifact-name` | Uploaded artifact containing the three source-security JSON reports |

Artifact contents:

```text
combined-source-security-scan-results.json
trivy-secret-results.json
semgrep-results.json
```

## Combined JSON format

```json
{
  "schemaVersion": 1,
  "status": "findings",
  "secretScanning": {
    "status": "passed",
    "findings": 0,
    "report": "trivy-secret-results.json"
  },
  "sast": {
    "status": "findings",
    "findings": 2,
    "report": "semgrep-results.json"
  }
}
```

The overall and scanner statuses are `passed`, `findings`, or `error`. The raw Trivy report uses `.Results[].Secrets[]`; the raw Semgrep report uses `.results[]`.

Example raw Trivy result:

```json
{
  "Results": [
    {
      "Target": "src/config.py",
      "Secrets": [{"RuleID": "aws-access-key-id", "Category": "AWS"}]
    }
  ]
}
```

Example raw Semgrep result:

```json
{
  "version": "1.0.0",
  "results": [
    {
      "check_id": "python.lang.security.audit",
      "path": "app/main.py",
      "extra": {"severity": "WARNING", "message": "Example finding"}
    }
  ],
  "errors": []
}
```
