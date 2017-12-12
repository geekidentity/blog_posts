---
categories: beam

tags: 
  - Apache Beam

title: Apache Beam Java SDK 快速开始

date: 2017-02-08

---
本快速入门将指导您完成第一个Beam pipeline，以便在您选择的runner 上运行使用Beam的Java SDK编写的WordCount。
- 设置开发环境
- 获取WordCount代码
- 运行WordCount
- 检查结果
- 下一步

## 设置开发环境
1. 下载并安装[Java 开发工具包（JDK）](http://www.oracle.com/technetwork/java/javase/downloads/index.html)1.7或更高版本。 验证是否已设置[JAVA_HOME](https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/envvars001.html)环境变量并指向JDK 安装目录。
1. 按照指定操作系统的[Maven安装指南](http://maven.apache.org/install.html)，下载并安装 [Apache Maven](http://maven.apache.org/download.cgi)。

## 获取WordCount代码

获取WordCount pipeline 拷贝的最简单方法是使用以下命令生成一个简单的Maven项目，其中包含Beam的WordCount示例，并针对最新的Beam版本进行构建：

``` bash
$ mvn archetype:generate \
      -DarchetypeRepository=https://repository.apache.org/content/groups/snapshots \
      -DarchetypeGroupId=org.apache.beam \
      -DarchetypeArtifactId=beam-sdks-java-maven-archetypes-examples \
      -DarchetypeVersion=LATEST \
      -DgroupId=org.example \
      -DartifactId=word-count-beam \
      -Dversion="0.1" \
      -Dpackage=org.apache.beam.examples \
      -DinteractiveMode=false
```

Maven 将创建目录word-count-beam，其中包含一个简单的pom.xml和一系列示例pipelines，用于文本文件中的字进行计数。

``` bash
$ cd word-count-beam/
 
$ ls
pom.xml src
 
$ ls src/main/java/org/apache/beam/examples/
DebuggingWordCount.java WindowedWordCount.java  common
MinimalWordCount.java   WordCount.java
```

有关这些示例中使用的Beam概念的详细介绍，请参见[WordCount示例演练](https://beam.apache.org/get-started/wordcount-example)。 这里，我们只关注执行WordCount.java。

## 运行WordCount

单个Beam pipeline 可以在Beam runners上运行，包括 [ApexRunner](https://beam.apache.org/documentation/runners/apex), [FlinkRunner](https://beam.apache.org/documentation/runners/flink), [SparkRunner](https://beam.apache.org/documentation/runners/spark) 和 [DataflowRunner](https://beam.apache.org/documentation/runners/dataflow).。  [DirectRunner](https://beam.apache.org/documentation/runners/direct)是一个常用的入门指南，因为它在本地运行，不需要特殊的设置。

在选择要使用的runner 之后：
1. 确保已完成任何特定于runner的设置。
2. 构建命令行：
    1. 使用--runner = <runner>（默认为DirectRunner）指定特定runner
    2. 添加runner 运行所需的选项
    3. 选择runner 可以访问的输入文件和输出位置。 （例如，如果正在外部集群上运行pipeline ，则无法访问本地文件。）
3. 运行你的第一个WordCount pipeline。
4. 
以Spark为例（其他示例请看官网文档）：

``` bash
$ mvn compile exec:java -Dexec.mainClass=org.apache.beam.examples.WordCount \
     -Dexec.args="--runner=SparkRunner --inputFile=pom.xml --output=counts" -Pspark-runner
```

## 检查结果

一旦pipeline完成，你可以查看输出。 你会注意到可能有多个输出文件以count为前缀。 这些文件的确切数目由运行程序决定，使其能够灵活地执行高效的分布式执行。

```
$ ls counts*
```

当您查看文件的内容时，您会看到它们包含唯一字词和每个字词的出现次数。 文件中的元素的顺序可能不同，因为beam 模型通常不保证排序，以再次允许runner 优化效率。

``` bash
$ more counts*
beam: 27
SF: 1
fat: 1
job: 1
limitations: 1
require: 1
of: 11
profile: 10
...
```

## 下一步

- 在[WordCount示例演练](https://beam.apache.org/get-started/wordcount-example)中了解有关这些WordCount示例的更多信息。
- 深入了解我们最喜欢的[文章和演示文稿](https://beam.apache.org/documentation/resources)。
- 加入Beam [用户@](https://beam.apache.org/documentation/resources)邮件列表。

如果您遇到任何问题，请随时与我们联系！