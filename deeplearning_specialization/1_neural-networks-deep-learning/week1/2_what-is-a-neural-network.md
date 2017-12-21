---
categories: 深度学习工程师

tags: 
  - AI
  - 深度学习工程师
  - 神经网络和深度学习
  - 学习笔记

title: 神经网络和深度学习-第一周深度学习概论-第二节：什么是神经网络

date: 2017-12-13
---

本系列博客是吴恩达(Andrew Ng)[深度学习工程师](http://mooc.study.163.com/smartSpec/detail/1001319001.htm) 课程笔记。全部课程请查看[吴恩达(Andrew Ng)深度学习工程师课程目录](http://blog.geekidentity.com/deeplearning_specialization/catalogues/)

# 什么是神经网络

“深度学习”指的是训练神经网络，那么什么是神经网络呢？

我们从房价预测的例子开始，假设我们有一个 6 个房间的数据集，已知房屋的面积（单位是平方英尺或平方米）和价格。我们需要找到一个根据房屋面积预测房价的函数。

如果你熟悉线性回归（Linear Regression），你可以用这些数据来拟合一条直线：
![image](http://blog.geekidentity.com/images/deeplearning_specialization/neural-networks-deep-learning/week1/2_what-is-a-neural-network/what_is_neural_network_01.png)

但我们知道，价格永远不能为负，因此这条直线不太合适，因此我们让这条线弯曲，让它结束在0 点（原点）。下面这条粗的蓝线，就是你想要的函数：根据房屋面积预测价格
![image](http://blog.geekidentity.com/images/deeplearning_specialization/neural-networks-deep-learning/week1/2_what-is-a-neural-network/what_is_neural_network_02.png)


你可以把这个房屋加个拟合函数，看成是一个非常简单的神经网络，这几乎是最简单的神经网络了：如下图，我们把房屋的面积( x )作为神经网络的输入，通过节点（小圆圈），最后输出了价格( y )。
![image](http://blog.geekidentity.com/images/deeplearning_specialization/neural-networks-deep-learning/week1/2_what-is-a-neural-network/what_is_neural_network_neuron.png)

这个小圆圈就是一个独立的神经元，你的网络实现了这个函数的功能，这个神经元所做的任务是：输入面积，完成线性运算，取不小于0 的值，最后得到输出的预测价格。在神经网络的文献中，我们会经常看到这个函数：函数一开始是0，然后就是一条直线，这个函数称作ReLU(rectified linear unit, 修正线性单元) 函数，修正指取值不小于0 的值，这就是这个函数长这样的原因。
![image](http://blog.geekidentity.com/images/deeplearning_specialization/neural-networks-deep-learning/week1/2_what-is-a-neural-network/what_is_neural_network_03.png)

> 不理解ReLU 函数不要担心，在课程后面，我们还会看到它。

这是一个单神经元网络，规模很小的神经网络，大一点的神经网络是把这些单个神经元堆叠起来形成的。所以你可以把这些神经元想象成单独的乐高积木，通过搭建积木来构建一个更大的神经网络。

来看一个例子，如下图所示，我们不仅用房屋面积来预测价格，现在还有一些房屋的其它特征，知道了一些别的信息，比如卧室的数量。你可能会想到，有一个很重要的因素会影响房屋价格：就是“家庭人数”，这个房能住下一个三口之家、四口之家或五口之家，这个性质和面积大小相关，还有卧室数量能否满足家庭人数需要。你可能知道邮编（有些国家叫作邮政编码），邮编或许能作为一个特征，说明了步行化程序（意思是周边基础设施非常完善，去商店、学校等走着就可以了）。另外根据邮政编码和富裕程度（在美国是这样的）体现了附近学校的质量。我画的每一个小圈都可能是一个ReLU或其他非线性函数，基于房屋面积和卧室数量你可以估算家庭人口基于邮编，可以评估步行化程度和学校质量，最后你可能会想人们愿意在房屋上花多少钱和他们关注什么息息相关。在这个例子中，家庭人口、步行化程度以及学校质量都能帮助你预测房屋价格。
![image](http://blog.geekidentity.com/images/deeplearning_specialization/neural-networks-deep-learning/week1/2_what-is-a-neural-network/what_is_neural_network_house_price_prediction_neuron.png)

在这个例子中x 是所有的这四个输入，y是预测的价格。把这些独立的神经元叠加起来，这些简单的预测器成为了一个稍大一点的神经网络。神经网络的一部分神奇之处在于，当你实现它之后，你要做的只是输入x

就能得到输出y，不管训练集有多大，所有的中间过程，它都会自己完成。那么你实际上做的如下图所示：
![image](http://blog.geekidentity.com/images/deeplearning_specialization/neural-networks-deep-learning/week1/2_what-is-a-neural-network/what_is_neural_network_house_price_prediction_neuron_2.png)

这有四个输入的神经网络，输入的特征可能是卧室的数量、邮政编码和周边富裕程度。已知这些输入的特征，神经网络的工作就是预测对应的价格。同时也注意到中间的这些圈圈，在一个神经网络中它们被叫做“隐藏单元”，它们每个的输入都同时来自四个特征。比如说，我们不会具体说第一个结点表示家庭人口或者说家庭人口仅取决于特征$x_1$和$x_2$；我们会说：神经网络，你自己决定这个节点是什么，我们只给你四个输入特征，随便你怎么计算。因此我们说输入层（最左边的x）、隐藏层（中间的一层）在神经网络中连接数是很高的，因为输入的每一个特征都连接到了中间的每个圆圈。值得注意的是，神经网络只有你喂给它足够多的关于x、y的数据，给到足够的x、y训练样本神经网络非常擅长于计算从x到y的精确映射函数。

这就是一个基本的神经网络，你可能发现自己的神经网络在监督学习的环境下是如此有效和强大，也就是说你只要尝试输入一个x，即可把它映射成y，像我们在房价预测例子中看到的。在下一节中，你会看到更多监督学习的例子，有些例子会让你觉得，你的神经网络对你的应用场合非常有帮助
