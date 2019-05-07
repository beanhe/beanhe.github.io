---
layout: post
title: ubuntu上制作qcow2镜像
category: openstack 
tags: [qcow2,openstack]
---

```
apt-get install qemu-kvm virt-viewer
qemu-img create -f qcow2 /tmp/trusty.qcow2 10G

virt-install --virt-type kvm --name utrusty --ram 1024 \
		--cdrom=/root/ubuntu-14.04.3-server-amd64.iso \
		--disk /tmp/trusty.qcow2,format=qcow2 \
		--network network=default

若出现报错：
Starting install…
ERROR internal error process exited while connecting to monitor: char device redirected to /dev/pts/2
kvm: -drive file=/home/muge0913/workstation/kvm/test.img,if=none,id=drive-ide0-0-0,format=raw: could not open disk image
/home/muge0913/workstation/kvm/test.img: Permission denied

则需要修改/etc/libvirt/qemu.conf文件加入如下内容：

user = "root"
group = "root"
dynamic_ownership = 0

修改后保存重启libvirt-bin

service libvirt-bin restart

然后开始安装
```
