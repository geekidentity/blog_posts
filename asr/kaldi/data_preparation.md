---
categories: kaldi

tags: 
  - AI
  - 语音识别
  - Kaldi

title: Kaldi官方文档（中文版） - 数据准备
date: 2017-08-09
---

# 介绍

运行完示例脚本（参见[Kaldi教程](http://blog.geekidentity.com/kaldi/kaldi_tutorial/)）后，你可能需要设置Kaldi来运行自己的数据。 本节将介绍如何准备数据。 我们假设你的示例脚本是最新版本（在示例目录中为“s5”目录的示例，例如egs/rm/s5/）。除了阅读本页内容，你还可以参考这些目录中的数据准备脚本。顶层的run.sh脚本（例如egs/rm/s5/run.sh ）最前面的几个命令与数据准备的各个阶段有关。子目录中下local/ 的部分都是与数据集相关的。 例如，在资源管理（Resource Management, RM）中设置脚本是local/rm_data_prep.sh。 如果使用RM，如果使用RM命令如下：

```
local/rm_data_prep.sh /export/corpora5/LDC/LDC93S3A/rm_comp || exit 1;

utils/prepare_lang.sh data/local/dict '!SIL' data/local/lang data/lang || exit 1;

local/rm_prepare_grammar.sh || exit 1;
```

而对于WSJ，命令是：

```
wsj0=/export/corpora5/LDC/LDC93S6B
wsj1=/export/corpora5/LDC/LDC94S13B

local/wsj_data_prep.sh $wsj0/??-{?,??}.? $wsj1/??-{?,??}.?  || exit 1;

local/wsj_prepare_dict.sh || exit 1;

utils/prepare_lang.sh data/local/dict "<SPOKEN_NOISE>" data/local/lang_tmp data/lang || exit 1;

local/wsj_format_data.sh || exit 1;
```
在WSJ脚本中有更多的命令与训练语言模型（而不是使用LDC提供的模型）相关，但上面的命令是最重要的，不然数据都没准备好，训练个毛。

数据准备阶段的输出由两部分组成。 一部分与“数据”相关（如目录：data/train/），一部分与“语言”相关（如目录：data/lang/）。 “数据”部分与你的录音文件有关，“语音”部分包含与语言本身相关相关，例如字典，音素集合以及Kaldi需要的有关音素的各种额外信息。 如果要准备要使用现有系统和现有语言模型进行语音识别，则“数据”部分就是您需要接触的所有部分。

# 数据准备 - “数据”部分

作为数据准备的“数据”部分的示例，请查看其中一个示例的“data/train”目录（假设您已经运行一次脚本）。 注意：目录名称“data/train”没有什么特别之处。其他目录，如“data/eval2000”（测试集），它们基本上也具有类似的文件目录格式（“基本上”是因为在测试目录中有会有一些“sgm”和“glm”文件，它们用于sclite评分）。我们看一个具体的例子Switchboard，查看egs/swbd/s5 中的内容。

```
s5# ls data/train
cmvn.scp  feats.scp  reco2file_and_channel  segments  spk2utt  text  utt2spk  wav.scp
```

并不是所有的文件同样重要。
对一个没有“segmentation”信息的简单设置（例如一个文件里只有一段发音），您只需创建你自己的“utt2spk”，“text”和“wav.scp”以及有可能需要的“segments” 和“reco2file_and_channel”，其余的将由标准脚本创建。

我们将从您需要创建的文件开始描该目录中所有的文件，。

## 您需要自己创建的文件

文件“text”包含每段发音的标注。

```
s5# head -3 data/train/text
sw02001-A_000098-001156 HI UM YEAH I'D LIKE TO TALK ABOUT HOW YOU DRESS FOR WORK AND
sw02001-A_001980-002131 UM-HUM
sw02001-A_002736-002893 AND IS
```

每行上的第一个元素是发音ID（utterance-id），它是一个任意的文本字符串，但是如果你的设置中有说话人的信息，则应该将说话人的ID设置为utterance-id的前缀; 这对于这些文件的排序很重要。 utterance-id后面的是每个句话的标注。 您不必确保此文件中的所有单词都在您的词汇表中; 超出的词汇单词将被映射到data/lang/oov.txt 中。

需要说明的是，当您对utt2spk和spk2utt文件进行排序时顺序的一致性。例如：从utt2spk文件中提取的speaker-ids列表与其字符串排序顺序相同。要做到这一点，最简单的方法就是将说话人的id作为“完全”的前缀，不过在这个例子中，我们使用了一个下划线来分隔“说话人”和“发音”部分，一般来说，使用“-”是更安全的。这是因为它的ASCII值更小; 如果speaker-id的长度有所不同，在某些情况下，当使用标准的“C”字符串排序时，speaker-ids 及其对应的发音ID最终可能会以不同的顺序排序，这将导致崩溃。 另一个重要的文件是wav.scp。 在例子Switchboard中，

```
s5# head -3 data/train/wav.scp
sw02001-A /home/dpovey/kaldi-trunk/tools/sph2pipe_v2.5/sph2pipe -f wav -p -c 1 /export/corpora3/LDC/LDC97S62/swb1/sw02001.sph |
sw02001-B /home/dpovey/kaldi-trunk/tools/sph2pipe_v2.5/sph2pipe -f wav -p -c 2 /export/corpora3/LDC/LDC97S62/swb1/sw02001.sph |
```
该文件的格式为：

```
<recording-id> <extended-filename>
```
extended-filename 应该是真实的文件名，或在本示例中是提取wav格式文件的命令。扩展文件名后的管道符表明这个命令被解释为一个管道。过会将会解释“recording-id”，但首先指出"segments"文件并不存在，在wav.scp文件每一行的第一项是正好的utterance id。wav.scp文件必须是单声道的(mono)；如果wav文件有多个声道，则必须在wav.scp中使用sox命令提取其中一个声道。

在Switchboard 设置中我们有segments文件，所有我们下面将讨论segments文件。

```
s5# head -3 data/train/segments
sw02001-A_000098-001156 sw02001-A 0.98 11.56
sw02001-A_001980-002131 sw02001-A 19.8 21.31
sw02001-A_002736-002893 sw02001-A 27.36 28.93
```

"segments"文件的格式为：

```
<utterance-id> <recording-id> <segment-begin> <segment-end>
```
segment-begin 和 segment-end单位都是秒。指明了一段发音在一段录音中的时间偏移量。“recording-id” 和在“wav.scp”中使用的是同一个标识字符串。再次声明文件"reco2file_and_channel"只是在你用NIST的sclite工具对结果进行评分（计算错误率）的时候使用：，这只是一个任意的标识字符串，你可以随便指定。文件"reco2file_and_channel"只是在你用NIST的sclite工具对结果进行评分（计算错误率）的时候使用：

```
s5# head -3 data/train/reco2file_and_channel
sw02001-A sw02001 A
sw02001-B sw02001 B
sw02005-A sw02005 A
```
格式为：

```
<recording-id> <filename> <recording-side (A or B)>
```
filename通常是 .sph文件的名称，没有后缀，但通常也可以是任何其他在“stm”文件中的标识符。recording-side是电话通话中两个通道的概念，如果不是，最好使用"A"，如果你不知道“stm”文件，或你对“stm”文件一无所知，那么你可能不需要"reco2file_and_channel"文件。

最后一个你需要自己创建的文件是“utt2spk”，该文件指明了一段发音是哪个说话人说的。

```
s5# head -3 data/train/utt2spk
sw02001-A_000098-001156 2001-A
sw02001-A_001980-002131 2001-A
sw02001-A_002736-002893 2001-A
```
格式为：

```
<utterance-id> <speaker-id>
```
注意：说话人ID不需要和说话人真实的姓名一致 -- 只要大致能猜出来就行。在这个示例中，我们假设每一个通话侧（电话每一侧）对应一个说话人。这可能并不完全正确 -- 有时候一个人将电话给另一个人通话，但有时候一个人可能在不同的对话中出现。但这对我们的目标已经足够了。如果你完全没有关于说话人的信息，你可以把发音编号当做说话人编号。那么对应的文件格式就变为<utterance-id> <utterance-id>。

在一些设置中会有一些其他文件，这些文件只是偶尔使用，且并不在Kaldi的系统构建中。在RM设置中文件看起来像这样：

```
s5# head -3 ../../rm/s5/data/train/spk2gender
adg0 f
ahh0 m
ajp0 m
```
该文件根据说话人的性别将说话人ID映射到“m”或“f”。

所有的文件应该被排序。如果没有排序，运行脚本时会发生错误。
在[The Table concept](http://www.kaldi-asr.org/doc/io.html#io_sec_tables)中我们解释了为什么需要进行排序。它与I/O框架有关；排序的最终原因是因为排序后的文件可以在一些不支持 fseek()的流中，例如，含有管道的命令，提供类似于随机存取查找的功能。许多Kalid程序会从其他Kaldi命令读取多个管道，读取不同类型的对象，然后对不同输入做一些类似于“合并然后排序”的处理。既然要合并排序， 当然需要输入是经过排序的。小心确保你的shell环境变量LC_ALL定义为“C”。例如，在bash中，你需要这样做：

```
export LC_ALL=C
```
如果你不这样做，这些文件的排序方式会与C++排序字符串的方式不一样，Kaldi就会崩溃。这一点我已经再三强调过了！

如果你的数据中包含NIST提供的测试集，其中有“stm”和“glm”文件可以用作计算WER，那么你可以直接把这些文件拷贝到数据目录下，并分别命名为“stm”和“glm”。注意，我们把评分脚本score.sh（可以计算WER）放到local/下， 这意味着该脚本是与数据集相关的。不是所有的示例设置下的评分脚本都能识别stm和glm文件。能够使用这些文件的一个例子在Switchboard设置里，即egs/swbd/s5/local/score_sclite.sh。如果检测到你有stm和glm文件该脚本会被顶层的评分脚本egs/swbd/s5/local/score.sh调用。

## 不需要手动创建的文件

数据目录下的其他文件可以由前述你提供的文件所生成。你可以用如下的一条命令创建“spk2utt”文件（ 这是一条从egs/rm/s5/local/rm_data_prep.sh中摘取的命令）：

```
utils/utt2spk_to_spk2utt.pl data/train/utt2spk > data/train/spk2utt
```
这是因为utt2spk和spk2utt文件中包含的信息是一样的。spk2utt文件的格式是：<speaker-id> <utterance-id1> <utterance-id2> ...

下面我们讲一讲 feats.scp文件：

```
s5# head -3 data/train/feats.scp
sw02001-A_000098-001156 /home/dpovey/kaldi-trunk/egs/swbd/s5/mfcc/raw_mfcc_train.1.ark:24
sw02001-A_001980-002131 /home/dpovey/kaldi-trunk/egs/swbd/s5/mfcc/raw_mfcc_train.1.ark:54975
sw02001-A_002736-002893 /home/dpovey/kaldi-trunk/egs/swbd/s5/mfcc/raw_mfcc_train.1.ark:62762
```
这个文件指向已经提取好的特征——在这个例子中我们所使用的是MFCC。feats.scp文件的格式是：

```
<utterance-id> <extended-filename-of-features>
```

每一个特征文件保存的都是Kaldi格式的矩阵。在这个例子中，矩阵的维度是13（译者注：即列数；行数则和你的文件长度有关，标准情况下帧长20ms，帧移10ms，所以一行特征数据对应10ms的音频数据。但在Kaldi中，实际返回的帧数可能比你预想的要小一点。例如，你的音频是5.68s，返回的通常是565帧而不是568。这和Kaldi中处理不足以分帧的剩余数据的方式有关，目前这种方式是兼容HTK的。你需要一直向下传递window size和frame shift的值，才能准确算出每一帧的timing。这种方式在实际使用中有不少问题。）。第一行的“extended filename”， 
即/home/dpovey/kaldi-trunk/egs/swbd/s5/mfcc/raw_mfcc_train.1.ark:24，意思是，打开存档（archive）文件
/home/dpovey/kaldi-trunk/egs/swbd/s5/mfcc/raw_mfcc_train.1.ark， fseek()定位到24（字节），然后开始读数据。

feats.scp文件由如下命令创建：

```
steps/make_mfcc.sh --nj 20 --cmd "$train_cmd" data/train exp/make_mfcc/train $mfccdir 
```

该句被顶层的“run.sh”脚本调用。命令中一些shell变量的定义，请查阅对应run.sh。
$mfccdir 是.ark文件将被写入的目录，由用户自定义。

data/train下最后一个未讲到的文件是“cmvn.scp”。该文件包含了倒谱均值和方差归一化的统计量，以说话人编号为索引。每个统计量集合都是一个矩阵，在本例中是2乘以14维。在我们的例子中，有：

```
s5# head -3 data/train/cmvn.scp
2001-A /home/dpovey/kaldi-trunk/egs/swbd/s5/mfcc/cmvn_train.ark:7
2001-B /home/dpovey/kaldi-trunk/egs/swbd/s5/mfcc/cmvn_train.ark:253
2005-A /home/dpovey/kaldi-trunk/egs/swbd/s5/mfcc/cmvn_train.ark:499
```

与feats.scp不同，这个scp文件是以说话人编号为索引，而不是发音编号。该文件由如下的命令创建：

```
steps/compute_cmvn_stats.sh data/train exp/make_mfcc/train $mfccdir
```
(这个例句来自egs/swbd/s5/run.sh).

因为数据准备阶段的错误会影响后续脚本的运行，所以我们有一个脚本来判断数据目录的文件格式是否正确。运行如下的例子：

```
utils/validate_data_dir.sh data/train
```
你可能会发现下面这个命令也很有用：

```
utils/fix_data_dir.sh data/train
```

（当然可对任何数据目录使用该命令，而不只是data/train）。该脚本会修复排序错误，并会移除那些被指明需要特征数据或标注，但是却找不到被需要的数据的那些发音（utterances）。

# 数据准备-- “lang”目录

现在我们关注一下数据准备的“lang”这个目录。

```
s5# ls data/lang
L.fst  L_disambig.fst  oov.int	oov.txt  phones  phones.txt  topo  words.txt
```

除data/lang，可能还有其他目录拥有相似的文件格式：例如有个目录被命名为“data/lang_test”，其中包含和data/lang完全一样的信息，但是要多一个G.fst文件。该文件是一个FST形式的语言模型：

```
s5# ls data/lang_test
G.fst  L.fst  L_disambig.fst  oov.int  oov.txt	phones	phones.txt  topo  words.txt
```

注意，lang_test/ 由拷贝lang/目录而来，并加入了G.fst。每个这样的目录都似乎只包含为数不多的几个文件。但事实上不止如此，因为其中phones是一个目录而不是文件：

```
s5# ls data/lang/phones
context_indep.csl  disambig.txt         nonsilence.txt        roots.txt    silence.txt
context_indep.int  extra_questions.int  optional_silence.csl  sets.int     word_boundary.int
context_indep.txt  extra_questions.txt  optional_silence.int  sets.txt     word_boundary.txt
disambig.csl       nonsilence.csl       optional_silence.txt  silence.csl
```

phones目录下有许多关于音素集的信息。同一类信息可能有三种不同的格式，分别以.csl、.int和.txt结尾。幸运的是，作为一个Kaldi用户，你没有必要去一一手动创建所有这些文件，因为我们有一个脚本“utils/prepare_lang.sh”能够根据更简单的输入为你创建所有这些文件。在讲述该脚本和所谓更简单的输入之前，有必要先解释一下“lang”目录下到底有些什么内容。之后我们将解释如何轻松创建该目录。如果用户不需要理解Kaldi是如何工作的，而是秉着快速建立识别系统的目的，那么可以跳过下面的Creating the "lang" directory 这一节 

# “lang”目录下的内容

首先是有文件hones.txt和words.txt。这些都是符号表（symbol-table）文件，符合OpenFst的格式定义。其中每一行首先是一个文本项，接着是一个数字项：

```
s5# head -3 data/lang/phones.txt
<eps> 0
SIL 1
SIL_B 2
s5# head -3 data/lang/words.txt
<eps> 0
!SIL 1
-'S 2
```

在Kaldi中，这些文件被用于在这些音素符号的文本形式和数字形式之间进行转换。
大多数情况下，只有脚本utils/int2sym.pl、utils/sym2int.pl和OpenFst中的程序
fstcompile和fstprint会读取这些文件。

文件L.fst 是FST形式的发音字典（L，见["Speech Recognition with Weighted Finite-State Transducers"](http://www.cs.nyu.edu/~mohri/pub/hbka.pdf) by Mohri, Pereira and Riley, in Springer Handbook on SpeechProcessing and Speech Communication, 2008），
其中，输入是音素，输出是词。文件 L_disambig.fst也是发音字典，但是还包含了为消歧而引入的符号，诸如#1、#2 之类，以及为自环（self-loop）而引入的 #0 。#0 能让消岐符号“通过”（pass through）整个语法（译者注：n元语法，即我们的语言模型。另外前面这句话我实在不知道该怎么翻译）。更多解释见Disambiguation symbols 。但是不管明白与否，你其实不用自己手动去引入这些符号。

文件 data/lang/oov.txt 仅仅只有一行：

```
s5# cat data/lang/oov.txt
<UNK>
```

在训练过程中，所有词汇表以外的词都会被映射为这个词（译者注：UNK即unknown）。“<UNK>”本身并没有特殊的地方，也不一定非要用这个词。重要的是需要保证这个词的发音只包含一个被指定为“垃圾音素”（garbage phone）的音素。该音素会与各种口语噪声对齐。在我们的这个特别设置中，该音素被称为<SPN>，就是“spoken noise”的缩写：

```
s5# grep -w UNK data/local/dict/lexicon.txt
<UNK> SPN
```
文件oov.int 包含<UNK>的整数形式（从words.txt中提取的），在本设置中是221。你可能已经注意到了，在Resource Management设置中，oov.txt里有一个静音词，在RM设置中被称为“!SIL”。在这种情况下，我们从词汇表中任意选一个词（放入oov.txt）——因为训练集中没有oov词，所以选哪个都不起作用。

文件data/lang/topo则含有如下数据：

```
s5# cat data/lang/topo
<Topology>
<TopologyEntry>
<ForPhones>
21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49 50 51 52 53 54 55 56 57 58 59 60 61 62 63 64 65 66 67 68 69 70 71 72 73 74 75 76 77 78 79 80 81 82 83 84 85 86 87 88 89 90 91 92 93 94 95 96 97 98 99 100 101 102 103 104 105 106 107 108 109 110 111 112 113 114 115 116 117 118 119 120 121 122 123 124 125 126 127 128 129 130 131 132 133 134 135 136 137 138 139 140 141 142 143 144 145 146 147 148 149 150 151 152 153 154 155 156 157 158 159 160 161 162 163 164 165 166 167 168 169 170 171 172 173 174 175 176 177 178 179 180 181 182 183 184 185 186 187 188
</ForPhones>
<State> 0 <PdfClass> 0 <Transition> 0 0.75 <Transition> 1 0.25 </State>
<State> 1 <PdfClass> 1 <Transition> 1 0.75 <Transition> 2 0.25 </State>
<State> 2 <PdfClass> 2 <Transition> 2 0.75 <Transition> 3 0.25 </State>
<State> 3 </State>
</TopologyEntry>
<TopologyEntry>
<ForPhones>
1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20
</ForPhones>
<State> 0 <PdfClass> 0 <Transition> 0 0.25 <Transition> 1 0.25 <Transition> 2 0.25 <Transition> 3 0.25 </State>
<State> 1 <PdfClass> 1 <Transition> 1 0.25 <Transition> 2 0.25 <Transition> 3 0.25 <Transition> 4 0.25 </State>
<State> 2 <PdfClass> 2 <Transition> 1 0.25 <Transition> 2 0.25 <Transition> 3 0.25 <Transition> 4 0.25 </State>
<State> 3 <PdfClass> 3 <Transition> 1 0.25 <Transition> 2 0.25 <Transition> 3 0.25 <Transition> 4 0.25 </State>
<State> 4 <PdfClass> 4 <Transition> 4 0.75 <Transition> 5 0.25 </State>
<State> 5 </State>
</TopologyEntry>
</Topology>
```

这个文件指明了我们所用HMM模型的拓扑结构。在这个例子中，一个“真正”的音素内含3个发射状态，呈标准的三状态从左到右拓扑结构——即“Bakis”模型。（发射状态即能“发射”特征矢量的状态，与之对应的就是那些“假”的仅用于连接其他状态的非发射状态）。音素1到20是各种静音和噪声。之所以会有这么多，是因为对词中的不同位置的同一音素进行了区分（word-position-dependency）。这种情况下，实际上这里的静音和噪声音素大多数都用不上。不考虑在词位话，应该只有5个代表静音和噪声的音素。所谓静音音素（silence phones）有更复杂的拓扑结构。每个静音音素都有一个起始发射状态和一个结束发射状态，中间还有另外三个发射状态。你不用手动创建data/lang/topo。

data/lang/phones/下有一系列的文件，指明了音素集合的各种信息。这些文件大多数有三个不同版本：一个“.txt”形式，如：

```
s5# head -3 data/lang/phones/context_indep.txt
SIL
SIL_B
SIL_E
```
一个“.int”形式，如:

```
s5# head -3 data/lang/phones/context_indep.int
1
2
3
```
以及一个“.csl”形式，内含一个冒号分割的列表：

```
s5# cat data/lang/phones/context_indep.csl
1:2:3:4:5:6:7:8:9:10:11:12:13:14:15:16:17:18:19:20
```

三种形式的文件包含的是相同的信息，所以我们只关注人们更易阅读的“.txt”形式。文件"context_indep.txt"包含一个音素列表，用于建立文本无关的模型。也就是说，对这些音素，我们不会建立需要参考左右音素的上下文决策树。实际上，我们建立的是更小的决策树，只参考中心音素和HMM状态。这依赖于“roots.txt”，下面将会介绍到。关于决策树的更深入讨论见 [How decision trees are used in Kaldi](http://www.kaldi-asr.org/doc/tree_externals.html) 。

```
文件 context_indep.txt包含所有音素，包括那些所谓“假”音素：例如静音（SIL），口语噪声（SPN），非口语噪声（NSN）和笑声（LAU）：
```

```
# cat data/lang/phones/context_indep.txt
SIL
SIL_B
SIL_E
SIL_I
SIL_S
SPN
SPN_B
SPN_E
SPN_I
SPN_S
NSN
NSN_B
NSN_E
NSN_I
NSN_S
LAU
LAU_B
LAU_E
LAU_I
LAU_S
```

因为考虑了词位，这些音素都有许多变体。不是所有的变体都会被实际使用。这里， SIL代表静音词，会被插入到发音字典中（是一个词而不是一个词的一部分，可选的）；SIL_B则代表一个静音音素，应该出现在一个词的开端（这种情况应该永不出现）；SIL_I代表词内静音（也很少存在）；SIL_E代表词末静音（不应该存在）；而SIL_S则表示一种被视为“单独词”（singleton word）的静音，意指这个音素只对应一个词——当你的发音字典中有“静音词”且标注中有明确的静音段时会有用。

silence.txt和onsilence.txt分别包含静音音素列表和非静音音素列表。这两个集合是互斥的，且如果合并在一起，应该是音素的总集。在本例中，silence.txt与context_indep.txt的内容完全一致。我们说“非静音”音素，是指我们将要估计各种线性变换的音素。所谓线性变换是指全局变换，如LDA和MLLT，以及说话人自适应变换，如fMLLR。 根据之前的实验，我们相信，加入静音对这些变换没有影响。我们的经验是，把噪声和发声噪声都列为“静音”音素，而其他传统的音素则是“非静音”音素。在Kaldi中我们还没有通过实验找到一个最佳的方法来这样做。

```
s5# head -3 data/lang/phones/silence.txt
SIL
SIL_B
SIL_E
s5# head -3 data/lang/phones/nonsilence.txt
IY_B
IY_E
IY_I
```
disambig.txt包含一个“消岐符号”列表 (见 [Disambiguation symbols](http://www.kaldi-asr.org/doc/graph.html#graph_disambig)):

```
s5# head -3 data/lang/phones/disambig.txt
#0
#1
#2
```
这些符号会出现在phones.txt 中，被当做音素使用。

optional_silence.txt只含有一个音素。该音素可在需要的时候出现在词之间：

```
s5# cat data/lang/phones/optional_silence.txt
SIL
```
（可选静音列表中的）音素出现在词之间的机制是，在发音字典的FST中，可选地让该音素 出现在每个词的词尾（以及每段发音的段首）。该音素必须在phones/中指明而不是仅仅出现在L.fst中。这个原因比较复杂，这里就不讲了。

文件sets.txt包含一系列的音素集，在聚类音素时被分组（被当做同一个音素），以便建立文本相关问题集（在Kaldi中，建立决策树时使用自动生成的问题集，而不是具有语言语义的问题集）。本设置中，sets.txt将每个音素的所有不同词位的变体组合为一行：

```
s5# head -3 data/lang/phones/sets.txt
SIL SIL_B SIL_E SIL_I SIL_S
SPN SPN_B SPN_E SPN_I SPN_S
NSN NSN_B NSN_E NSN_I NSN_S
```
文件extra_questions.txt包含那些自动产生的问题集之外的一些问题：

```
s5# cat data/lang/phones/extra_questions.txt
IY_B B_B D_B F_B G_B K_B SH_B L_B M_B N_B OW_B AA_B TH_B P_B OY_B R_B UH_B AE_B S_B T_B AH_B V_B W_B Y_B Z_B CH_B AO_B DH_B UW_B ZH_B EH_B AW_B AX_B EL_B AY_B EN_B HH_B ER_B IH_B JH_B EY_B NG_B
IY_E B_E D_E F_E G_E K_E SH_E L_E M_E N_E OW_E AA_E TH_E P_E OY_E R_E UH_E AE_E S_E T_E AH_E V_E W_E Y_E Z_E CH_E AO_E DH_E UW_E ZH_E EH_E AW_E AX_E EL_E AY_E EN_E HH_E ER_E IH_E JH_E EY_E NG_E
IY_I B_I D_I F_I G_I K_I SH_I L_I M_I N_I OW_I AA_I TH_I P_I OY_I R_I UH_I AE_I S_I T_I AH_I V_I W_I Y_I Z_I CH_I AO_I DH_I UW_I ZH_I EH_I AW_I AX_I EL_I AY_I EN_I HH_I ER_I IH_I JH_I EY_I NG_I
IY_S B_S D_S F_S G_S K_S SH_S L_S M_S N_S OW_S AA_S TH_S P_S OY_S R_S UH_S AE_S S_S T_S AH_S V_S W_S Y_S Z_S CH_S AO_S DH_S UW_S ZH_S EH_S AW_S AX_S EL_S AY_S EN_S HH_S ER_S IH_S JH_S EY_S NG_S
SIL SPN NSN LAU
SIL_B SPN_B NSN_B LAU_B
SIL_E SPN_E NSN_E LAU_E
SIL_I SPN_I NSN_I LAU_I
SIL_S SPN_S NSN_S LAU_S
```

你可以看到，所谓一个问题就是一组音素。前4个问题是关于普通音素的词位信息，后面五个则是关于“静音音素”的。 “静音”音素也可能没有像_B 这样的后缀，比如SIL。这些可被作为发音字典中可选的静音词的表示，即不会出现在某个词中，而是单独成词。在具有语调和语气的设置中，extra_questions.txt 可以包含与之相关的问题集。

word_boundary.txt解释了这些音素与词位的关联情况：

```
s5# head  data/lang/phones/word_boundary.txt
SIL nonword
SIL_B begin
SIL_E end
SIL_I internal
SIL_S singleton
SPN nonword
SPN_B begin
```

这和音素中的后缀（_B等等）是相同的信息，但我们并不想给音素的文本形式附加这样的强制限制——记住一件事，Kaldi的可执行程序从不使用音素的文本形式，而是整数形式。（译者注：即Kaldi内部使用的都是音素的整数标号来表示和传递音素）。所以我们使用文件word_boundary.txt来指明各音素与词位间的对应关系。 建立这种对应关系的原因是因为我们需要这些信息从音素网格中恢复词的边界（例如，lattice-align-words需要读取word_boundary.txt的整数版本word_boundary.int）。找出词的边界是有用的，其中之一是用作NIST的sclite评分，该工具需要词的时间标记。还有其他的后续处理需要这些信息。

roots.txt文件包含如何建立音素上下文决策树的信息：

```
head data/lang/phones/roots.txt
shared split SIL SIL_B SIL_E SIL_I SIL_S
shared split SPN SPN_B SPN_E SPN_I SPN_S
shared split NSN NSN_B NSN_E NSN_I NSN_S
shared split LAU LAU_B LAU_E LAU_I LAU_S
...
shared split B_B B_E B_I B_S
```

暂时你可以忽略”shared“和”split“——这些与我们建立决策树时的具体选项有关(更多信息见[How decision trees are used in Kaldi](http://www.kaldi-asr.org/doc/tree_externals.html) ). 像SIL SIL_B SIL_E SIL_I SIL_S这样，几个音素出现在同一行的意义是，在决策树中它们都有同一个“共享根”（shared root），因此状态可在这些音素间共享。对于带语气语调的系统，通常所有与语气和语调相关的音素变体都会出现在同一行。此外，一个HMM中的3个状态（对静音来说有5个状态）共享一个根，且决策树的建立过程需要知道状态（的共享情况）。 HMM状态间共享决策树根节点，这就是roots文件中“shared”代表的意思。

# 建立"lang"目录

data/lang/目录下有很多不同的文件，所以我们提供了一个脚本为你创建这个目录，你只需要提供一些相对简单的输入信息：

```
utils/prepare_lang.sh data/local/dict "<UNK>" data/local/lang data/lang
```
这里，输入目录是data/local/dict/，<UNK>需要在字典中，是标注中所有OOV词的映射词（映射情况会写入data/lang/oov.txt中）。data/local/lang/只是脚本使用的一个临时目录，data/lang/才是输出文件将会写入的地方。

作为数据准备者，你需要做的事就是创建data/local/dict/这个目录。该目录包含以下信息：

```
s5# ls data/local/dict
extra_questions.txt  lexicon.txt nonsilence_phones.txt  optional_silence.txt  silence_phones.txt
```
（实际上还有一些文件我们没有列出来，但那都是在创建目录时所遗留下的临时文件，可以忽略）。下面的这些命令可以让你知道这些文件中大概都有些什么：

```
s5# head -3 data/local/dict/nonsilence_phones.txt
IY
B
D
s5# cat data/local/dict/silence_phones.txt
SIL
SPN
NSN
LAU
s5# cat data/local/dict/extra_questions.txt
s5# head -5 data/local/dict/lexicon.txt
!SIL SIL
-'S S
-'S Z
-'T K UH D EN T
-1K W AH N K EY
```

正如你看到的，本设置（Switchboard）中，这个目录下的内容都非常简单。我们只是分别列出了“真正”的音素和“静音”音素，一个叫extra_questions.txt的空文件，以及一个有如下格式的lexicon.txt：

```
<word> <phone1> <phone2> ...
```
注意：lexicon.txt中，如果一个词有不同发音，则会在不同行中出现多次。如果你想使用发音概率，你需要建立exiconp.txt而不是 lexicon.txt。lexiconp.txt中第二域就是概率值。

注意，一个通常的作法是，对发音概率进行归一化，使最大的那个概率值为1，而不是使同一个词的所有发音概率加起来等于1。 这样可能会得到更好的结果。 如果想在顶层脚本中找一个与发音概率相关的脚本，请在egs/wsj/s5/run.sh目录下搜索pp。

需要注意的是，在这些输入中，没有词位信息，即没有像_B和_E<这样的后缀。
这是因为脚本prepare_lang.sh会添加这些后缀。

从空的extra_questions.txt件中你会发现，可能还有些潜在的功能我们没有利用。

这其中就包括重音和语调标记。 对具有不同重音和语调的同一音素，你可能会想用不同的标记去表示。为展示如何这样做，我们看看在另外一个设置egs/wsj/s5/的这些文件。结果如下：

```
s5# cat data/local/dict/silence_phones.txt
SIL
SPN
NSN
s5# head data/local/dict/nonsilence_phones.txt
S
UW UW0 UW1 UW2
T
N
K
Y
Z
AO AO0 AO1 AO2
AY AY0 AY1 AY2
SH
s5# head -6 data/local/dict/lexicon.txt
!SIL SIL
<SPOKEN_NOISE> SPN
<UNK> SPN
<NOISE> NSN
!EXCLAMATION-POINT  EH2 K S K L AH0 M EY1 SH AH0 N P OY2 N T
"CLOSE-QUOTE  K L OW1 Z K W OW1 T
s5# cat data/local/dict/extra_questions.txt
SIL SPN NSN
S UW T N K Y Z AO AY SH W NG EY B CH OY JH D ZH G UH F V ER AA IH M DH L AH P OW AW HH AE R TH IY EH
UW1 AO1 AY1 EY1 OY1 UH1 ER1 AA1 IH1 AH1 OW1 AW1 AE1 IY1 EH1
UW0 AO0 AY0 EY0 OY0 UH0 ER0 AA0 IH0 AH0 OW0 AW0 AE0 IY0 EH0
UW2 AO2 AY2 EY2 OY2 UH2 ER2 AA2 IH2 AH2 OW2 AW2 AE2 IY2 EH2
s5#
```

你可能已经注意到了，nonsilence_phones.txt中的某些行，一行中
有多个音素。这些是同一元音的与重音相关的不同表示。 注意，在CMU版的字典中，
每个音素有4种表示：例如，UW UW0 UW1 UW2。基于某些原因。其中一种表示没有数字后缀。行中音素的顺序没有关系。通常，我们建议将每个“真实音素”的不同形式都组织在单独的一行中。我们使用CMU字典中的重音标记。文件extra_questions.txt中只有一个问题
包含所有的“静音”音素（实际上这是不必要的，只是脚本prepare_lang.sh 会添加这么一个问题），以及一个涉及不同重音标记的问题。
这些问题对利用重音标记信息来说是必要的，因为在nonsilence_phones.txt中每个音素的不同重音表示都在同一行，这确保了他们在 data/lang/phones/roots.txt 和
data/lang/phones/sets.txt也属同一行，这又反过来确保了它们共享同一个（决策）树
根，并且不会有决策问题弄混它们。因此，我们需要提供一个特别的问题，能为决策树的建立过程提供一种区分音素的方法。 注意：我们在sets.txt和roots.txt中将音素分组放在一起的原因是，这些同一音素的不同重音变体可能缺乏足够的数据去稳健地估计一个单独的决策树，或者是产生问题集时需要的聚类信息。 像这样把它们组合在一起，我们可以确保当数据不足以对它们分别估计决策树时，这些变体能在决策树的建立过程中“聚集在一起”（stay together）。

写到这里我们需要提一点，脚本utils/prepare_lang.sh支持很多选项。下面是该脚本的用法，可让你们了解这些选项都有哪些：

```
usage: utils/prepare_lang.sh <dict-src-dir> <oov-dict-entry> <tmp-dir> <lang-dir>
e.g.: utils/prepare_lang.sh data/local/dict <SPOKEN_NOISE> data/local/lang data/lang
options:
     --num-sil-states <number of states>             # default: 5, #states in silence models.
     --num-nonsil-states <number of states>          # default: 3, #states in non-silence models.
     --position-dependent-phones (true|false)        # default: true; if true, use _B, _E, _S & _I
                                                     # markers on phones to indicate word-internal positions.
     --share-silence-phones (true|false)             # default: false; if true, share pdfs of
                                                     # all non-silence phones.
     --sil-prob <probability of silence>             # default: 0.5 [must have 0 < silprob < 1]
```

一个可能的重要选项是--share-silence-phones。该选项默认是false。 如果该选项被设为true， 所有静音音素——如静音、发声噪声、噪声和笑声——的概率密度函数（PDF，高斯混合模型）都会共享，只有模型中的转移概率不同。现在还不清楚为什么这样做有用，但我们发现这对IARPA的BABEL项目中的广东话数据集非常有效。该数据集非常乱，其中有很长的未标注的部分，我们试着将其与一个特别标记的音素对齐。我们怀疑训练数据可能没能成功正确对齐，而且基于某些不明原因，将上述选项设置为true则改变了结果。

另外一个可能的重要选项是“--sil-prob”。 通常，对这些选项我们所作的实验都不多，所以对具体如何设置也不能给出非常详细的建议。

# 创建语言模型或者语法文件

前面的关于如何创建lang/目录的教程没有涉及如何产生G.fst文件。该文件是语言模型——或者称为语法——的有限状态转换器格式的表示，我们解码时需要它。 实际上，在一些设置中，为做不同的测试，我们可能会有许多“lang”目录。这些目录中有不同的语言模型和字典。以华尔街日报（WSJ）的设置为例：

```
s5# echo data/lang*
data/lang data/lang_test_bd_fg data/lang_test_bd_tg data/lang_test_bd_tgpr data/lang_test_bg \
 data/lang_test_bg_5k data/lang_test_tg data/lang_test_tg_5k data/lang_test_tgpr data/lang_test_tgpr_5k
```
根据我们使用的语言模型的不同——是统计语言模型还是别的种类的语法形式——生成G.fst的步骤会不同。 在RM设置中，使用的是二元语法，只允许某些词对。我们将总概率值1分配给所有向外的弧，以确保每个语法状态的概率和为1。在 local/rm_data_prep.sh中有这样一句代码：

```
local/make_rm_lm.pl $RMROOT/rm1_audio1/rm1/doc/wp_gram.txt  > $tmpdir/G.txt || exit 1;
```
脚本local/make_rm_lm.pl会建立一个FST格式的语法文件（文本格式，不是二进制格式）。
该文件包含如下形式的行：

```
s5# head data/local/tmp/G.txt
0    1    ADD    ADD    5.19849703126583
0    2    AJAX+S    AJAX+S    5.19849703126583
0    3    APALACHICOLA+S    APALACHICOLA+S    5.19849703126583
```

到 www.openfst.org 上查阅更多关于OpenFst的信息（他们有一个很详细的教程）。 脚本local/rm_prepare_grammar.sh会将文本格式的语法文件转换为二进制文件G.fst。所用命令如下：

```
fstcompile --isymbols=data/lang/words.txt --osymbols=data/lang/words.txt --keep_isymbols=false \
    --keep_osymbols=false $tmpdir/G.txt > data/lang/G.fst
```

如果你要建立自己的语法文件，你也应做类似的事。

注意：这种过程只适用于一类语法：用上述方法你不能创建上下文无关的语法，因 为这类语法不能被表示为OpenFst格式。 在WFST框架下还是有办法这么做（见Mike Riley最近关于push down transducers的研究工作），但是在Kaldi中我们还没实现这些功能。

在WSJ设置中，我们使用了一个统计语言模型。脚本 local/wsj_format_data.sh将WSJ数据库提供的ARPA格式的语言模型文件转换为OpenFst格式的。 脚本中关键的命令如下：