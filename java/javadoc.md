---
categories: Java

tags: 
  - Java

title: 文档注释规范

date: 2016-12-20
---

# 包描述
我们功能一般是分包开发的，在包下创建一个package.html，在包里简要描述本包的功能。
模板：

```
<html>
<body>
package description
</body>
</html>
```

# 类描述
可以用较长的文字说明该类是的功能，最好的方式是其他人只读类描述就能知道这个类是做什么的，应该有什么样的方法。
基本模板：

```
<h1>标题</h1>
<p>主要内容进行<strong>强调</strong>，更多信息查看{@link Object}</p>
```

在eclipse中设置创建类时的模板
在eclipse菜单

Window -> Preferences -> Java -> Code Style –> Code Templates

中，选择Comments中的Types，编辑

```
<p>Copyright (c).</p>
@author author
@since 
@version 
```

# 方法描述
方法描述要说明方法做了什么，有什么影响，使用时有什么要注意的；可以不用说具体怎么做。
@param  @return  @throws都要进行说明

# 成员变量描述
简要说明成员变量的意义。

```
/**
 * field description
 */
private String field
```
