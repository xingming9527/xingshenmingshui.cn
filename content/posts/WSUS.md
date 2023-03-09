---
title: "Windows更新服务（WSUS）"
categories: ["Windows"]
tags: 
  - Windows
  - WSUS
date: 2022-02-11T09:16:09+08:00
lastmod: 2023-01-24T09:16:09+08:00
draft: false
---


## WSUS介绍

Windows操作系统层面的漏洞比较多，而且国内外安全人员跟进也特别快，各种POC利用工具特别多，稍不留神就被黑了，所以window下的补丁管理尤其重要。
微软自带就有免费的很简单可依赖的补丁管理软件：WSUS。

WSUS（Windows Server Update Services），它在以前Windows Update Services的基础上有了很大的改善。目前的版本可以更新更多的Windows补丁，同时具有报告功能和导向性能，管理员还可以控制更新过程。

WSUS是典型的**CS架构**，也支持**分布式部署**，不过单台WSUS服务生产环境负责一千台以内的Windows服务器的补丁管理与分发没有任何问题。

### WSUS简单网络架构

![WSUS_20230308223125047](https://img.ivansli.com/images/WSUS_20230308223125047.png)

### WSUS功能

- 审批更新
	- 更新可以被自动审批（重要的更新自动审批）
		- 更新服务 --> 选项 --> 自动审批
	- 更新应该先被测试，之后再被审批用于生产环境
	- 不需要的更新，可以被拒绝。
	- 可能导致问题的更新，应该被删除
- 根据组进行更新
	- 更新服务 --> 计算机 --> 所有计算机 --> 各个计算机组
	- 可以配合AD域进行组管理。（部门管理）
	- 比如一天更新一个组（需要手动了，灰度发布）
- 配置自动安装（不提醒）
	- 下文《配置自动更新》章节

### WSUS安装

添加角色和功能  --> 选择Windows更新服务 --> 其他都默认下一步下一步

### 配置向导

语言：只保留中文简体（如果域内有英语版服务器，需要勾选英语）
选择产品：只更新Windows要更新的产品。（比如Windows7、Windows10等等）
选择分类：“安全更新程序”、"关键更新程序"，其他按需来。
配置同步计划：自动同步（定时）

就算不小心点错了，也可以在`更新服务 --> 选项`中重新配置。

## 客户端配置自动更新

window服务器自带WSUS客户端，即自动更新服务，只要配置下内部WSUS地址即可；

### 配置更新服务器
#### 组策略配置更新服务器（单台）

![WSUS_20230308205036510](https://img.ivansli.com/images/WSUS_20230308205036510.png)

配置WSUS地址
![WSUS_20230308205105324.png](https://img.ivansli.com/images/WSUS_20230308205105324.png)

#### AD域组策略下发（需加入AD域）

暂无。
可以参考[组策略：域控制器组策略：部署软件自动下发静默安装到域计算机上（分发软件、软件分发）](http://zh-cjh.com/gechangjia/2054.html) + 本文的《组策略配置更新服务器》章节。

#### reg脚本配置（多台未加AD域的服务器）

也可以通过[[ansible]]的Windows模块，去批量执行，需要Windows开启winrm。

```reg
[HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate]
"WUServer"="http://你的IP"
"WUStatusServer"="http://你的IP"
"ElevateNonAdmins"=dword:00000001
"TargetGroupEnabled"=dword:00000001
```

```reg
[HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU]
"NoAutoUpdate"=dword:00000000
'设置为0，表明自动升级
"AUOptions"=dword:00000004
'设置为4，表示下载后自动安装。3是提醒安装
"ScheduledInstallDay"=dword:00000000
'设置为0，表示每天都检测升级
"UseWUServer"=dword:00000001
'启用内部WSUS。
```

### 配置自动更新

计算机配置 --> 策略 --> 管理模板 --> Windows组件 --> windows更新 --> 配置自动更新
（可以在AD组策略中，批量下发配置。）
![WSUS_20230309001735150.png](https://img.ivansli.com/images/WSUS_20230309001735150.png)


## WSUS隔离网络中解决方案

- 1、需要在隔离网络中再部署一台WSUS服务器
- 2、通过U盘等移动介质，把补丁包复制到隔离网络中WSUS服务器上。
	- 补丁包位置：`C:\wsus\WsusContent`
- 3、导出更新和日志，和补丁包复制到隔离网络中WSUS服务器上。
```cmd
C:\>C:\Program Files\Update Services\Tools\wsusutil export c:\test.cab c:\test.log
正在导出更新。请不要终止该程序。
```
- 4、在隔离网络中WSUS服务器导入。
```cmd
C:\>wsusutil import c:\test.cab c:\test.log
正在导入更新。请不要终止该程序。
已成功导入所有更新。
```




