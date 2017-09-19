---
title: install-ceph-Luminous-on-centos7-with-ceph-deploy
date: 2017-09-19 18:14:52
tags:
---
Hostname | IP Address | Role | OS
-|-|-|-
ceph-node1 | 10.211.55.81 | deploy+mon1+osd1 | CentOS 7.4.1708
ceph-node2 | 10.211.55.82 | mon2+osd2 | CentOS 7.4.1708
ceph-node3 | 10.211.55.83 | mon3+osd3 | CentOS 7.4.1708
# 1.预检
## 1.1 设置所有节点的 yum 仓库
```
yum clean all
rm -rf /etc/yum.repos.d/*.repo
curl http://mirrors.aliyun.com/repo/Centos-7.repo > /etc/yum.repos.d/CentOS-Base.repo
curl http://mirrors.aliyun.com/repo/epel-7.repo > /etc/yum.repos.d/epel.repo
yum makecache
```
## 1.2 安装 CEPH 部署工具
把 Ceph 仓库添加到 ceph-deploy 管理节点，然后安装 ceph-deploy 。
### 1.2.1 把软件包源加入软件仓库
```
sudo vim /etc/yum.repos.d/ceph.repo

[ceph-noarch]
name=Ceph noarch packages
baseurl=http://download.ceph.com/rpm-luminous/el7/noarch
enabled=1
gpgcheck=1
type=rpm-md
gpgkey=https://download.ceph.com/keys/release.asc
```
### 1.2.2 更新软件库并安装 ceph-deploy
```
sudo yum update
sudo yum install -y ceph-deploy
```
## 1.3 CEPH 节点安装
### 1.3.1 安装 NTP
在所有 Ceph 节点上安装 NTP 服务（特别是 Ceph Monitor 节点），以免因时钟漂移导致故障
```
sudo yum install -y ntp ntpdate ntp-doc 
ntpdate 0.cn.pool.ntp.org
```
### 1.3.2 部署节点无密码登录所有 Ceph 节点
```
ssh-keygen
ssh-copy-id root@10.211.55.81
ssh-copy-id root@10.211.55.82
ssh-copy-id root@10.211.55.83
```
### 1.3.3 修改hosts文件
```
vim /etc/hosts

10.211.55.81 ceph-node1
10.211.55.82 ceph-node2
10.211.55.83 ceph-node3
```
### 1.3.4 关闭 SELINUX
```
sudo setenforce 0
```
# 2. 创建集群
## 2.1 创建集群
```
ceph-deploy new ceph-node1 ceph-node2 ceph-node3
```
## 2.2 安装 Ceph
```
ceph-deploy install  ceph-node1 ceph-node2 ceph-node3  --repo-url=http://mirrors.aliyun.com/ceph/rpm-luminous/el7/ --gpg-url=http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-7
```
## 2.3 配置初始 monitor(s)、并收集所有密钥
```
ceph-deploy mon create-initial
```
## 2.4 擦净磁盘
```
ceph-deploy disk zap ceph-node1:sdb
ceph-deploy disk zap ceph-node2:sdb
ceph-deploy disk zap ceph-node3:sdb
```
## 2.5 创建 OSD
```
ceph-deploy osd create ceph-node1:sdb
ceph-deploy osd create ceph-node2:sdb
ceph-deploy osd create ceph-node3:sdb
```
## 2.6 
```
ceph-deploy admin ceph-node1 ceph-node1 ceph-node2 ceph-node3
```
## 2.7
```
ceph-deploy mgr create ceph-node1
```
