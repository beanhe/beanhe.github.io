---
layout: post
title: Redis Getshell漏洞
category: redis,db
tags: [__beanhe,redis,db]
---

### Redis getshell漏洞

#### 环境：
  - OS: 阿里云Ubuntu 14.04
  - Redis版本: 3.2.8； Cluster: 1 Master + 1 Slave

#### 故障描述和解决
- 开发同事反应`API`报错，信息如下：

```
-MISCONF Redis is configured to save RDB snapshots, but is currently not able to persist on disk. Commands that may modify the data set are disabled. Please check Redis logs for details about the error.
```

- 很明确是`redis`故障，跑到服务器查看`redis`发现两台服务均运行中，转而查看日志，发现`slave`确实存在上述报错，`master`也有报错，内容不同，如下：

```
13947:C 21 May 06:25:20.054 # Failed opening the RDB file root (in server root dir /var/spool/cron) for saving: Permission denied
22680:M 21 May 06:25:20.154 # Background saving error
```

- 网络上关于`background saving error`有解决方案如下，这个方案看起来可以使`redis`停止报错，但是并未从根本解决，因为它只是让`redis`遇到报错时停止写入，也就是直接忽略这个错误而已

```
redis 127.0.0.1:6379> config set stop-writes-on-bgsave-error no
```

- 继续来看`master`日志，感觉很疑惑，因为我的`RDB file`是默认的位于`/var/lib/redis`目录下的`dump.rdb`文件，可是报错中的`background save`的目标却是`/var/spool/cron`目录下的`root`文件，查看两台服务器的配置文件也没有错，再通过`config get`登入`redis`分别查看，发现`slave`的`config get dir`确实是`/var/spool/cron`，`config get dbfilename`结果为`root`，`master`正常
- 接着查看`slave`的`/var/lib/redis`目录发现了`root`文件和另外一个很危险的非空文件`authorized_keys`，然后再去`google`了一下这个报错，发现一个官方[issue](https://github.com/antirez/redis/issues/3594)，看来是中招了，还好是才开的阿里云测试服务器并未有任何业务数据，而且`redis-server`默认的运行用户是`redis`而不是`root`，幸免于难。
- 故障发生的原因很明显，就是因为`redis-server`运行的太`open`了，外网可以自由访问，并且没有配置密码，这种故障会导致`redis`被`block`中，无法处理和响应请求，无法通过`service redis-server stop`这种方式停止，只能使用`kill -9`来关闭：

```
Is your Redis server accessible to the public? If so, this could be an
attempted attack.

On Nov 6, 2016 5:11 PM, "azurit" notifications@github.com wrote:

We are experiencing really weired behavior of Redis server, which is
randomly appearing (at least it looks like that). When problem happens,
Redis is locked, not responding and even won't stop (must be killed with
signal 9). This is appearing in logs until server is killed and started
again:

20704:M 06 Nov 14:41:17.090 * 10 changes in 300 seconds. Saving...
20704:M 06 Nov 14:41:17.091 * Background saving started by pid 5333
5333:C 06 Nov 14:41:17.091 # Failed opening the RDB file root (in server
root dir /var/spool/cron) for saving: Read-only file system
20704:M 06 Nov 14:41:17.191 # Background saving error

Note that error 'Read-only file system' means that systemd is not allowing
Redis to write to that directory (it is running in different mount
namespace, the filesystem is really mounted in read-write mode).

I was also strace-ing it and Redis is really trying to create temporary
files in /var/spool/cron and is ignoring settings in configuration file:
dbfilename dump.rdb
dir /var/lib/redis

System is Debian Jessie, package is from dotdeb.org, 64bit OpenStack VPS.

Any hints?

—
You are receiving this because you are subscribed to this thread.
Reply to this email directly, view it on GitHub
#3594, or mute the thread
https://github.com/notifications/unsubscribe-auth/AFx1_KhYzEZupnh0VfATUQqnHbBfH3YEks5q7e4PgaJpZM4KqlBk
.
```


- 既然是由于运行配置过于`开放`产生的故障，所以避免方式也很简单，我的解决步骤也比较粗暴：
  - 删除非法文件：`/var/lib/redis/root`，`/var/lib/redis/authorized_keys`
  - 备份好`/var/lib/redis`目录
  - `kill -9`停止服务
  - 配置`iptables`规则只放行两台`服务器`之间的`redis`流量
  - 重新启动`redis`服务