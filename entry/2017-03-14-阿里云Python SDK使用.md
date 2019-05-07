---
layout: post
title: 阿里云Python SDK使用
category: 阿里云
tags: [__beanhe,python,阿里云]
---
    
#### 阿里云Python SDK使用
以`aliyun-python-sdk-ecs`为例，阿里云SDK使用分三步：
- 初始化Client
首先[点我](https://account.aliyun.com/login/login.htm?spm=5176.7926452.196042.3.UNlTh0&oauth_callback=http%3A%2F%2Fak-console.aliyun.com%2Findex)获取自己的access ID/SECRET，然后初始化Client
```
from aliyunsdkcore import client
clt = client.AcsClient('<你的access id>', '<你的access secret>', '<默认的regionid>')
```
- 初始化request
以获取instance列表为例
```
from aliyunsdkecs.request.v20140526 import DescribeInstancesRequest
request = DescribeInstancesRequest.DescribeInstancesRequest()
request.set_accept_format('json')
```
- 发起调用
```
result = clt.do_action(request)
```
所有请求均按照这个步骤来调用