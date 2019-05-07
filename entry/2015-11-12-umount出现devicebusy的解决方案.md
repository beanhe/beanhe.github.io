---
layout: post
title: Umount出现device busy的解决方案
category: linux
tags: [linux,mount]
---
- 使用fuser命令将所有占用挂载目录的进程杀掉

	```
	fuser -m -k /mount/point
	```
