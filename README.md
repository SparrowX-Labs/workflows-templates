# SparrowX reusable workflows

Reusable GitHub Actions workflows for the SparrowX Kubernetes platform. Application repositories use these workflows for validation, image supply-chain controls, Argo CD synchronization, and post-deployment checks.

## Workflow catalog

The current implementation contains these 18 reusable workflows:

| Workflow | Purpose |
| --- | --- |
| `pre-build.yaml` | Detects whether a new image is required. |
| `test.yaml` | Runs Python or Node tests, optionally with PostgreSQL. |
| `source-security-scan.yaml` | Runs secret scanning and Semgrep SAST. |
| `pr-quality-gate.yaml` | Evaluates pull-request source-security results. |
| `build.yaml` | Builds and optionally publishes an immutable Git-SHA image. |
| `image-security-scan.yaml` | Scans images and generates an SBOM. |
| `ci-quality-gate.yaml` | Evaluates image-security results. |
| `sign-image.yaml` / `verify-image-signature.yaml` | Signs and verifies Cosign/Sigstore attestations. |
| `attest-image.yaml` | Publishes image provenance attestation. |
| `promote-image.yaml` / `copy-image.yaml` | Promotes an image between ECR environments by digest. |
| `argosync.yaml` | Waits for and synchronizes an Argo CD application. |
| `smoke-test.yaml` / `api-test.yaml` | Validates the deployed service. |
| `dast.yaml` | Runs the configured OWASP ZAP scan. |
| `dev-quality-gate.yaml` / `prod-quality-gate.yaml` | Makes environment deployment quality decisions. |

The Markdown files in this repository document workflow contracts. The active contract set matches the workflow files above; older standalone deploy, deployment-guard, image-metadata, and image-resolution workflows were consolidated or removed and are not part of the current interface.

## Delivery flow

```text
pull request → test → source scan → build → PR quality gate
main         → build → image scan → CI gate → sign → attest → promote metadata
development  → verify → Argo CD sync → smoke/API/DAST → dev quality gate
production   → copy by digest → verify → Argo CD sync → smoke/API tests → prod quality gate
```

This is **Build Once, Promote Many**. The production deployment uses the image digest validated in development. The application repositories keep environment-specific Helm values for `dev` and `prod`, while Argo CD applies them as namespaces on the same EKS cluster.

## Consumers

The templates are used by the Python APIs, `web-portal`, `backstage`, and `backstage-pyactions`. The scaffolding templates in `backstage-pyactions` are the source of the standard workflow composition for newly generated services.

## Application variables and secrets

Application repositories inherit shared organization configuration. Each application only needs to define `HEALTH_CHECK_PATH`, the public endpoint used by deployment checks, and may define a repository-level `AWS_ROLE_NAME` to override the shared default IAM role. Shared organization variables are `AWS_ACCOUNT_ID`, `AWS_REGION`, `DEV_BASE_URL`, and `PROD_BASE_URL`.

`ARGOCD_SERVER` and `ARGOCD_AUTH_TOKEN` are organization secrets granted to the ten selected repositories that use them: `EKS-HelmCharts`, `crossplane`, `customer-api`, `notification-api`, `task-api`, `billing-api`, `reporting-api`, `web-portal`, `backstage`, and `backstage-pyactions`. The platform Helm workflows additionally consume `ARGOCD_ADMIN_PASSWORD_HASH`, `ARGOCD_ADMIN_PASSWORD_MTIME`, and `GRAFANA_ADMIN_PASSWORD`.

## License

This is a proprietary portfolio project. See [LICENSE.md](LICENSE.md).
