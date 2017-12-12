---
categories: beam

tags: 
  - Apache Beam

title: Apache Beam 综述

date: 2017-02-07

---
# Apache Beam 综述
Apache Beam 是一个开源的统一编程模型，您可以使用它来创建数据处理管道pipeline。 你首先要构建一个程序，使用一个开源的Beam SDK定义管道。 然后，pipeline 由Beam支持的分布式处理后端之一执行，包括Apache Apex，Apache Flink，Apache Spark和Google Cloud Dataflow。

Beam对于尴尬的并行数据处理任务特别有用，其中问题可以分解为可以独立和并行处理的许多较小的数据束。 您还可以使用Beam 进行提取，变换和加载（ETL）任务和纯数据集成。 这些任务对于在不同存储介质和数据源之间移动数据，将数据转换为更理想的格式或将数据加载到新系统上是有用的。
 
# Apache Beam SDKs
Beam SDK 提供了统一的编程模型，可以表示和变换任何大小的数据集，无论输入是来自批处理数据源的有限数据集还是来自流数据源的无限数据集。 Beam SDK使用相同的类来表示有界和无界数据，并且相同的转换操作该数据。 您使用您选择的Beam SDK构建一个定义数据处理管道的程序。
Beam目前支持以下特定语言的SDK：

语言|SDK状态
---|---
Java	|积极开发中
Python	|即将来临
其他	|待定
# Apache Beam Pipeline Runners (Beam管道运行器)
Beam 管道运行器将您用Beam程序定义的数据处理管道转换为与您选择的分布式处理后端兼容的API。 当您运行Beam程序时，您需要为要执行管道的后端指定适当的运行程序。

Beam目前支持使用以下分布式处理后端的Runners：

Runner|Status
---|---
Apache Apex	|开发中
Apache Flink	|开发中
Apache Spark	|开发中
Google Cloud Dataflow	|开发中
注意：您也可以在本地执行pipeline 以进行测试和调试。
# 开始 Apache Beam
开始为您的数据处理任务使用Beam。
1. 学习Java SDK或Python SDK的快速入门。
1. 有关介绍SDK的各种功能的示例，请参阅WordCount示例演练。