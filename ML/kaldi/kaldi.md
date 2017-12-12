# Kaldi 模型训练与测试流程

# 先决条件

要学习Kaldi 有一些先决条件要具备

* Linux:
    Kalid本身是在Linux下开发和测试运行的，虽然官方给了在Windows下用VS编译运行的方法，但我没有成功，坑比较多，除非你对Windows VS很熟，否则建议直接在Linux下搞。
* C++: Kaldi所有相关算法的实现都是用C++做的，如果要用Kaldi做自己的产品，C++是必须会的。

* 机器学习已经入门：搞Kaldi机器学习基本的东西要知道，不然，你都不知道你在干什么。

# 目录结构

首先介绍一下运行Kaldi项目的目录结构

```
|-- cmd.sh // 运行配置目录，设置Kaldi运行的环境变量，例如使用什么类型的队列
|-- conf  // 配置文件目录，mfcc、等参数的配置
|-- data  // Kaldi运行所产生的数据
|-- exp  // Kaldi每一步训练的模型数据及测试数据
|-- local  //存放run.sh 中调用的脚本工具，需要自己写
|-- mfcc  // mfcc数据
|-- path.sh //将Kaldi 工具和库目录添加到PATH
|-- run.sh // top层脚本，运行该脚本训练数据和测试， 需要自己写
|-- steps // kaldi 脚本工具， 复制到工程目录下
|-- tools // kaldi 脚本工具， 复制到工程目录下
`-- utils // kaldi 脚本工具， 复制到工程目录下

```

# 运行脚本

run.sh为总的运行脚本

```bash

```


## 语料准备

要做训练首先要有语料

Kaildi默认只支持wav格式的文件，为了方便，我们将为每个wav文件建立一个相同文件名的txt文件，在其放置标记后的内容，并做分词（why?）。

开放的语料主要的thchs30和aishell

## 词典准备

词典准备需要下面五个文件

* lexicon.txt
* extra_questions.txt
* nonsilence_phones.txt
* optional_silence.txt
* silence_phones.txt

local/tinet_prepare_dict.sh 是准备词典的，只需要提供lexicon.txt，该脚本就会将其他文件也自动生成。

调用
```bash
local/tinet_prepare_dict.sh $data/resource_aishell
```

脚本将会在data/local/dict文件夹下创建这五个文件。




## 需要我们创建的文件

有些文需要我们自己去创建，这些文件是Kaldi训练模型所需要的。
在data目录下创建train、test 文件夹，并在每个文件夹创建如下文件

* wav.scp
* utt2spk.scp
* spk2utt.scp
* text
* word.txt

### wav.scp


文件格式如下所示：

```
<recording-id> <extended-filename>
```

wav.scp文件的目的就是给每个录音文件指定一个唯一ID。

完成后会在data目录下创建两个文件夹 train、test

# MFCC

下面要计算wav的mfcc

在data目录下创建文件夹mfcc，并将train、test目录拷贝过来

接下来分别创建train和test的mfcc。

```
steps/make_mfcc.sh --nj $n --cmd "$train_cmd" data/mfcc/train exp/make_mfcc/train mfcc/train

steps/make_mfcc.sh --nj $n --cmd "$train_cmd" data/mfcc/test exp/make_mfcc/test mfcc/test
```

make_mfcc.sh的使用如下所示：

```
./steps/make_mfcc.sh 
Usage: ./steps/make_mfcc.sh [options] <data-dir> [<log-dir> [<mfcc-dir>] ]
e.g.: ./steps/make_mfcc.sh data/train exp/make_mfcc/train mfcc
Note: <log-dir> defaults to <data-dir>/log, and <mfccdir> defaults to <data-dir>/data
Options: 
  --mfcc-config <config-file>                      # config passed to compute-mfcc-feats 
  --nj <nj>                                        # number of parallel jobs
  --cmd (utils/run.pl|utils/queue.pl <queue opts>) # how to run jobs.
  --write-utt2num-frames <true|false>     # If true, write utt2num_frames file.
```

data/mfcc/train 为数据目录，exp/make_mfcc/train 为日志目录，mfcc/train是计算出的mfcc。

make_mfcc.sh 会在data/mfcc/{train,test} 创建feats.scp文件，内容如下所示：

```
000000 /lab/kaldi/kaldi_lab/mfcc/train/raw_mfcc_train.1.ark:7
000001 /lab/kaldi/kaldi_lab/mfcc/train/raw_mfcc_train.1.ark:3376
000002 /lab/kaldi/kaldi_lab/mfcc/train/raw_mfcc_train.1.ark:5705
```

这个文建指向了很多我们提取到的特征的原文件，而这也正是我们在一些脚本使用的。该文件的格式是

```
<utterance-id> <extended-filename-of-features>
```
其中 000000 /lab/kaldi/kaldi_lab/mfcc/train/raw_mfcc_train.1.ark:7
 表示从这个文件的第7个位置开始读（使用 fseek() 函数）。

## cmvn

cmvn.scp 保存了倒谱归一化均值和方差（cepstral mean and variance normalization）的统计信息.

```
<speaker-id> <extended-filename-of-cmvn>
```


compute_cmvn_stats.sh 命令如下所示

```

compute_cmvn_stats.shsteps/

Usage: steps/compute_cmvn_stats.sh [options] <data-dir> [<log-dir> [<cmvn-dir>] ]
e.g.: steps/compute_cmvn_stats.sh data/train exp/make_mfcc/train mfcc
Note: <log-dir> defaults to <data-dir>/log, and <cmvn-dir> defaults to <data-dir>/data
Options:
 --fake          gives you fake cmvn stats that do no normalization.
 --two-channel   is for two-channel telephone data, there must be no segments 
                 file and reco2file_and_channel must be present.  It will take
                 only frames that are louder than the other channel.
 --fake-dims <n1:n2>  Generate stats that won't cause normalization for these
                  dimensions (e.g. 13:14:15)
```


compute_cmvn_stats.sh 在数据目录下创建cmvn.scp

该文件格式是

```
<speaker-id> <extended-filename-of-cmvn>
```
