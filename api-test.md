# Deployed API Test

Reusable workflow: `.github/workflows/api-test.yaml`

## Purpose

Runs repository-owned API tests against a deployed service. Test failures are reported through outputs and the job summary; the test command itself does not fail the job. A caller can use the `status` output in a later quality gate, which can classify the result as blocking or advisory with its `api-test-blocking` input.

## Usage

```yaml
jobs:
  api-test:
    needs: smoke-test
    uses: SparrowX-ECS/workflows-templates/.github/workflows/api-test.yaml@<version>
    with:
      runtime: python
      base-url: ${{ vars.DEV_BASE_URL }}
      test-path: api-tests/dev
      test-command: pytest -q api-tests/dev
      requirements-file: requirements-dev.txt
```

## Inputs

| Input | Required | Default | Expected values / description |
|---|---:|---|---|
| `base-url` | yes | — | Deployed service origin, including protocol and domain |
| `runtime` | yes | — | `python` or `node` |
| `test-command` | yes | — | Repository-owned command that runs the deployed tests |
| `test-path` | yes | — | Test path shown in the summary |
| `python-version` | no | `3.12` | Python version when `runtime` is `python` |
| `node-version` | no | `22` | Node.js version when `runtime` is `node` |
| `requirements-file` | no | `requirements-dev.txt` | Python dependency file |

## Outputs

| Output | Values | Description |
|---|---|---|
| `status` | `passed`, `failed` | Test command result |
| `passed` | non-negative integer | Number of tests that passed |
| `total` | non-negative integer | Total parsed test count |

For Python, counts are parsed from pytest terminal output. The total includes passed, failed, error, skipped, xfailed, and xpassed tests. Node output supports common `passed`/`failed`/`total` summary formats.

## Reports and summary

No JSON report is generated. The workflow writes `## Deployed API Tests` to `$GITHUB_STEP_SUMMARY`, including runtime, URL, test path, status, and `passed/total`.
