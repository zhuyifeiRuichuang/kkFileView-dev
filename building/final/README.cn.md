# kkFileView 最终镜像

本目录构建实际运行 kkFileView 的 **最终镜像**。它在预构建的基础镜像（`kkfileview-base`）之上叠加应用，因此沉重的 LibreOffice/JRE 环境只需构建一次。

## 文件说明

| 文件 | 作用 |
|------|------|
| `Dockerfile` | 基于 `kkfileview-base` 构建最终镜像，叠加 Maven 发行包，并设置启动入口。 |
| `README.md` / `README.cn.md` | 本文档（英文 / 中文）。 |

## 工作原理

1. **FROM 基础镜像** —— `ghcr.io/zhuyifeiruichuang/kkfileview-base:latest` 提供 OS + JRE 21 + LibreOffice + 中文字体。基础镜像不区分版本，始终使用 `:latest`。
2. **ADD 发行包** —— `server/target/kkFileView-*.tar.gz`（由 `mvn package` 产出）解压到 `/opt`，得到 `/opt/kkFileView-<版本>`。
3. **版本无关软链** —— `/opt/kkfileview -> /opt/kkFileView-<版本>`，使部署挂载路径与版本无关。
4. **HEALTHCHECK** —— 使用 `bash /dev/tcp` 对 8012 端口做原生检查（基础镜像无 curl）。`start_period` 为 120s 以容忍冷启动。
5. **ENTRYPOINT** —— 以 `-Dspring.config.location=/opt/kkfileview/config/application.properties` 启动 jar，始终使用外部挂载的配置文件。

## 构建

```shell
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  --file building/final/Dockerfile \
  --tag ghcr.io/zhuyifeiruichuang/kkfileview:5.0.2 \
  --tag ghcr.io/zhuyifeiruichuang/kkfileview:latest \
  --push .
```

> 最终镜像的 `Dockerfile` **不需要** `--build-arg` 来指定基础镜像——它始终拉取 `kkfileview-base:latest`。版本号 tag（如 `5.0.2`）只应用于最终镜像，基础镜像不区分版本。

> 通常你无需手动构建：该镜像由 `release.yml` 工作流在 `mvn package` 之后自动构建并推送。

## 被谁使用

发布后的镜像被 `deploy/docker`（Compose）与 `deploy/k8s`（Kubernetes）下的部署清单所使用。
