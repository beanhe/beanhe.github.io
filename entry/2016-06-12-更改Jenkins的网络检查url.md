---
layout: post
title: 更改Jenkins网络连接检查URL
category: jenkins 
tags: [jenkins]
---
- 默认情况下Jenkins通过google来检查网络连接，但是在大天朝，google被一些众所周知的原因和谐了，因此要想再墙内正常使用jenkins，可以通过修改配置文件来改变这个检查URL，修改方法为，将`/var/lib/jenkins/updates/default.json`文件中的`"connectionCheckUrl":"http://www.google.com/"`更改为`"connectionCheckUrl":"http://www.baidu.com/"`即可，更改完成后需要重启生效
