# kkFileView Docker Standalone Deployment

This directory provides a Docker Compose deployment. Runtime configuration is mounted via a volume for flexible overrides.

## Files

| File | Purpose |
|------|---------|
| `docker-compose.yml` | Service definition. Pulls `ghcr.io/zhuyifeiruichuang/kkfileview:latest` and mounts the config file read-only. |
| `application.properties` | Runtime configuration (a copy of the server default). Edit this to customize. |
| `README.md` / `README.cn.md` | This documentation (English / Chinese). |

## Usage

```bash
# 1) Edit application.properties in this directory as needed
# 2) Start
docker compose up -d
# 3) Access
open http://localhost:8012
```

## Configuration (flexible)

- **Image**: `ghcr.io/zhuyifeiruichuang/kkfileview:latest`. The tag follows the repository version; pin it to `:5.0.2` for reproducibility.
- **Config mount**: mounted at the fixed in-container path `/opt/kkfileview/config/application.properties`. This path is version-independent (the image uses a symlink), so upgrading the image version does **not** require changing this directory.
- **Env overrides**: many options support `${KK_*}` environment-variable overrides; you can also inject them via the `environment:` block in `docker-compose.yml`.
- **Port**: defaults to `8012`. To change it, update both the `ports` mapping and `server.port` in `application.properties`.

## Health check

The image ships a native `HEALTHCHECK` (a `bash /dev/tcp` TCP connectivity check on port 8012, because the base image has no curl). Docker Compose reuses it via the `healthcheck` block. `start_period` is 120s to tolerate JVM + LibreOffice cold start.

```bash
docker ps                                                   # STATUS shows healthy / starting
docker inspect -f '{{.State.Health.Status}}' kkfileview
```

> The base image (ubuntu:24.04) has no curl/wget, hence `bash /dev/tcp` is used (zero extra dependencies). Application-level HTTP 200 validation is covered by the CI smoke test and the k8s `httpGet` probes.
