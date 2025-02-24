---
layout: post
title: "Proxmeox ve 系统分区使用心得"
date:   2024-12-14
tags: [linux]
comments: true
author: kamust
---

## 分区

一般默认安装的pve系统会给硬盘分为两个主要的分区local和local-lvm，这使得硬盘空间被分割，不方便使用，特别是许多个人用户会把pve安装到U盘里，让本就不多的空间被浪费。根据本文方法就可以设置为一个分区。

系统安装时会要求用户设置5个分区的大小，用户需要设置maxvz为0。

![part](https://kamust.github.io/images/pve_user_exp/part.png)

这5个分区含义分别如下

+ hdsize：要使用的硬盘的总大小。一般是全部。
+ swapsize：交换空间swap的大小。默认值是已安装内存的大小，限制为4-8 GB。
+ maxroot：操作系统的/root的最大大小。
+ minfree：磁盘预留空间，pve系统会保留这部分空间防止系统允许时崩溃。
+ maxvz：LVM 存储池的最大大小。未手动指定，PVE 会自动分配剩余空间给它。

在系统安装完成后使用ssh登录到pve系统上，执行如下两条命令。

``` sh
lvextend -l +100%FREE /dev/mapper/pve-root
resize2fs /dev/mapper/pve-root
```

## 使用cloud版本的linux系统

推荐在pve上使用cloud版本的linux系统作为虚拟机，这样就能通过cloud-init配置系统的ip、账号等，非常方便。

### 各发行版cloud系统下载地址

#### centos：
<https://cloud.centos.org/centos/>

#### ubuntu:
<https://cloud-images.ubuntu.com/releases/>

#### debian:
<https://cloud.debian.org/images/cloud/>

#### fedora:
<https://alt.fedoraproject.org/cloud/>

有些发行版提供多种cloud版本，推荐使用 genericcloud

+ nocloud 普通虚拟机镜像，不含cloud-init，可以使用root登录。
+ generic 适用于多种云服务提供商的通用镜像，它包含了一些常见的云相关软件包和设置，例如cloud-init，cloud-guest-utils等。它可以根据不同的云平台进行调整和优化。
+ genericcloud 是一种类似于generic的镜像，但是它不含硬件驱动所以更小。（推荐pve中不需要直通的vm使用）

### 安装cloud版本系统

1. 通过sftp等方式把下载的镜像传入pve，如 /tmp/debian-12-genericcloud-amd64.qcow2

2. 在管理网页上新建一个虚拟机，无需设置硬盘，其他按需配置，记住这个虚拟机的id

3. 使用root账户登录ssh，执行如下命令

``` sh
#            虚拟机id     下载的镜像位置         指定虚拟机硬盘的存储池  格式为 raw 或 qcow2
qm importdisk 100 /tmp/debian-12-genericcloud-amd64.qcow2 local --format=qcow2
```

4. 在管理网页上为虚拟机添加 VirtIO 硬件、cloud-init设备，在选项里设置新增的硬盘为第一启动项。

5. 配置cloud-init参数后即可使用。

## 退出集群

有时候会不小心把pve系统加入或创建集群，可按如下方法退出

``` sh
# 停止服务
systemctl stop pve-cluster.service && systemctl stop corosync.service 

# 设置为本地模式
pmxcfs  -l

# 删除 corosync 配置文件
rm -f /etc/pve/corosync.conf && rm -rf /etc/corosync/*

# 重新服务
killall pmxcfs && systemctl start pve-cluster.service 

```
