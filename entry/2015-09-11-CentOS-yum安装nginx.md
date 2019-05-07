---
layout: post
title: CentOS yum安装nginx
category: web
tags: [CentOS,yum,nginx]
---
- 先添加nginx的yum源

```
rpm -ivh http://nginx.org/packages/centos/6/noarch/RPMS/nginx-release-centos-6-0.el6.ngx.noarch.rpm
```

- 然后就可以进行安装了 

```
yum install -y nginx
```
