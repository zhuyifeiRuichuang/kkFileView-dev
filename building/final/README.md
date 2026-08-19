# kkFileView Final Image

This directory builds the **final** runtime image that actually runs kkFileView. It is layered on top of the prebuilt base image (`kkfileview-base`) so the heavy LibreOffice/JRE environment is built only once.

## Files

| File | Purpose |
|------|---------|
| `Dockerfile` | Builds the final image from `kkfileview-base`, adds the Maven distribution package, and sets the entrypoint. |
| `README.md` / `README.cn.md` | This documentation (English / Chinese). |

## How it works

1. **ARG `KK_VERSION`** — the version number (e.g. `5.0.2`). It selects the matching base image `ghcr.io/zhuyifeiruichuang/kkfileview-base:<KK_VERSION>` and must match the release tag / GitHub Release.
2. **FROM base** — `ghcr.io/zhuyifeiruichuang/kkfileview-base:${KK_VERSION}` provides OS + JRE 21 + LibreOffice + CJK fonts.
3. **ADD distribution** — `server/target/kkFileView-*.tar.gz` (produced by `mvn package`) is extracted into `/opt`, yielding `/opt/kkFileView-<version>`.
4. **Version-independent symlink** — `/opt/kkfileview -> /opt/kkFileView-<version>` so deployment mount paths stay fixed regardless of version.
5. **HEALTHCHECK** — a native check using `bash /dev/tcp` on port 8012 (the base image has no curl). `start_period` is 120s to tolerate cold start.
6. **ENTRYPOINT** — starts the jar with `-Dspring.config.location=/opt/kkfileview/config/application.properties` so the externally mounted config file is always used.

## Build

```shell
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  --file building/final/Dockerfile \
  --build-arg KK_VERSION=5.0.2 \
  --tag ghcr.io/zhuyifeiruichuang/kkfileview:5.0.2 \
  --tag ghcr.io/zhuyifeiruichuang/kkfileview:latest \
  --push .
```

> Normally you do **not** build this by hand: the `release.yml` workflow builds and pushes it automatically after `mvn package`.

## Consumed by

The published image is consumed by the deployment manifests under `deploy/docker` (Compose) and `deploy/k8s` (Kubernetes).
