---
categories: git

tags: 
  - git

title: gitignore配置

date: 2017-01-15
---

项目中有一些东西不需要提交到git中，可以用.gitignore指定哪些文件不提交。
## 配置语法：

- 以斜杠“/”开头表示目录；

- 以星号“*”通配多个字符；

- 以问号“?”通配单个字符

- 以方括号“[]”包含单个字符的匹配列表；

- 以叹号“!”表示不忽略(跟踪)匹配到的文件或目录；

此外，git 对于 .ignore配置文件是按行从上到下进行规则匹配的，意味着如果前面的规则匹配的范围更大，则后面的规则将不会生效；

## 示例

```
# 以'#'开始的行为注释.
# 忽略掉所有文件名是 foo.txt的文件.
# 忽略所有生成的 html文件,
*.html
# foo.html是手工维护的，所以例外.
!foo.html
# 忽略所有.o和 .a文件.
*.[oa]
target/
# 忽略目录 target 下的全部内容；注意，不管是根目录下的 /target/ 目录，还是某个子目录 /child/target/ 目录，都会被忽略；

/target/
# 忽略根目录下的 /target/ 目录的全部内容；

/*
!.gitignore
!/fw/bin/
!/fw/sf/

# 忽略全部内容，但是不忽略 .gitignore 文件、根目录下的 /fw/bin/ 和 /fw/sf/ 目录；
```
对Maven构建的Java项目这里提供一个方便的模板。

一个项目中可能有很多模块，对多模块项目建议只在项目根目录下建立一下.gitignore文件，方便维护。

```
target/
.classpath
.settings/
.project
```
