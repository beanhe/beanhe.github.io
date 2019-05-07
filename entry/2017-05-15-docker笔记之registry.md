---
layout: post
title: docker笔记之registry
category: docker,虚拟化
tags: [__beanhe,docker,虚拟化]
---

##### docker命令笔记 —— register

- 认证Private docker registry

```
# -u指定用户名
# https://hub.kubiops.com为private docker registry地址
docker login -u docker https://hub.kubiops.com
```

- 输入以上命令回车后，会提示输入用户名和密码，成功后会有`Login Succeeded`提示，最终会在当前用户的`~`目录的`.docker`目录下创建配置文件`config.json`，内容形如：

```
root@default:~/.docker# cat config.json
{
	"auths": {
		"hub.kubiops.com": {
			"auth": "ZG9ja2VyOmFsbGlxbW9uZXk="
		}
	}
}
```

- push本地image到private registry

```
# 查看本地image列表
root@default:~# docker images
REPOSITORY                                            TAG                 IMAGE ID            CREATED             SIZE
kubeguide/redis-master                        latest              405a0b586f7e        21 months ago     419 MB

# 这里以push kubeguide/redis-master:latest为例
# 首先使用docker tag 命令重命名image，加上目标registry地址
root@default:~# docker tag 405a0b586f7e hub.kubiops.com/kubeguide/redis-master:latest
# 命令中的新image名(kubeguide/redis-master)和tag(latest)不必与本地的保持一致，示例中简单点，保持一致，运行这条命令后再检查本地的image列表结果如下，发现多了一条记录，除了repository列不同，其他都一样
root@default:~# docker images
REPOSITORY                                            TAG                 IMAGE ID            CREATED             SIZE
hub.kubiops.com/kubeguide/redis-master             latest              405a0b586f7e        21 months ago       419 MB
kubeguide/redis-master                                latest              405a0b586f7e        21 months ago       419 MB

# 最后push这条新的image
root@default:~# docker push hub.kubiops.com/kubeguide/redis-master
The push refers to a repository [hub.kubiops.com/kubeguide/redis-master]
62432aff2935: Pushed
5f70bf18a086: Mounted from kubeguide/guestbook-php-frontend
9c3b55ccdff5: Pushed
a49f65f29e42: Pushed
914412fa1ebd: Pushed
7c52ec8c6e19: Pushed
03ad52bfffbb: Pushed
706ff50bb519: Pushed
32cb0ce92981: Pushed
541bc77c741d: Pushed
bf721bc7e47e: Pushed
1bff2ae568be: Pushed
latest: digest: sha256:0084ebf7b2a1217858372944d5b2470d03b4cdc5692d3b91f4b5acc4d694473f size: 4882
```

- 远程查看private registry的image列表

```
[root@CentOS-7 ~]# curl -s https://<username>:<password>@hub.kubiops.com/v2/_catalog |python -m json.tool
{
    "repositories": [
        "kubeguide/redis-master"
    ]
}
```

- 从private registry pull image

```
# 别忘了如果有认证需要先docker login，认证成功后执行下面的操作
docker pull hub.kubiops.com/kubeguide/redis-master
```