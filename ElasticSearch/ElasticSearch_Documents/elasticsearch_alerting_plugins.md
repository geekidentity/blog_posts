---
categories: elasticsearch

tags: 
  - Elasticsearch
  - Elasticsearch插件

title: Elasticsearch插件和集成[5.5] -- 报警插件

date: 2017-07-13
---

报警插件允许Elasticsearch监视索引，并在违反阈值时触发报警。

## 核心报警插件

核心报警插件是：

### X-Pack

[X-Pack](https://www.elastic.co/products/x-pack/alerting)包含Elasticsearch的报警和通知产品，可让您根据数据更改采取行动。它是围绕着原则设计的，如果您可以在Elasticsearch中查询某些内容，可以提醒它。 简单地定义一个查询，条件，日程安排以及要采取的操作，而X-Pack将完成其余操作。