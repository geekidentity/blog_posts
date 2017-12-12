---
categories: spring

tags: 
  - Spring
  - Java

title: Spring MVC ant路径匹配原则

date: 2017-03-14
---

## 简介
 

## 匹配原则
Ant风格按以下规则匹配URL：

通配符 | 说明
---|---
? | 匹配一个字符
* | 匹配零个或多个字符
** | 匹配路径中的零个或多个目录
{spring:[a-z]+}} | 配置正则表达式 [a-z]+ 作为spring的路径变量(4.3)

## 例子

URL路径 | 说明
--- | ---
com/t?st.jsp | 匹配com/test.jsp，但也匹配com/tast.jsp或com/txst.jsp
com/*.jsp | 匹配com目录中的所有.jsp文件，不包括子文件夹
com/**/test.jsp | 匹配com路径下的所有test.jsp文件包括子文件夹
org/springframework/\*\*/*.jsp | 匹配org/springframework路径下的所有.jsp文件，包括子文件夹
org/**/servlet/bla.jsp | 匹配 org/springframework/servlet/bla.jsp 也包括 org/springframework/testing/servlet/bla.jsp 和 org/servlet/bla.jsp
com/{filename:\\\w+}.jsp} | 将匹配com/test.jsp并将值test分配给filename变量(4.3)

## 注意

**按最长匹配原则匹配**

**4.3表示Spring MVC4.3中才引入的特性。**