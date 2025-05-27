# Docker Build

Build a Docker image while utilizing [layer caching](https://docs.docker.com/build/cache/) backed from the image repository. Image tags will be automatically created based upon the relevant PR, branch name, and commit SHA.

When using this action we recommend utilizing a separate image repositories for development and production (e.g.`temporary/my-image` and `permanent/my-image`) to make it easier to separate temporary images from permanent images meant for end users. The `beacon-biosignals/docker-build` action is used to build temporary images under development. Once a temporary image is ready for production it can be promoted to be permanent by using `docker tag`/`docker push` or [`regctl image copy --digest-tags`](https://github.com/regclient/regclient/blob/main/docs/regctl.md#registry-commands) (if you want the digest to be identical across registries) to transfer the image.

Note that although [Docker does support using GitHub Actions cache](https://docs.docker.com/build/cache/backends/gha/) as a layer cache backend the GHA cache limit for a repository is 10 GB which is quite limiting for larger Docker images.

## Example

```yaml
---
on:
  pull_request: {}
  # Trigger this build workflow on "main". See `from-scratch`
  push:
    branches:
      - main
jobs:
  example:
    # These permissions are needed to:
    # - Get the workflow run: https://github.com/beacon-biosignals/docker-build#permissions
    permissions: {}
    runs-on: ubuntu-latest
    steps:
      - name: Build image
        uses: beacon-biosignals/docker-build@v1
        with:
          image-repository: temporary/my-image
          context: .
          # Example of passing in Docker `--build-arg`
          build-args: |
            JULIA_VERSION=1.10
            PYTHON_VERSION=3.10
          # Example of passing in Docker `--secret`
          build-secrets: |
            github-token=${{ secrets.token || github.token }}
          # Build images from scratch on "main". Ensures that caching doesn't result in using insecure system packages.
          from-scratch: ${{ github.ref == 'refs/heads/main' }}
          cache-mount-ids: |
            /var/cache/apt
            /var/lib/apt
```

## Inputs

| Name                 | Description | Required | Example |
|:---------------------|:------------|:---------|:--------|
| `image-repository`   | The Docker image repository to push the build image and cached layers. | Yes | `temporary/my-image` |
| `context`            | The Docker build context directory. Defaults to `.`. | No | `./my-image` |
| `dockerfile`         | Path to the Dockerfile. Defaults to `${context}/Dockerfile`. | No | `./my-image.Dockerfile` |
| `build-args`         | List of [build-time variables](https://docs.docker.com/reference/cli/docker/buildx/build/#build-arg). | No | <pre><code>HTTP_PROXY=http://10.20.30.2:1234 &#10;FTP_PROXY=http://40.50.60.5:4567</code></pre> |
| `build-secrets`      | List of [secrets](https://docs.docker.com/engine/reference/commandline/buildx_build/#secret) to expose to the build. | No | <pre><code>GIT_AUTH_TOKEN=mytoken</code></pre> |
| `from-scratch`       | Do not read from the cache when building the image. Writes to caches will still occur. Defaults to `false`. | No | `false` |
| `cache-mount-ids`    | List of build [cache mount IDs or targets](https://docs.docker.com/reference/dockerfile/#run---mounttypecache) to preserve across builds. | No | <pre><code>/var/cache/apt&#10;/var/lib/apt</code></pre> |

## Outputs

| Name               | Description | Example |
|:-------------------|:------------|:--------|
| `image`            | Reference to the build image including the digest. | `temporary/my-image@sha256:37782d4e1c24d8f12047039a0d3512d1b6059e306a80d5b66a1d9ff60247a8cb` |
| `image-repository` | The Docker image repository where the image was pushed to. | `temporary/my-image` |
| `digest`           | The built Docker image digest. | `sha256:37782d4e1c24d8f12047039a0d3512d1b6059e306a80d5b66a1d9ff60247a8cb` |
| `tags`             | JSON list of tags associated with the built Docker image. | `branch-my-branch`, `sha-152cb14`, `pr-123` |
| `commit-sha`       | The Git commit SHA used to build the image. | `152cb14643b50529b229930d6124e6bbef48668d` |

## Permissions

The follow [job permissions](https://docs.github.com/en/actions/using-jobs/assigning-permissions-to-jobs) are required to run this action:

```yaml
permissions:
  packages: write  # Only required when using the GitHub Container registry: https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry
```