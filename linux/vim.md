---
categories: linux
tags:
  - Linux
  - VIM

title: 配置 vimrc
date: 2017-11-18
---
# 配置 vimrc

在Home 目录下创建一个 .vimrc 文件来设置Vim属性。

```
set number             # 显示行号 
set autoindent         # 自动缩进 
set nowrap             # 不换行
```

删除一行或多行

```
dd ： (输入两次 d，下同)删除当前行；
5dd ：删除当前行开始的5行； 
dG ：(先输入d，然后按 shift 键输入g)删除当前行至最后一行的所以行。
```
