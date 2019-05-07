---
layout: post
title: Ubuntu安装Oracle-java
category: java
tags: [java]
---
- 添加ppa:

```
 sudo add-apt-repository ppa:webupd8team/java
```

- 更新源

```
sudo apt-get update
```

- 安装java

```
sudo apt-get install oracle-java8-installer
```

- 多版本切换

```
sudo update-java-alternatives -s java-8-oracle
```
