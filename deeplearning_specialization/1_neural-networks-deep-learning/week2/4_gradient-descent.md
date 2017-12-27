---
categories: 深度学习工程师

tags: 
  - AI
  - 深度学习工程师
  - 学习笔记
  - 梯度下降


title: 神经网络和深度学习-第二周神经网络基础-第四节：梯度下降法

date: 2017-12-27
---

本系列博客是吴恩达(Andrew Ng)[深度学习工程师](http://mooc.study.163.com/smartSpec/detail/1001319001.htm) 课程笔记。全部课程请查看[吴恩达(Andrew Ng)深度学习工程师课程目录](http://blog.geekidentity.com/deeplearning_specialization/catalogues/)

在上一节中学习了损失函数，损失函数是衡量单一训练样例的效果，成本函数用于衡量参数w和b的效果，在全部训练集上来衡量。下面我们讨论如何使用梯度下降法，来训练和学习训练集上的参数w和b，使得$J(w,b)$尽可能地小。

![](http://blog.geekidentity.com/images/deeplearning_specialization/neural-networks-deep-learning/week2/4_gradient-descent/gradient-descent-function.png)

这个图中的横轴表示空间参数w和b，在实践中，w可以是更高维的。成本函数$J(w,b)$是在水平轴w和b上的曲面，曲面的高度表示了$J(w,b)$在某一点的值，我们所想要做的就是找到这样的w和b，使其对应的成本函数J值是最小值。可以看到成本函数$J$是一个凸函数，因此我们的成本函数$J(w,b)$之所以是凸函数，其性质是我们使用logistic回归的个特定成本函数$J$的重要原因之一。为了找到更好的参数值，我们要做的就是用某初始值初始化w和b，用图上最上面的小红点表示。

对于logistic回归而言几乎任意初始化方法都有效，通用用0来进行初始化，但对于logistic回归，我们通常不这么做。因为函数是凸的无论在哪里初始化，都应到达同一点或大致相同的点。梯度下降法所做的就是从初始点开始朝最陡的下坡方向走，就像图里一样沿着红点一直走，直到到达或接近全局最优解。