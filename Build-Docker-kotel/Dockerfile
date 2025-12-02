# 第一阶段：前端构建阶段
FROM node:18.20.8-bookworm AS builder

# 配置环境变量
ENV TZ=Asia/Shanghai

WORKDIR /home
# 从GitHub拉取项目
RUN apt-get install -y git; \
    git clone https://gitee.com/witersen/SvnAdminV2.0.git; \
    npm config set registry https://registry.npmmirror.com;

WORKDIR /home/SvnAdminV2.0
# 复制前端源码并构建
COPY Build-Docker-kotel/01.web/ 01.web/
COPY Build-Docker-kotel/02.php/ 02.php/

RUN cd 01.web; \
    rm -rf package-lock.json; \
    npm install; \
    npm run build; \
    # 验证构建结果 \
    if [ ! -d "dist" ]; then echo "构建失败"; exit 1; fi;

# 第二阶段：最终运行环境阶段
FROM rockylinux/rockylinux:9

# LABEL MAINTAINER = "www.witersen.com 2023-07-23"

# 配置环境变量
ENV TZ=Asia/Shanghai \
    LANG=C.UTF-8 \
    LC_ALL=C.UTF-8

# 复制前端构建产物（从第一阶段）
COPY --from=builder /home/SvnAdminV2.0/01.web/dist/ /var/www/html/

# 复制其他应用文件（从第一阶段）
COPY --from=builder /home/SvnAdminV2.0/02.php /var/www/html/
COPY --from=builder /home/SvnAdminV2.0/03.cicd/svnadmin_docker/data/ /templete/
COPY Build-Docker-kotel/favicon.ico /var/www/html/favicon.ico

# 复制启动脚本
COPY Build-Docker-kotel/Start.sh /home/Start.sh

# 修改SQLite数据库登录不要验证码
COPY Build-Docker-kotel/svnadmin.db /templete/svnadmin.db

# 安装系统依赖和运行环境
RUN set -eux \
    # 配置时区 \
    echo "${TZ}" > /etc/timezone; \
    dnf install -y https://rpms.remirepo.net/enterprise/remi-release-9.rpm; \
    dnf module enable -y php:remi-8.4; \
    # 安装运行时依赖 \
    dnf install -y  \
        --setopt=install_weak_deps=False --nodocs \
        httpd \
        mod_dav_svn \
        mod_ldap \
        subversion \
        subversion-tools \
        php \
        php-common \
        php-cli \
        php-fpm \
        php-json \
        php-mysqlnd \
        php-pdo \
        php-process \
        php-gd \
        php-bcmath \
        php-ldap \
        php-mbstring \
        cyrus-sasl \
        cyrus-sasl-lib \
        cyrus-sasl-plain \
        cronie \
        at \
        glibc-langpack-en \
        glibc-langpack-zh \
        glibc-all-langpacks \
        glibc-locale-source \
        langpacks-zh_CN \
        procps; \
    # 清理安装缓存 \
    rm -rf /var/cache/dnf; \
    dnf autoremove -y; \
    dnf clean all; \
    rm -rf \
            /var/cache/dnf \
            /opt /mnt /usr/games /usr/share/man /usr/share/doc \
            /tmp/* /var/tmp/*; \
    # 解决启动时的域名警告问题 \
    echo "ServerName localhost" >> /etc/httpd/conf/httpd.conf; \
    mkdir -p /run/php-fpm/; \
    # PHP安全配置 \
    sed -i 's/expose_php = On/expose_php = Off/g' /etc/php.ini; \
    sed -i 's/listen.acl_users = apache,nginx/listen.acl_users = apache/g' /etc/php-fpm.d/www.conf; \
    chown -R apache:apache /var/www/html; \
    # 赋予启动脚本执行权限 \
    chmod +x /home/Start.sh
# 创建字符映射文件（解决UTF-8找不到的问题）
RUN if [ ! -f /usr/share/i18n/charmaps/UTF-8 ]; then \
    if [ -f /usr/share/i18n/charmaps/UTF-8.gz ]; then \
        gunzip -c /usr/share/i18n/charmaps/UTF-8.gz > /usr/share/i18n/charmaps/UTF-8; \
    else \
        echo "UTF-8 charmap not found, creating from locale source"; \
        localedef --list-archive | grep UTF-8 || localedef -i en_US -f UTF-8 en_US.UTF-8; \
    fi \
    fi

# 生成中文locale（使用正确的语法）
RUN localedef -c -i zh_CN -f UTF-8 zh_CN.UTF-8 2>/dev/null || \
    localedef -i zh_CN -f UTF-8 zh_CN.UTF-8 2>/dev/null || \
    echo "Locale generation may have warnings, continuing..."

# 设置环境变量
ENV LANG=zh_CN.UTF-8
ENV LC_ALL=zh_CN.UTF-8
ENV LC_CTYPE=zh_CN.UTF-8

# USER apache
WORKDIR /home/svnadmin

# 暴露服务端口
EXPOSE 80
EXPOSE 443
EXPOSE 3690

# 启动服务
CMD ["/home/Start.sh"]
