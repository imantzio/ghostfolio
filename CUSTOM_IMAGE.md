# Custom Ghostfolio application image

This repository is the application/image repository. It contains upstream
Ghostfolio, the local precise-quantity customization, the Dockerfile, and the
GitHub Actions workflow that publishes the customized image. Raspberry Pi host
configuration belongs in the separate `smart-house` repository.

## Branches and releases

- `custom` is the stable customized application branch.
- `build/multiarch-image` prepares AMD64 and ARM64 image publication.
- Release tags use `<upstream-version>-custom.<revision>`, for example
  `3.72.0-custom.4`.

Pushing a release tag invokes `.github/workflows/publish-custom-image.yml` and
publishes `ghcr.io/imantzio/ghostfolio:<tag>` for both `linux/amd64` and
`linux/arm64`. The build stage runs on `$TARGETPLATFORM`, so Prisma and native
dependencies are present for the final architecture rather than downloaded at
container startup.

The deployment repository references a release tag; it does not copy this
source tree or build the application on the Raspberry Pi.
