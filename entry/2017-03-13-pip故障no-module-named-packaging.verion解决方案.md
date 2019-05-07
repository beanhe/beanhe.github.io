---
layout: post
title: pip故障'no module named packaging.verion'解决方案
category: python
tags: [__beanhe,pip,python]
---
    
#### ubuntu 14.04 pip命令报错：'ImportError: No module named packaging.version'

- 原因：pip包出现bug
- 修复方案：

```
wget https://bootstrap.pypa.io/get-pip.py
python get-pip.py
```