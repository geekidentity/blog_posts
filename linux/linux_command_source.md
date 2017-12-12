---
categories: linux

tags: 
  - Linux
  - Linux命令

title: Linux命令 -- source（点命令）

date: 2017-07-14
---

source filename [arguments]在当前shell环境中从filename读取并执行命令，并返回从filename执行的最后一个命令的退出状态。直接执行shell文件则当前shell会fork/exec 一个子shell去执行filename中的命令，而子shell中从父shell中继承了环境变量，但是执行后不会改变父shell的环境变量。

如果文件名不包含斜杠，将会在PATH（环境变量）配置的目录中查找该文件。在PATH中搜索的文件无需执行。

当bash不是posix模式时，如果在PATH中找不到文件，则搜索当前目录。 如果关闭shopt builtin 命令的sourcepath 选项，则不会搜索PATH。 如果提供任何参数，它们将成为执行文件名时的定位参数（positional parameters）。 否则定位参数（positional parameters）不变。

返回状态是脚本中退出的最后一个命令的状态（如果没有执行任何命令，则为0），如果找不到filename或者不能读取，则返回false。


source命令又叫点命令，在需要用到source的情况下，直接换成'.'即可注意两个点之间有空格。

```Bash
. ./shell.sh
```
