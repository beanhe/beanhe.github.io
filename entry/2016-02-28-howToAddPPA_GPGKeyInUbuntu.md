---
layout: post
title: howToAddPPA_GPGKeyInUbuntu
category: linux 
tags: [ubuntu]
---
- 常用方法：使用add-apt-repository（以添加nginx stable源为例）：

```
add-apt-repository ppa:nginx/stable
```

- 使用apt-key命令：

```
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys C300EE8C 
```
- 上面命令的最后一个字段在[Lauchpad](https://launchpad.net)对应的项目页面中找到，如[https://launchpad.net/~nginx/+archive/ubuntu/stable](https://launchpad.net/~nginx/+archive/ubuntu/stable)中，对应内容如下：

```
Signing key:
1024R/C300EE8C
```

- 取`/`后的那串key即可
