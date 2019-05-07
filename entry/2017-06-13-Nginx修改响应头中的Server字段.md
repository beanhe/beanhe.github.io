---
layout: post
title: Nginx修改Response Header中的Server字段
category: web
tags: [__beanhe,web,nginx]
---
    
#### Nginx修改Response Header中的Server字段

- 先看一段示例：

```
$ curl -I http://www.dictionary.com/
HTTP/1.1 200 OK
Content-Length: 51257
Content-Type: text/html; charset=utf-8
Server: Microsoft-IIS/8.0
X-AspNet-Version: 4.0.30319
X-AspNetMvc-Version: 5.2
X-Powered-By: ASP.NET
X-Varnish: 124740207 32891
Expires: Tue, 13 Jun 2017 06:27:45 GMT
Cache-Control: max-age=0, no-cache, no-store
Pragma: no-cache
Date: Tue, 13 Jun 2017 06:27:45 GMT
Connection: keep-alive
Set-Cookie: mseg=71; expires=Fri, 11-Jun-2027 06:27:45 GMT; path=/; domain=.dictionary.com
X-Check-Cacheable: YES

$ curl -I http://www.kubiops.com/
HTTP/1.1 200 OK
Server: nginx
Date: Tue, 13 Jun 2017 06:23:15 GMT
Content-Type: text/html; charset=UTF-8
Content-Length: 33988
Connection: keep-alive

$ curl -I https://www.baidu.com/
HTTP/1.1 200 OK
Server: bfe/1.0.8.18
Date: Tue, 13 Jun 2017 06:22:03 GMT
Content-Type: text/html
Content-Length: 277
Connection: keep-alive
Last-Modified: Mon, 13 Jun 2016 02:50:43 GMT
ETag: "575e1f83-115"
Cache-Control: private, no-cache, no-store, proxy-revalidate, no-transform
Pragma: no-cache
Accept-Ranges: bytes

```

- 上面代码中分别使用`curl`命令访问`www.dictionary.com`、本站和百度，通过`-I`参数使结果只显示响应头部，注意结果中的`Server`字段，分别是`Microsoft-IIS/8.0`，`nginx`，`bfe/1.0.8.18`。很明显`dictionary`站点首页服务器是Windows系统，且使用IIS作为Web服务器，并且版本为8.0，服务器的信息一览无遗，本站则使用`nginx`，百度的`bfe/1.0.8.18`如果不深究，完全不知道是什么，其实这个信息对于普通用户来讲并没有什么用处，但是对于一些攻击者来说，就大有用处了，特别是`dictionary`主站，这个字段将系统类型，Web服务器软件以及版本完全暴露出来，显然很不安全，如果此服务器版本有已知漏洞暴露出来，那么直接通过这个字段就可以进行攻击；另外两个则相对安全，起码可以有缓冲时间来预防，本站暴露了Web服务器软件为`nginx`其他信息则无法通过此方法判断（当然有其他方案获取系统版本，不在本文讨论范围内），百度的结果则无法直接判断系统类别，只能了解服务器软件为`bfe`版本为`1.0.8.19`，显然不是常见Web服务器，若要攻击这个字段对于不了解`bfe`的攻击者来说显然无用。由此可见，这个`Server`字段的结果对于`Web`服务器的安全防御有直接的影响，对于`Nginx`来说，默认是会显示其版本的，本站之所以只显示了`nginx`，其实是通过配置文件来实现的：只需要将`nginx`配置中`server_tokens`值设置为`off`然后重启`nginx`即可对服务器版本进行隐藏（如本站）。但是无法对`nginx`这个值进行修改或隐藏

- 如果要实现对`Server`字段的修改，有如下几种常见方案：

##### 方案一： 修改源码，重新编译安装
  - 其实在`nginx`的源码中，`Server`字段是写死在`src/http/ngx_http_header_filter_module.c`中的，相关内容如下：

```
static u_char ngx_http_server_string[] = "Server: nginx" CRLF;
static u_char ngx_http_server_full_string[] = "Server: " NGINX_VER CRLF;
static u_char ngx_http_server_build_string[] = "Server: " NGINX_VER_BUILD CRLF;
```

  - 既然如此，如果要修改这个字段结果，则只需要修改这个文件内容然后重新编译即可实现，如将上述文件修改为：

```
static u_char ngx_http_server_string[] = "Server: KubiopsServer" CRLF;
static u_char ngx_http_server_full_string[] = "Server: " NGINX_VER CRLF;
static u_char ngx_http_server_build_string[] = "Server: " NGINX_VER_BUILD CRLF;
```

##### 方案二：使用ngx_header_more模块
  - 还有另外一种方式可以对此字段进行随时自定义：使用[ngx_header_more](https://github.com/openresty/headers-more-nginx-module#installation)模块(./configure --prefix=/opt/nginx --add-module=/path/to/headers-more-nginx-module)，通过这种访问编译安装好`nginx`，则可以直接在配置文件加入如下配置即可实现对`Server`字段的自定义：

```
server_tokens off;
more_set_headers 'Server: KubiopsServer';
```

  - 注意，上述方案需要`perl`的支持

- 当然还可以使用一些在`nginx`基础上进行修改过并且集成了`ngx_header_more`模块或有相同功能模块的软件，如`OpenResty`等Nginx修改Response Header中的Server字段