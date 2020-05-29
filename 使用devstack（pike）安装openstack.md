---
title: 使用devstack（pike）安装openstack
date: 2020-05-19 09:28:34
tags: OpenStack
---



[toc]



# 环境和工具

ubuntu16.04 + devstack（pike）







- 更改apt/或者yum源

- 更改pip源

- 提前下载好glance、neutron、nova、keystone、cinder等镜像，放在files文件夹下

- git clone devstack

- 创建stack用户，赋予用户权限

- ./stack.sh 安装不成功./unstack.sh ./clean.sh

  

  **<font color=yellow>the most beautiful thing is not keep time but retain memories.</font>**

参考[陈沙克](http://www.chenshake.com/local-conf-devstack-profile-parameter-description/)大佬说明解释 Local.conf

[toc]

### 开始

第一行文件[[local | localrc]]

### 修改git源

默认Devstack会从github下载所有需要的代码，包括OpenStack。这其实是导致Devstack安装时间太长的一个重要原因。

目前 git.trystack.cn 提供OpenStack的所有github的mirror。对于Devstack来说，只需要在配置文件增加3行就可以。

```shell
# use TryStack git mirror
GIT_BASE=http://git.trystack.cn
NOVNC_REPO=http://git.trystack.cn/kanaka/noVNC.git
SPICE_REPO=http://git.trystack.cn/git/spice/spice-html5.git
```

### 主机IP

```shell
HOST_IP=192.168.27.128
```

### 镜像下载

安装devstack的时候，默认会下载相应的镜像，这些镜像都在国外，我们可以指定连接来下载相关镜像。

```shell
# Define images to be automatically downloaded during the DevStack built process.
DOWNLOAD_DEFAULT_IMAGES=False
IMAGE_URLS=http://download.cirros-cloud.net/0.3.4/cirros-0.3.4-x86_64-disk.img
```

### ipv4

```shell
# only support IP v4
SERVICE_IP_VERSION=4
```

### keystone 版本

现在默认就是支持keystone v3，除非你需要使用v2。

```shell
# only support keystone v2
ENABLE_IDENTITY_V2=True
```

### 网络

默认Devstack会创建一个网络，如果你不需要创建

```shell
#not create default network
NEUTRON_CREATE_INITIAL_NETWORKS=False
```

我们也可以指定相关的网络

```shell
# instead of default network
FLOATING_RANGE="192.168.27.0/24"
FIXED_RANGE="10.0.0.0/24"
Q_FLOATING_ALLOCATION_POOL=start=192.168.27.102,end=192.168.27.110
PUBLIC_NETWORK_GATEWAY="192.168.27.2"
```

### 指定版本安装

对于普通用户，想了解某个版本的功能，可以在配置文件指定版本

```shell
# Branches
KEYSTONE_BRANCH=stable/liberty
NOVA_BRANCH=stable/liberty
NEUTRON_BRANCH=stable/liberty
SWIFT_BRANCH=stable/liberty
GLANCE_BRANCH=stable/liberty
CINDER_BRANCH=stable/liberty
```

我们使用的Devstack，也需要使用相同的版本，这样才能避免安装失败可能性。

```shell
git clone http://git.trystack.cn/openstack-dev/devstack -b stable/liberty
```

默认大家都是使用devstack的master。

### Neutron

这是最复杂的地方，目前devstack默认的网络还是nova network，所以你要采用Neutron，你必须

```shell
# Enabling Neutron (network) Service
disable_service n-net
enable_service q-svc
enable_service q-agt
enable_service q-dhcp
enable_service q-l3
enable_service q-meta
enable_service q-metering
enable_service neutron
```

vlan模式

```shell
# VLAN configuration.
Q_PLUGIN=ml2
ENABLE_TENANT_VLANS=True
TENANT_VLAN_RANGE=1100:2999
```

### 离线安装

当我们修改参数，重新运行devstack的时候，这个时候，你不希望重新下载git和操作系统的update

```shell
# Work offline
#OFFLINE=True
# Reclone each time
RECLONE=no
```

