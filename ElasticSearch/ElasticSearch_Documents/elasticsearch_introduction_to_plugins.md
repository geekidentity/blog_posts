---
categories: elasticsearch

tags: 
  - Elasticsearch
  - Elasticsearch插件

title: Elasticsearch插件和集成[5.5] -- 插件简介

date: 2017-07-12
---

# 插件简介

插件是以自定义方式增强核心Elasticsearch 功能的一种方式。 它们包括添加自定义映射类型，自定义分析器，本地脚本，自定义发现等。

插件包含JAR文件，但也可能包含脚本和配置文件，并且必须安装在群集中的每个节点上。 安装完成后，必须重新启动每个节点才能使插件变得可用。

> 安装具有自定义集群状态元数据的插件（如X-Pack）需要完全集群重新启动。 仍然可以通过滚动重新启动来升级此类插件。

本文档区分了两类插件：

## 核心插件

此类别标识作为Elasticsearch项目一部分的插件。 与Elasticsearch同时交付，其版本号与Elasticsearch本身的版本号一致。 这些插件由Elastic团队维护，并有惊人的社区成员（对于开源插件）的赞赏。可以在[Github项目](https://github.com/elastic/elasticsearch)页面上报告问题和提交bug。

## 社区贡献

此类别标识Elasticsearch项目之外的插件。 它们由个人开发者或私人公司提供，并拥有自己的许可证以及自己的版本管理系统。问题和bug报告通常可以在社区插件的网站上报告。

有关编写自己的插件的建议，请参阅[插件作者的帮助](https://www.elastic.co/guide/en/elasticsearch/plugins/5.4/plugin-authors.html)。

