---
layout: post
title: elasticsearch2.x以上版本安装bigdesk插件
category: elasticsearch
tags: [elasticsearch,bigdesk]
---
- Bigdesk是一个很实用的elasticsearch插件，用于实时监控elasticsearch服务器运行状况，包括JVM/CPU/OS以及其服务本身的相关指标，其界面如下图：
- ![预览图](/static/img/blog/elasticsearchBigdesk1.png)

- 但是随着elasticsearch更新（更新速度简直丧病啊），这个插件的官方版本已无法正常通过`bin/plugin install`命令进行安装了，然后这无法阻挡程序员的脚本，有位小伙伴已成功突破限制使其可以正常安装了，然而不幸的是，通过测试这个改良版虽然可以正常安装，但仍然无法正常使用，打开插件页面会有如下报错：
- ![报错](/static/img/blog/elasticsearchBigdesk2.png)

- 哈，不就是一个js弹窗吗，突破你焉用大神之手，说干就干，以下是elasticsearch2.x以上版本安装并正常使用bigdesk的整个过程，在这之前，先贴出原是插件和所使用插件的代码首页以示尊敬：
- 原始插件：[lukas-vlcek/bigdesk](https://github.com/lukas-vlcek/bigdesk)
- 本教程所基于插件：[jayant2014/bigdesk](https://github.com/jayant2014/bigdesk)

- 首先使用`elasticsearch`的`plugin`命令安装改良后的插件:

```
bin/plugin install jayant2014/bigdesk
```

- 安装完成后进入插件目录`plugins/bigdesk`，进入报错对应的js目录`_site/js/store`,打开`BigdeskStore.js`文件定位到142行，将其中的`major == 1`改为`major >= 1`保存退出即可，接下来就是见证奇迹的时刻（好土的台词啊，但还是要说的），打开`http://<ES_SERVER_IP>:<ES_SERVER_PORT>/_plugin/bigdesk`页面即可看到对应的实时监控页面了。OK, that's all,来，夸我夸我，O(∩_∩)O。
