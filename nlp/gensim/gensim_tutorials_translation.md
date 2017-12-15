---
categories: Gensim

tags: 
  - Gensim
  - NLP
  - 翻译

title: Gensim官方教程翻译——快速入门

date: 2017-12-15
---
原文：https://radimrehurek.com/gensim/tutorial.html

本教程按照一系列的例子组织，用以突出gensim的各种功能。本教程的受众是熟悉[Python](http://www.python.org/)，已经[安装了gensim](http://radimrehurek.com/gensim/install.html)，而且阅读过[介绍](http://blog.geekidentity.com/nlp/gensim/gensim_introduction_translation/)的读者。

这些例子由以下几部分组成：

- 语料库与向量空间
  - 从字符串到向量
  - 语料库流-一次一个文档
  - 语料库格式
  - 与NumPy和SciPy的兼容性
- 主题与转换
  - 转换接口
  - 可用的转换
- 相似性查询
  - 相似性接口
  - 接下来做什么？
- 对于英文维基百科的实验
  - 准备语料库
  - 潜在语义分析
  - 隐含狄利克雷分配
- 分布式计算
  - 为什么需要分布式计算？
  - 先决条件
  - 核心概念
  - 可用的分布式算法

# 准备

所有的例子都可以复制到你的Python解释器窗口。[IPython](http://ipython.scipy.org/)的cpaste命令对于复制-粘贴代码片段十分方便，包括无意义的前导“>>>”字符。

Gensim使用Python标准的日志类来记录不同优先级的各种事件，想要激活日志（可选的），运行如下代码：

```python
>>> import logging
>>> logging.basicConfig(format='%(asctime)s : %(levelname)s : %(message)s', level=logging.INFO)
```

# 快速示例

首先，让我们导入gensim并创建一个小小的语料库，其中有9个（每行代表一个）文档和12个属性[[1\]](http://www.cs.bham.ac.uk/~pxt/IDA/lsa_ind.pdf)：

```python
>>> from gensim import corpora, models, similarities
>>>
>>> corpus = [[(0, 1.0), (1, 1.0), (2, 1.0)],
>>>           [(2, 1.0), (3, 1.0), (4, 1.0), (5, 1.0), (6, 1.0), (8, 1.0)],
>>>           [(1, 1.0), (3, 1.0), (4, 1.0), (7, 1.0)],
>>>           [(0, 1.0), (4, 2.0), (7, 1.0)],
>>>           [(3, 1.0), (5, 1.0), (6, 1.0)],
>>>           [(9, 1.0)],
>>>           [(9, 1.0), (10, 1.0)],
>>>           [(9, 1.0), (10, 1.0), (11, 1.0)],
>>>           [(8, 1.0), (10, 1.0), (11, 1.0)]]
```
在gensim中语料库只是一个对象，我们能够通过不断迭代从中取出用稀疏向量代表的文档。在这里，我们使用元组列表。如果你不熟悉[向量空间模型](https://en.wikipedia.org/wiki/Vector_space_model)，我们将会在下一个教程[《语料库与向量空间》](https://en.wikipedia.org/wiki/Vector_space_model)填平**原始字符串**、**语料库**、**稀疏向量**之间的鸿沟。

如果你熟悉向量空间模型，你可能会知道解析文档并将其转换为向量的方式对后续应用程序的质量有重大影响。

> 在这个例子中，整个语料作为一个Python List被存在内存中。但是，语料库接口仅仅规定一个语料库必须支持迭代取出其文档。对于特别大的语料库，最好将整个语料库存在硬盘上，并且按序一次取出一篇文档。所有的操作和转换通过一种语料库大小无关的方式实现，内存依赖较低。  

接下来，让我们初始化一个*转换*

```python
>>> tfidf = models.TfidfModel(corpus)
```

转换就是将文档的一种向量表示方式转换为另一种向量表示方式（以便我们从特定的角度更好地分析数据）：

```python
>>> vec = [(0, 1), (4, 1)]
>>> print(tfidf[vec])
[(0, 0.8075244), (4, 0.5898342)]
```

在此，我们使用了[Tf-Idf](https://en.wikipedia.org/wiki/Tf%E2%80%93idf)，这是一种简单的转换。它要求输入的文档用带有词频的词袋(bag-of-words)的方法表示，可以用来降低常用词的权重（相对地提高了罕见词的权重）。它还会把结果向量的长度调整为单位长度（指[欧几里得范数](https://en.wikipedia.org/wiki/Norm_%28mathematics%29#Euclidean_norm)）。

转化方法详情，请看教程[主题与转换](https://radimrehurek.com/gensim/tut2.html)

为了将整个语料库通过Tf-idf转化并索引，以便相似度查询，需要做如下准备：

```python
>>> index = similarities.SparseMatrixSimilarity(tfidf[corpus], num_features=12)
```

为了查询我们需要的向量vec相对于其他所有文档的相似度，需要：

```python
>>> sims = index[tfidf[vec]]
>>> print(list(enumerate(sims)))
[(0, 0.4662244), (1, 0.19139354), (2, 0.24600551), (3, 0.82094586), (4, 0.0), (5, 0.0), (6, 0.0), (7, 0.0), (8, 0.0)]
```

如何解释这些输出呢？文档编号为零（第1个文档）与vec的相似度为0.466=46.6%，第二个文档与vec的相似度为19.1%，依次类推。 因此，根据Tfidf文档表示方法和余弦相似度方法，与我们的查询文档vec最相似的文档为3号文档，相似度达到82.1%。注意：4-8号文档与vec没有任何公共的属性，因此相似度为0.0。了解详细请看[《相似度查询》](http://radimrehurek.com/gensim/tut3.html)教程。

[1] 这与Deerwester等人在 [Deerwester et al. (1990): Indexing by Latent Semantic Analysis](http://www.cs.bham.ac.uk/~pxt/IDA/lsa_ind.pdf), Table2使用的语料库相同。
[2] http://blog.csdn.net/questionfish/article/details/46725475
