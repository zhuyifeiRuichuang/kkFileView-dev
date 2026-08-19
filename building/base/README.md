# kkFileView Base Image

This directory builds the **base** image (`kkfileview-base`) that bundles the heavy, rarely-changing runtime environment: Ubuntu 24.04, OpenJDK 21 (JRE), LibreOffice (no-GUI), and CJK fonts. Building it once and reusing it across releases drastically speeds up the final image build and avoids rebuilding LibreOffice every time the application code changes.

## Files

| File | Purpose |
|------|---------|
| `Dockerfile` | Installs OS + JRE 21 + LibreOffice + CJK fonts. Contains **no** application code. |
| `fonts/.gitkeep` | Keeps the (otherwise empty) fonts directory under version control. |
| `README.md` / `README.cn.md` | This documentation (English / Chinese). |

## Image registry & tag

The base image is published to GitHub Container Registry:

```
ghcr.io/zhuyifeiruichuang/kkfileview-base:<version>
```

The `<version>` matches the repository version (e.g. `5.0.2`). It is also aliased as `:latest`.

## Built automatically (recommended)

You normally do **not** build the base image by hand. The `build-base.yml` workflow rebuilds and pushes `ghcr.io/zhuyifeiruichuang/kkfileview-base:latest` **only when files under `building/base/**` change** (path filter), so the expensive LibreOffice layer is not rebuilt on every code change. At release time, `release.yml` reuses that `:latest` and aliases it to `:<version>` via `imagetools create` (a zero-layer re-tag — no rebuild).

To force a rebuild manually:

```shell
gh workflow run build-base.yml
# or simply push any change under building/base/**
```

## Manual build (local)

> Example tag `5.0.2`. The maintained Dockerfile is cross-platform aware; to build an arm64 image, run the same command on an arm64 machine.

```shell
docker build --tag ghcr.io/zhuyifeiruichuang/kkfileview-base:5.0.2 .
```

## Cross-platform build

`docker buildx` can build multiple architectures on a single machine. For example, add `--platform=linux/arm64` to build an arm64 image — convenient when you lack arm64 hardware.

> This project supports `linux/amd64` and `linux/arm64` only.
> The buildx builder driver can be the default `docker` type, or `docker-container` to build architectures in parallel (not covered here). See [Docker Buildx](https://docs.docker.com/buildx/working-with-buildx/#build-multi-platform-images).

**Prerequisites** (amd64 host example): enable buildx and Linux QEMU user-mode. WSL2 + recent Docker Desktop on Windows already satisfies these.

1. Install the buildx plugin (Docker >= 19.03). Skip if present. See https://github.com/docker/buildx.
2. Enable QEMU user-mode and install emulators (Linux kernel >= 4.8):

```shell
docker run --privileged --rm tonistiigi/binfmt --install all
```

Example cross-platform build & push:

```shell
docker buildx build --platform=linux/amd64,linux/arm64 -t ghcr.io/zhuyifeiruichuang/kkfileview-base:5.0.2 --push .
```
