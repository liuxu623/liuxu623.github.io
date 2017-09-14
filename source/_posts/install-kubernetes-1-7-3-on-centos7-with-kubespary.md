---
title: 使用kubespary在centos7上安装kubernetes集群
date: 2017-09-14 18:59:00
tags: 
- kubernetes
- k8s
- kubespary
---
### 1. 节点信息
IP Address | Role | OS
-|-|-
10.211.55.2 | deploy  | macOS Sierra 10.12.6
10.211.55.73 | etcd1 | CentOS 7.3.1611
10.211.55.74 | etcd2 | CentOS 7.3.1611
10.211.55.75 | etcd3 | CentOS 7.3.1611
10.211.55.76 | master1 | CentOS 7.3.1611
10.211.55.77 | master2 | CentOS 7.3.1611
10.211.55.78 | master3 | CentOS 7.3.1611
10.211.55.79 | node1 | CentOS 7.3.1611
10.211.55.80 | node2 | CentOS 7.3.1611
### 2. 在部署节点上安装ansible
```
brew install ansbile
```
### 3. 免密码登录所有节点
```
ssh-copy-id root@10.211.55.73
ssh-copy-id root@10.211.55.74
ssh-copy-id root@10.211.55.75
ssh-copy-id root@10.211.55.76
ssh-copy-id root@10.211.55.77
ssh-copy-id root@10.211.55.78
ssh-copy-id root@10.211.55.79
ssh-copy-id root@10.211.55.80
```
### 4. 下载kubespray
```
git clone https://github.com/kubernetes-incubator/kubespray.git
```
### 5. 修改inventory.cfg
```
cd kubespray
vim inventroy/inventroy.cfg

[all]
etcd1    ansible_host=10.211.55.64 ansible_user=root ip=10.211.55.64
etcd2    ansible_host=10.211.55.65 ansible_user=root ip=10.211.55.65
etcd3    ansible_host=10.211.55.66 ansible_user=root ip=10.211.55.66
master1    ansible_host=10.211.55.67 ansible_user=root ip=10.211.55.67
master2    ansible_host=10.211.55.68 ansible_user=root ip=10.211.55.68
master3    ansible_host=10.211.55.69 ansible_user=root ip=10.211.55.69
node1    ansible_host=10.211.55.70 ansible_user=root ip=10.211.55.70
node2    ansible_host=10.211.55.71 ansible_user=root ip=10.211.55.71

[etcd]
etcd1
etcd2
etcd3

[kube-master]
master1
master2
master3

[kube-node]
node1
node2

[k8s-cluster:children]
kube-node
kube-master
```
### 6. 关闭防火墙
```
ansible -i inventory/inventory.cfg all -a "systemctl stop firewalld"
ansible -i inventory/inventory.cfg all -a "systemctl disable firewalld"
```
### 7.设置http代理
如果不能访问gcr.io等registry，需要设置http代理
```
vim inventory/group_vars/all.yml

## Set these proxy values in order to update docker daemon to use proxies
http_proxy: "http://10.211.55.2:1080"
https_proxy: "http://10.211.55.2:1080"
no_proxy: ""
```
### 8. 安装kubernetes
```
ansible-playbook -i inventory/inventory.cfg cluster.yml
```