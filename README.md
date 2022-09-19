# vSign GitHub Action

This action enables you to sign and verify container images using `vsign`.
`vsign` verifies the integrity of the `vsign` release during installation.

## Usage

This action currently supports GitHub-provided Linux, macOS and Windows runners (self-hosted runners may not work).

Add the following entry to your Github workflow YAML file:

```yaml
uses: zosocanuck/vsign-action@main
with:
  vsign-release: 'v0.1.0' # optional
```

Example using a pinned version:

```yaml
jobs:
  test_vsign_action:
    runs-on: ubuntu-latest

    permissions: {}

    name: Install vsign and test presence in path
    steps:
      - name: Install vsign
        uses: zosocanuck/vsign-action@main
        with:
          cosign-release: 'v0.1.0'
      - name: Check install!
        run: vsign version
```

Example using the default version:

```yaml
jobs:
  test_vsign_action:
    runs-on: ubuntu-latest

    permissions: {}

    name: Install vsign and test presence in path
    steps:
      - name: Install vsign
        uses: zosocanuck/vsign-action@main
      - name: Check install!
        run: vsign version
```

This action does not need any GitHub permission to run, however, if your workflow needs to update, create or perform any
action against your repository, then you should change the scope of the permission appropriately.

For example, if you are using the `gcr.io` as your registry to push the images you will need to give the `write` permission
to the `packages` scope.

Example of a simple workflow:

```yaml
jobs:
  test_vsign_action:
    runs-on: ubuntu-latest

    permissions:
      contents: read
      packages: write
      id-token: write # needed for signing the images with GitHub OIDC Token **not production ready**

    name: Install vsign and test presence in path
    steps:
      - uses: actions/checkout@master
        with:
          fetch-depth: 1

      - name: Install vsign
        uses: zosocanuck/vsign-action@main

      - name: Set up QEMU
        uses: docker/setup-qemu-action@v1
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v1

      - id: docker_meta
        uses: docker/metadata-action@v3.6.0
        with:
          images: ghcr.io/sigstore/sample-honk
          tags: type=sha,format=long

      - name: Build and Push container images
        uses: docker/build-push-action@v2
        with:
          platforms: linux/amd64,linux/arm/v7,linux/arm64
          push: true
          tags: ${{ steps.docker_meta.outputs.tags }}
          labels: ${{ steps.docker_meta.outputs.labels }}

      - name: Sign image with a key
        run: |
          vsign sign --image ghcr.io/sample/app --mechanism 64
        env:
          TAGS: ${{ steps.docker_meta.outputs.tags }}
          VSIGN_URL: https://tpp.example.com
          VSIGN_TOKEN: ${{secrets.VSIGN_TOKEN}}
          VSIGN_PROJECT: my\\project
```

### Optional Inputs
The following optional inputs:

| Input | Description |
| --- | --- |
| `vsign-release` | `vsign` version to use instead of the default. |
| `install-dir` | directory to place the `vsign` binary into instead of the default (`$HOME/.vsign`). |
| `use-sudo` | set to `true` if `install-dir` location requires sudo privs. Defaults to false. |