# Docker Versioning for Illarion

![GitHub](https://img.shields.io/github/license/Illarion-eV/Illarion-Docker-Version)

This generates several variables that can be used when creating docker images.

## Usage

```yml
jobs:
  push:
    runs-on: ubuntu-latest

    steps:
      - uses: Illarion-eV/Illarion-Docker-Version@v1
        id: docker-vars
        with:
          registry-secret: ${{ secrets.GITHUB_TOKEN }}
```

### Input Variables

* **`image-name`** - The base name of the image, including the registry but excluding any tags. It defaults to "ghcr.io"
  for the registry and the repository owner and name for the image name.
* **`latest-on-tags`** - Set to `true` or `false`, this option allows to set if builds of tags should get the `latest` tag.
  This default to `true`.
* **`latest-on-branches`** - A regular expression that needs to match the name of the branch that is currently build,
  to have the action set the `latest` tag. This default to an empty string, meaning the `latest` tag is not set for any
  branch.
* **`registry-secret`** - This variable may be populated with the secret that should be used to authenticate with the
  registry. This action doesn't perform any authentication, it's just use to set a variable determining if the secret
  is available.

### Output Variables

* **`version`** - The version information for the image. This value is meant to populate the
  `org.opencontainers.image.version` label.
* **`tags`** - A comma separated list of the image name along with all the tags that are genenerated for the image.
  This is meant to be provided to the `tags` parameter of the `docker/build-push-action@v2`.
* **`created`** - The date this image was created. This is meant to populate the `org.opencontainers.image.created`
  label.
* **`docker-secret`** - The same value that was provided in the `registry-secret` input.
* **`has-docker-secret`** - A boolean value that is set `true` in case the input `registry-secret` is not empty.

### Example Usage

```yml
jobs:
  push:
    runs-on: ubuntu-latest

    steps:
      - uses: Illarion-eV/Illarion-Docker-Version@v1
        id: docker-vars
        with:
          registry-secret: ${{ secrets.GITHUB_TOKEN }}

      - name: Login to GitHub
        if: ${{ steps.docker-vars.outputs.has-docker-secret }}
        uses: docker/login-action@v1
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ steps.docker-vars.outputs.docker-secret }}

      - name: Build and push
        uses: docker/build-push-action@v2
        with:
          push: ${{ steps.docker-vars.outputs.has-docker-secret }}
          tags: ${{ steps.docker-vars.outputs.tags }}
          labels: |
              org.opencontainers.image.version=${{ steps.docker-vars.outputs.version }}
              org.opencontainers.image.created=${{ steps.docker-vars.outputs.created }}
```