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
