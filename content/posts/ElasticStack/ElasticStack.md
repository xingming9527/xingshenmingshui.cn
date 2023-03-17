---
title: "ElasticStack"
categories: ["ElasticStack", "ELK"]
tags: 
  - ElasticStack
  - ELK
  - 日志监控
date: 2022-03-15T09:16:09+08:00
lastmod: 2022-03-17T09:16:09+08:00
draft: false
---

## 一、ELK Stack介绍

### 1.1、需求背景

- 1、开发人员为了排除问题，经常上线服务器去查询项目日志。
- 2、服务器越来越多，项目越来越多，日志类型越来越多。

传统方式，就是上每一台服务器上去执行：`awk sed grep access.log`。输出文本或CSV，但是对人的视觉还不是很好。
人视觉好的，就是用图 + 文。其实就是动态图。[在线画图工具](https://echartsjs.com/)（需要前端知识功底）

### 1.2、ELK Stack介绍

如果你没有听说过Elastic Stack，那你一定听说过ELK，实际上ELK是三款软件的简称，分别是Elasticsearch、 Logstash、Kibana组成，在发展的过程中，又有新成员Beats的加入，所以就形成了Elastic Stack。所以说，ELK是旧的称呼，Elastic Stack是新的名字。

![ElasticStack_20230314142150525](https://img.ivansli.com/images/ElasticStack_20230314142150525.png)

> 2016年秋季，因为有新成员Beats加入，再加上统一版本，ELK官方将产品线命名为ElasticStack。
> 以前的版本比较混乱，每个产品的版本号都不一样，Elasticsearch和Logstash目前是2.3.4；Kibana是4.5.3；Beats是1.2.3；版本号太乱了有没有，什么版本的ES用什么版本的Kibana？有没有兼容性问题？

##  二、Elastic Stack架构（技术栈）

**本系列文章采用的是7.9.1版本，注意各个大的版本均有所差别。版本不同可能会导致各种问题。**

![ElasticStack_20230314142430788](https://img.ivansli.com/images/ElasticStack_20230314142430788.png)

- 数据收集
	- [Beats](./Beats.md)：轻量型采集器的平台，从边缘机器向Logstash和ES发送数据。 
		- [Filebeat](./Filebeat.md)：轻量型日志采集器。是Beats的一员
	- [Logstash](./Logstash.md)：重量型日志采集器，集日志收集、过滤（`filter`）、output输出（输出至redis、kafka、ES等等）于一身。
		- 功能越强大，性能要求就越高，所以就有了Beats平台。
		- Filter：过滤，将日志格式化。有丰富的过滤插件：Grok正则捕获、Date时间处理、Json编解码、Mutate数据修改等。
- 数据处理与存储
	- [ElasticSearch](./ElasticSearch.md)：搜索、分析和存储数据。
		- Elasticsearch是一个分布式的RESTful风格的搜索和数据分析引擎，能够解决不断涌现出的各种用例。作为Elastic Stack的核心，它集中存储您的数据，帮助您发现意料之中以及意料之外的情况。（ElasticStack的核心）
- 日志展示
	- [Kibana](./Kibana.md)：数据可视化。

## 三、Elastic Stack综合案例







## 参考资料

- [Elastic Stack（ELK）从入门到实践 - 黑马程序员（视频）](https://www.bilibili.com/video/BV1iJ411c7Az)
	- [Elastic Stack（ELK）从入门到实践 - 黑马程序员（笔记非常全面，完全可以只看笔记，不懂再看视频）](https://gitee.com/moxi159753/LearningNotes/tree/master/ElasticStack)
	- **90%参考这个笔记，10%是我自行补充方面理解的笔记。（包括修正错误）**


