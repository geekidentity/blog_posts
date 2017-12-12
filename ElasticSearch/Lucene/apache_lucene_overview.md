# Welcome to Apache Lucene

Apache Lucene项目是一个开源搜索软件，包括：
* [Lucene Core](https://lucene.apache.org/core/)：我们的旗舰子项目提供基于Java的索引和搜索技术，以及拼写检查，高亮显示和高级分析/标记功能。
* [Solr](https://lucene.apache.org/solr)：Solr是使用Lucene Core构建的高性能搜索服务器，具有XML/HTTP和 JSON/Python/Ruby API，高亮显示，分面搜索，缓存，复制和Web管理界面。
* [PyLucene](https://lucene.apache.org/pylucene/index.html)：是Lucene Core 项目的Python 端口与包装。

# Apache Lucene Core

Apache Lucene是一种完全用Java编写的高性能，全功能的文本搜索引擎库。这是一种适用于几乎所有需要全文搜索的应用程序的技术，尤其是跨平台的。

Apache Lucene是一个可免费下载的开源项目。

## Lucene 特点

Lucene通过简单的API提供强大的功能：

### 可扩展，高性能索引

* 在现代的硬件条件下，处理能力超过150GB/小时
* 更小的内存要求 -- 只要1MB的堆空间
* 增量索引与批次索引一样快
* 索引大小约为文本索引大小的20-30％

### 强大，准确和高效的搜索算法

* 排名搜索 - 最好的结果首先返回
* 许多强大的查询类型：短语查询，通配符查询，邻近查询，范围查询等
* 字段(fielded)搜索（例如标题，作者，内容）
* 按任何字段进行排序
* 具有合并结果的多重索引搜索
* 允许同时更新和搜索
* 灵活的分面(faceting)，高亮显示，连接和结果分组
* 快速的，高效内存和错字容忍的suggesters
* 可插拔的排名模型，包括[矢量空间模型( Vector Space Model)](http://en.wikipedia.org/wiki/Vector_Space_Model)和[Okapi BM25](http://en.wikipedia.org/wiki/Okapi_BM25)
* 可配置存储引擎（codecs）

### 跨平台解决方案

* 作为[Apache License](http://www.apache.org/licenses/LICENSE-2.0.html)下的开源软件，您可以在商业和开源程序中使用Lucene
* 100％纯Java
* 索引兼容使其可以在其他编程语言中实现

# Solr

Solr具有高可靠性，可扩展性和容错性，可提供分布式索引，复制和负载平衡查询，自动故障转移和恢复，集中配置等功能。 Solr为世界上许多最大的互联网站点提供搜索和导航功能。

# PyLucene

## 什么是PyLucene

PyLucene是访问Java Lucene的Python扩展。它的目标是允许您通过Python使用Lucene的文本索引和搜索功能。它是与2017年4月6日最新版本的Java Lucene版本6.5.0兼容的API。

PyLucene不是Lucene端口(port)，而是围绕Java Lucene的Python包装。 PyLucene将一个带有Lucene的Java虚拟机嵌入到Python进程中。 PyLucene Python扩展，一个名为lucene的Python模块由JCC生成。

PyLucene是由[JCC](https://lucene.apache.org/pylucene/jcc/index.html)构建的，它是一个C++ 代码生成器，可以通过Java的Native Invocation Interface（JNI）从Python调用Java类。JCC的来源包括PyLucene来源。

有关PyLucene的更多信息和文档，请参阅[此处](https://lucene.apache.org/pylucene/features.html)。

## 要求

PyLucene支持macOS，Linux，Solaris和Windows。

PyLucene需要Python 2.x版（x >= 3.5）或Python 3.x（ x >= 3）和Java 1.8版本。构建PyLucene需要GNU Make，最新版本的Ant能够构建Java Lucene和C++编译器。 建议使用 [setuptools](http://pypi.python.org/pypi/setuptools)。

有关从源头构建JCC的更多信息，请参阅[JCC安装说明](https://lucene.apache.org/pylucene/jcc/install.html)。

有关从源代码构建PyLucene的更多信息，请参阅[PyLucene安装说明](https://lucene.apache.org/pylucene/install.html)。