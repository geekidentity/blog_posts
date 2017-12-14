---
categories: AI

tags: 
  - AI

title: 1 - 关于Kaldi项目
date: 2017-08-06
---

Kaldi的代码现在 https://github.com/kaldi-asr/kaldi 。 可以使用命令`git clone https://github.com/kaldi-asr/kaldi` 检出最新版本或在github上直接下载。

该网站还包含了Kaldi模型构建，即运行Kaldi方式(recipes)的输出。请注意，这些模型构建都是相当旧的，因为文档的模型构建部分在从svn切换到git之前进行的，而我们还没有更新该站点来处理新的git存储库。 我们没有立即计划将更多的Kaldi模型上传到本网站，或修改它以与新的git存储库一起使用，因为我们发现处理那些想要使用从本网站下载的模型的人往往会占用了很多时间，影响力相对较小。 这可能会在未来发生变化（以年为单位）。

要浏览可用的模型构建，请单击模型。

# 关于Kaldi项目

## 什么是Kaldi

Kaldi是一个用C++ 编写的语音识别工具包，在Apache License v2.0下授权。Kaldi目的在于语音识别研究人员使用。有关更详细的历史和贡献者名单，请参阅[Kaldi项目的历史](http://kaldi-asr.org/doc/history.html)。

## Kaldi名字由来

据传说，Kaldi是发现咖啡厂的埃塞俄比亚旅行者。

## Kaldi与其他工具包

Kaldi的目标和范围与[HTK](http://htk.eng.cam.ac.uk/)相似。 目标是使用C ++编写的现代而灵活的代码，易于修改和扩展。重要功能包括：

* 与有限状态转换器（Finite State Transducers, FST）的代码级集成
    * 我们针对OpenFst工具包进行编译（使用它作为类库）。
* 广泛的线性代数支持
    * 我们包括一个包含标准[BLAS](http://www.netlib.org/blas/)和[LAPACK](http://www.netlib.org/lapack/)例程的[矩阵库](http://kaldi-asr.org/doc/matrix.html)。
* 可扩展设计
    * 我们尽可能地以最通用的形式提供我们的算法。例如，我们的解码器在一个对象上进行模板化，该对象提供由（frame, fst-input-symbol）元组索引的分数。这意味着解码器可以从任何合适的分数来源工作，例如神经网络。
* 开放license
    * 该代码在Apache 2.0下授权，这是限制性最低的许可证之一。
* 完整的方案
    * 我们的目标是提供建立语音识别系统的完整方案，这些系统由广泛可用的语料库（如语言数据联盟（LDC）提供）提供。
* 发布完整方案的目标是Kaldi的一个重要方面。由于该代码可以根据允许修改和重新发布的许可证公开提供，因此我们希望鼓励人们以类似格式与Kaldi自己的示例脚本一起发布其代码及其脚本目录。

考虑到时间限制，我们试图使Kaldi的文档尽可能完整，但在短期内，我们无法希望生成与HTK相同的文档。特别是在HTKBook中有很多介绍性的材料，解释了未分类的统计语音识别，这可能不会出现在Kaldi的文档中。 Kaldi的大部分文档都是以专家的方式进行编写的。 在未来，我们希望使它更容易访问，同时考虑到我们的目标受众是语音识别研究人员或研究人员的培训。 一般来说，Kaldi不是一个“for dummies”的语音识别工具包。 它将允许您进行多种不合理的操作。

## Kaldi的特性

在本节中，我们尝试总结Kaldi工具包的一般性质。

* 我们强调通用算法和通用方法
    * 使用“通用算法”，我们的意思是线性变换，而不是某种特定于语言的变体。但是，如果更具体的算法是有用的，那么我们并不打算太教条。
    * 我们想要可以在任何数据集上运行的方案，而不是必须定制的方案。
* 我们更喜欢可靠的算法
    * 方案的设计方式，原则上它们绝对不会以灾难性的方式失败。 尽管如此，尽管它们在“正常情况下”并不失败（尽管FST的重量推动）通常会有所帮助，但可能会在某些情况下崩溃或使事情变得更糟例）。
* Kaldi代码经过彻底测试。
    * 目标是使所有或几乎所有的代码具有相应的测试。
* 我们试图使案例更简单。
    * 构建一个大型语音工具包时，代码可能成为很少使用的替代品的森林，存在危险。 我们试图通过以下方式构建工具包来避免这种情况。 每个命令行程序通常用于有限的一组情况（例如，解码器可能适用于GMM）。 因此，当您添加新类型的模型时，您将创建一个新的命令行解码器（调用相同的底层模板代码）。
* Kaldi代码很容易理解。
    * 尽管Kaldi工具包整体可能会变得非常大，但我们的目标是让每一个部分都可以理解，而不需要太多的努力。 如果提高单个作品的可理解性，我们将接受一些代码重复。
* 卡尔迪代码容易重用和重构。
    * 我们的目标是尽可能松散地耦合工具包。 一般来说，这意味着任何给定的头应该需要#include尽可能少的其他头文件。 特别地，矩阵库仅依赖于另一个子目录中的代码，因此它可以独立于Kaldi的其余所有部分使用。

## 项目状况

目前，我们有大多数标准技术的代码和脚本，包括所有标准线性变换，MMI，增强的MMI和MCE辨别性训练，以及特征空间辨别训练（如fMPE，但基于提升的MMI）。 我们有“华尔街日报”和“资源管理”以及“配电板”的工作方法。 由于词汇和语言模型问题，交换机配方还没有提供最先进的结果 - 我们不使用任何外部数据源。

注意：在我们打算使用Kaldi（“v1”等主要版本的版本号）的早期阶段之后，我们意识到这些类型的版本不能很好地与自然的开发风格相结合，这是非常连续的。 目前我们只保留“主”开发分支，这是您应该使用的版本。 另外，经常做“git拉”来保持最新; 有关详细信息，请参阅下载和安装Kaldi。

## 在论文中引用Kaldi

如果您想在论文中引用Kaldi，可以使用以下参考。

```
@INPROCEEDINGS{
         Povey_ASRU2011,
         author = {Povey, Daniel and Ghoshal, Arnab and Boulianne, Gilles and Burget, Lukas and Glembek, Ondrej and Goel, Nagendra and Hannemann, Mirko and Motlicek, Petr and Qian, Yanmin and Schwarz, Petr and Silovsky, Jan and Stemmer, Georg and Vesely, Karel},
       keywords = {ASR, Automatic Speech Recognition, GMM, HTK, SGMM},
          month = dec,
          title = {The Kaldi Speech Recognition Toolkit},
      booktitle = {IEEE 2011 Workshop on Automatic Speech Recognition and Understanding},
           year = {2011},
      publisher = {IEEE Signal Processing Society},
       location = {Hilton Waikoloa Village, Big Island, Hawaii, US},
           note = {IEEE Catalog No.: CFP11SRW-USB},
}
```
论文可以在[这里](http://publications.idiap.ch/downloads/papers/2012/Povey_ASRU2011_2011.pdf)找到。

# 其他Kaldi相关资源（以及如何获得帮助）

可以找到Kaldi知识的主要地方是本网站和代码库中。

代码库包含Kaldi代码; 安装脚本; 以及位于子目录egs/中的多个不同数据集的示例脚本。

Kaldi的[项目页面](http://kaldi-asr.org/)包含一些有用的资源; 请参阅 kaldi-asr.org/forums.html 上的有关帮助论坛和电子邮件列表的信息。

# 下载并安装Kaldi

## 下载Kaldi

我们现在已经转型为GitHub，以便将来的发展。 你首先需要安装Git。 最新版本的Kaldi，可能包括未完成和实验功能，可以通过键入shell来下载：

```
git clone https://github.com/kaldi-asr/kaldi.git kaldi --origin upstream
   cd kaldi
```
如果要获取更新和错误修复，您可以访问一些检出目录，然后键入

```
git pull
```
## 安装kaldi

顶级安装说明在文件INSTALL中。 对于Windows，Windows/INSTALL中有单独的说明。 另请参见[构建过程（如何编译Kaldi）](http://kaldi-asr.org/doc/build_setup.html)，它解释了构建过程在内部的工作原理。

示例脚本在目录egs/ 下

# Kaldi 版本

Kaldi在其一生中有三种不同的版本控制方法。 原来Kaldi是一个基于Subversion（svn）的项目，并且在Sourceforge上托管。 然后Kaldi被移动到github，而在一段时间，唯一可用的版本号是提交的git hash。

2017年1月，我们推出了版本号计划。 Kaldi的第一个版本是5.0.0，表彰这个项目已经存在了很长时间了。 基本方案是主要/次要/补丁，但是“补丁”版本号也可能包含功能（通常是后向兼容的）。 每当向github上合并Kaldi时，“补丁数”就会自动增加。

当进行相对较大的更改或不兼容的更改时，我们只打算更改主要或次要版本号。 卡尔迪版本5.1正在准备中。 完成后（大概在2017年2月初），最新版本的5.0.x将被备份到名为“5.0”的分支，“主”将指向5.1.0版本。 根据需求，我们可能会继续使用修正等更新5.0分支。

我们总是打算推荐Kaldi用户查看最新版本的“master”，因为支持多个版本会增加我们的工作量。

# 安装和运行Kaldi所需的软件

## 理想的计算环境

首先，我们将介绍理想的计算环境，然后我们将说出运行Kaldi所需的最低限度。 理想的计算环境是运行Sun GridEngine（SGE）的Linux机器集群（任何主要分布集群），可通过NFS或某些类似的网络文件系统访问共享目录。 在理想情况下，网格上的某些计算机将具有可用于神经网络训练的NVidia GPU，您可以通过向qsub添加一些额外的选项来将其保留在队列中。 有关更多信息，请参[阅Kaldi中的并行化](http://kaldi-asr.org/doc/queue.html)。

前段时间，我们开始了一个名为[Kluster](https://sourceforge.net/projects/kluster/)的项目，向您展示如何在Amazon EC2上创建此类集群; 然而，这不是很好的维护; 麻省理工学院的[StarCluster](http://star.mit.edu/cluster/)是一个更大更好的支持项目，提供相同的功能。 大多数脚本应适用于基于Debian或Red Hat的本地托管的群集; 您可以调查[Rocks](http://www.rocksclusters.org/wordpress/)，旨在帮助您设置这样的集群。

## 最小计算环境

运行Kaldi的最小计算环境是任何类Unix环境; 并且可以在单个机器上运行它，尽管速度会更慢，但是您可以减少一些示例脚本中使用的作业数量，以避免耗尽机器的内存。

Kaldi最好在Debian和Red Hat Linux上进行测试，但可以在任何Linux发行版，Cygwin或Mac OSX上运行。

Kaldi的脚本已经被写成这样一种方式，如果用不同语法（如Tork）的类似机制替换SGE，那么应该比较容易使它工作; 我们还提供了一个“愚蠢”的替代品，您可以在没有排队系统时使用（在脚本中搜索run.pl和ssh.pl）。

过去Kaldi已经在Windows上编译了; 然而，示例脚本不会在那里工作，并且我们不是非常主动地维护代码或Windows构建脚本的Windows兼容性（当我们被告知关于它们时，我们会修复问题）。

## 需要的软件包

以下是安装Kaldi所需的一些包的一个非详尽的列表。 完整列表并不重要，因为安装脚本会告诉您您丢失了什么。

* Git：这需要下载Kaldi和其他依赖的软件。
* wget是安装下面描述的一些非Kaldi组件所必需的
* 示例脚本需要标准的UNIX实用程序，例如bash，perl，awk，grep和make。
* ​

如果您的系统上安装了ATLAS线性代数包，也可能会有帮助。 大多数系统已经有这个（你也可以通过简单的命令，如“yum search atlas”或“apt-cache search libatlas”）搜索linux中的软件包进行安装; 最好的方法是忽略现在的这个要求，看看你在安装Kaldi时是否有问题。

## Kaldi安装的软件包

以下工具和库随附在tools/ 目录中的安装脚本，因此您不必自己安装它们（注意：这是一个非详尽的列表）。

* OpenFst：我们针对此编译并大量使用它。
* IRSTLM：这是一个语言建模工具包。一些示例脚本需要它，但它并没有与Kaldi紧密集成;我们可以将任何Arpa格式的语言模型转换为FST。
  IRSTLM构建过程需要automake，aclocal和libtoolize（ 相应的包是automake和libtool）。
  注意：一些示例脚本现在使用SRILM;我们可以轻松安装，虽然您仍然需要在线注册才能下载。
* SRILM：一些示例脚本使用它。它通常是比IRSTLM更好和更完整的语言建模工具包;唯一的缺点是许可证，这不是免费的商业用途。您必须在下载页面上输入您的名称才能下载，因此安装脚本需要进行人工交互。
* sph2pipe：这是将sph格式文件转换为其他格式，如wav。使用LDC数据的示例脚本是必需的。
* sclite：这是为了得分，没有必要，因为我们有自己的简单的计分程序（compute-wer.cc）。
* ATLAS，线性代数库。这仅仅是标头的需要;在典型的设置中，我们预计ATLAS将在您的系统上。但是，如果您的系统上尚不能编译ATLAS，只要您的计算机没有启用CPU限制。
* CLAPACK，线性代数库（我们下载头文件）。这仅在您没有ATLAS的系统上有用，而是使用CLAPACK进行编译。
* OpenBLAS：这是ATLAS或CLAPACK的一个同义词。脚本默认情况下不使用脚本，但是我们提供安装脚本，以便您可以将其与ATLAS进行比较（它比ATLAS更为积极地维护）。

# 法律资料

http://kaldi-asr.org/doc/legal.html

