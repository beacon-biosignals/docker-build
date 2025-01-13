# Docker Build

Build a Docker image while utilizing [layer caching](https://docs.docker.com/build/cache/)
backed from the image repository. [Docker does support using GitHub Actions cache](https://docs.docker.com/build/cache/backends/gha/)
as a layer cache backend but the default cache limit for a repository is 10 GB which is
quite small for Docker images.

We recommend utilizing a separate image repositories for deployment and production (e.g.`temporary/my-image` and `permanent/my-image`) to make it easier to separate temporary images from permanent images meant for end users. Promoting temporary images to be permanent can be done with `docker push` or [`regctl image copy --digest-tags`](https://github.com/regclient/regclient/blob/main/docs/regctl.md#registry-commands) if you want the digest to be identical across registries.

## Example

```yaml
---
jobs:
  example:
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
          # Build images from scratch on `main`. Ensures system packages have latest security fixes.
          from-scratch: ${{ github.ref == 'refs/heads/main' }}
```

## Inputs

| Name                 | Description | Required | Example |
|:---------------------|:------------|:---------|:--------|
| `image-repository`   | The Docker image repository to push the build image and cached layers. | Yes | `temporary/my-image` |
| `context`            | The Docker build context directory. Defaults to `.`. | No | `./my-image` |
| `build-args`         | List of [build-time variables](https://docs.docker.com/reference/cli/docker/buildx/build/#build-arg). | No | <pre><code>HTTP_PROXY=http://10.20.30.2:1234&#10;FTP_PROXY=http://40.50.60.5:4567</code></pre> |
| `build-secrets`      | List of [secrets](https://docs.docker.com/engine/reference/commandline/buildx_build/#secret) to expose to the build. | No | `GIT_AUTH_TOKEN=mytoken` |
| `from-scratch`       | Do not use cache when building the image. Defaults to `false`. | No | `false` |

## Outputs

| Name               | Description | Example |
|:-------------------|:------------|:--------|
| `image`            | Reference to the build image including the digest. | `temporary/my-image@sha256:37782d4e1c24d8f12047039a0d3512d1b6059e306a80d5b66a1d9ff60247a8cb` |
| `image-repository` | The Docker image repository where the image was pushed to. | `temporary/my-image` |
| `digest`           | The built Docker image digest. | `sha256:37782d4e1c24d8f12047039a0d3512d1b6059e306a80d5b66a1d9ff60247a8cb` |
| `tags`             | JSON list of tags associated with the built Docker image. | `branch-main`, `sha-152cb14` |
| `commit-sha`       | The Git commit SHA used to build the imag. | `152cb14643b50529b229930d6124e6bbef48668d` |

## Permissions

No [job permissions](https://docs.github.com/en/actions/using-jobs/assigning-permissions-to-jobs) are required to run this action.
