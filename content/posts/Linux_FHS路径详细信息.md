---
title: "Linux_FHS路径详细信息"
categories: ["Linux"]
tags: 
  - FHS
date: 2022-02-10T09:16:09+08:00
lastmod: 2023-02-24T09:16:09+08:00
draft: false
---

## FHS介绍
因为Linux开源，全世界N多人一起开发。
当A用`/etc`当配置文件路径，B用`/conf`当配置文件路径，会有众多问题。（就如秦始皇统一度量衡和文字，全国交流就方便很多）
FHS就是Linux开源界的目录标准，全称为：Filesystem Hierarchy Standard（文件系统层次化标准）

## 实际应用中

1. 只读居多，一般不会新增文件夹。所以了解一些常用的目录即可。
2. 一般编译的nginx、php、以及公司开发的软件包。都有统一的目录存放。方便数据迁移的时候，直接拷贝整个路径。例如：

```shell
/opt/nginx
/opt/php
/opt/java
/opt/tomcat
```


## Linux常用路径功能介绍

```shell

Linux下应用程序的组成部分：
     二进制程序：bin，sbin
     库文件：lib，lib64
     配置文件：/etc
     帮助文件：/uer/share/man,/uer/local/share/man


/bin：存储基本的用户命令(系统启动时必须要用到命令)
/boot：boot loader（引导）相关的静态文件，内核以及ramdisk文件
/dev：设备相关的文件
/etc：主机特有的系统配置文件
/home：用户的家目录（可选）
/lib：基本的共享库文件，以及内核模块文件
	  .so  库文件    .ko内核模块文件
/lib64：64bits系统的基本共享库文件
/media：可移除设备的挂载点
/mnt：可挂载文件系统的临时使用的挂载点
/opt：第三方应用程序的安装位置
/root：管理员root用户的家目录
/sbin：系统管理相关的二进制程序
	  注意：执行设定操作时，此目录下的命令仅具有管理权限的用户可执行
/srv：当前系统的服务相关的数据
/tmp：临时文件目录，具有特殊权限设定
/usr：可被共享的、只读数据的存放位置
	bin：二进制程序（共享的命令）
	include：C程序用到的头文件
	lib：库文件（应用与/usr文件用到的库文件
	lib64：64位库文件
	local：本地程序目录结构
	share：独立于平台架构的数据
	sbin：管理程序

	src：源代码目录

/var：可变文件目录
	cache：应用程序的缓存数据
	lib：可变的状态信息库
	local：为/usr/local下的程序准备的可变数据目录
	lock：锁文件目录
	log：日志文件
	run：运行进程时相关的数据，如守护进程的PID文件
	tmp：系统重启之后需要留存的临时文件目录

/proc：内核及进程相关信息的虚拟文件系统接口
	 伪文件系统：本身不是文件系统，但输出为文件系统

/sys：伪文件系统，硬件设备相关信息的虚拟文件系统接口

/usr/local：
	bin
	etc
	lib
	lib64
	libexec
	sbin
	share
```

## Linux常用文件介绍

```text
/proc/cpuinfo：CPU硬件信息
/proc： /proc：内核及进程相关信息的虚拟文件系统接口
/etc/sysconfig/network-scripts/ifcfg-eth0 ：网卡eth0配置文件
/etc/passwd：密码配置文件
/etc/issue：发行版系统版本
/etc/null：一个数据黑洞，所有丢给它的数据，全部默默丢弃
/etc/passwd：用户及其属性信息
/etc/group：组及其属性信息：
/etc/shadow：用户密码及其相关属性
/etc/gshadow：组密码及其相关属性
/usr/share/man ：程序手册存放位置
/etc/man.config：man手册的配置文件
/uer/local/share/man：程序手册存放位置
/usr/share/doc/COMMAND-VERSION:很多应用程序自带帮助文档
/etc/default/useradd   ：放的添加用户时默认的配置文件
/etc/login.defs：用户时默认的配置信息文件。
/etc/shells：bash的路径配置文件

```
