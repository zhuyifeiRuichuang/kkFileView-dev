# 基础镜像：仅打包运行环境（JRE 21 + LibreOffice + 字体），不含应用代码。
# 不区分版本，只保留 :latest tag；由 build-base.yml 在 building/base/** 变更时重建。
# 安全补丁：每次构建都 --no-cache 全量重建 + --pull 拉最新 ubuntu:24.04，
# apt-get update 安装最新 Ubuntu 仓库包（含 CVE 修复），不受缓存命中影响。

FROM ubuntu:24.04

# 单层安装全部运行时组件（合并 RUN 减少镜像层数）：
#   - openjdk-21-jre       应用运行时（与 pom.xml 的 Java 21 匹配）
#   - tzdata / locales     时区与中文 locale
#   - xfonts-utils / fontconfig   字体工具（mkfontscale / fc-cache）
#   - libreoffice-nogui    Office 文档转换引擎（无 GUI 依赖）
#   - fonts-liberation / fonts-crosextra-carlito / fonts-crosextra-caladea
#     Ubuntu 仓库内的度量兼容字体，替代 ttf-mscorefonts-installer：
#     后者依赖 SourceForge 在线下载且经常失败（构建脆弱点）；
#     Liberation ≈ Arial/Times/Courier，Carlito ≈ Calibri，Caladea ≈ Cambria
#   - ttf-wqy-microhei / ttf-wqy-zenhei / xfonts-wqy   中文字体
# 不做国内镜像源替换：GitHub Actions runner 在海外，默认 archive.ubuntu.com 更快。
# 安装后立即清理 apt 缓存，控制镜像体积。
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
        openjdk-21-jre \
        tzdata \
        locales \
        xfonts-utils \
        fontconfig \
        libreoffice-nogui \
        fonts-liberation \
        fonts-crosextra-carlito \
        fonts-crosextra-caladea \
        ttf-wqy-microhei \
        ttf-wqy-zenhei \
        xfonts-wqy && \
    echo 'Asia/Shanghai' > /etc/timezone && \
    ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    localedef -i zh_CN -c -f UTF-8 -A /usr/share/locale/locale.alias zh_CN.UTF-8 && \
    locale-gen zh_CN.UTF-8 && \
    apt-get autoremove -y && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# 内置一些常用的中文字体，避免普遍性乱码
ADD fonts/* /usr/share/fonts/chinese/

# 刷新字体缓存（fontconfig + X11 字体索引）
RUN cd /usr/share/fonts/chinese && \
    mkfontscale && \
    mkfontdir && \
    fc-cache -fv

ENV LANG=zh_CN.UTF-8 LC_ALL=zh_CN.UTF-8
