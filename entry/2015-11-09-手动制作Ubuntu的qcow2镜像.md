---
layout: post
title: 手动制作Ubuntu的qcow2镜像
category: kvm,openstack
tags: [__beanhe,kvm,qcow2]
---
    
- 前提：
	- 操作系统：Ubuntu(带图形化)
	- Ubuntu镜像：此处以14.04为例，其官方镜像下载地址为：[地址](http://archive.ubuntu.com/ubuntu/dists/trusty/main/installer-amd64/current/images/netboot/mini.iso)
- 步骤：
	- 安装kvm级virt-manager

	```
	apt-get install -y kvm virt-manager
	```

	- 下载Ubuntu镜像：

	```
	wget http://archive.ubuntu.com/ubuntu/dists/trusty/main/installer-amd64/current/images/netboot/mini.iso
	```

	- 创建qcow2格式的磁盘快文件

	```
	qemu-image create -f qcow2 /tmp/trusty.qcow2 10G
	```

	- 更改配置文件qemu.conf，将下面的几行注释去掉，并把`dynamic_ownership`的值改为0以禁止libvirtd动态修改文件的归属

	```
	# vi /etc/qemu.conf
	user = "root"
	group = "root"
	dynamic_ownership = 0
	service libvirt-bin restart
	```
	【注】如果没有更改上面的配置，执行下一步则会出现`Could not open '/root/images/mini.iso': Permission denied`的报错（即使是root用户）

	- 开始制作：

	```
	virt-install --virt-type kvm --name trusty --ram 1024 \
	--cdrom=./mini.iso --disk /tmp/trusty.qcow2,format=qcow2 \
	--network network=default --os-type=linux --os-variat=ubuntutrusty
	如果使用vnc进行远程制作，则在上面的命令中加上
	--graphics vnc,listen=0.0.0.0 --noautoconsole
	```

	- 接下来直接启动virt-manager图形化界面进行安装即可