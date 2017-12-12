---
categories: lucene

tags: 
  - Lucene
  - 翻译

title: Elasticsearch插件和集成[5.5] -- 插件管理

date: 2017-07-13
---

Apache Lucene中有一个类`MemoryIndex`，该类是一个高性能单文档，内存全文搜索索引。

# 概述

MemoryIndex类是RAMDirectory类大部分功能的替代。它旨在实现实时匹配的最大效率，将实时流式应用程序中的结构化和模糊全文搜索结合起来，例如基于Nux XQuery的XML消息队列，用于博客/新闻馈送的发布订阅系统，文本聊天，数据采集和分发系统 ，应用级路由器，防火墙，分类器等。而不是通过巨大的持久性数据归档（历史搜索）来定位全息搜索偶然查询，而是通过相对较小的瞬时实时数据（预期搜索）来查找大量查询的全文搜索。例如在

```Java
float score = search(String text, Query query)
```
每个实例最多可以容纳一个Lucene“document”，其中document包含零个或多个“fields”，每个fields具有名称和全文值。 根据分析器（Analyzer）实施的策略，将全文值在addField()上进行标记化（拆分和转换）为零个或多个索引项（也称为单词）。 例如，Lucene分析器可以在空格上分割，默认分析器不区分大小写，会全部转换为小写，忽略诸如“他”，“在”，“和”（停止词）之间具有很少歧视价值的常见术语，减少了将词分解为自然语言学的词根形式， 诸如“钓鱼”被缩减为“鱼”（词干），解析同义词/拐点/叙词表（索引和/或查询）等。有关详细信息，请参阅[Lucene Analyzer简介](http://today.java.net/pub/a/today/2003/07/30/LuceneIntro.html)。

可以对此类运行任意Lucene查询 - 请参阅Lucene查询语法以及查询解析器规则。 请注意，Lucene查询选择字段名称和关联（索引）标记术语，而不是原始全文 - 后者不存储，而是在标记化后立即丢弃。

有关搜索技术的一些有趣的背景信息，请参阅Bob Wyman的前瞻性搜索，Jim Gray的“A Call to Arms” - 定制订阅以及Tim Bray的“On Search”系列。

# 使用示例


```Java
Analyzer analyzer = new SimpleAnalyzer(version);
MemoryIndex index = new MemoryIndex();
index.addField("content", "Readings about Salmons and other select Alaska fishing Manuals", analyzer);
index.addField("author", "Tales of James", analyzer);
QueryParser parser = new QueryParser(version, "content", analyzer);
float score = index.search(parser.parse("+author:james +salmon~ +fish* manual~"));
if (score > 0.0f) {
    System.out.println("it's a match");
} else {
    System.out.println("no match found");
}
System.out.println("indexData=" + index.toString());
```
# XQuery使用示例


```
(: An XQuery that finds all books authored by James that have something to do with "salmon fishing manuals", sorted by relevance :)
declare namespace lucene = "java:nux.xom.pool.FullTextUtil";
declare variable $query := "+salmon~ +fish* manual~"; (: any arbitrary Lucene query can go here :)
 
for $book in /books/book[author="James" and lucene:match(abstract, $query) > 0.0]
let $score := lucene:match($book/abstract, $query)
order by $score descending
return $book
```

# 线程安全保证

MemoryIndex对于添加或查询通常不是线程安全的。 但是，在调用了 freeze() 之后，查询是线程安全的。

# 性能说明

在内部，有一个面向高效索引和搜索的新数据结构，以及无缝插入Lucene框架的必要支持代码。

此类为非常小的文本（例如，10个字符），以及用于大文本（例如，10 MB）之间的一切有很好的表现。 通常，它比RAMDirectory快10-100倍。 请注意，RAMDirectory在时间和空间上对于中小型文本具有特别大的效率开销。 使用N个令牌索引一个字段在最好的情况下取O（N），在最坏的情况下取O（N logN）。 内存消耗可能大于RAMDirectory。

单个MemoryIndex上的许多简单术语查询的吞吐量示例：MacBook Pro，jdk 1.5.0_06，服务器VM上的约500000个查询/秒。

如果您对性能瓶颈在哪感到好奇，请使用无扰动的'-server -agentlib:hprof=cpu=samples,depth=10'标志运行java 1.5，然后研究跟踪日志并将其hotspot预告片与其呼叫相关联 堆栈头（见hprof跟踪）。