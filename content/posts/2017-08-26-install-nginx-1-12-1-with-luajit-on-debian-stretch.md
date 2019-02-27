---
title: "在 Debian Stretch 上安装 nginx 1.12.1 + LuaJIT"
date: 2017-08-26T22:11:57+08:00
categories: ["linux"]
tags: ["nginx", "debian"]
---

最近将 VM 实例的系统更换到了 Debian Stretch，重新安装了 `nginx`，新增了 `Lua` 模块，记录一下安装过程。

由于 `lua-nginx-module` 不支持高版本的 `openssl`，因此这里下载了 1.0.2l 版本的 `openssl`。

本次安装计划将 nginx 安装在 `/apps/nginx/1.12.1` 目录下，并以 `www` 用户运行 `nginx`。

安装所需依赖项目放在 `/apps/src` 目录下。

<!--more-->

## 准备工作

创建目录：

```sh
mkdir -p /apps/{nginx,src}
```

下载所需项目，包括：

* [nginx 1.12.1](http://nginx.org/en/download.html)
* [LuaJIT 2.0.5](http://luajit.org/download.html)
* [ngx-devel-kit 0.3.0](https://github.com/simpl/ngx_devel_kit/releases)
* [lua-nginx-module 0.10.10](https://github.com/chaoslawful/lua-nginx-module/releases)
* [pcre 8.38](http://www.pcre.org/)
* [zlib 1.2.11](http://zlib.net/)
* [openssl 1.0.2l](https://www.openssl.org)

```sh
cd /apps/src
curl -LS http://nginx.org/download/nginx-1.12.1.tar.gz | tar -xzv
curl -LS http://luajit.org/download/LuaJIT-2.0.5.tar.gz | tar -xzv
curl -LS https://github.com/simpl/ngx_devel_kit/archive/v0.3.0.tar.gz | tar -xzv
curl -LS https://github.com/openresty/lua-nginx-module/archive/v0.10.10.tar.gz | tar -xzv
curl -LS ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.38.tar.gz | tar -xzv
curl -LS http://zlib.net/zlib-1.2.11.tar.gz | tar -xzv
curl -LS https://www.openssl.org/source/openssl-1.0.2l.tar.gz | tar -xzv
```

安装编译工具：

```sh
apt install build-essential
```

## 创建 www 用户

```sh
useradd -d /apps/www -U -s /usr/sbin/nologin www
```

## 安装 LuaJIT

```sh
cd /apps/src/LuaJIT-2.0.5
make
make install PREFIX=/apps/LuaJIT/2.0.5
echo 'export LUAJIT_LIB=/apps/LuaJIT/2.0.5/lib' >> /etc/profile.d/LuaJIT.sh
echo 'export LUAJIT_INC=/apps/LuaJIT/2.0.5/include/luajit-2.0' >> /etc/profile.d/LuaJIT.sh
ln -s /apps/LuaJIT/2.0.5/lib/libluajit-5.1.so.2 /lib
```

## 安装 nginx

```sh
cd /apps/src/nginx-1.12.1
./configure \
--user=www \
--group=www \
--prefix=/apps/nginx/1.12.1 \
--with-openssl=../openssl-1.0.2l \
--with-http_ssl_module \
--http-client-body-temp-path=tmp/client_body_temp \
--http-proxy-temp-path=tmp/proxy_temp \
--http-fastcgi-temp-path=tmp/fastcgi_temp \
--http-uwsgi-temp-path=tmp/uwsgi_temp \
--http-scgi-temp-path=tmp/scgi_temp \
--with-pcre=/apps/src/pcre-8.38 \
--with-zlib=/apps/src/zlib-1.2.11 \
--add-module=../ngx_devel_kit-0.3.0 \
--add-module=../lua-nginx-module-0.10.10
make
make install
cd /apps/nginx/1.12.1
mdkir tmp
```

## 配置用于 systemd 的 nginx.service

```sh
cat <<"EOF" > /apps/nginx/nginx.service
[Unit]
Description=The NGINX Server
After=network.target

[Service]
Type=forking
PIDFile=/apps/nginx/1.12.1/logs/nginx.pid
ExecStartPre=/apps/nginx/1.12.1/sbin/nginx -t
ExecStart=/apps/nginx/1.12.1/sbin/nginx
ExecReload=/bin/kill -HUP $MAINPID
ExecStop=/bin/kill -QUIT $MAINPID
PrivateTmp=false

[Install]
WantedBy=multi-user.target
EOF

systemctl enable /apps/nginx/nginx.service
systemctl start nginx.service
```

## 在 nginx 中使用 lua：

在 `nginx.conf` 中加入：

```sh
location /ping {
    content_by_lua '
        ngx.header.content_type = "text/plain"
        ngx.say("pong")
    ';
}
```

重载 nginx 配置文件 `systemctl reload nginx.service`，之后访问 `curl http://host/ping` 即可看到由 lua 返回的结果。

另外，在生产环境中，注意配置 nginx 是否开启 gzip 压缩，关闭 server_tokens 等选项。
