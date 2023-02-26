---
title: "burpsuite使用指南"
categories: ["burpsuite", "web扫描器"]
tags: 
  - hugo
  - blog
  - burpsuite
  - web扫描器
date: 2022-07-15T09:16:09+08:00
lastmod: 2023-02-24T09:16:09+08:00
draft: false
---

## 一、burpsuite介绍

**Web安全工具中的瑞士军刀**，是所有web扫描器中最优秀的一款。（都要会使用）
由PortSwigger公司研发。有Burp Suite Community Edition（burp社区版）和[Burp Suite Professional](https://portswigger.net/burp/pro)（burp专业版）

既然是web扫描器最优秀的一款，就有主动扫描与被动扫描的功能。
一般是通过爬虫为主+爆破为辅，进行主动扫描，效率大大提高。

## 二、burpsuite下载安装

1. 下载jdk，不过kali自带open jdk。[oracle jdk官方下载地址](https://www.oracle.com/cn/java/technologies/downloads/#java11)
2. [burpsuite jar下载地址（jar好破解）](https://portswigger.net/burp/releases/)，[注册机下载地址](https://github.com/h3110w0r1d-y/BurpLoaderKeygen)，下载完成后运行，输入注册机的密钥后选择 **Manual activation手动激活**
3. 运行命令`java -jar BurpLoaderKeygen.jar`（通过注册机运行burpsuite）


