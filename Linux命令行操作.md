---
title: Linux命令行操作
date: 2017-11-18 00:21:52
tags: 
- Linux
comments: true
toc: true
mathjax: true
urlname:
categories:
thumbnail:
---

### 目录操作
**创建目录**

```
mkdir
```
**切换目录**

```
cd ..
cd
```
**移动目录**

```
mv /home/test /var/tmp
```
**删除目录**

```
rm -rf var/test
```
**查看目录**

```
ls
```
##文件操作

**创建文件**

```
touch ~/testfile
```
**删除文件**

```
rm ~/testfile
```
**复制文件**

```
cp ~/testfile ~/testnewfile
```
**查看文件内容**

```
cat ~/.bash_history
```
##过滤、管道与重定向
**过滤文件内内容**

```
grep 'root' /etc/pwd
```
**过滤文件夹内内容**

```
grep -r 'linux' /var/log
```
**管道**
管道的作用是将上一个命令的输出和下一个命令的输入连接起来

```
cat ~/.bash_history | grep 'ls' 
```
**重定向**
使用>或者<将内容重定向文件中 

```shell
 echo 'hello,world!'> ~/test.txt
```

### 运维常用命令

**ping**
四个ping包

```
ping -c 4 www.baidu.com
```
**网络状态**

```
netstat -lt
netstat -tulpn
```
**过滤得到当前系统ssh信息**

```
ps -aux | grep 'ssh'
```

```

----------

anaconda切换环境
conda info -e
activate python2

----------

```