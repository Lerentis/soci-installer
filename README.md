# soci-installer GitHub Action

This Action installs the SOCI binary during a GitHub Actions run.

## Usage

This action currently supports GitHub-provided Linux, runners (self-hosted runners may or may not work).
MacOS and Windows binaries are currently not build, therfore this action can not install those.

Add the following entry to your Github workflow YAML file:

```yaml
uses: lerentis/soci-installer@v1.0.0
with:
  soci-release: 'v0.3.0' # optional
```

Example using a pinned version:

```yaml
jobs:
  example:
    runs-on: ubuntu-latest

    permissions: {}

    name: Install soci
    steps:
      - name: Install soci
        uses: lerentis/soci-installer@v1.0.0
        with:
          soci-release: 'v0.3.0'
      - name: Check install!
        run: soci --version
```

Example using the default version:

```yaml
jobs:
  example:
    runs-on: ubuntu-latest

    permissions: {}

    name: Install soci
    steps:
      - name: Install soci
        uses: lerentis/soci-installer@v1.0.0
      - name: Check install!
        run: soci --version
```

This action does not need any GitHub permission to run, however, if your workflow needs to update, create or perform any
action against your repository, then you should change the scope of the permission appropriately.

For example, if you are using the `gcr.io` as your registry to push the images you will need to give the `write` permission
to the `packages` scope.

Example of a simple workflow:

```yaml
jobs:
  build-image:
    runs-on: ubuntu-latest

    permissions:
      contents: read
      packages: write
      id-token: write # needed for signing the images with GitHub OIDC Token

    name: build-image
    steps:
      - uses: actions/checkout@v3.5.2
        with:
          fetch-depth: 1

      - name: Install soci
        uses: lerentis/soci-installer@v1.0.0

      - name: Set up QEMU
        uses: docker/setup-qemu-action@v2.1.0

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2.5.0

      - name: Login to GitHub Container Registry
        uses: docker/login-action@v2.1.0
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - id: docker_meta
        uses: docker/metadata-action@v4.4.0
        with:
          images: ghcr.io/lerentis/test
          tags: type=sha,format=long

      - name: Build and Push container images
        uses: docker/build-push-action@v4.0.0
        id: build-and-push
        with:
          platforms: linux/amd64,linux/arm/v7,linux/arm64
          push: true
          tags: ${{ steps.docker_meta.outputs.tags }}

      - name: Create and push soci index
        run: |
          soci create "${TAGS}"
          soci push "${TAGS}"
        env:
          TAGS: ${{ steps.docker_meta.outputs.tags }}

```

### Optional Inputs
The following optional inputs:

| Input | Description |
| --- | --- |
| `soci-release` | `soci` version to use instead of the default. |
| `install-dir` | directory to place the `soci` binary into instead of the default (`$HOME/.soci`). |
