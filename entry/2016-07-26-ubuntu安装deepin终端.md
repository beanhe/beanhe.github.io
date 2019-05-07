---
layout: post
title: ubuntu安装deepin终端
category: linux 
tags: [Ubuntu,Mint,deepin]
---
- [Deepin Linux](https://www.deepin.org/)是由我大天朝的深之度科技有限公司(来自大武汉,没错,就是那个以前做盗版windows的深度)基于linux开发出来的操作系统,目前在国际小有名气,并不像某些拿着政府补贴的公司:做一个主题就对外吹嘘是一个自主研发的操作系统,喜欢Linux/Unix操作系统的各位小伙伴可以尝试下,绝对不会让你失望,在这套真正的国产操作系统里面有很多优秀的软件,比如可以正常运行的QQ,LINUX版本的WPS,LINUX版本的网易云音乐以及咱正要讨论的深度终端,不知道有没有小伙伴使用过Mac OS X中的[item2](https://www.iterm2.com/)终端,那家伙简直是DevOps神器啊,可惜这东西只支持Mac系统,不过别担心,这不是有深度么

- 深度终端很简洁,在Ubuntu中(此方法适用Ubuntu 14.10 Utopic/14.04 Trusty/12.04 Precise/Linux Mint 17.1等linux发行版)安装深度终端方法也很简单,就是常用的添加其对应的PPA源,安装即可,具体如下:

	```
	sudo add-apt-repository ppa:noobslab/deepin-sc
	sudo apt-get update
	sudo apt-get install deepin-terminal
	```
- 安装完成后可以使用快捷键操作(支持自定义),下面是一些快捷键的截图:
	![Deepin终端快捷键](https://wiki.deepin.org/images/1/10/Deepin-terminal1.png)

- 支持多workspace,使用`Ctrl+/`组合键可以新建workspace,使用`Ctrl+Shift+<`和`Ctrl+Shift+>`组合键可以在workspace间切换,如图:
	![Deepin多workspace](/static/img/blog/deepinTerminal1.png)

- 支持保存常用的SSH登陆,直接上图:
	![Deepin保存SSH连接信息](/static/img/blog/deepinTerminal2.png) 

- 另外深度终端还支持一个特有的quake模式,这个模式可以让你直接进入全屏模式,或者说叫HUD模式,此时整个终端铺满全屏切半透明,在某些特定情况下很实用,进入方式是直接使用名ing` deepin-terminal --quake mode`回车进入

- 总而言之,深度终端在传统终端基础上加了很多实用功能,值得推荐!
