# kkFileView 基础镜像

本目录构建 **基础镜像**（`kkfileview-base`），它打包了体积大、很少变动的运行环境：Ubuntu 24.04、OpenJDK 21（JRE）、LibreOffice（无 GUI 版）以及中文字体。只构建一次并在各次发版间复用，可大幅加速最终镜像构建，避免在每次应用代码变更时都重编译 LibreOffice。

## 文件说明

| 文件 | 作用 |
|------|------|
| `Dockerfile` | 安装 OS + JRE 21 + LibreOffice + 中文字体，**不含**任何应用代码。 |
| `fonts/.gitkeep` | 让（原本为空的）fonts 目录纳入版本控制。 |
| `README.md` / `README.cn.md` | 本文档（英文 / 中文）。 |

## 镜像仓库与 tag

基础镜像发布到 GitHub Container Registry：

```
ghcr.io/zhuyifeiruichuang/kkfileview-base:latest
```

基础镜像**不区分版本**——只保留一个 `:latest` tag。这是有意设计：基础镜像仅包含 OS 运行时（Ubuntu + JRE + LibreOffice + 字体），与版本无关。版本号 tag 只在最终镜像（`kkfileview`）上区分。

## 自动构建（推荐）

通常你无需手动构建基础镜像。`build-base.yml` 工作流**仅在 `building/base/**` 下的文件变更时**才会重建并推送 `ghcr.io/zhuyifeiruichuang/kkfileview-base:latest`（路径过滤），因此昂贵的 LibreOffice 层不会在每次代码变更时重构建。发版时，`release.yml` 只需检查 `:latest` 是否存在（若缺失则从源码兜底构建）——基础镜像不创建任何版本号 tag。

如需手动强制重建：

```shell
gh workflow run build-base.yml
# 或者直接向 building/base/** 推送任意改动
```

## 本地手动构建

> 基础镜像无论版本如何，只使用 `:latest` tag。本项目维护的 Dockerfile 已考虑跨平台；若要构建 arm64 镜像，在 arm64 机器上执行同样的命令即可。

```shell
docker build --tag ghcr.io/zhuyifeiruichuang/kkfileview-base:latest .
```

## 跨平台构建

`docker buildx` 支持在一台机器上构建多种架构镜像。例如，执行 `docker buildx build` 时加上 `--platform=linux/arm64` 即可构建 arm64 镜像——对没有 arm64 硬件却需要 arm64 镜像的用户非常方便。

> 当前本项目仅支持构建 `linux/amd64` 与 `linux/arm64` 两种架构。
> buildx 的 builder driver 可以使用默认的 `docker` 类型，若使用 `docker-container` 类型可并行构建多种架构（此处不展开）。参考 [Docker Buildx](https://docs.docker.com/buildx/working-with-buildx/#build-multi-platform-images)。

**前提要求**（以 amd64 主机为例）：需开启 docker 的 buildx 特性与 Linux QEMU 用户模式。使用 WSL2 且安装了较新 Docker Desktop 的 Windows 用户已满足这些前提。

1. 安装 docker buildx 客户端插件（Docker >= 19.03）。若已安装可跳过。参考 https://github.com/docker/buildx。
2. 开启 QEMU 用户模式并安装其他平台模拟器（Linux 内核 >= 4.8）：

```shell
docker run --privileged --rm tonistiigi/binfmt --install all
```

跨平台构建并推送示例：

```shell
docker buildx build --platform=linux/amd64,linux/arm64 -t ghcr.io/zhuyifeiruichuang/kkfileview-base:latest --push .
```
