---
layout: post
title: ubuntu中update-rc.d工具使用
category: linux
tags: [__beanhe,ubuntu,linux,system-v]
---
    
#### Ubuntu中System-v启动脚本管理工具update-rc.d

使用`update-rc.d`之前先要确保脚本存放在`/etc/ini.t.d`目录下
简单讲服务(以foobar为例)加入启动项直接使用命令:

```
update-rc.d foobar defaults
```
`defaults`参数代表指定的服务会在`2345`的运行级别(runlevel)启动，在`016`的运行级别关闭，并且默认的启动顺序和关闭顺序均为`20`，更详细的指定脚本启动和关闭的`runlevel`可以使用如下命令：

```
update-rc.d foobar start 20 2 3 4 5 . stop 20 0 1 6 .
```
上面的命令代表`foobar`在`2345`运行级别下会启动并且启动顺序为20，在`016`运行级别下回关闭且关闭顺序为20。

要移除系统启动或关闭的脚本可使用命令选项`remove`，如下

```
update-rc.d -f foobar remove
```

添加启动或关闭脚本后会在对应级别的脚本目录`/etc/rc?.d`创建对应的软连接，其中`?`代表运行级别。通过`remove`命令移除后其中对应的软连接也会被移除，所以可以连接为只要`/etc/rc?.d`中有对应脚本就会在对应级别生效