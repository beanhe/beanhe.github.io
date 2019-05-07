---
layout: post
title: Mac OS X安装MySQL-Python包
category: mac
tags: [__beanhe,mac,pytohn,mysql]
---
    
#### Mac OS X 安装MySQL-Python

- MySQL-Python依赖`mysql_config`，在Mac OS X中直接`pip install MySQL-Python`会报错`mysql_config not found`，所以首先需要安装依赖，在Mac OS X中依赖包可以使用`homebrew`安装，命令如下：

```
brew install mysql-connector-c
pip install MySQL-Python
```

- 在Ubuntu中依赖包为`libmysqlclient-dev`，故安装方式如下：

```
apt-get install -y libmysqlclient-dev
pip install MySQL-Python
```