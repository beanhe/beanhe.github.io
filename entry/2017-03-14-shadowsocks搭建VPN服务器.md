---
layout: post
title: shadowsocks搭建socks5代理服务器
category: python
tags: [__beanhe,python,shadowsocks,vpn,socks5]
---
    
##### 使用shadowsocks配置socks5代理服务器

- 常见VPN服务器搭建较繁琐，`shadowsocks`提供了很简便的方式，具体使用参考如下步骤

- shadowsocks官网：[点我](https://shadowsocks.org)
- 安装shadowsocks

```
sudo pip install shadowsocks
```

- 创建服务器配置文件`shadowsocks.json`,按如下内容进行配置

```
{
    "server":"<对外端口IP地址>",
    "server_port":<服务器端口>,
    "local_port":9080,
    "password":"<访问密码>",
    "timeout":600,
    "method":"rc4-md5"
}
```

- 配置文件中`server`用于指定对外网口的IP地址，`server_port`用于指定对外服务端口，`local_port`指定本地端口，`password`指定服务器访问密码，`timeout`指定连接超时时间，单位`秒`，`method`用于指定加密算法，支持"bf-cfb", "aes-256-cfb", "des-cfb", "rc4", etc. Default is table, which is not secure. "aes-256-gcm"等各种类型加密算法

- 配置完成后即可使用命令如下命令启动服务：

```
sudo ssserver -c shadowsocks.json -d start
```

- 连接`shadowsocks`很方便，只需要在设备安装对应的客户端，各平台客户端下载地址请点击[官网](http://shadowsocks.org/en/download/clients.html)，配置输入认证信息和认证方式即可进行连接