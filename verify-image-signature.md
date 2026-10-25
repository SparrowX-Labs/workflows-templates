# Verify Image Signature

Reusable workflow: `.github/workflows/verify-image-signature.yaml`

## Purpose

Verifies that an ECR image has a valid Cosign keyless signature and that the signature certificate matches the expected identity and OIDC issuer. This workflow is intended for CD before deployment or promotion.

## Usage

```yaml
jobs:
  verify-image-signature:
    uses: SparrowX-ECS/workflows-templates/.github/workflows/verify-image-signature.yaml@<version>
    with:
      env-params-file: helm-values-dev.yaml
      image-tag: dev-candidate
      certificate-identity-regexp: >-
        https://github.com/SparrowX-ECS/.*/.github/workflows/.*@refs/heads/main
```

The workflow resolves the supplied tag to its current ECR digest and verifies the immutable `registry/repository@sha256:digest` URI. It does not verify a mutable tag directly.

The identity expression must be restricted to the trusted organization, repository, and workflow that are allowed to sign images. Do not use `.*` for the entire identity in a production caller.

## Inputs

| Input | Required | Description |
|---|---:|---|
| `env-params-file` | yes | Application values file containing `projectName`, `environment`, and `serviceName` |
| `image-tag` | yes | ECR image tag to resolve and verify |
| `certificate-identity-regexp` | yes | Expected Sigstore certificate identity regular expression |
| `certificate-oidc-issuer` | no | Expected issuer; defaults to `https://token.actions.githubusercontent.com` |

The caller must provide AWS repository/organization variables `AWS_REGION`, `AWS_ACCOUNT_ID`, and `AWS_ROLE_NAME`. The AWS role must be able to pull the image and its OCI signature artifact from ECR.

## Outputs

| Output | Values | Description |
|---|---|---|
| `status` | `passed`, `failed` | Whether the signature and certificate constraints verified successfully |

No JSON file is generated. Cosign verification output is written to the workflow log and the summary records the verification constraints and result.

The summary has this shape:

```text
## Image Signature Verification
| Item | Value |
| Image | registry/repository@sha256:digest |
| Certificate identity | configured regular expression |
| OIDC issuer | configured issuer |
| Result | passed or failed |
```

## Verification command

The workflow executes:

```bash
aws ecr describe-images --repository-name <repository> --image-ids imageTag=<tag>
cosign verify <registry/repository@sha256:digest> \
  --certificate-identity-regexp '<trusted-identity-regexp>' \
  --certificate-oidc-issuer 'https://token.actions.githubusercontent.com'
```

Verification fails if the image has no signature, the signature is invalid, the certificate identity does not match, or the OIDC issuer is unexpected.
