---
layout: post
title: python insecure platform warning
category: python
tags: [python,error]
---
> Python 软件包运行或者pip安装的时候出现如下报错：

```
/usr/lib/python2.6/site-packages/pip-7.1.2-py2.6.egg/pip/_vendor/requests/packages/urllib3/util/ssl_.py:90: InsecurePlatformWarning: A true SSLContext object is not available. This prevents urllib3 from configuring SSL appropriately and may cause certain SSL connections to fail. For more information, see https://urllib3.readthedocs.org/en/latest/security.html#insecureplatformwarning.
```
- 解决方案：

```
pip install -U requests[security]

```
