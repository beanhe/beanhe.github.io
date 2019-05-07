---
layout: post
title: ubuntu14.04、15.10、16.04安装OpenResty
category: web
tags: [__beanhe,nginx,web,openresty,lua]
---

### Ubuntu 14.04安装OpenResty

- 脚本如下

```
#!/bin/bash
apt-get -y update
apt-get -y install nginx-extras build-essential libpcre3-dev libssl-dev libgeoip-dev libpq-dev libxslt1-dev libgd2-xpm-dev
wget -c https://openresty.org/download/openresty-1.11.2.3.tar.gz
tar zxvf openresty-1.11.2.3.tar.gz
cd openresty-1.11.2.3

./configure \
--sbin-path=/usr/sbin/nginx \
--conf-path=/etc/nginx/nginx.conf \
--error-log-path=/var/log/nginx/error.log \
--http-client-body-temp-path=/var/lib/nginx/body \
--http-fastcgi-temp-path=/var/lib/nginx/fastcgi \
--http-log-path=/var/log/nginx/access.log \
--http-proxy-temp-path=/var/lib/nginx/proxy \
--http-scgi-temp-path=/var/lib/nginx/scgi \
--http-uwsgi-temp-path=/var/lib/nginx/uwsgi \
--lock-path=/var/lock/nginx.lock \
--pid-path=/var/run/nginx.pid \
--with-sha1=/usr/include/openssl \
--with-md5=/usr/include/openssl \
--with-luajit \
--with-pcre-jit \
--with-debug \
--with-file-aio \
--with-http_auth_request_module \
--with-http_addition_module \
--with-http_dav_module \
--with-http_flv_module \
--with-http_gunzip_module \
--with-http_geoip_module \
--with-http_gzip_static_module \
--with-http_image_filter_module \
--with-http_mp4_module \
--with-http_postgres_module \
--with-http_random_index_module \
--with-http_realip_module \
--with-http_secure_link_module \
--with-http_ssl_module \
--with-http_stub_status_module \
--with-http_sub_module \
--with-http_xslt_module \
--with-http_v2_module \
--with-ipv6 

make
make install
apt-get -y autoclean
apt-get -y autoremove
```

- OpenResty 文档：[链接](http://openresty-reference.readthedocs.io/en/latest/Lua_Nginx_API/)