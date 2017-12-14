---
categories: Kaldi

tags: 
  - AI
  - 语音识别
  - Kaldi
  - ASR

title: Kaldi官方文档（中文版） - Kaldi教程
date: 2017-08-09
---

# Kaldi教程

## 先决条件

本教程假定您使用HMM-GMM方法并了解语音识别的基础知识。 简要介绍一下：M. Gales and S. Young (2007)。 “隐马尔可夫模型在语音识别中的应用”信号处理的基础与趋势1（3）：195-304。HTK Book也是一个很好的资源，但除非你有很强的数学背景，而且非常专注 我们不鼓励尝试在机构设置之外学习语音识别，本教程的目标受众是语音识别研究人员，毕业生或正在研究该领域的高级本科生。

我们假设你知道C++，并且至少熟悉shell脚本，最好使用bash或类似的shell。 本教程假定您正在使用类似UNIX的环境或Cygwin（尽管Kaldi不一定会在所有这些环境中编译和运行）。

此外，重要的是，本教程假设您可以以原始形式从LDC访问来自语言数据联盟（Linguistic Data Consortium, LDC）的资源管理（RM）CD上的数据。 也就是说，我们假设这些数据位于你的系统上。 我们获得了目录号LDC93S3A。 它也有两个独立的部分。 要注意，因为以前有不同布局的RM数据的不同分布。

系统要求相当基本。 我们假设你有工具，包括wget，git，svn，awk，perl等，或者你知道如何安装它们。 安装过程中最困难的部分涉及数学库ATLAS; 如果这还没有作为系统上的库安装，那么您必须编译它，这就要求关闭CPU限制，这可能需要root权限。 我们提供所有安装步骤的脚本和详细说明。 当脚本失败时，仔细阅读输出，因为它试图提供如何解决问题的指导。 请告知我们，如果任何时候有问题，不管怎么说; 请参阅[其他Kaldi相关资源（以及如何获得帮助）](http://kaldi-asr.org/doc/other.html)。

我们尝试提供一些想法，执行教程的每个步骤需要花费多长时间。 如果完成本教程的时间有限，我们建议您尝试遵守已发布的时间表，如有必要，请跳过步骤，并避免以下连接到我们在文本中提供的更多信息。 这将有助于确保您获得平衡的概述。 您可以随时更详细地查看材料。 如果本课程将在课堂设置中给出，那么有必要事先通过相关系统上的教程来验证是否安装了所有先决条件。

## 开始 (15 minutes)

第一步是下载并安装Kaldi。 我们将使用该工具包的第1版，以便本教程不会过时。 但是，请注意，“trunk”（始终是最新的）中的代码和脚本更易于安装，并且通常更好。 如果使用“trunk”代码，您还可以尝试使用最新的脚本，这些脚本位于目录“egs/rm/s5”中，而不是本教程中提到的“s3”脚本。 但请注意，如果您这样做，教程的某些方面可能已过期。

假设安装了Git，以获取可以键入的最新代码

```
git clone https://github.com/kaldi-asr/kaldi.git kaldi-trunk --origin golden
```

然后cd到kaldi-trunk。 看看INSTALL文件并按照说明（它指向两个子目录）。 仔细看看安装脚本的输出，因为他们试图指导你做什么。 一些安装错误是非致命的，并且安装脚本会告诉你这样（即有一些它安装的东西是很好的但不是真正需要的）。 “最好的情况”是你做的：

```
cd kaldi-trunk/tools/; make; cd ../src; ./configure; make
```
但是，如果没有发生这种情况，则会有备份计划（例如，您可能需要在计算机上安装一些软件包，或者在tools /中运行install_atlas.sh，或手动运行工具/ INSTALL中的一些步骤，或者为配置脚本提供选项 在src/）。 如果有问题，在构建过程中可能会有一些信息（如何编译Kaldi），这将有助于您; 否则，请随时联系维护人员（其他Kaldi相关资源（以及如何获得帮助）），我们将很乐意提供帮助。

## 版本控制与Git（5分钟）

http://kaldi-asr.org/doc/tutorial_git.html

## 发行概况（20分钟）

在我们跳入示例脚本之前，让我们花几分钟时间查看Kaldi发行版中的其他内容。 转到kaldi-1目录并list所有内容。 有几个文件和子目录。 重要的子目录是“tools/”，“src/”和“egs/”，我们将在下一节中看到。 我们将概述“tools/”和“src/”。

### tools/目录（10分钟）

目录“tools/”是我们以各种方式安装Kaldi所依赖的东西的地方，切换到目录tools/ 并list所有内容，您将看到各种文件和子目录，大部分是由make命令安装的内容。 文件INSTALL提供如何安装这些工具的说明。

最重要的子目录是OpenFst的子目录。 cd到openfst/。 这是一个具有版本号的实际目录的软链接。列出openfst目录。如果安装成功，将会有一个带有已安装的二进制文件的bin/ 目录，以及带有库的lib/ 目录（我们需要这两个目录）。 最重要的代码是include/fst/ 目录。 如果您想了解Kaldi，您将需要了解OpenFst。 为此，最好的出发点是 http://www.openfst.org/ 。

现在，只需查看文件include/fst/fst.h。 这包括抽象FST类型的一些声明。 你可以看到有很多模板涉及。 如果不熟悉这些模板，你可能会在理解这个代码时遇到麻烦。

切换到目录bin/，或将其添加到路径中。 我们将从这里执行一些简单的示例指令。

将以下命令粘贴到shell中：

```shell
# arc 格式: src dest ilabel olabel [weight]
# 最后一行的格式: state [weight]
# 每一行可能以任何顺序发生，除了初始状态必须是第一行
# 未指定的权重默认为0.0（对于library-default权重类型）
cat >text.fst <<EOF
0 1 a x .5
0 1 b y 1.5
1 2 c z 2.5
2 3.5
EOF
```

以下命令创建符号表; 将它们粘贴到shell中。

```shell
cat >isyms.txt <<EOF
<eps> 0
a 1
b 2
c 3
EOF

cat >osyms.txt <<EOF
<eps> 0
x 1
y 2
z 3
EOF
```

注意：要使以下步骤正常工作，如果您的PATH上没有当前目录，可能需要输入：

```bash
export PATH=.:$PATH
```
接下来创建一个二进制格式的FST：

```
fstcompile --isymbols=isyms.txt --osymbols=osyms.txt text.fst binary.fst
```

我们来执行一个例子命令：

```
fstinvert binary.fst | fstcompose - binary.fst > binary2.fst
```

所得到的WFST，binary2.fst应该与binary.fst相似，但weights的两倍。 你可以打印出来，看看：

```
fstprint --isymbols=isyms.txt --osymbols=osyms.txt binary.fst
fstprint --isymbols=isyms.txt --osymbols=osyms.txt binary2.fst
```
该示例从 www.openfst.org 上提供的更长的教程进行了修改。 完成此操作后，请输入以下内容：

```
rm *.fst *.txt -f
```

### src/ 目录（10分钟）

将目录更改回到顶层（kaldi）并进入src/。 列出目录。 您将看到一些文件和大量的子目录。 看看Makefile。 在顶部，它设置变量SUBDIRS。 这是包含代码的子目录的列表。 请注意，其中一些以“bin”结尾， 这些是包含可执行文件（代码和可执行文件在同一目录中）的那些。 其他目录包含内部代码。

您可以看到Makefile中的一个targets是“test”。 输入“make test”。 该命令进入各种子目录并在其中运行测试程序。 所有的测试都应该成功。 如果你感觉很幸运，你也可以输入“make valgrind”。 这与内存检查程序运行相同的测试，并且花费更长时间，但会发现更多的错误。 如果这不行，忘掉它; 现在不重要 如果时间过长，请用ctrl-c停止。

将目录更改为base/。 看看Makefile。 注意这一行：

```
include ../kaldi.mk
```

每当调用子目录中的Makefile（就像C #include指令）时，这一行包含文件../kaldi.mk所有内容。看看文件../kaldi.mk。它将包含一些与valgrind相关的规则（用于内存调试），然后会包含一些特定于系统的配置，例如CXXFLAGS等变量。看看是否有-O选项（例如-O0）。默认情况下，标志-O0和-DKALDI_PARANOID被禁用，因为它们会减慢速度（您可能希望使其能够更好地调试）。再看 base/Makefile 。顶部的声明“all:”告诉Make“all”是顶级目标（因为kaldi.mk中有目标，我们不希望这些目标成为顶级目标）。因为“all”的依赖关系取决于之后定义的变量，所以我们有另一个语句（目标在 default_rules.mk 中定义），我们定义“all”取决于什么。寻找它。定义了其他几个目标，从“干净”开始。寻找他们为了使“干净”你可以输入“make clean”。目标.valgrind不是你从命令行调用的东西;您将键入“make valgrind”（目标在kaldi.mk中定义）。调用所有这些目标，即键入“make clean”和其他目标相同，并注意在执行此操作时会发出什么命令。

在base/目录中的Makefile中：选择TESTFILES中列出的二进制文件之一，并运行它。 然后简要查看相应的.cc文件。 kaldi-math-test是一个很好的例子（注意：这不包括Kaldi中的大部分数学函数，它们是矩阵向量相关函数，位于../matrix/中）。 请注意，有很多断言，使用宏KALDI_ASSERT。 这些测试程序设计为出现错误状态（如果有问题，不应该依赖人工检查输出）。

看看头文件kaldi-math.h。您将看到我们的编码实践的一些要素。 请注意，我们所有的本地#includes都是相对于src/ 目录（所以我们#include base/kaldi-types.h 即使我们已经在 base/ 目录中）。 请注意，我们#define定义的所有宏，除了我们正确确定具有正常值的标准值，其余都以KALDI_开头。 这是一个预防措施，以避免与其他代码库的宏定义冲突（因为#defines不限制自己到kaldi命名空间）。 注意函数名称的样式：LikeThis()。 我们的风格一般都是基于这个，符合OpenFst的，但是有一些区别。

查看代码风格的其他元素，这将帮助您了解Kaldi代码，cd到 ../util，并查看text-utils.h。 请注意，这些函数的输入始终是第一个，通常是const引用，而输出（或被修改的输入）始终为最后，并且是指针参数。 不允许使用非常量引用作为函数参数。 如果您有兴趣，可以稍后阅读更多关于编码风格的Kaldi特定元素。 现在，请注意，有一个具有相当具体规则的编码风格。

将目录更改为../gmmbin并键入

```
./gmm-init-model
```

它打印出的用法，这应该给你一个通用的想法，如何调用Kaldi程序。 请注意，虽然有一个可用于传递配置文件的-config选项，但一般来说，Kaldi不像HTK那样配置驱动，而且这些文件没有被广泛使用。 您将看到一个二进制选项。 一般来说，Kaldi文件格式包含二进制和测试形式，而-binary选项控制它们的写入方式。 然而，这仅仅控制单个对象（例如声学模型）的写入方式。 对于整个对象集合（例如，特征文件的集合），我们将会有一个不同的机制。 键入

```
./gmm-init-model >/dev/null
```

你看到了什么，这告诉你Kaldi对日志类型输出有什么影响？ 使用消息所在的地方与所有错误和日志消息相同的位置，并且有一个原因，当您开始查看脚本时，应该会变得很明显。

要了解构建过程的一些内容，请转到../matrix，然后键入

```
rm *.o
make
```

看看传递给编译器的选项。 这些最终由../kaldi.mk中设置的变量控制，而这些变量又由../configure决定。 还要查看链接选项，当它创建matrix-lib-test时传入。 你会得到一些信息，它连接到哪些数学库（这在某种程度上取决于系统）。 有关如何使用外部矩阵库的更多信息，可以阅读[External Matrix库](http://www.kaldi-asr.org/doc/matrixwrap.html)。

将目录更改为一级（src/目录），并查看“configure”文件。如果您熟悉automake生成的“配置”文件，您会注意到它不是其中之一。它是手动生成的。在其中搜索“makefiles/”并快速扫描出现该字符串的所有位置（例如，键入“shell”，然后键入“/makefiles[enter]”，然后键入“n”以查看稍后的实例）。您将看到它在子目录“makefiles/”中使用带有后缀.mk的某些文件。这些基本上是kaldi.mk的“原型”版本。看看其中一个原型，例如makefile/cygwin.mk，看看它们包含的内容。对于更可预测的系统，它只是将系统特定的makefile与makefiles/kaldi.mk.common连接起来，并将其写入kaldi.mk。对于Linux，它必须做更多的事情，因为有这么多的发行版。这大部分涉及到找到数学库的安装位置。如果您在构建过程中遇到问题，则一个解决方案是尝试手动修改kaldi.mk。为了做到这一点，你应该了解Kaldi如何利用外部数学库（参见[外部矩阵库  External matrix libraries](http://www.kaldi-asr.org/doc/matrixwrap.html)）。

## 运行示例脚本（40分钟）

### 入门和先决条件。

本教程的下一阶段是开始运行资源管理（Resource Management）示例脚本。将目录更改为最高级别（我们称之为kaldi-1），然后更改为egs/。看看该目录中的README.txt文件，具体看看资源管理部分。它提到对应于该语料库的LDC目录编号。这可能有助于您获得LDC的数据。如果由于某些原因无法获取数据，只需继续阅读本教程，并执行您可以在没有数据的情况下执行的步骤，并且您仍然可以从中获取一些值。最好的情况是系统上有一些目录，比如 /export/corpora5/LDC/LDC93S3A/rm_comp ，其中包含三个子目录;称它们为rm1_audio1，rm1_audio2和rm2_audio。这些将对应于LDC数据分发中的三个原始磁盘。这些说明假设你的shell是bash。如果你有一个不同的shell，这些命令将不起作用或应该被修改（只需键入“bash”进入bash，一切都应该工作）。

现在将目录更改为 rm/，浏览文件README.txt以查看整体结构，并将cd转换为s5/。 这是与工具包版本5中的主要功能对应的基本实验步骤。

在s5/ 中，列出目录并浏览到RESULTS文件，以便在那里您会有一些想法，（稍后，您应该可以验证您获得的结果与那里的内容相似）。 我们将要查看的主文件是run.sh. 注意：run.sh不是直接从shell运行的; 这个想法是你手动一个一个运行这些命令。

### 数据准备

我们首先需要配置作业是需要在本地运行还是在Oracle GridEngine上运行。有关如何执行此操作的说明在cmd.sh中。

如果您没有安装GridEngine，或者如果您在较小的数据集上运行实验，请在您的shell上执行以下命令。

```
train_cmd="run.pl"
decode_cmd="run.pl"
```

如果您已安装GridEngine，则应使用queue.pl文件，该参数指定GridEngine所在的位置。 在这种情况下，您将执行以下命令（参数-q是一个示例，您将要将其替换为GridEngine详细信息）

```
train_cmd="queue.pl -q all.q@a*.clsp.jhu.edu"
decode_cmd="queue.pl -q all.q@[ah]*.clsp.jhu.edu"
```

下一步是从RM语料库创建测试和训练集。 为此，在shell上运行以下命令（假设您的数据位于/export/corpora5/LDC/LDC93S3A/rm_comp中）：

```
local/rm_data_prep.sh /export/corpora5/LDC/LDC93S3A/rm_comp 
```

如果一切正常，会显示：“RM_data_prep succeeded”。 如果没有，您可以在脚本失败的地方找到问题。

现在列出当前目录的内容，你应该会看到一个名为“data”的新目录被创建。 进入新创建的数据目录并列出内容。 您应该看到三种主要类型的文件夹：

* local：包含当前数据的字典。
* train：从语料库分割的数据用于训练。
* test_*：从语料库分割的数据用于测试目的。

让我们花一点时间看看创建的数据文件。 这应该可以让您了解Kaldi所期望输入的数据。 （另请参考：[详细数据准备指南](http://www.kaldi-asr.org/doc/data_prep.html)）

假设您在data/目录中，请执行以下命令：

```
cd local/dict
head lexicon.txt
head nonsilence_phones.txt
head silence_phones.txt
```

这些将会让您了解通用数据准备过程的输出将如何。 您应该理解的是，并不是所有这些文件都是“native”Kaldi格式，即并不是所有这些文件都可以被Kaldi的C++程序读取，并且需要在Kaldi可以使用之前使用OpenFST工具进行处理。

* lexicon.txt：这是词典。
* \*silence\*.txt：这些文件包含有关哪些电话是无声的，哪些不是。

 现在回到数据目录并将目录更改为/train。 然后执行以下命令查看此目录中文件的输出：

```
head text
head spk2gender.map
head spk2utt
head utt2spk
head wav.scp
```

* text - 此文件包含将被Kaldi使用的话语和话语ID之间的映射。该文件将变成一个整数格式 - 仍然是一个文本文件，但这些单词被替换为整数。
* spk2gender.map - 此文件包含说话者和他们的性别之间的映射。这也是参与训练的独特用户的列表。
* spk2utt - 这是扬声器标识符与与扬声器相关联的所有话语标识符之间的映射。
* utt2spk - 这是一个一对一的话语标识和对应的说话人标识符之间的映射。
* wav.scp - 当进行特征提取时，这个文件实际上是由Kaldi程序直接读取的。它被解析为一组键值对，其中键是每行上的第一个字符串。该值是一种“扩展文件名”，您可以猜测它是如何工作的。由于它是用于读取的，所以我们将这种类型的字符串称为“rxfilename”（用于写我们使用术语wxfilename）。如果您好奇，请参阅[扩展文件名](http://www.kaldi-asr.org/doc/io.html#io_sec_xfilename)：rxfilenames和wxfilenames。请注意，虽然我们使用扩展名.scp，但这不是HTK意义上的脚本文件（即不被视为命令行参数的扩展）。

train文件夹和test_*文件夹的结构是一样的。 然而，train数据的大小明显大于测试数据。 您可以通过返回到数据目录并执行以下命令来验证这一点，查看train和test集字数：

```
wc train/text test_feb89/text
```

下一步是创建Kaldi使用的原始语言文件。 在大多数情况下，这些将是整数格式的文本文件。 确保你回到s5目录并执行以下命令：

```
utils/prepare_lang.sh data/local/dict '!SIL' data/local/lang data/lang 
```

这将在本地文件夹中创建一个名为lang的新文件夹，该文件夹将包含描述相关语言的FST。脚本中它将数据中创建的一些文件转换为Kaldi读取的更为规范化的格式。 此脚本在data/lang/ 目录中创建其输出。 我们下面提到的文件将在该目录中。

此脚本创建的前两个文件称为words.txt和phones.txt（在目录data/lang/）中。 这些是OpenFst格式的符号表，并且表示从字符串到整数和后面的映射。仔细看这些文件; 因为它们很重要，而且经常被使用，所以你需要了解它们的内容。 它们与我们之前在分发概述中遇到的符号表格式格式相同。

查看具有后缀.csl（在data/lang/phones）的文件。 这些是冒号分隔的整数id的非沉默列表，分别是静音，手机。 它们有时作为程序命令行上的选项（例如指定静音手机的列表）以及其他目的需要。

查看phones.txt（在data/lang/）。 该文件是一个电话符号表，它还处理标准FST方案中使用的“消歧符号（disambiguation symbols）”。 这些符号通常称为＃1，＃2等; 参见“[加权有限状态转换器的语音识别](http://www.cs.nyu.edu/~mohri/pub/hbka.pdf)”一文。 我们还添加了一个符号＃0，它替代了语言模型中的epsilon转换; 有关更多信息，请参阅[消歧符号](http://www.kaldi-asr.org/doc/graph.html#graph_disambig)。 有多少消歧符号？ 在一些方案中，消歧符号的数量与共享相同发音的单词的最大数量相同。 在我们的方案中还有一些; 你可以在[这里](http://www.kaldi-asr.org/doc/graph.html#graph_disambig)找到更多的解释。

文件L.fst是FST格式的编译词典。 要看看它中有什么样的信息，你可以（从s5 /），做：

```
fstprint --isymbols=data/lang/phones.txt --osymbols=data/lang/words.txt data/lang/L.fst | head
```
如果bash找不到命令fstprint，则需要将OpenFST的安装路径添加到PATH环境变量中。 只需运行脚本path.sh就可以这样做：

```
. ./path.sh
```
下一步是使用上一步创建的文件创建一个描述语言语法的FST。 为此，请返回目录s5并执行以下命令：

```
 local/rm_prepare_grammar.sh
```
如果成功，这应该返回消息“Succeeded preparing grammar for RM.”。 将在 /data/lang中创建一个名为G.fst的新文件。

### 特征提取

下一步是提取训练特征。 在run.sh中搜索“mfcc”，并运行相应的三行脚本（您必须首先决定要将功能放在哪里，并相应地修改示例）。 确保您决定放置功能的目录有很多空间。 假设我们决定将功能放在/my/disk/rm_mfccdir上，我们会做一些类似的事情：

```
export featdir=/my/disk/rm_mfccdir
# make sure featdir exists and is somewhere you can write.
# can be local if you want.
mkdir $featdir
for x in test_mar87 test_oct87 test_feb89 test_oct89 test_feb91 test_sep92 train; do \
  steps/make_mfcc.sh --nj 8 --cmd "run.pl" data/$x exp/make_mfcc/$x $featdir; \
  steps/compute_cmvn_stats.sh data/$x exp/make_mfcc/$x $featdir; \
done
```

运行这些作业。 他们并行使用多个CPU，并应在快速机器上大约两分钟内完成。 您可以根据机器的CPU数量更改-nj选项（指定要运行的作业数）。 查看文件 exp/make_mfcc/train/make_mfcc.1.log 以查看创建MFCC的程序的日志记录输出。 在顶部，您将看到命令行（除非指定了-print-args = false，否则Kaldi程序将始终回显命令行）。

在脚本步骤 /make_mfcc.sh中，查看调用split_scp.pl的行。 你可以猜测这是什么。

键入

```
wc $featdir/raw_mfcc_train.1.scp 
wc data/train/wav.scp
```

你可以确认。

接下来看一下调用compute-mfcc-feats的行。 这些选择应该是相当清楚的。 涉及配置文件的选项是可以在Kaldi中使用的机制来传递配置选项，如HTK配置文件，但实际上很少使用。 位置参数（以“scp”和“ark，scp”开头的参数需要更多的解释。

在我们解释一下之前，再次查看脚本中的命令行，并使用以下方式检查输入和输出：

```
head data/train/wav.scp
head $featdir/raw_mfcc_train.1.scp
less $featdir/raw_mfcc_train.1.ark
```
小心 .ark文件包含二进制数据（如果您的终端看不到，您可能需要键入“reset”）。

通过列出文件，您可以看到.ark文件相当大（因为它们包含实际的数据）。 您可以通过键入更方便地查看这些存档文件之一（假设您在s5目录中并运行脚本path.sh）：

```
copy-feats ark:$featdir/raw_mfcc_train.1.ark ark,t:- | head
```
您可以从此命令中删除“,t”修饰符，如果您喜欢，请再次尝试 - 但是将其更改为“less”可能会很好，因为数据将为二进制。 查看相同数据的另一种方法是：

```
copy-feats scp:$featdir/raw_mfcc_train.1.scp ark,t:- | head
```

这是因为这些归档文件和脚本文件都表示相同的数据（从技术上讲，归档文件仅表示其八分之一，因为我们将其分为八个部分）。 请注意这些命令中的“scp:”和“ark:”前缀。 Kaldi不会试图弄清楚某些东西是从数据本身的脚本文件还是归档格式，实际上Kaldi从来没有尝试从文件后缀中执行操作。 这是为了一般的哲学原因，并且还防止与管道的不良相互作用（因为管道通常不具有名称）。

现在键入以下命令：

```
head -10 $featdir/raw_mfcc_train.1.scp | tail -1 | copy-feats scp:- ark,t:- | head
```
这打印出第十个训练文件中的一些数据。 请注意，在“scp:- ”中，“ - ”表示从标准输入读取，而“scp”则将其解释为脚本文件。

接下来，我们将描述哪些脚本和归档文件实际上是。 我们想要做的第一点是代码以同样的方式看到它们。 对于用户级别调用代码的一个特别简单的例子，键入以下命令：

```
tail -30 ../../../src/featbin/copy-feats.cc
```
您可以看到，这个程序的实际工作的部分只是三行代码（实际上有两个分支，每个代码有三行）。 如果您熟悉OpenFst中的StateIterator类型，您将注意到，我们迭代的方式是相同的样式（我们尽可能地像OpenFst一样作为样式兼容）。

基础脚本和归档是表的概念。 表基本上是由唯一字符串（例如话语标识符）索引的一组有序的项目（例如特征文件）。 一个表不是真正的C++对象，因为我们有独立的C++对象来访问数据，这取决于我们是写，迭代还是随机访问。 这些类型的示例，其中所讨论的对象是浮点矩阵（Matrix <BaseFloat>），是：

```
BaseFloatMatrixWriter
RandomAccessBaseFloatMatrixReader
SequentialBaseFloatMatrixReader
```
这些类型都是实际上是模板类的typedef。 这里不再详细说明了。 脚本（.scp）文件或存档（.ark）文件都被视为数据表。 格式如下：

* .scp格式是一个纯文本格式，每行有个key，然后是一个“扩展文件名”，告诉Kaldi哪里可以找到数据。
* 存档格式可以是文本或二进制文件（您可以使用“，t”修饰符写入文本模式;二进制是默认值）。 格式为：键（例如发音id），然后是空格，然后是对象数据。

关于脚本和档案的几点通用点：

* 指定如何读取表（归档或脚本）的字符串称为rspecifier;例如“ark:gunzip -c my/dir/foo.ark.gz |”。
* 指定如何编写表（归档或脚本）的字符串称为wspecifier;例如“ark,t:foo.ark”。
* 档案可以并置在一起，仍然是有效档案（没有“中心索引”）。
* 该代码可以顺序地或通过随机访问读取脚本和存档。用户级代码只知道是迭代还是进行查找;它不知道是访问脚本还是存档。
* Kaldi不会尝试在档案中表示对象类型;您必须提前知道对象类型
* 归档和脚本文件不能包含类型的混合。
* 通过随机访问来读取存档可能是无效的，因为代码可能必须缓存内存中的对象。
* 为了有效地随机访问存档，您可以使用“ark，scp”写入机制（例如用于将mfcc功能写入磁盘）来写出相应的脚本文件。然后，您将通过scp文件访问它。
* 另一种避免代码在存档中进行随机访问时将缓存一堆内容的方法是告知代码归档归档并按排序顺序调用（例如“ark，s，cs： - ”） 。
* 读取和写入存档的类型将以Holder类型进行模板化，该类型是“知道如何”读取和写入有问题的对象。

这里我们刚刚给出了一个概述，可能会提出更多的问题，而不是提供答案; 这只是为了让您了解所涉及的各种问题。 有关详细信息，请参阅[Kaldi I/O机制](http://www.kaldi-asr.org/doc/io.html)。

为了让您了解如何在管道中使用存档和脚本文件，请键入以下命令，并尝试了解发生了什么：

```
head -1 $featdir/raw_mfcc_train.1.scp | copy-feats scp:- ark:- | copy-feats ark:- ark,t:- | head
```
它可能有助于顺序运行这些命令，并观察发生的情况。 使用复制，记住pipe输出，因为您可能会列出很多内容（在ark文件的情况下可能是二进制文件）。

最后，为了方便起见，我们将所有测试数据合并到一个目录中。 我们将对这个平均的步骤进行所有的测试。 以下命令还将合并speakers，注意重复和重新生成这些speakers的统计信息，以便我们的工具不会抱怨。 通过运行以下命令（从s5目录）执行此操作。

```
utils/combine_data.sh data/test data/test_{mar87,oct87,feb89,oct89,feb91,sep92}
steps/compute_cmvn_stats.sh data/test exp/make_mfcc/test $featdir
```
我们还创建训练数据的一个子集（train.1k），每个speaker将只保留1000个话语。我们将用于训练。执行以下命令执行此操作：

```
utils/subset_data_dir.sh data/train 1000 data/train.1k 
```

### 单音训练

下一步是训练单声道模型。 如果您安装Kaldi的磁盘不大，您可能想要将exp/a 软链接到一个大磁盘上的某个目录（如果运行所有的实验，并且不清理，它可以达到几个千兆字节）。 键入

```
nohup steps/train_mono.sh --nj 4 --cmd "$train_cmd" data/train.1k data/lang exp/mono &
```
您可以通过下面命令来查看最新的输出

```
tail nohup.out
```

您可以以这种方式运行时间更长的作业，以便即使我们断开shell连接，也可以完成运行。实际上，这个脚本的标准输出和错误实际上只有很少的输出; 其中大部分都是在exp/mono/中记录文件。

当它正在运行时，查看文件数据/lang/topo。 该文件会被立即创建。 其中一个phones具有与其他phones不同的拓扑。 看看data/phones.txt，以便从numeric ID中找出哪个phones。 请注意，拓扑文件中的每个条目都有一个最终状态，没有转换。 拓扑文件中的约定是第一个状态是初始状态（概率为1），最后一个状态是最终状态（概率为1）。

键入

```
gmm-copy --binary=false exp/mono/0.mdl - | less
```
并查看模型文件。 您将看到它包含顶部拓扑文件中的信息，然后在模型参数之前的其他一些事情。 约定是.mdl文件包含两个对象：TransitionModel类型的一个对象，其中包含作为HmmTopology类型的成员变量的拓扑信息，以及相关模型类型的一个对象（在这种情况下，类型为AmGmm）。 通过“包含两个对象”，我们的意思是对象具有标准形式的写入和读取功能，我们称这些函数将对象写入文件。 对于这样的对象，这不是表的一部分（即没有涉及到“ark：”或“scp：”），写入是二进制或文本模式，可以由标准的命令行选项–binary=true或 –binary=false（不同的程序具有不同的默认值）。 对于表（即归档和脚本），二进制或文本模型由说明符中的“,t”选项控制。

浏览模型文件，看看它包含什么样的信息。 在这一点上，我们不会更详细地介绍Kaldi的模型如何表示; 请参阅[HMM拓扑结构和转换模型](http://www.kaldi-asr.org/doc/hmm.html)以了解更多。

我们将提到一个重要的一点，但是，在Kaldi中的p.d.f.由数字id表示，从零开始（我们称之为pdf-id）。 他们没有“名字”，就像HTK一样。 .mdl文件没有足够的信息来映射与上下文相关的phones和pdf-id。 有关该信息，请参阅tree 文件：do

```
copy-tree --binary=false exp/mono/tree - | less
```

请注意，这是一个单音(monophone)“tree”，所以它的内容非常琐碎 - 它没有进行任何“分割”。虽然这种树形格式并不是非常人性化的，但是我们已经收到了很多关于树型格式的问题，所以我们将会解释一下。本段的其余部分可以由休闲读者（ casual reader）跳过。在“ToPdf”之后，树形文件包含一个多态类型EventMap的对象，它可以被认为是存储一组代表上下文和HMM状态的整数（键值）对映射到数字pdf ID。来自EventMap的是类型ConstantEventMap（表示树的叶子），TableEventMap（表示某种查找表）和SplitEventMap（表示树分割）。在这个文件exp/mono/tree 中，“CE”是ConstantEventMap的标记（对应于树的叶子），“TE”是TableEventMap的标记（没有“SE”或SplitEventMap，因为这个是单声道的情况）。 “TE 0 49”是TableEventMap的开始，它在键零上“分割”（表示在单声道情况下长度为1的电话上下文向量中的第零个电话位置）。在括号中，由EventMap类型的49个对象进行跟踪。第一个为NULL，表示指向EventMap的零指针，因为将电话号码零保留给“epsilon”。示例非NULL对象是字符串“TE -1 3（CE 33 CE 34 CE 35）”，它表示在键-1上分割的TableEventMap。该键表示拓扑文件中指定的PdfClass，在本例中与HMM状态索引相同。该手机具有3个HMM状态，因此分配给该键的值可以取值0,1或2.括号内有三个类型为ConstantEventMap的对象，每个对象都表示树的叶子。

//TODO 剩下的没弄。。。

## 阅读和修改代码（半小时）

当triphone系统构建运行时，我们将需要一点时间来浏览代码的某些部分。 您将从本教程的这一部分中获得的主要内容是有关代码如何组织以及依赖关系结构的一些思想; 以及修改和调试代码的一些经验。 如果您想更深入地了解代码，我们建议您遵循主文档页面上的链接，我们将按主题组织更详细的文档。

### 常用工具

转到顶级目录（kaldi），然后进入src/。首先查看文件[base/kaldi-common.h](http://www.kaldi-asr.org/doc/kaldi-common_8h.html)（从shell或从编辑器查看它）。#includes 包含了几乎每个Kaldi程序使用的base/ 目录中的一些东西。大多数人可以从文件名中猜出提供的事物的类型：error-logging macros, typedefs,数学函数（如随机数生成）和其他#define。但这是一个被剥离的帮助库;在[util/common-utils.h](http://www.kaldi-asr.org/doc/common-utils_8h.html) 中有一个更完整的集合，包括命令行解析和处理扩展文件名（例如管道）的I / O函数。花几秒钟浏览util / common-utils.h，看看#includes是什么。我们将帮助程序分到base/目录的原因是为了使我们可以最小化对matrix/ 目录的依赖（这本身很有用）; matrix/ 目录只依赖于base /目录。查看matrix/Makefile并搜索base/ 以查看如何指定。在Makefile中查看这种类型的规则可以让您深入了解工具包的结构。

### Matrix库（和修改和调试代码）

现在看文件matrix/matrix-lib.h。 看看它包含什么文件。 这提供了矩阵库中事物种类的概述。 这个库基本上是一个用于BLAS和LAPACK的C++包装器，如果这意味着任何东西都可以给你（如果没有，不用担心）。 文件sp-matrix.h和tp-matrix.h分别涉及对称打包矩阵和三角填充矩阵。 快速浏览文件matrix/kaldi-matrix.h。 你将会了解矩阵代码的样子。它由一个表示矩阵的C++类组成。 如果您有兴趣，我们将在这里提供矩阵库中的小型教程。 你可能会注意到代码中似乎是一个奇怪的评论风格，评论由三个斜杠（///）开始。 这些类型的推荐，并以开头的方式阻止注释

```
/**
```
由Doxygen软件解释，自动生成文档。 它还会生成您正在读取的页面（此类型的文档的来源位于src / doc /）。