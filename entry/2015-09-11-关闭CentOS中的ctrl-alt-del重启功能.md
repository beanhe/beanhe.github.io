---
layout: post
title: 关闭CentOS中的ctrl+alt+del重启功能
category: linux 
tags: [CentOS,inittab]
---
> 只需要注释相应文件中的配置行即可，并且修改文件后立即生效，不需要重启：

- 对于CentOS 6之前的版本，相关配置在`/etc/inittab`中，到此文件中注释如下行即可：

```
ca:12345:ctrlaltdel:/sbin/shutdown -t1 -a -r now
```

- 对于CentOS 6之后的版本，相关配置在`/etc/init/control-alt-delete.conf`中，到此文件中注释如下行即可：

```
start on control-alt-delete
```
