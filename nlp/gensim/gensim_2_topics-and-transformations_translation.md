---
categories: Gensim

tags: 
  - Gensim
  - NLP
  - 翻译

title: Gensim官方教程翻译（二）——主题与转换（Topics and Transformations）

date: 2017-12-17
---
如果想要开启日志，别忘记设置：

```python
>>> import logging
>>> logging.basicConfig(format='%(asctime)s : %(levelname)s : %(message)s', level=logging.INFO)
```

## 转换接口

在之前的教程[《语料库与向量空间》](http://blog.geekidentity.com/nlp/gensim/gensim_1_corpora-and-vector-spaces_translation/)中，我们创建了一个用向量流表示文档的语料库。为了继续征程，让我们启动gensim并使用该语料库。

```python
>>> from gensim import corpora, models, similarities
>>> dictionary = corpora.Dictionary.load('/tmp/deerwester.dict')
>>> corpus = corpora.MmCorpus('/tmp/deerwester.mm')
>>> print(corpus)
MmCorpus(9 documents, 12 features, 28 non-zero entries)
```

在本次教程中，我将会向你展示如何将文档从一种向量表示方式转换到另一种。这个处理是为了两个目的：

1. 将语料库中隐藏的结构发掘出来，发现词语之间的关系，并且利用这些结构、关系使用一种新的、更有语义价值的（这是我们最希望的）方式描述其中的文档。
2. 使得表示方式更加简洁。这样不仅能提高效率（新的表示方法一般消耗较少的资源）还能提高效果（忽略了边际数据趋势、降低了噪音）。

### 创建一个转换

转换（transformations）是标准的Python对象，通常通过训练语料库的方式初始化

```python
>>> tfidf = models.TfidfModel(corpus) # 第一步 -- 初始化一个模型
```

我们使用了[前一个教程](http://blog.geekidentity.com/nlp/gensim/gensim_1_corpora-and-vector-spaces_translation/)中用过的语料库来初始化（训练）这个转换模型。不同的转换可能需要不同的初始化参数；在Tfidf案例中，“训练”仅仅是遍历提供的语料库然后计算所有属性的文档频率（译者注：在多少文档中过）。训练其他模型，例如潜在语义分析或隐含狄利克雷分配，更加复杂，因此耗时也多。

> 作者注：转换常常是在两个特定的向量空间之间进行。训练与后续的转换必须使用相同的向量空间（=有相同的属性，且编号相同）。如若不然，例如使用不同的预处理方法处理字符串、使用不同的属性编号、需要Tfidf向量的时候却输入了词袋向量，将会导致转换过程中属性匹配错误，进而使输出结果无意义并可能引发异常。

### 转换向量

从现在开始，tfidf将被视为只读的对象，可以用它来转换将任何采用旧表示方法的向量（词袋整数计数）转换为新的表示方法（Tfidf 实数权重）：

```python
>>> doc_bow = [(0, 1), (1, 1)]
>>> print(tfidf[doc_bow]) # 第二步 -- 使用模型转换向量
[(0, 0.70710678), (1, 0.70710678)]
```

或者对整个语料库实施转换：

```python
>>> corpus_tfidf = tfidf[corpus]
>>> for doc in corpus_tfidf:
...     print(doc)
[(0, 0.57735026918962573), (1, 0.57735026918962573), (2, 0.57735026918962573)]
[(0, 0.44424552527467476), (3, 0.44424552527467476), (4, 0.44424552527467476), (5, 0.32448702061385548), (6, 0.44424552527467476), (7, 0.32448702061385548)]
[(2, 0.5710059809418182), (5, 0.41707573620227772), (7, 0.41707573620227772), (8, 0.5710059809418182)]
[(1, 0.49182558987264147), (5, 0.71848116070837686), (8, 0.49182558987264147)]
[(3, 0.62825804686700459), (6, 0.62825804686700459), (7, 0.45889394536615247)]
[(9, 1.0)]
[(9, 0.70710678118654746), (10, 0.70710678118654746)]
[(9, 0.50804290089167492), (10, 0.50804290089167492), (11, 0.69554641952003704)]
[(4, 0.62825804686700459), (10, 0.45889394536615247), (11, 0.62825804686700459)]
```

在这个特殊的情况中，被转换的语料库与用来训练的语料库相同，但是这仅仅是偶然。一旦转换模型被初始化了，它可以用来转换任何向量（当然最好使用与训练语料库相同的向量空间 | 注：即语言环境），即使它们并没有在训练语料库中出现。这是通过潜在语义分析的调入（folding in）、隐含狄利克雷分配的主题推断（topic inference）等得到的。

> 调用model[corpus]只能在旧的corpus文档流的基础上创建一个包装-真正的转化是在迭代文档时即时计算的。我们可以通过调用corpus_transformed = model[corpus]一次转化整个语料库，因为这样意味着我们要将结果存入内存中，这与gensim内存无关的设计理念不太符合。如果你还要多次迭代转化后的corpus_transformed，负担将会十分巨大。你可以先将结果[序列化并存储到硬盘上](http://radimrehurek.com/gensim/tut1.html#corpus-formats)再做需要的操作。

转换也可以被序列化，还可以一个（转换）叠另一个，像一串链条一样：

```python
>>> lsi = models.LsiModel(corpus_tfidf, id2word=dictionary, num_topics=2) # 初始化一个LSI转换
>>> corpus_lsi = lsi[corpus_tfidf] # 在原始语料库上加上双重包装: bow->tfidf->fold-in-lsi
```

这里我们利用[潜在语义索引（LSI）](http://en.wikipedia.org/wiki/Latent_semantic_indexing) 将Tf-Idf语料转化为一个潜在2-D空间（2-D是因为我们设置了num_topics=2）。现在你可能想知道：2潜在维度意味着什么？让我们利用`models.LsiModel.print_topics()`来检查一下这个过程到底产生了什么变化吧：

```python
>>> lsi.print_topics(2)
topic #0(1.594): -0.703*"trees" + -0.538*"graph" + -0.402*"minors" + -0.187*"survey" + -0.061*"system" + -0.060*"response" + -0.060*"time" + -0.058*"user" + -0.049*"computer" + -0.035*"interface"
topic #1(1.476): -0.460*"system" + -0.373*"user" + -0.332*"eps" + -0.328*"interface" + -0.320*"response" + -0.320*"time" + -0.293*"computer" + -0.280*"human" + -0.171*"survey" + 0.161*"trees"
```

（这些主题将会记录在日志中，想要了解如何激活日志，请看开头的注解）

根据LSI来看，“tree”、“graph”、“minors”都是相关的词语（而且在第一主题的方向上贡献最多），而第二主题实际上与所有的词语都有关系。如我们所料，前五个文档与第二个主题的关联更强，而其他四个文档与第一个主题关联最强：

```python
>>> for doc in corpus_lsi: # both bow->tfidf and tfidf->lsi transformations are actually executed here, on the fly
...     print(doc)
[(0, -0.066), (1, 0.520)] # "Human machine interface for lab abc computer applications"
[(0, -0.197), (1, 0.761)] # "A survey of user opinion of computer system response time"
[(0, -0.090), (1, 0.724)] # "The EPS user interface management system"
[(0, -0.076), (1, 0.632)] # "System and human system engineering testing of EPS"
[(0, -0.102), (1, 0.574)] # "Relation of user perceived response time to error measurement"
[(0, -0.703), (1, -0.161)] # "The generation of random binary unordered trees"
[(0, -0.877), (1, -0.168)] # "The intersection graph of paths in trees"
[(0, -0.910), (1, -0.141)] # "Graph minors IV Widths of trees and well quasi ordering"
[(0, -0.617), (1, 0.054)] # "Graph minors A survey"
```

模型的持久可以借助`save()`和`load()`函数完成：

```python
>>> lsi.save('/tmp/model.lsi') # same for tfidf, lda, ...
>>> lsi = models.LsiModel.load('/tmp/model.lsi')
```

下一个问题可能就该是：这些文档之间确切的相似度是多少呢？能否将相似性形式化，以便给定一个文档，我们能够根据其他文档与该文档的相似度排序呢？敬请阅读下个教程——《相似度查询》。

## 可用的转换

Gensim实现了几种常见的向量空间模型算法：

### [词频-逆向文档频率（Term Frequency * Inverse Document Frequency， Tf-Idf）](http://en.wikipedia.org/wiki/Tf%E2%80%93idf)

需要一个词袋形式（整数值）的训练语料库来实现初始化。转换过程中，他将会接收一个向量同时返回一个相同维度的向量，在语料库中非常稀有的属性的权重将会提高。因此，他会将整数型的向量转化为实数型的向量，同时让维度不变。而且。你可以选择是否将返回结果标准化至单位长度（欧几里得范数）。

```
 >>> model = tfidfmodel.TfidfModel(bow_corpus, normalize=True)
```

### [潜在语义索引（Latent Semantic Indexing，LSI，or sometimes LSA）](http://en.wikipedia.org/wiki/Latent_semantic_indexing)

将文档从词袋或TfIdf权重空间（更好）转化为一个低维的潜在空间。对于我们上面用到的玩具级的语料库，我们使用了2潜在维度，但是在真正的语料库上，推荐200-500的目标维度为“金标准”。[[1\]](http://radimrehurek.com/gensim/tut2.html#id6)

```
>>> model = lsimodel.LsiModel(tfidf_corpus, id2word=dictionary, num_topics=300)1
```

LSI训练的独特之处是我们能在任何继续“训练”，仅需提供更多的训练文本。这是通过对底层模型进行增量更新，这个过程被称为“在线训练”。正因为它的这个特性，输入文档流可以是无限大——我们能在以只读的方式使用计算好的模型的同时，还能在新文档到达时一直“喂食”给LSI“消化”！

```
>>> model.add_documents(another_tfidf_corpus) # 现在LSI已经使用tfidf_corpus + another_tfidf_corpus进行过训练了
>>> lsi_vec = model[tfidf_vec] # 将新文档转化到LSI空间不会影响该模型
>>> ...
>>> model.add_documents(more_documents) # tfidf_corpus + another_tfidf_corpus + more_documents
>>> lsi_vec = model[tfidf_vec]
>>> ...123456
```

有关在无限大的流中，如何让LSI逐渐“忘记”旧的观测结果，详情请看[gensim.models.lsimodel](http://radimrehurek.com/gensim/models/lsimodel.html#module-gensim.models.lsimodel)的帮助文档。如果你不怕麻烦，有几个参数可以控制LSI算法的影响速度、内存占用和数值精度等。 
gensim使用一个新颖的在线增量流分布式训练算法（还挺拗口的…），我曾将该方法发表在[[5\]](http://radimrehurek.com/gensim/tut2.html#id10)中。gensim内部执行了一个来自Halko等[[4\]](http://radimrehurek.com/gensim/tut2.html#id9)的随机多通道算法（stochastic multi-pass algorithm）来加速核心（in-core）部分的计算。参考[《在英文维基百科上的实验》](http://radimrehurek.com/gensim/wiki.html)教程了解如何通过计算机集群分布式计算来提高速度。

### [随机映射（Random Projections，RP）](http://www.cis.hut.fi/ella/publications/randproj_kdd.pdf)

目的在于减小空维度。这是一个非常高效（对CPU和内存都很友好）方法，通过抛出一点点随机性，来近似得到两个文档之间的Tfidf距离。推荐目标维度也是成百上千，具体数值要视你的数据集大小而定。

```
>>> model = rpmodel.RpModel(tfidf_corpus, num_topics=500)1
```

### [隐含狄利克雷分配（Latent Dirichlet Allocation, LDA）](http://en.wikipedia.org/wiki/Latent_Dirichlet_allocation)

也是将词袋计数转化为一个低维主题空间的转换。LDA是LSA（也叫多项式PCA）的概率扩展，因此LDA的主题可以被解释为词语的概率分布。这些分布式从训练语料库中自动推断的，就像LSA一样。相应地，文档可以被解释为这些主题的一个（软）混合（又是就像LSA一样）。

```
>>> model = ldamodel.LdaModel(bow_corpus, id2word=dictionary, num_topics=100)1
```

gensim使用一个基于[[2\]](http://radimrehurek.com/gensim/tut2.html#id7)的快速的在线LDA参数估计实现，修改并使其可以在计算机集群上以分布式模式运行。

### [分层狄利克雷过程（Hierarchical Dirichlet Process，HDP）](http://jmlr.csail.mit.edu/proceedings/papers/v15/wang11a/wang11a.pdf)

是一个无参数贝叶斯方法（注意：这里没有num_topics参数）：

```
>>> model = hdpmodel.HdpModel(bow_corpus, id2word=dictionary)1
```

gensim使用一种基于[[3\]](http://radimrehurek.com/gensim/tut2.html#id8)的快速在线来实现。该算法是新加入gensim的，并且还是一种粗糙的学术边缘产品——小心使用。 
增加新的VSM转化（例如新的权重方案）相当平常；参见[API参考](http://radimrehurek.com/gensim/apiref.html)或者直接参考我们的[源代码](https://github.com/piskvorky/gensim/blob/develop/gensim/models/tfidfmodel.py)以获取信息与帮助。

值得一提的是，这些模型增量模型，无需一次将所有的训练语料库全部放到内存中。在关心内存的同时，我还在不断改进[分布式计算](http://radimrehurek.com/gensim/distributed.html)，来提高CPU效率。如果你自觉能够贡献一份力量（测试、提供用例或代码），请让[我知道](http://blog.csdn.net/questionfish/article/details/radimrehurek@seznam.cz)。

下一个教程，我们将会讲《相似度查询》。

[1]	Bradford. 2008. An empirical study of required dimensionality for large-scale latent semantic indexing applications.
[2]	Hoffman, Blei, Bach. 2010. Online learning for Latent Dirichlet Allocation.
[3]	Wang, Paisley, Blei. 2011. Online variational inference for the hierarchical Dirichlet process.
[4]	Halko, Martinsson, Tropp. 2009. Finding structure with randomness.
[5]	Řehůřek. 2011. Subspace tracking for Latent Semantic Analysis.
[6]   http://blog.csdn.net/questionfish/article/details/46742403