---
title: "在 Debian Stretch 上编译安装 PHP7.2.2"
date: 2018-02-28T21:59:38+08:00
categories: ["PHP"]
tags: ["PHP"]
---

## 准备工作

首先安装依赖的组件：

```sh
apt install pkg-config \
libxml2-dev \
libssl-dev \
zlib1g-dev \
libcurl4-gnutls-dev \
libjpeg-dev \
libpng-dev \
libgd-dev \
libxslt1-dev \
```

<!--more-->

执行 `ln -s /usr/include/x86_64-linux-gnu/curl/ /usr/local/include/curl` 链接编译需要的 curl 文件到对应的位置。

## 下载 PHP 源码

从 `http://cn2.php.net/get/php-7.2.2.tar.gz/from/this/mirror` 下载 `PHP` 源码并解压。

```sh
cd /usr/local/src
curl http://cn2.php.net/get/php-7.2.2.tar.gz/from/this/mirror | tar -xzv
```

## 编译安装

进入源码所在目录，将 `PHP` 编译安装到 `/usr/local/php/7.2.2` 目录：

```sh
cd /usr/local/src/php-7.2.2

./configure --prefix=/usr/local/php/7.2.2 \
--with-config-file-path=/usr/local/php/7.2.2/etc \
--with-curl \
--with-freetype-dir \
--with-gd \
--with-gettext \
--with-iconv-dir \
--with-kerberos \
--with-libxml-dir \
--with-mysqli \
--with-openssl \
--with-pcre-regex \
--with-pdo-mysql \
--with-pdo-sqlite \
--with-pear \
--with-png-dir \
--with-jpeg-dir \
--with-xmlrpc \
--with-xsl \
--with-zlib \
--with-libzip \
--enable-fpm \
--enable-bcmath \
--enable-libxml \
--enable-inline-optimization \
--enable-mbregex \
--enable-mbstring \
--enable-opcache \
--enable-pcntl \
--enable-shmop \
--enable-soap \
--enable-sockets \
--enable-sysvsem \
--enable-xml \
--enable-zip \
--enable-mysqlnd

make
make install
```

## 配置 PHP

将 `PHP` 的配置文件 `php.ini` 从源码目录拷贝到安装目录：

```sh
cp /usr/local/src/php-7.2.2/php.ini-production /usr/local/php/7.2.2/etc/php.ini
```

处理 `php-fpm` 配置文件：

```sh
cp /usr/local/php/7.2.2/etc/php-fpm.conf.default /usr/local/php/7.2.2/etc/php-fpm.conf
cp /usr/local/php/7.2.2/etc/php-fpm.d/www.conf.default /usr/local/php/7.2.2/etc/php-fpm.d/www.conf
```

根据自己需要配置好 `php.ini`、 `php-fpm.conf` 以及 `www.conf` 文件。

将 `systemd` 启动用到的服务文件拷贝到安装目录并启用：

```sh
cp /usr/local/src/php-7.2.2/sapi/fpm/php-fpm.service /usr/local/php/

systemctl enable /usr/local/php/php-fpm.service
systemctl start php-fpm.service
```

## 结束语

这样就编译安装好了 `PHP` `7.2.2`。

