# 最终镜像：在基础镜像之上叠加 kkFileView 发行包并启动服务
#
# 构建时必须传入 --build-arg KK_VERSION=<版本号>，该版本号同时用于
# 1) 拉取对应的基础镜像 ghcr.io/<owner>/kkfileview-base:<KK_VERSION>
# 2) 与发版 tag、GitHub Release 保持一致
#
# 用法示例：
#   docker buildx build \
#     --platform linux/amd64,linux/arm64 \
#     --file building/final/Dockerfile \
#     --build-arg KK_VERSION=5.0.2 \
#     --tag ghcr.io/<owner>/kkfileview:5.0.2 \
#     --tag ghcr.io/<owner>/kkfileview:latest \
#     --push .

ARG KK_VERSION
FROM ghcr.io/zhuyifeiruichuang/kkfileview-base:${KK_VERSION}

# 将 Maven 打包好的发行包（含 bin/*.sh、bin/*.jar、config/）解压到 /opt
ADD server/target/kkFileView-*.tar.gz /opt/

# 建立版本无关的软链 /opt/kkfileview -> /opt/kkFileView-<版本>
# 这样部署时无论版本如何升级，配置挂载路径都固定为
# /opt/kkfileview/config/application.properties
RUN ln -sfn "$(ls -d /opt/kkFileView-* | head -n1)" /opt/kkfileview

ENV KKFILEVIEW_BIN_FOLDER=/opt/kkfileview/bin \
    LANG=zh_CN.UTF-8 \
    LC_ALL=zh_CN.UTF-8

EXPOSE 8012

# 通过 -Dspring.config.location 指定外部配置，便于挂载 application.properties 实现灵活配置
# 通配符 /opt/kkFileView-*/bin/kkFileView-*.jar 由 sh 展开，天然适配版本号变化
ENTRYPOINT ["sh", "-c", "exec java -Dfile.encoding=UTF-8 -Dspring.config.location=/opt/kkfileview/config/application.properties -jar /opt/kkFileView-*/bin/kkFileView-*.jar"]
