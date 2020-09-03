---
title: 搭建P4RuntimeAPI
toc: true
thumbnail: 'https://pic.downk.cc/item/5eda03efc2a9a83be5241038.jpg'
comments: true
tags:
  - P4Runtime
date: 2020-08-31 19:34:08
urlname:
categories:
---

# 依赖安装:dependency    

### grpcpp

```shell
cd
[sudo] apt-get install build-essential autoconf libtool pkg-config
[sudo] apt-get install cmake
```

##### Clone the repository (including submodules)

```shell
git clone -b v1.31.1 https://github.com/grpc/grpc
cd grpc
git submodule update --init
```

### PI

An implementation framework for a P4Runtime server

```she
cd
sudo apt install  libboost-thread-dev libjudy-dev
git clone https://github.com/p4lang/PI.git
git submodule update --init --recursive
```

安装Bmv2及P4编译器（非必须，主要要安装p4编译器p4c）

```shell
cd ~/PI
git clone https://github.com/p4lang/behavioral-model.git
cd behavioral-model
./install_deps.sh
./autogen.sh
./configure
make
sudo make install
```

安装Protobuf

```shell
git clone https://github.com/google/protobuf.git
cd protobuf/
git checkout tags/v3.6.1
./autogen.sh
./configure
注意：此处如果有一个警告，没有googletest的配置信息的话，需要删除third_part/googletest,从https://github.com/google/googletest/releases下载源文件，解压，重新执行第四步
make
[sudo] make install
[sudo] ldconfig
```

安装gRPC

```shell
git clone https://github.com/google/grpc.git
cd grpc/
git checkout tags/v1.17.2
git submodule update --init --recursive
make
[sudo] make install
[sudo] ldconfig
```

注意：这里grpc/third_party/bloaty里面有一个libfuzze的子模块下不下来，更改.gitmodules的地址会造成git tree检测分支报错，我这里直接就在github上下载下来放到grpc/third_party/bloaty/third_party里，进行下一步make了。

安装sysrepo（非必须）

```shell
sudo apt installbuild-essential cmake libpcre3-dev libavl-dev libev-dev libprotobuf-c-dev protobuf-c-compiler
cd ~/PI
git clone https://github.com/CESNET/libyang.git
cd libyang
git checkout v0.16-r1
mkdir build
cd build
cmake ..
make
[sudo] make install
cd ~/PI
git clone https://github.com/sysrepo/sysrepo.git
cd sysrepo
git checkout v0.7.5
mkdir build
cd build
cmake -DCMAKE_BUILD_TYPE=Release -DBUILD_EXAMPLES=Off -DCALL_TARGET_BINS_DIRECTLY=Off ..
make
[sudo] make install
```

##### Building p4runtime.proto

```shell
./autogen.sh
./configure --with-proto
make
make check
[sudo] make install
```

如果在make报错，在这里一直报一个protoc版本的错误，则修改~/PI/proto/cpp_out/p4/config/v1第17行，删掉。如果不行，重新装一遍成功。

### t4p4s

```shell
git clone https://github.com/P4ELTE/t4p4s
把t4p4s_mod文件夹里的内容拷贝至t4p4s内
```

# 生成桩模块

```shell
./install.sh
```

# 编译源文件以及起pi_server

```shell
./compile.sh -D
sudo ./pi_server
```

# 连接至ONOS

1. 将P4Runtime_GRPCPP/onos目录下的t4p4s文件复制到 ~/onos/apps下
2. vim ~/onos/tools/build/bazel/modules.bzl
   在//apps/p4-tutorial/mytunnel后一行加上
   "//apps/t4p4s/l2fwdgen:onos-apps-t4p4s-l2fedgen-oar":[],
3. 在~/onos下重新bazel build onos 
4. bazel run onos-local -- clean 

# 连接T4P4s