---
categories: elasticsearch

tags: 
  - Elasticsearch
  - Elasticsearch插件

title: Elasticsearch插件和集成[5.5] -- API扩展插件

date: 2017-07-13
---

API扩展插件通过添加新的API或功能（通常与搜索或映射相关）来向Elasticsearch添加新功能。

## 社区提供了API扩展插件

我们社区贡献了一些插件：

* [carrot2插件](https://github.com/carrot2/elasticsearch-carrot2)：结果与[carrot2](http://project.carrot2.org/) 聚类（by Dawid Weiss）
* [Elasticsearch Trigram加速正则表达式过滤器](https://github.com/wikimedia/search-extra):(by Wikimedia Foundation/Nik Everett）
* [Elasticsearch Experimental Highlighter](https://github.com/wikimedia/search-highlighter): (by Wikimedia Foundation/Nik Everett)
* [实体解析插件](https://github.com/YannBrrd/elasticsearch-entity-resolution)：使用[Duke](http://github.com/larsga/Duke)进行重复检测（by Yann Barraud）
* [SQL语言插件](https://github.com/NLPchina/elasticsearch-sql/)：允许使用SQL查询Elasticsearch （by nlpcn）
* [Elasticsearch Taste 插件](https://github.com/codelibs/elasticsearch-taste)：Mahout基于Taste的协同过滤实现（by CodeLibs Project）
* [WebSocket更改反馈插件](https://github.com/jurgc11/es-change-feed-plugin)（by ForgeRock/Chris Clifton）