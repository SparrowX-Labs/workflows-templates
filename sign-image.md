# Sign Image

Reusable workflow: `.github/workflows/sign-image.yaml`

## Purpose

Signs an ECR container image with keyless Cosign signing. The workflow uses GitHub Actions OIDC with Sigstore, so no private signing key is stored in GitHub secrets. The signature is published as an OCI artifact associated with the image in ECR.

## Usage

Call it after the image has been built and pushed and after CI security checks pass:

```yaml
jobs:
  sign-image:
    needs: [build, image-security-scan, ci-quality-gate]
    uses: SparrowX-ECS/workflows-templates/.github/workflows/sign-image.yaml@<version>
    with:
      env-params-file: ecs-parameters-dev.yaml
      image-tag: ${{ needs.build.outputs.image-tag }}
```

## Inputs

| Input | Required | Description |
|---|---:|---|
| `env-params-file` | yes | ECS parameter file containing `.image.nameSpace` and `.image.repository` |
| `image-tag` | yes | Immutable commit-based ECR tag to sign |

The caller must provide AWS repository/organization variables `AWS_REGION`, `AWS_ACCOUNT_ID`, and `AWS_ROLE_NAME`. The OIDC role must be able to authenticate to and push OCI signature artifacts to ECR.

The workflow logs in to ECR, resolves the tag to its current digest, and signs the resulting immutable `registry/repository@sha256:digest` reference. A tag is used only to look up the digest; Cosign never signs the mutable tag reference.

## Outputs

| Output | Values | Description |
|---|---|---|
| `status` | `passed`, `failed` | Whether Cosign successfully published the image signature |

No JSON file is generated. The signature is stored in ECR and the workflow summary records the image and result.

The summary has this shape:

```text
## Image Signing
| Item | Value |
| Tool | Cosign keyless signing |
| Image | registry/repository@sha256:digest |
| Digest | sha256:digest |
| Signature storage | OCI signature in Amazon ECR |
| Result | passed or failed |
```

## Signing model

Cosign keyless signing obtains an ephemeral certificate from Sigstore Fulcio and records transparency-log evidence in Rekor. The signing identity is later constrained during verification. The signing command is:

```bash
cosign sign --yes <registry/repository@sha256:digest>
```

The workflow uses `sigstore/cosign-installer@v4.1.2` and does not run Docker-in-Docker.
