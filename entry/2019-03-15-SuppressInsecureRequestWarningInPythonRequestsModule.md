---
layout: post
title: 禁止Python requests输出Suppress InsecureRequestWarning: Unverified HTTPS request提醒信息
category: python
tags: [__beanhe,python]
---

### 禁止Python requests输出`InsecureRequestWarning: Unverified HTTPS request`提醒信息

- 背景：在使用`Python`的`requests`模块发送`HTTPS`请求时，可以禁用`SSL`认证，`requests`里面直接指定:`verify = False`即可，但是即便已经指定，`Python`仍然会出现`InsecureRequestWarning: Unverified HTTPS request`的提醒，一下是禁止次提醒的方案
- 方案：
	- 通用方案：设置环境变量`export PYTHONWARNINGS="ignore:Unverified HTTPS request"`
	- 根据`requests`版本使用代码禁止：
		- 当`requests`版本`>=2.16.0`时，禁止代码如下：
		
		```
		import urllib3
		urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)
		```
		
		- 当`requests`版本`<2.16.0`时，代码入下：
		
		```
		import requests
		from requests.packages.urllib3.exceptions import InsecureRequestWarning
		requests.packages.urllib3.disable_warnings(InsecureRequestWarning)
		```