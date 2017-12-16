---
categories: Gensim

tags: 
  - Gensim
  - NLP
  - 翻译
  - 文本相似度

title: Gensim官方教程翻译（三）——相似度查询（Similarity Queries）

date: 2017-12-17
---

如果想要开启日志，别忘记设置：

```python
>>> import logging
>>> logging.basicConfig(format='%(asctime)s : %(levelname)s : %(message)s', level=logging.INFO)12
```

# 相似度接口

在之前的教程[《语料库与向量空间》](http://blog.geekidentity.com/nlp/gensim/gensim_1_corpora-and-vector-spaces_translation/)和[《主题与转换》](http://blog.geekidentity.com/nlp/gensim/gensim_2_topics-and-transformations_translation/)中，我们了解了创建在向量空间创建一个语料库意味着什么，如何在不同的向量空间之间转换。我们所做的一切都是为了一个共同目标：决定文档对之间的相似度或者一篇特定文档和其他文档之间的相似度（例如用户输入与已索引文档）。 
为了展示如何在gensim中做这项工作，让我们使用与之前相同的语料库（这个语料库真的来自于1990年的Deerwester等的[《 Indexing by Latent Semantic Analysis》](http://www.cs.bham.ac.uk/~pxt/IDA/lsa_ind.pdf)）：

```python
>>> from gensim import corpora, models, similarities
>>> dictionary = corpora.Dictionary.load('/tmp/deerwester.dict')
>>> corpus = corpora.MmCorpus('/tmp/deerwester.mm') # comes from the first tutorial, "From strings to vectors"
>>> print(corpus)
MmCorpus(9 documents, 12 features, 28 non-zero entries)12345
```

为了模仿Deerwester的例子，我们首先用这个微型语料库来定义一个2维LSI空间：

```python
lsi = models.LsiModel(corpus, id2word=dictionary, num_topics=2)1
```

现在假设一个用户键入查询“Human computer interaction”，我们应该对我们的九个语料库文档按照与该输入的关联性逆向排序。与现代搜索引擎不同，我们仅集中关注一个单一方面的可能的相似性——文本（单词）的语义关联性。没有超链接，没有随机行走静态排列，只是在关键词布尔匹配的基础上进行了语义扩展。

```python
>>> doc = "Human computer interaction"
>>> vec_bow = dictionary.doc2bow(doc.lower().split())
>>> vec_lsi = lsi[vec_bow] # convert the query to LSI space
>>> print(vec_lsi)
[(0, -0.461821), (1, 0.070028)]12345
```

此外，我们将会考虑[余弦相似性](http://en.wikipedia.org/wiki/Cosine_similarity)来决定两个向量的相似性。余弦相似性是一种向量空间模型的标准方法，但是对于表示概率分布的向量，其他相似度方法可能更好。

## 初始化查询结构

为了准备相似度查询，我们需要输入所有我们我们需要比较的文档。在本例中，他们是LSI训练中用到的被转换到2-D的LSA空间的9个文档。但是这只是偶然，我们也可能索引完全不同的语料库。

```python
>>> index = similarities.MatrixSimilarity(lsi[corpus]) # transform corpus to LSI space and index it1
```

> 警告：`similarities.MatrixSimilarity`类仅仅适合能将所有的向量都在内存中的情况。例如，如果一个百万文档级的语料库使用该类，可能需要2G内存与256维LSI空间。 
> 如果没有足够的内存，你可以使用`similarities.Similarity`类。该类的操作只需要固定大小的内存，因为他将索引切分为多个文件（称为碎片）存储到硬盘上了。它实际上使用了`similarities.MatrixSimilarity`和`similarities.SparseMatrixSimilarity`两个类，因此它也是比较快的，虽然看起来更加复杂了。

索引也可以通过标准的`save()`和`load()`函数来存储到硬盘上。

```
>>> index.save('/tmp/deerwester.index')
>>> index = similarities.MatrixSimilarity.load('/tmp/deerwester.index')12
```

所有的索引类都可以用这种方法（`similarities.Similarity`,`similarities.MatrixSimilarity`和`similarities.SparseMatrixSimilarity`)。下面也是，索引可以是任何一个索引类。如果有疑问，可以使用similarities.Similarity，因为它是最具扩展性的版本，它也支持之后增加更多的文档索引。

## 实施查询

为了获得我们的查询文档相对于其他9个经过索引的文档的相似度：

```
>>> sims = index[vec_lsi] # perform a similarity query against the corpus
>>> print(list(enumerate(sims))) # print (document_number, document_similarity) 2-tuples
[(0, 0.99809301), (1, 0.93748635), (2, 0.99844527), (3, 0.9865886), (4, 0.90755945),
(5, -0.12416792), (6, -0.1063926), (7, -0.098794639), (8, 0.05004178)]1234
```

余弦方法返回的相似度在-1~1之间（越大越相似），所以第一个文档分数为0.99809301等。 
使用一些标准Python函数，我们将这些相似性倒序排列，并且获得了查询“Human computer interaction”的结果。

```
>>> sims = sorted(enumerate(sims), key=lambda item: -item[1])
>>> print(sims) # print sorted (document number, similarity score) 2-tuples
[(2, 0.99844527), # The EPS user interface management system
(0, 0.99809301), # Human machine interface for lab abc computer applications
(3, 0.9865886), # System and human system engineering testing of EPS
(1, 0.93748635), # A survey of user opinion of computer system response time
(4, 0.90755945), # Relation of user perceived response time to error measurement
(8, 0.050041795), # Graph minors A survey
(7, -0.098794639), # Graph minors IV Widths of trees and well quasi ordering
(6, -0.1063926), # The intersection graph of paths in trees
(5, -0.12416792)] # The generation of random binary unordered trees1234567891011
```

（出处结果的注释是我加的，为了方便观察。） 
需要注意，使用标准的布尔全文搜索将不会返回编号分别为2和4的文档（The EPS user interface management system”，”Relation of user perceived response time to error measurement”），因为他们与“Human computer interaction”没有任何相同的单词。可是，在使用了LSI后，我们可以看到他们都得到了比较高的分数（2号文档是最相似的），直观感觉上他们都更加符合我们查询的“computer-human”（“人机”）相关主题。事实上，将语义概括化是我们首先使用转换和主题模型的原因。

# 接下来做什么

恭喜，你已经完成了教程-现在你知道gensim如何工作了。^_^为了钻研详细内容，你可以通篇浏览[API文档](http://radimrehurek.com/gensim/apiref.html)、阅读[《维基百科实验》](http://radimrehurek.com/gensim/wiki.html)或者可能试一试gensim的[分布式计算](http://radimrehurek.com/gensim/distributed.html)。

Gensim是一个比较成熟的工具包，很多公司、个人已经成功将该工具应用于快速原型和生产。但是也并不意味着他是完美的。

- 还有一些部分可以用更加高效地方式实现（例如用c实现），或者更好地利用并行（多台机器内核）。
- 随时都有可能发布新的算法；你可以通过[讨论](http://groups.google.com/group/gensim)与[贡献代码](https://github.com/piskvorky/gensim/wiki/Developer-page)帮助gensim跟上时代的步伐。
- 欢迎并且感激你的反馈（不仅仅是代码）：[创意贡献](https://github.com/piskvorky/gensim/wiki/Ideas-&-Features-proposals)、[Bug反馈](https://github.com/piskvorky/gensim/issues)或者仅仅是贡献你的使用经理及问题。 
  Gensim并没有野心成为一个能包含自然语言处理（NLP）（或者仅仅是机器学习）领域的一切的框架。他的任务只是帮助NLP实践者能轻松地尝试流行的主题模型算法应用于大型数据集，并且帮助研究人员设计新算法原型。