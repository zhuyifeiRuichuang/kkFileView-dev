# kkFileView Docker 单机部署

本目录提供 Docker Compose 部署方式，配置文件通过卷挂载实现灵活配置。

## 使用

```bash
# 1) 按需修改同目录 application.properties
# 2) 启动
docker compose up -d
# 3) 访问
open http://localhost:8012
```

## 说明

- 镜像：`ghcr.io/zhuyifeiruichuang/kkfileview:latest`（tag 与代码仓库版本号一致，可锁定为 `:5.0.2`）。
- 配置挂载到容器内固定路径 `/opt/kkfileview/config/application.properties`，该路径与镜像版本无关（镜像内已建软链），升级版本无需改动本目录。
- 配置文件中大量项支持 `${KK_*}` 环境变量覆盖，也可在 `docker-compose.yml` 的 `environment` 中注入。
- 默认端口 `8012`，如需修改请同步改 `ports` 映射与 `application.properties` 的 `server.port`。

## 健康检查

容器内置原生 `HEALTHCHECK`（基于镜像内 bash 对 8012 端口做 TCP 连通性检查，校验服务监听已就绪），`docker ps` 可看到 `healthy` 状态；Docker Compose 亦通过 `healthcheck` 字段复用该机制。`start_period` 设为 120s 以容忍 JVM + LibreOffice 冷启动。

```bash
docker ps                              # STATUS 列显示 healthy / starting
docker inspect -f '{{.State.Health.Status}}' kkfileview
```

> 说明：基础镜像（ubuntu:24.04）未安装 curl/wget，故使用 bash /dev/tcp 实现，零额外依赖。应用级的 HTTP 200 校验由 CI 冒烟测试（curl）与 k8s 的 httpGet 探针负责。
