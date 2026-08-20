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
ghcr.io/zhuyifeiruichuang/kkfileview-base:latest
```

The base image **does not differentiate by version** — it only carries a single `:latest` tag. This is intentional: the base image contains only the OS runtime (Ubuntu + JRE + LibreOffice + fonts), which is version-independent. The final image (`kkfileview`) carries per-release version tags.

## Built automatically (recommended)

You normally do **not** build the base image by hand. The `build-base.yml` workflow rebuilds and pushes `ghcr.io/zhuyifeiruichuang/kkfileview-base:latest`:

1. **On file change** — only when files under `building/base/**` change (path filter), so the expensive LibreOffice layer is not rebuilt on every code change.
2. **Manually** — via `workflow_dispatch`.

Every build uses `--no-cache` + `--pull` to force a full rebuild and pull the latest `ubuntu:24.04` base layer, so `apt-get update` always installs the newest Ubuntu repo packages (including LibreOffice / OpenJDK CVE fixes) — no reliance on inline cache that could skip the apt layer.

At release time, `release.yml` also fully rebuilds the base image from source (same `--no-cache` + `--pull`), so the released final image is always built on a base with the latest security patches. No version tags are created for the base image.

To force a rebuild manually:

```shell
gh workflow run build-base.yml
# or simply push any change under building/base/**
```

## Runtime notes

- **No mirror swap** — the Dockerfile does not replace `archive.ubuntu.com` with a mirror: GitHub Actions runners are hosted overseas where the default source is fastest.
- **Fonts** — `ttf-mscorefonts-installer` (a flaky SourceForge download) is replaced with Ubuntu repo fonts that are metrically compatible: `fonts-liberation` (≈ Arial/Times/Courier), `fonts-crosextra-carlito` (≈ Calibri), `fonts-crosextra-caladea` (≈ Cambria), plus `ttf-wqy-*` CJK fonts.

## Manual build (local)

> The base image uses a single `:latest` tag regardless of version.

```shell
docker build --tag ghcr.io/zhuyifeiruichuang/kkfileview-base:latest .
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
docker buildx build --platform=linux/amd64,linux/arm64 -t ghcr.io/zhuyifeiruichuang/kkfileview-base:latest --push .
```
