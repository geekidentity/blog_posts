---
categories: Gensim

tags: 
  - Gensim
  - NLP
  - 翻译

title: Gensim官方教程翻译（一）——语料库与向量空间（Corpora and Vector Spaces）

date: 2017-12-16
---

本教程在[这里](https://github.com/piskvorky/gensim/blob/develop/docs/notebooks/Corpora_and_Vector_Spaces.ipynb)可以作为Jupyter Notebook使用。

如果你想记录日志，请不要忘记设置：

```python
>>> import logging
>>> logging.basicConfig(format='%(asctime)s : %(levelname)s : %(message)s', level=logging.INFO)
```

## 从字符串到向量

这次，让我们从用字符串表示的文档开始：

```python
>>> from gensim import corpora
>>>
>>> documents = ["Human machine interface for lab abc computer applications",
>>>              "A survey of user opinion of computer system response time",
>>>              "The EPS user interface management system",
>>>              "System and human system engineering testing of EPS",
>>>              "Relation of user perceived response time to error measurement",
>>>              "The generation of random binary unordered trees",
>>>              "The intersection graph of paths in trees",
>>>              "Graph minors IV Widths of trees and well quasi ordering",
>>>              "Graph minors A survey"]
```

这是一个由9篇文档组成的微型语料库，每个文档仅有一个句子组成。

### 标记化

首先，让我们对这些文档进行标记化处理，删除常用词（利用停用词表）和整个语料库中仅仅出现一次的词：

```python
>>> # 删除停用词并分词
>>> # 译者注：这里只是例子，实际上还有其他停用词  
>>> #         处理中文时，请借助 Py结巴分词 https://github.com/fxsjy/jieba 
>>> stoplist = set('for a of the and to in'.split())
>>> texts = [[word for word in document.lower().split() if word not in stoplist]
>>>          for document in documents]
>>>
>>> # 去除仅出现一次的单词
>>> from collections import defaultdict
>>> frequency = defaultdict(int)
>>> for text in texts:
>>>     for token in text:
>>>         frequency[token] += 1
>>>
>>> texts = [[token for token in text if frequency[token] > 1]
>>>          for text in texts]
>>>
>>> from pprint import pprint  # pretty-printer
>>> pprint(texts)
[['human', 'interface', 'computer'],
 ['survey', 'user', 'computer', 'system', 'response', 'time'],
 ['eps', 'user', 'interface', 'system'],
 ['system', 'human', 'system', 'eps'],
 ['user', 'response', 'time'],
 ['trees'],
 ['graph', 'trees'],
 ['graph', 'minors', 'trees'],
 ['graph', 'minors', 'survey']]
```

你处理文件的方式可能会有所不同。在这里我仅利用空格切分字符串来标记化，并将它们都转成小写。实际上，我是在用这个特殊（简单且效率低）的设置来模仿Deerwester等人的原始LSA文章中的实验。[[1\]](http://radimrehurek.com/gensim/tut1.html#id3)

### 生成特征字典

处理文档的方法应该视应用情形、语言而定，因此我决定不用接口对处理方法做任何的限制。取而代之的是，一个文档必须由其中提取出来的特征（featrues）表示，而不仅仅是其字面的字符串形式表示；如何提取这些特征由你来决定（可以是单词、文档长度数量等）。接下来，我将描述一个通用的、常规目的的方法（称为词袋 bag-of-words），但是请记住不同应用领域应使用不同的属性。如若不然，渣进滓出（[garbage in, garbage out](http://en.wikipedia.org/wiki/Garbage_In,_Garbage_Out)）。
为了将文档转换为向量，我们将会用一种称为[词袋](http://en.wikipedia.org/wiki/Bag_of_words)（bag-of-words）的文档表示方法。这种表示方法，每个文档由一个向量表示，该向量的每个元素都代表这样一个问-答对：

*“‘系统’这个单词出现了多少次？1次。”*

我们最好用这些问题的（整数）编号来代替这些问题。问题与编号之间的映射，我们称其为字典（Dictionary）。**

```python
>>> dictionary = corpora.Dictionary(texts)
>>> dictionary.save('/tmp/deerwester.dict')  # 把字典保存起来，方便以后使用
>>> print(dictionary)
Dictionary(12 unique tokens)
```

上面这些步骤，我们利用[gensim.corpora.dictionary.Dictionary](http://radimrehurek.com/gensim/corpora/dictionary.html#gensim.corpora.dictionary.Dictionary)类为每个出现在语料库中的单词分配了一个独一无二的整数编号id。这个操作收集了单词计数及其他相关的统计信息。在结尾，我们看到语料库中有12个不同的单词，这表明每个文档将会用12个数字表示（即12维向量）。如果想要查看单词与编号之间的映射关系：

```python
>>> print(dictionary.token2id)
{'minors': 11, 'graph': 10, 'system': 5, 'trees': 9, 'eps': 8, 'computer': 0,
'survey': 4, 'user': 7, 'human': 1, 'time': 6, 'interface': 2, 'response': 3}
```

### 产生稀疏文档向量

为了真正将标记化的文档转换为向量，需要：

```python
>>> new_doc = "Human computer interaction"
>>> new_vec = dictionary.doc2bow(new_doc.lower().split())
>>> print(new_vec)  # "interaction"没有在dictionary中出现，因此会被忽略
[(0, 1), (1, 1)]
```

函数doc2bow() 简单地对每个不同单词的出现次数进行了计数，并将单词转换为其编号，然后以稀疏向量的形式返回结果。因此，稀疏向量[(0, 1), (1, 1)]表示：在“Human computer interaction”中“computer”(id 0) 和“human”(id 1)各出现一次；其他10个dictionary中的单词没有出现过（隐含的）。

```python
>>> corpus = [dictionary.doc2bow(text) for text in texts]
>>> corpora.MmCorpus.serialize('/tmp/deerwester.mm', corpus) # 存入硬盘，以备后需
>>> pprint(corpus)
[(0, 1), (1, 1), (2, 1)]
[(0, 1), (3, 1), (4, 1), (5, 1), (6, 1), (7, 1)]
[(2, 1), (5, 1), (7, 1), (8, 1)]
[(1, 1), (5, 2), (8, 1)]
[(3, 1), (6, 1), (7, 1)]
[(9, 1)]
[(9, 1), (10, 1)]
[(9, 1), (10, 1), (11, 1)]
[(4, 1), (10, 1), (11, 1)]
```

（通过上面的操作，我们看到了这次我们得到的语料库。）到现在为止，我们应该明确，上面的输出表明：对于前六个文档来说，编号为10的属性值为0表示问题“文档中‘graph’出现了几次”的答案是“0”；而其他文档的答案是1。事实上，我们得到了[《快速入门》](http://blog.geekidentity.com/nlp/gensim/gensim_tutorials_translation/)中的示例语料库。

## 语料库流——一次一个文档

需要注意的是，上面的语料库整个作为一个Python List存在了内存中。在这个简单的例子中，这当然无关紧要。但是我们因该清楚，假设我们有一个百万数量级文档的语料库，我们不可能将整个语料库全部存入内存。假设这些文档存在一个硬盘上的文件中，每行一篇文档。Gemsim仅要求一个语料库可以每次返回一个文档向量：

```python
>>> class MyCorpus(object):
>>>     def __iter__(self):
>>>         for line in open('mycorpus.txt'):
>>>             # assume there's one document per line, tokens separated by whitespace
>>>             yield dictionary.doc2bow(line.lower().split())
```

请在这里下载示例文件[mycorpus.txt](http://radimrehurek.com/gensim/mycorpus.txt)。

这里假设的在一个单独的文件中每个文档占一行不是十分重要；你可以改造 iter 函数来适应你的输入格式，无论你的输入格式是什么样的，例如遍历文件夹、解析XML、访问网络等等。你仅需在每个文档中解析出一个由标记（tokens）组成的干净列表，然后利用dictionary将这些符号转换为其id，最后在iter函数中产生一个稀疏向量即可。

```python
>>> corpus_memory_friendly = MyCorpus() # 没有将整个语料库载入内存  
>>> print(corpus_memory_friendly)  
<__main__.MyCorpus object at 0x10d5690>  
```

现在的语料库是一个对象。我们没有定义任何打印它的方法，所以仅能打印该对象在内存中的地址，对我们没什么帮助。为了查看向量的组成，让我们通过迭代的方式取出语料库中的每个文档向量（一次一个）并打印：

```python
>>> for vector in corpus_memory_friendly: # 一次读入内存一个向量
...     print(vector)
[(0, 1), (1, 1), (2, 1)]
[(0, 1), (3, 1), (4, 1), (5, 1), (6, 1), (7, 1)]
[(2, 1), (5, 1), (7, 1), (8, 1)]
[(1, 1), (5, 2), (8, 1)]
[(3, 1), (6, 1), (7, 1)]
[(9, 1)]
[(9, 1), (10, 1)]
[(9, 1), (10, 1), (11, 1)]
[(4, 1), (10, 1), (11, 1)]
```

虽然输出与普通的Python List一样，但是现在的语料库对内存更加友好，因为一次最多只有一个向量寄存于内存中。你的语料库现在可以想多大就多大啦！

相似的，为了构造dictionary我们也不必将全部文档读入内存：

```python
>>> # 收集所有符号的统计信息
>>> dictionary = corpora.Dictionary(line.lower().split() for line in open('mycorpus.txt'))
>>> # 收集停用词和仅出现一次的词的id
>>> stop_ids = [dictionary.token2id[stopword] for stopword in stoplist
>>>             if stopword in dictionary.token2id]
>>> once_ids = [tokenid for tokenid, docfreq in dictionary.dfs.iteritems() if docfreq == 1]
>>> dictionary.filter_tokens(stop_ids + once_ids) # 删除停用词和仅出现一次的词
>>> dictionary.compactify() # 消除id序列在删除词后产生的不连续的缺口
>>> print(dictionary)
Dictionary(12 unique tokens)
```

这就是你需要为gensim准备的所有东西，至少从词袋（bag-of-words）模型的角度考虑是这样的。当然，我们用该语料库做什么事另外一个问题，我们并不清楚计算不同单词的词频是否真的有用。事实证明，它确实也没有什么用，我们将需要首先对这种简单的表示方法进行一个转换，才能计算出一些有意义的文档及文档相似性。
转换的内容将会在[下个教程](https://radimrehurek.com/gensim/tut2.html)讲解，在这之前，让我们暂时将注意力集中到语料库持久上来。

## 语料库格式

### 存储语料库

我们有几种文件格式来序列化一个向量空间语料库（~向量序列），并存到硬盘上。Gemsim通过之前提到的语料库流接口（streaming corpus interface）实现了这些方法，用一个惰加载方式来将文档从硬盘中读出（或写入）。一次一个文档（document），不会将整个语料库读入主内存。
所有的语料库格式中，一种非常出名的文件格式就是 [Market Matrix](http://math.nist.gov/MatrixMarket/formats.html)格式。想要将语料库保存为这种格式：

```python
>>> from gensim import corpora
>>> # 创建一个小语料库
>>> corpus = [[(1, 0.5)], []]  # 让一个文档为空，作为它的heck
>>>
>>> corpora.MmCorpus.serialize('/tmp/corpus.mm', corpus)
```

其他格式还有[Joachim’s SVMlight](http://svmlight.joachims.org/)、[Blei’s LDA-C](http://www.cs.princeton.edu/~blei/lda-c/)、[GibbsLDA++](http://gibbslda.sourceforge.net/)等：

```python
>>> corpora.SvmLightCorpus.serialize('/tmp/corpus.svmlight', corpus)
>>> corpora.BleiCorpus.serialize('/tmp/corpus.lda-c', corpus)
>>> corpora.LowCorpus.serialize('/tmp/corpus.low', corpus)
```

### 载入语料库

相反地，从一个Matrix Market文件载入语料库：

```python
>>> corpus = corpora.MmCorpus('/tmp/corpus.mm')
```

语料库对象是流式的，因此你不能直接将其打印出来

```python
>>> print(corpus)
MmCorpus(2 documents, 2 features, 1 non-zero entries)
```

如果你真的想看语料库中的内容：

```python
>>> # 将语料库全部导入内存的方法
>>> print(list(corpus)) # 调用list()将会把所有的序列转换为普通Python List
[[(1, 0.5)], []]
```

或者

```python
>>> # 另一种利用流接口，一次只打印一个文档
>>> for doc in corpus:
...     print(doc)
[(1, 0.5)]
[]
```

第二种方法显然更加内存友好，但是如果只是为了测试与开发，没有什么比调用list(corpus)更简单了。(*^_^*)

### 语料库格式转换

想将这个 Matrix Market格式的语料库存为Blei’s LDA-C格式，你只需：

```python
>>> corpora.BleiCorpus.serialize('/tmp/corpus.lda-c', corpus)
```

这种方式，gensim可以被用作一个内存节约型的**I/O格式转换器**：你只要用一种文件格式流载入语料库，然后直接保存成其他格式就好了。增加一种新的格式简直是太容易了，请参照我们[为SVMlight语料库设计的代码](https://github.com/piskvorky/gensim/blob/develop/gensim/corpora/svmlightcorpus.py)。

## 与NumPy和SciPy的兼容性

Gensim包含了许多[高效的工具函数](http://radimrehurek.com/gensim/matutils.html)来帮你实现语料库与numpy矩阵之间互相转换：

```python
>>> corpus = gensim.matutils.Dense2Corpus(numpy_matrix)
>>> numpy_matrix = gensim.matutils.corpus2dense(corpus, num_terms=number_of_corpus_features)
```

以及语料库与scipy稀疏矩阵之间的转换：

```python
>>> corpus = gensim.matutils.Sparse2Corpus(scipy_sparse_matrix)
>>> scipy_csc_matrix = gensim.matutils.corpus2csc(corpus)
```

想要更全面的参考（例如，压缩词典的大小、优化语料库与NumPy/SciPy数组的转换），参见[API文档](http://radimrehurek.com/gensim/apiref.html)，或者继续阅读下一篇[《主题与转换》](https://radimrehurek.com/gensim/tut2.html)教程。

[1]	该语料库与 [Deerwester et al. (1990): Indexing by Latent Semantic Analysis](http://www.cs.bham.ac.uk/~pxt/IDA/lsa_ind.pdf), Table 2 中使用的相同

[2]	http://blog.csdn.net/questionfish/article/details/46739207