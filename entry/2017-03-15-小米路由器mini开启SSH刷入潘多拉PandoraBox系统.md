---
layout: post
title: 小米路由器mini开启SSH，刷入潘多拉(PandoraBox)系统 
category: 刷机
tags: [__beanhe,路由器,ssh,刷机]
---

#### 小米路由器mini开启SSH，刷入潘多拉(PandoraBox)系统

_已测试过[OpenWrt](https://openwrt.org/)，无法成功，刷入后mini一直红灯，数次测试，成功刷入[PandoraBox](http://downloads.openwrt.org.cn/PandoraBox/)，具体版本直接点击本文链接下载即可_

#### 准备
- 刷机前做好备份，刷机失败直接到[官网](http://miwifi.com/miwifi_download.html)下载系统镜像刷回去（有配置备份直接页面操作恢复配置）即可
- 需要的软件包：
 - [系统包](http://bigota.miwifi.com/xiaoqiang/rom/r1cm/miwifi_r1cm_all_46ed1_0.4.36.bin "miwifi_r1cm_all_46ed1_0.4.36.bin") （其他版本均被做了手脚，无法成功输入SSH）
 - [SSH补丁](http://d.miwifi.com/rom/ssh)，记住页面里的root密码，后面要用到
 - [PandoraBox系统镜像](http://downloads.openwrt.org.cn/PandoraBox/Xiaomi-Mini-R1CM/testing/PandoraBox-ralink-mt7620-xiaomi-mini-squashfs-sysupgrade-r876-20150520.bin)
 - SSH客户端：
   - `Windows` : PuTTY, XShell, GitBash, WinScp等
   - Unix/Linux/Mac等`*nix`: 自带SSH客户端（本文以此为例，Windows用户自行google对应软件使用方法）
   
- 注：下载的数据包除了潘多拉(PandoraBox)镜像可以自定义命名，其他均要使用文中名称，否则会失败
 
#### 步骤
 - 先刷入系统包，然后刷入SSH包，**顺序不能错，方法都一样，只是文件名不同**。刷系统包和SSH补丁包步骤如下：
  - 将下载的工具包bin文件复制到U盘（FAT/FAT32格式）的根目录下，保证文件名为：**系统包：miwifi.bin; SSH补丁：miwifi_ssh.bin**
  - 断开小米路由器的电源，将U盘插入USB接口；
  - 按住reset按钮之后重新接入电源，指示灯变为黄色闪烁状态即可松开reset键；
  - 等待一段时间后（系统包大概几分钟，SSH补丁几十秒）安装完成之后，小米路由器会自动重启，之后可以连入默认WIFI进行初始化配置也可以不配置。然后在你下载SSH补丁的页面获取root密码之后您就可以准备刷`Pandora`了

- 使用SSH客户端工具将`PandoraBox`镜像（下文用`pandorabox.bin`代替）传入路由器的`/tmp`目录下，如`scp`（windows中的客户端均可实现）命令：

```
scp pandorabox.bin root@192.168.31.1:/tmp
```

- 使用`ssh`客户端进入路由器，刷入`pandora.bin`，如使用`ssh`（windows中的客户端均可实现）命令：

```
ssh root@192.168.31.1
mtd -r write /tmp/pandorabox.bin firmware
```

输入以上命令后就等他自己操作了，刷完会有`Rebooting...`提示，此时等他重启直到路由器指示灯变成蓝色即可完成，默认的`WIFI`热点名类似`PandoraBox_xxxxx`的格式，不需要密码，连上后通过`192.168.1.1`进入管理页面，默认用户名/密码为`root/admin`，至此刷机完成
