
# Spark 技术背景

在Spark 之前，大多数集群编程模型（如MapReduce、Dryad等）是基于非循环的数据流模型。即从稳定的物理存储（如HDFS）中加载记录，记录被传入由一组确定性操作构成的DAG（Directed Acyclic Graph，有向无环图），然后写回稳定存储。DAG 数据流图有够在运行时自动实现任务调度和故障恢复。

虽然非循环数据流是一种非常强大的抽象方法，但仍有些应用无法使用这种方式描述。这类应用包括：
1. 机器学习和图应用中常用的迭代算法（每一步对数据执行相似的函数）；
2. 交互式数据挖掘工具（用户反复查询一个数据子集）。

基于数据流的框架并不明确支持工作集，所以需要将数据输出到磁盘，然后在每次查询时重新加载，这会带来较大的开销。针对上述问题，Spark 实现了一种分布式内存抽象，称为弹性分布式数据集（Resilient Distributed Dataset, RDD）。
它支持基于工作集的应用，同时具有数据流模型的特点：自动容错、位置感知性调度和可伸缩性。RDD 允许用户在执行多个查询时显式地将工作集缓存在内存中，后续的查询能够重用工作集，这极大提升了查询松速度。

RDD提供了一种高度受限的共享内存模型，即RDD是只读记录分区的集合，只能通过在其他RDD执行确定的转换操作（如map、join和groupBy）而创建，然而这些限制使得实现容错的开销很低。与分布式共享内存系统需要付出高昂代价的检查点和回滚机制不同，RDD通过Lineage来重建丢失的分区。

# Spark 优点

1. 速度：与Hadoop的MapReduce相比，Spark基于内存的运算要快100倍以上；而基于硬盘的运算也要快10倍以上。Spark实现了高效的DAG执行引擎，可以通过基于内存来高效处理数据流。
2. 易用：Spark支持Scala、Python和Java的API，还支持大量高级算法，使用户可能 快速构建不同的应用。而且Spark还支持交互式的Python和Scala，方便原型的快速开发。
3. 通用：Spark提供了统一的解决方案。Spark可以用于批处理、交互式查询（Spark SQL）、实时流处理（Spark Streaming）、机器学习（Spark MLlib）和图计算（Spark GraphX）。这些不同类型的处理都可以在同一个应用中无缝使用。
![image](http://blog.geekidentity.com/images/spark_introduce/spark_1.png)

4. 可融合性：Spark可以方便地与其他开源产品融合。

# Spark 架构综述

Spark整体架构如下图所示。其中，Driver是用户编写的数据处理逻辑，这个逻辑中包含用户创建的SparkContent。SparkContent是用户逻辑与Spark集群主要的交互接口，它会和Cluster Manager交互，包括向它申请资源等。Cluster Manager负责集群的资源管理和调度，现在支持Standalone、Apache Mesos和Hadoop YARN。Worker Node是集群中可以执行计算任务的节点。Excutor是在一个Worker Node上为某应用启动的一个进程，该进程负责运行任务，并且负责将数据存在内存或者磁盘上。Task是被送到某个Executor上的计算单元。每个应用都有各自独立的Executor，计算最终在计算节点的Executor中执行。
![image](http://spark.apache.org/docs/latest/img/cluster-overview.png)


用户程序从最开始的提交到最终的计算执行，需要经历以下几个阶段：

1. 用户程序从最开始的提交到最终的计算执行，新创建的Spark Content实例会连接到Cluster Manager。Cluster Manager会根据用户提交时设置的CPU和内存等信息为本次提交分配计算资源，启动Executor。

2. Driver会将用户程序划分为不同的执行阶段，每个执行阶段由一组完全相同的Task组成，这些Task分别作用于待处理数据的不同分区。在阶段划分完成和Task创建后，Driver会向Executor发送Task。

3. Executor在接收到Task后，会下载Task的运行时依赖，在准备好Task的执行环境后，会开始执行Task，并且将Task运行状态汇报给Driver。

4. Driver会根据收到的Task运行状态来处理不同的状态更新。Task分为两种：一种是Shuffle Map Task，它实现数据的重新洗牌，洗牌的结果保存在节点的文件系统中；另外一种是Result Task，它负责生成结果数据。

5. Driver会不断地调用Task，将Task发送到Executor执行，在所有的Task都正解执行或者超过执行次数的限制仍然没的执行成功时停止。

# Spark 核心组件概述

## Spark Streaming

Spark Streaming基于Spark Core实现了可扩展、高吞吐和容错的实时数据流处理。现在支持的数据源有kafka、Flume、Twitter、ZerorMQ、Kinesis、HDFS、S3、ElasticSearch和TCP socket。处理后的结果可以存储到HDFS、Database或者Dashboard中，如下图所示：
![image](http://spark.apache.org/docs/latest/img/streaming-arch.png)

Spark Streaming是将流式计算分解成一系列短小的批处理作业。这里的批处理引擎是Spark，也就是把Spark Streaming的输入数据按照批处理尺寸（如1秒）分成一段一段的数据（Stream），每一段数据都转换成Spark中的RDD，然后将Spark Streaming中对DStream的转换操作变为针对Spark中对RDD的转换操作，将RDD经过操作变成中间结果保存在内存中，整个流式计算可以根据业务的需求对中间结果进行叠加，或者存储到外部设备。
![image](http://spark.apache.org/docs/latest/img/streaming-flow.png)

## MLlib

MLlib是Spark对常用机器学习算法的实现库，同时含有相关的测试和数据生成器，包括分类、回归、聚类协同过滤、降维以及底层基本的优化元素。

## Spark SQL

Spark SQL最常见的用途之一就是作为一个从Spark平台获取数据的渠道。Spark SQL支持的数据源如下图所示：
![image](http://blog.geekidentity.com/images/spark_introduce/spark-sql.png)

数据源API通过Spark SQL提供了访问结构化数据的可插拔机制。这使数据源有了简便的途径进行数据转换并加入到Spark平台中。

## GraphX

Spark GraphX是Spark提供的关于图和图并行计算的API，它集ETL、试探性分析和迭代式的图计算于一体，并且在不失灵活性、易用性和容错性的前提下获得了很好的性能。现在GraphX已经提供了很多的算法，新的算法也在不断加入，而且很多的
算法都是由Spark的用户贡献的。

# 环境搭建

# RDD

RDD 是一种高度受限的共享内存模型，即RDD 是只读记录分区的集合，只能通过在其他RDD  执行确定的转换操作（如map、join、groupBy等）而创建。RDD含有如何从其他RDD衍生（即计算）出本RDD的相关信息（即Lineage），因此在RDD部分分区数据丢失的时候可以从物理存储的数据计算出相应的RDD分区。

1. 一组分片（Partition），即数据集的基本组成单位。对于RDD来说，每个分片都会被一个计算任务处理，并决定并行计算的
粒度。用户可以在创建RDD时指定RDD的分片个数，如果没有指定，那么就会采用默认值。默认值就是程序所分配到的CPU
Core的数目。

2. 一个计算每个分区的函数。Spark中RDD的计算是以分片为单位的，每个RDD都会实现compute函数以达到这个目的。
compute函数会对迭代器进行复合，不需要保存每次计算的结果。

3. RDD之间的依赖关系。RDD的每次转换都会生成一个新的RDD，所以RDD之间就会形成类似于流水线一样的前后依赖关
系。在部分分区数据丢失时，Spark可以通过这个依赖关系重新计算丢失的分区数据，而不是对RDD的所有分区进行重新计
算。

4. 一个Partitioner，即RDD的分片函数。当前Spark中实现了两种类型的分片函数，一个是基于哈希的HashPartitioner，另外一个
是基于范围的RangePartitioner。只有对于key-value的RDD，才会有Partitioner，非key-value的RDD的Parititioner的值是None。
Partitioner函数不但决定了RDD本身的分片数量，也决定了parent RDD Shuffle输出时的分片数量。

5. 一个列表，存储存取每个Partition的优先位置（preferred location）。对于一个HDFS文件来说，这个列表保存的就是每个
Partition所在的块的位置。按照“移动数据不如移动计算”的理念，Spark在进行任务调度的时候，会尽可能地将计算任务分配到
其所要处理数据块的存储位置。

## RDD操作

RDD支持两种操作：转换（trans-formation），即从现有的数据集创建一个新的数
据集；动作（action），即在数据集上进行计算后，返回一个值给Driver程序。例如，map就是一种转换，它将数据集每一个元
素都传递给函数，并返回一个新的分布式数据集表示结果。另一方面，reduce是一种动作，通过一些函数将所有元素叠加起
来，并将最终结果返回给Driver（还有一个并行的reduceByKey，能返回一个分布式数据集）。

## RDD转换

RDD中的所有转换都是惰性的，也就是说，它们并不会直接计算结果。相反的，它们只是记住这些应用到基础数据集（例如一个文件）上的
转换动作。只有当发生一个要求返回结果给Driver的动作时，这些转换才会真正运行。

# 与Hadoop MapReduce 优势

更准确地说，Spark是一个计算框架，而Hadoop中包含计算框架MapReduce和分布式文
件系统HDFS，Hadoop更广泛地说还包括在其生态系统上的其他系统，如Hbase、Hive等。
Spark是MapReduce的替代方案，而且兼容HDFS、Hive等分布式存储层，可融入
Hadoop的生态系统，以弥补缺失MapReduce的不足。

Spark相比Hadoop MapReduce的优势如下。

1. 中间结果输出

## 容错能力

# Spark运行逻辑

# 算子

# 现在的问题

# 改进与不足

# Spark调试

Application、Job、Stage、Task

# 代码