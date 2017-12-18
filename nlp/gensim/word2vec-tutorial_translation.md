---
categories: Gensim

tags: 
  - Gensim
  - NLP
  - 翻译
  - word2vec

title: Word2vec教程翻译

date: 2017-12-18
---

翻译自：https://rare-technologies.com/word2vec-tutorial/

我从来没有写过关于如何在gensim中使用word2vec的教程。它的使用非常简单，[API文档](http://radimrehurek.com/gensim/models/word2vec.html)简单明了，但我知道有些人更喜欢更详细的内容。这篇文章是一个教程和一些参考的例子。

# 准备输入

开始时，gensim的word2vec需要一系列句子（sentences）作为输入。 每个句子是一个单词列表（utf-8格式字符串）：

```python
# 导入模块，设置日志
import gensim, logging
logging.basicConfig(format='%(asctime)s : %(levelname)s : %(message)s', level=logging.INFO)
 
sentences = [['first', 'sentence'], ['second', 'sentence']]
# 在这两个句子上训练word2vec
model = gensim.models.Word2Vec(sentences, min_count=1)
```

将输入保存为Python内置list是很方便的，但是当输入很大时会占用大量的内存。

Gensim只要求迭代时输入必须按顺序提供句子即可，没有必要把所有的东西放在RAM中：我们可以提供一个句子，处理它，忘记它，加载另一个句子...

例如，如果我们的输入被分散在磁盘上的多个文件中，每行一个句子，那么就不需要将所有内容都加载到内存list中，我们可以按文件逐行处理输入文件（译者注：Gensim官方文档[语料库流——一次一个文档](http://blog.geekidentity.com/nlp/gensim/gensim_1_corpora-and-vector-spaces_translation#语料库流——一次一个文档)中也提到了类似方式）：

```python
class MySentences(object):
    def __init__(self, dirname):
        self.dirname = dirname
 
    def __iter__(self):
        for fname in os.listdir(self.dirname):
            for line in open(os.path.join(self.dirname, fname)):
                yield line.split()
 
sentences = MySentences('/some/directory') # 一个内存友好的迭代器
model = gensim.models.Word2Vec(sentences)
```

假设我们要进一步预处理文件中的单词 —— 转换为unicode，小写，删除数字，提取命名实体...所有这些都可以在MySentences迭代器中完成，而word2vec不需要知道。 所要做的就是输入一个句子（utf8-单词列表）。

注意：调用Word2Vec(sentences, iter=1)将在句子迭代器（sentences）上运行两遍（或者，通常运行iter + 1次;默认iter = 5）。 第一遍收集单词和它们的频率来建立一个内部的字典树结构。 第二次及以后的过程训练神经模型。 如果你的输入流是不可重复的（你只能遍历一次），那么这两（或者iter + 1）次遍历也可以手动启动，你可以用其他方式初始化词汇表：

```python
model = gensim.models.Word2Vec(iter=1)  # 一个空的模型， 不做任何训练
model.build_vocab(some_sentences)  # 可以是一个不能重复遍历的迭代器，迭代器只能遍历一次
model.train(other_sentences)  # 可以是一个不能重复遍历的迭代器，迭代器只能遍历一次
```

如果您对Python中的iterators, iterables 和generators不理解，请查看我们关于Python中[数据流](https://rare-technologies.com/data-streaming-in-python-generators-iterators-iterables/)的教程。

#训练

Word2vec接受的几个参数会影响训练速度和质量。

其中之一是内部字典剪枝。 在十亿单词的语料库中只出现一次或两次的词可能是没有用的的错字和垃圾。另外，没有足够的数据对这些出现次数很少的词做任何有意义的训练，所以最好忽略它们：

```python
model = Word2Vec(sentences, min_count=10)  # 默认值是 5
```

min_count的合理取值在0-100之间，具体取决于数据集的大小。

另一个参数是NN层的大小，它对应于训练算法的“自由度”

```python
model = Word2Vec(sentences, size=200)  # 默认值是 100
```

更大的size值需要更多的训练数据，但可能训练出更好（更准确）的模型。合理的取值在几十到几百。

最后一组主要参数（全部[参数列表](http://radimrehurek.com/gensim/models/word2vec.html#gensim.models.word2vec.Word2Vec)）用于训练并行化，以加速训练：

```python
model = Word2Vec(sentences, workers=4) # default = 1 worker = no parallelization
```

只有安装了[Cython](http://cython.org/)，workers参数才有效。没有Cython，因为[GIL](https://wiki.python.org/moin/GlobalInterpreterLock)你将只能使用一个核（word2vec训练将会[非常慢](http://radimrehurek.com/2013/09/word2vec-in-python-part-two-optimizing/)）。

# 内存

在其核心，word2vec模型参数被存储为矩阵（NumPy数组）。每个数组是**#vocabulary**（由min_count参数控制）乘以float（单精度，又称为4字节）的**#vocabulary**（size参数）。

RAM中有三个这样的矩阵（进行工作的时候，把这个数目减少到两个，甚至一个）。因此，如果您的输入包含100,000个词，并且您要求层（layer）大小= 200，则该模型将需要约100,000 * 200 * 4 * 3字节=〜229MB。

存储单词树需要一些额外的内存（100,000个单词需要几兆字节），但除非你的单词是非常冗长的字符串，否则内存主要将由上述三个矩阵占用。

# 评估

Word2vec训练是一个无监督的任务，没有什么好的方法来客观评估结果。 评估取决于你最终应用程序。

Google已经发布了大约两万个句法和语义测试示例的测试集，继“A is to B as C is to D”task：<https://raw.githubusercontent.com/RaRe-Technologies/gensim/develop/gensim/test/test_data/questions-words.txt>.

Gensim支持相同的评估集，格式完全相同：

```python
model.accuracy('/tmp/questions-words.txt')
2014-02-01 22:14:28,387 : INFO : family: 88.9% (304/342)
2014-02-01 22:29:24,006 : INFO : gram1-adjective-to-adverb: 32.4% (263/812)
2014-02-01 22:36:26,528 : INFO : gram2-opposite: 50.3% (191/380)
2014-02-01 23:00:52,406 : INFO : gram3-comparative: 91.7% (1222/1332)
2014-02-01 23:13:48,243 : INFO : gram4-superlative: 87.9% (617/702)
2014-02-01 23:29:52,268 : INFO : gram5-present-participle: 79.4% (691/870)
2014-02-01 23:57:04,965 : INFO : gram7-past-tense: 67.1% (995/1482)
2014-02-02 00:15:18,525 : INFO : gram8-plural: 89.6% (889/992)
2014-02-02 00:28:18,140 : INFO : gram9-plural-verbs: 68.7% (482/702)
2014-02-02 00:28:18,140 : INFO : total: 74.3% (5654/7614)
```

这个准确性需要一个[可选参数](http://radimrehurek.com/gensim/models/word2vec.html#gensim.models.word2vec.Word2Vec.accuracy)restrict_vocab，它限制了哪些测试示例需要考虑。

再一次，在这个测试集上的良好性能并不意味着word2vec会在你的应用程序中运行良好，反之亦然。 直接评估你的预期任务总是最好的。

# 存储和加载模型

您可以使用标准的gensim方法来存储/加载模型：

```python
model.save('/tmp/mymodel')
new_model = gensim.models.Word2Vec.load('/tmp/mymodel')
```

内部使用pickle，可选择将模型的内部大型NumPy矩阵直接从磁盘文件映射到虚拟内存中，以进行进程间内存共享。

另外，您可以使用原始C工具创建的文本和二进制格式：

```python
model = Word2Vec.load_word2vec_format('/tmp/vectors.txt', binary=False)
# 使用gzipped/bz2输入也可以，不需要解压：
model = Word2Vec.load_word2vec_format('/tmp/vectors.bin.gz', binary=True)
```

# 在线训练/恢复训练

用户可以加载模型，并继续用更多的句子进行训练：

```python
model = gensim.models.Word2Vec.load('/tmp/mymodel')
model.train(more_sentences)
```

您可能需要调整total_words参数以*train()*，具体取决于您想要模拟的学习速率衰减。

请注意，无法使用C工具load_word2vec_format()生成的模型继续训练。你仍然可以使用它们进行查询/相似性查询，但对于训练（单词树）而言至关重要的信息在此处不存在了。

# 使用模型

Word2vec支持几个词相似的任务，并且开箱即用：

```python
model.most_similar(positive=['woman', 'king'], negative=['man'], topn=1)
[('queen', 0.50882536)]
model.doesnt_match("breakfast cereal dinner lunch";.split())
'cereal'
model.similarity('woman', 'man')
0.73723527
```

如果你在应用程序中需要原始输出向量，则可以逐字（word-by-word）访问这些向量

```python
model['computer']  # raw NumPy vector of a word
array([-0.00449447, -0.00310097,  0.02421786, ...], dtype=float32)
```