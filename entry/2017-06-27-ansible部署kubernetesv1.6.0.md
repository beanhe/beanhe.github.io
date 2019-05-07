---
layout: post
title: ansible部署kubernetesv1.6.0
category: ansible,kubernetes,docker
tags: [__beanhe,ansible,kubernetes,docker]
---

### 使用Ansbile在Ubuntu14.04部署全功能Kubernetes

#### 背景

- 由于GFW的种种限制加上k8s本身部署的复杂性，想要在`Ubuntu14.04`部署`kubernetes`确实很麻烦，作为一个合格的`DevOps Engineer`:joy: :joy:，第一个想到的就是自动化实现了，所以就通过多次的修改和测试，发布了github上的[这个库](https://github.com/beanhe/k8s-deploy)。

#### 准备

- `docker私有库`(Private Registry)：用来存储被`***`的那些`gcr.io`镜像
- `Samba服务器`(共享服务器)：用来临时存储公共配置文件，也可以使用其他方式
- `下载服务器`：用来存储被`***`的那些软件包（如kubernetes-server-linux-amd64.tar.gz等）
- `ansbile`：不低于2.2.0.0
- 系统：`ubuntu14.04`
- 需要的软件包（请科学上网后下载至本地的下载服务器）：
	- https://storage.googleapis.com/kubernetes-release/release/v1.6.0/kubernetes-server-linux-amd64.tar.gz
	- https://github.com/coreos/etcd/releases/download/v3.1.5/etcd-v3.1.5-linux-amd64.tar.gz
	- https://github.com/coreos/flannel/releases/download/v0.7.1/flannel-v0.7.1-linux-amd64.tar.gz
	- https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
	- https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
	- https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64
- 需要的`docker`镜像（请科学上网后上传至本地的`docker registry`）：
	- gcr.io/google_containers/pause-amd64:3.0
	- gcr.io/google_containers/kubernetes-dashboard-amd64:v1.6.0
	- gcr.io/google_containers/k8s-dns-kube-dns-amd64:1.14.1
	- gcr.io/google_containers/k8s-dns-dnsmasq-nanny-amd64:1.14.1
	- gcr.io/google_containers/k8s-dns-sidecar-amd64:1.14.1
	- gcr.io/google_containers/elasticsearch:v2.4.1-2
	- gcr.io/google_containers/fluentd-elasticsearch:1.22
	- gcr.io/google_containers/kibana:v4.6.1-1
	- gcr.io/google_containers/heapster-grafana-amd64:v4.0.2
	- gcr.io/google_containers/heapster-amd64:v1.3.0-beta.1
	- gcr.io/google_containers/heapster-influxdb-amd64:v1.1.1

#### 功能

- 安装了`kubernetes`全部必要组件：`etcd`, `kube-apiserver`, `kube-controller-manager`, `kube-scheduler`, `kubelet`, `kube-proxy`
- 实现了`flanneld`对`docker`的网络控制
![](/files/201706/27171303482_flanneld.png)

- 实现了`k8s`的`RBAC`认证
![](/files/201706/27171421658_rbac.png)

- 实现了`dashboard`的部署
![](/files/201706/27171204746_dashboard1.png)

- 实现了基于`EFK`的日志收集
![efk](/files/201706/27171048033_efk.png "efk")

- 实现了 `kube-dns`的部署
![](/files/201706/27171447754_cluster-info.png)

- 实现了 `heapster`的部署
![](/files/201706/27171133579_grafana.png)

#### 部署方法

- `git clone https://github.com/beanhe/k8s-deploy.git`
- 按你的环境修改`group_vars/all.yml`中的变量，其中变量值形如`<xxxxx>`的是需要修改的， 其他变量可以根据自己的需求修改
	- 注：涉及网段和IP的请务必修改的与本地不冲突
- 根据你的`ansible`配置修改`ansible.cfg`
- 根据本地环境修改`hosts`文件中的IP地址
- 用本地环境中`ansible`的SSH私钥替换`conf/id_rsa`
- 运行命令`ansible-playbook playbook.yml`等待完成部署