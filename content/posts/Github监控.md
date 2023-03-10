---
title: "Github监控"
categories: ["数据防泄漏", "互联网企业安全"]
tags: 
  - 数据防泄漏
  - 互联网企业安全
  - github监控
  - github
date: 2022-08-16T09:16:09+08:00
lastmod: 2023-02-24T09:16:09+08:00
draft: false
---


**本文适用于所有代码托管网站**（代码不通用）

## GitHub监控目的

GitHub监控目的：（代码托管网站监控的目的）
1、收集和最新的exp或者poc
2、便于发现测试目标相关资产（代码）
3、账号密码等敏感信息收集：[GitHub信息收集语法](https://blog.csdn.net/somnuszhigang/article/details/88122473)，获取目标系统账号密码（比如mysql数据库账号密码、smtp账号密码，都是在代码配置文件中搜索到的）

## 监控最新的EXP发布及其他脚本

1、需要在GitHub上注册账号。
2、需要在[Server酱](https://sct.ftqq.com/)注册账号。（推送到微信上）

脚本不全，主要是微信推送的脚本部分不全
```python
# Title: wechat push CVE-2020
# Date: 2020-5-9
# Exploit Author: weixiao9188
# Version: 4.0
# Tested on: Linux,windows
# cd /root/sh/git/ && nohup python3 /root/sh/git/git.py &
# coding:UTF-8

import requests
import json
import time
import os
import pandas as pd

time_sleep = 60 #每隔 20 秒爬取一次
while(True):
	headers1 = {
	"User-Agent": "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.25 Safari/537.36 Core/1.70.3741.400 QQBrowser/10.5.3863.400"}

	#判断文件是否存在
	datas = []
	response1=None
	response2=None
	if os.path.exists("olddata.csv"):
		#如果文件存在则每次爬取 10 个
		df = pd.read_csv("olddata.csv", header=None)
		datas = df.where(df.notnull(),None).values.tolist()#将提取出来的数据中的 nan 转化为 None
		requests.packages.urllib3.disable_warnings()
		response1 = requests.get(url="https://api.github.com/search/repositories?q=CVE2020&sort=updated&per_page=10",headers=headers1,verify=False)
		response2 = requests.get(url="https://api.github.com/search/repositories?q=RCE&ssort=updated&per_page=10",headers=headers1,verify=False)
	else:
		#不存在爬取全部
		datas = []
		requests.packages.urllib3.disable_warnings()
		response1 = requests.get(url="https://api.github.com/search/repositories?q=CVE2020&sort=updated&order=desc",headers=headers1,verify=False)
		response2 = requests.get(url="https://api.github.com/search/repositories?q=RCE&ssort=updated&order=desc",headers=headers1,verify=False)

data1 = json.loads(response1.text)
data2 = json.loads(response2.text)
for j in [data1["items"],data2["items"]]:
	for i in j:
		s = {"name":i['name'],"html":i['html_url'],"description":i['description']}
		s1 =[i['name'],i['html_url'],i['description']]
		if s1 not in datas:
			#print(s1)
			后面微信推送脚本未完
```

监控今年出现的CVE编号
```python
response1 = requests.get(url="https://api.github.com/search/repositories?q=CVE2020&sort=updated&per_page=10",headers=headers1,verify=False)
```

监控RCE资产信息
```python
response2 = requests.get(url="https://api.github.com/search/repositories?q=RCE&ssort=updated&per_page=10",headers=headers1,verify=False)
```

## 发现测试目标相关资产

根据cms的相关信息去GitHub搜索。
比如ctcms关键词，比如相关URL路径。

## 账号密码等敏感信息收集
[GitHub信息收集语法](https://blog.csdn.net/somnuszhigang/article/details/88122473)

## 开源工具GitMiner

GitMiner是一款轻量级的github监控工具，地址：[https://github.com/UnkL4b/GitMiner](https://github.com/UnkL4b/GitMiner)

安装非常简单：
```shell
git clone http://github.com/danilovazb/GitMiner 
pip install -r requirements.txt
```

