# DAST

Reusable workflow: `.github/workflows/dast.yaml`

## Purpose

Runs a controlled OWASP ZAP scan against an already deployed service. The required `runtime` input selects the scan type:

- `python`: ZAP API scan against the service OpenAPI document, defaulting to `/openapi.json`.
- `node`: ZAP baseline web scan against the supplied base URL. This is the lower-impact passive path.

The workflow is intended for a development environment after deployment and smoke testing. The Python API scan may actively test endpoints, so only targets owned or explicitly authorized by the application team should be scanned.

## Usage

Python API:

```yaml
jobs:
  dast:
    needs: [deploy, smoke-test]
    uses: SparrowX-ECS/workflows-templates/.github/workflows/dast.yaml@<version>
    with:
      runtime: python
      base-url: ${{ vars.DEV_BASE_URL }}
      api-spec-path: /openapi.json
```

Node web application:

```yaml
jobs:
  dast:
    needs: [deploy, smoke-test]
    uses: SparrowX-ECS/workflows-templates/.github/workflows/dast.yaml@<version>
    with:
      runtime: node
      base-url: ${{ vars.DEV_BASE_URL }}
```

## Inputs

| Input | Required | Default | Description |
|---|---:|---|---|
| `runtime` | yes | — | `python` or `node`; selects the ZAP scan mode |
| `base-url` | yes | — | Deployed application URL, without a required trailing slash |
| `api-spec-path` | no | `/openapi.json` | OpenAPI path appended to `base-url` for Python API scans |

## Outputs

| Output | Description |
|---|---|
| `artifact-name` | Uploaded artifact containing normalized JSON, raw ZAP JSON, and an HTML report |
| `findings` | Number of findings in the normalized ZAP report |
| `scan-status` | `passed`, `findings`, or `error` |

Artifact contents:

```text
dast-reports/
├── combined-dast-results.json
├── zap-results.json
└── zap-report.html
```

## Combined report format

`combined-dast-results.json` is the stable downstream contract:

```json
{
  "schemaVersion": 1,
  "status": "findings",
  "runtime": "python",
  "scan": {
    "mode": "ZAP API scan",
    "target": "https://dev.example.com/openapi.json",
    "findings": 2,
    "high": 1,
    "medium": 1,
    "low": 0,
    "informational": 0,
    "report": "zap-results.json",
    "htmlReport": "zap-report.html"
  }
}
```

`status` is `passed`, `findings`, or `error`. ZAP risk codes are normalized as HIGH, MEDIUM, LOW, and informational counts. The raw report follows ZAP’s JSON report format and the HTML report is provided for human review.

## Tool and runtime behavior

The workflow uses the pinned ZAP action versions in `dast.yaml`. Python uses the ZAP API scan with OpenAPI input; Node uses the ZAP baseline scan against the web URL. Findings are report-only in this workflow; [`dev-quality-gate.yaml`](dev-quality-gate.md) applies blocking thresholds. API testing is a separate workflow documented in [`api-test.md`](api-test.md).
