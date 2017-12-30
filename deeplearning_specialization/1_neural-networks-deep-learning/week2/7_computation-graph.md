---
categories: 深度学习工程师

tags: 
  - AI
  - 深度学习工程师
  - 学习笔记
  - 计算图


title: 神经网络和深度学习-第二周神经网络基础-第七节：计算图

date: 2017-12-30
---

本系列博客是吴恩达(Andrew Ng)[深度学习工程师](http://mooc.study.163.com/smartSpec/detail/1001319001.htm) 课程笔记。全部课程请查看[吴恩达(Andrew Ng)深度学习工程师课程目录](http://blog.geekidentity.com/deeplearning_specialization/catalogues/)

可以说，一个神经网络的计算都是按照前向或反向传播过程来实现的。首先计算出神经网络的输出，紧接着进行一个反向传输操作，后者我们用来计算出对应梯度或者导数。而计算图解释了为什么用这样的方式这样实现。

为了阐明这个计算过程，我们举一个比logistic回归更加简单的，不那么正式的神经网络的例子。我们计算函数$J$：
$$
J(a,b,c)=3(a+bc)
$$
计算这个函数实际上有三个不同的步骤第一个首先是计算b乘以c，我们把它存储在变量u中：
$$
u=bc
$$
然后计算$v=a+u$，最后计算$J=3v$。我们可以把这三步画成如下计算图：

![](http://blog.geekidentity.com/images/deeplearning_specialization/neural-networks-deep-learning/week2/7_computation-graph/computation-graph-1.png)

可以看出，通过一个从左向右的过程，你可以计算出J的值。在接下的课程中我们会看到，为了计算导数从右到左的这个过程，和这个蓝色的过程相反。这会是用于计算导数最自然的方式。

![](http://blog.geekidentity.com/images/deeplearning_specialization/neural-networks-deep-learning/week2/7_computation-graph/computation-graph-2.png)