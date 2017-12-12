Kaldi数据准备所需要的文件（一个录音为一个完整的通话记录）

## wav.scp

给每个为每个录音文件编号，文件格式如下

```
<recording-id> <extended-filename>
```

recording-id 为录音文件唯一ID，extended-filename

## text


```
<utterance-id> <content>
```

## segments

```
<utterance-id> <recording-id> <segment-begin> <segment-end>
```

## utt2spk

```
<utterance-id> <speaker-id>
```

## spk2gender