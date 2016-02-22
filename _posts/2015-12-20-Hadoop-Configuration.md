---
layout: post
title: "Hadoop配置备忘录"
author: "yphuang"
date: 2015-12-20
categories: blog
tags: [Hadoop,Linux]

---

在这个人人都谈大数据的时代，如果不了解一点大数据的操作，都不好意思说自己是学统计的了。所以，今天让我们来一起学习在自己的电脑上使用多个虚拟机模拟配置hadoop集群，自娱自乐一番。


***

## 1.预备环境配置

### 硬件规格

根据『Hadoop权威指南』一书的介绍，2010年年中，运行Hadoop的datanode和tasktracker的典型机器具有以下规格：

- 处理器：两个四核2～2.5GHz CPU

- 内存：16～24GB ECC RAM

- 存储器：4× 1 TB SATA 硬盘

- 网络：千兆以太网


= =！赶紧存钱～

### 系统平台要求

Hadoop的主体由Java语言写成，能够在任意一个安装了JVM的平台上运行。但是，仍有部分代码需要在Unix环境下执行。Windows操作系统主要作为一个开发平台而非生产平台而存在。

因此，要在自己的电脑上搭建Hadoop集群环境，我们需要先安装VMware软件，然后，安装几个Linux虚拟机。

具体细节可以参考这里：

- [百度经验](http://jingyan.baidu.com/article/0bc808fc906bf91bd485b92a.html)

-[官方文档](http://partnerweb.vmware.com/GOSIG/Ubuntu_14_04.html)



## 2.集群的构建和安装

### 2.1 安装Java 

Hadoop的运行需要Java 6及以上的版本。首选最新稳定版本的Sun JDK。

安装JVM可以参考以下博客：

- CentOS系统：[Redhat Linux安装JDK 1.7](http://www.cnblogs.com/kerrycode/p/3197865.html)

- Ubuntu系统：[ubuntu 安装jdk 的两种方式](http://www.cnblogs.com/a2211009/p/4265225.html)

安装结束之后，可以在Console使用以下命令检查是否安装成功：

```
java -version

```
如未成功，则可能需要配置环境变量。

可参考：

- [linux配置java环境变量(详细) ](http://www.cnblogs.com/samcn/archive/2011/03/16/1986248.html) 


### 2.2 创建Hadoop用户

最好在操作系统上创建特定的Hadoop用户以区分Hadoop和本机上的其他服务。

Linux创建用户可以参考：

- [Linux 帳號管理與 ACL 權限設定](http://linux.vbird.org/linux_basic/0410accountmanager.php)

- [linux 新建用户、用户组 以及为新用户分配权限](http://www.blogjava.net/hello-yun/archive/2012/05/16/378295.html)

### 2.3 安装Hadoop

从[Apache Hadoop的发布页面](http://hadoop.apache.org/releases.html)下载Hadoop发布包，并在某一目录下解压缩。

```
# 进入 cd /usr/local
 cd /usr/local

# 解压hadoop包并重命名解压后的文件 
sudo tar -xvf hadoop-2.5.2.tar.gz
 mv  hadoop-2.5.2.tar.gz hadoop

```


### 2.4 SSH配置

#### xshell安装

Hadoop控制脚本依赖SSH来执行针对整个集群的操作。

那么，什么是SSH呢？

简单说，SSH是一种网络协议，用于计算机之间的加密登录。

进一步了解，可以参考：

- [SSH原理与运用（一）：远程登录](http://www.ruanyifeng.com/blog/2011/12/ssh_remote_login.html)

- <https://zh.wikipedia.org/wiki/Secure_Shell>

下面介绍具体如何配置SSH。

首先，需要在Windows系统上安装Xshell。

问题又来了，什么是xshell？

> Xshell is a powerful terminal emulator that supports SSH, SFTP, TELNET, RLOGIN and SERIAL. 

> Xshell offers many user friendly features that are not available in other terminal emulators.

> Xshell is free for home and school use.

进一步了解和下载安装软件，可以访问这里：

- <https://www.netsarang.com/products/xsh_overview.html>

- <https://www.netsarang.com/xshell_download.html>

#### SSH无密码验证配置(master和各个SLAVE)

为了支持无缝式工作，SSH安装好之后，需要允许hadoop用户无需键入密码即可登录集群内的机器。最简单的方法是创建一个公钥/私钥对，存放在NFS中，让整个集群共享该密钥对。

* Master机器上生成密码对
    + 生成密钥存放目录: `mkdir .ssh`
    + 进入目录： `cd ~/.ssh`
    + 生成密钥： `ssh-keygen -t rsa -f ~/.ssh/id_rsa`.
        - 最终，私钥存放在`-f`选项指定的文件当中，如：`~/.ssh/id_rsa`.公钥在相应的目录，以`.pub`为后缀，如：`~/.ssh/id_rsa.pub`.
    + 将密钥存放在指定文件中：`cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys`
    + 将密钥发送到slave1机器：`scp id_rsa.pub root@slave1:/root/hadoop`
    + 登录slave1：
        - `ssh root@slave1`
        - `cd hadoop`
        - 在slave1上共享密钥：`cat id_rsa.pub >> ~/.ssh/authorized_keys`
    + 在master验证无密钥进入：`ssh root@slave1`

以上是在master服务器上生成密钥，配置公钥到各slave机器，以便允许hadoop用户无需键入密码即可登录集群内的机器。


其实xshell也可以用生成密钥的方式登录Linux。具体可以参考这里：

- [Xshell配置ssh免密码登录-密钥公钥(Public key)与私钥(Private Key)登录](http://www.aiezu.com/system/linux/xshell_ssh_public-key_login.html)

- [xshell使用key登录Linux](http://www.live-in.org/archives/1368.html)

## 3. Hadoop配置

(待续)

## 参考文献：

- 『Hadoop权威指南』

