---
layout: post
title: 阿里云命令行工具aliyuncli
category: 阿里云
tags: [__beanhe,阿里云,python]
---

#### 阿里云命令行工具

阿里云命令行工具使用Python编写，Python2.6.5以上版本均可安装使用,通常直接使用`pip`即可
```
sudo pip install aliyuncli
```
在Ubuntu 14.04中由于pip安装Python `lib`路径与系统的不同，故安装完成后会出现不能正常使用的情况，不论输入什么命令，结果都是如下内容：
```
usage: aliyuncli <command> <operation> [options and parameters]
<aliyuncli> the valid command as follows:
```
解决方法如下（**注意由于所有阿里云Python安装包的路径都有这个故障，因此只要通过`pip`命令安装了新的阿里云模块都要执行如下命令，否则新安装的内容无法再命令行中正常使用**）：
```
cp -r /usr/local/lib/python2.7/dist-packages/aliyun* /usr/lib/python2.7/dist-packages/
```
解决后就可以使用命令行对阿里云的各种资源进行管理了，在这之前还需要进行认证信息的配置，每个阿里云账号均有一对认证`ID/SECRET`，可以进入自己的阿里云账号（[点我](https://account.aliyun.com/login/login.htm?spm=5176.7926452.196042.3.UNlTh0&oauth_callback=http%3A%2F%2Fak-console.aliyun.com%2Findex)）进行查看（首次查看或者长时间未登录后查看都需要短信认证），获得认证信息后，就可以使用`aliyuncli configure`命令进行配置了，命令输完回车后会有交互式的配置信息出现，如下：
```
$ aliyuncli configure
Aliyun Access Key ID [None]: <输入你的认证ID>
Aliyun Access Key Secret [None]: <输入你的认证SECRET>
Default Region Id [None]: <输入默认的Region>
Default output format [None]: <输入默认的命令结果输出格式(json/xml等)>
```
配置后会在当前用户的家目录创建一个配置目录`.aliyuncli`里面有对应的配置信息
初始化配置后并没有多少功能，要进行云资产管理，还需要安装对应的`sdk`之后才能使用`aliyuncli`命令行对对应的资源进行管理，阿里云Python SDK列表请参考：[链接](https://help.aliyun.com/document_detail/30003.html)
例如如果我们需要通过命令管理云服务器（ECS）则还需要安装`ecs`对应的`sdk`，即`pip install aliyun-python-sdk-ecs`，安装完成后即可进行云服务器的管理，如：
```
$ aliyuncli Ecs DescribeInstances
$ aliyuncli Ecs StartInstance --InstanceId your_instance_id
```