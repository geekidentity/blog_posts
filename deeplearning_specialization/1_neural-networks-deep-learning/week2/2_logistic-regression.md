---
categories: 深度学习工程师

tags: 
  - AI
  - 深度学习工程师
  - 学习笔记
  - 二分分类
  - Logistic
  - sigmoid函数


title: 神经网络和深度学习-第二周神经网络基础-第二节：Logistic回归

date: 2017-12-19
---

本节将会讲解logistic回归，logistic是一个学习算法，用在监督学习问题中输出标签y是0或1的时候，这是一个二元分类问题。

已知的输入特征向量x可能是一张图，你希望识别出这个图里是不是一只猫。你需要一个算法可以给出一个预测值我们说预测值$\hat{y}$，就是你对y的预测。更正式的说，你希望$\hat{y}$是一个概率，当输入特征x满足条件时y就是1。所以换句话说，如果x是图片，正如我们在上一节中看到，你希望$\hat{y}$能告诉你这个图里是猫的概率。所以x正如我们之前课程里所说的是一个$n_x$维向量：
$$
x \in R^{n_x}
$$
已知logistic回归的参数是w(也是$n_x$维向量，$w \in R^{n_x}$)而b就是一个实数，所以已知输入x、参数w和b，我们如何计算输出预测$\hat{y}$ ？你可以试试（但其实并不靠谱）$\hat{y}=w^T  x + b$，输入x的线性函数，事实上，如果你做线性回归就是这么算的。但这并不是一个非常好的二元分类算法，因为你希望$\hat{y}=1$，所以$\hat{y}$应该在0和1之间，但事实上这很难实现，因为$w^{T} x + b$可能比1大的多或者甚至是负值，这样的概率是没有意义的，你希望$\hat{y}$在0和1之间。所以在logisticl回归中我们的输出变成：$\hat{y}= \sigma(w^T x +b)$，加上一个sigmoid函数，$sigmoid(x)$函数的图形如下图所示：

![sigmoid函数](http://blog.geekidentity.com/images/deeplearning_specialization/neural-networks-deep-learning/week2/2_logistic-regression/sigmoid-function.png)

$sigmoid(x)$公式为：
$$
\sigma(z) = \frac{1}{1+e^{-z}} \quad z \in R
$$
要注意一些事项，如果z非常大，那么$e^{-z}$就很接近0那么$sigmoid(x)$无限接近1，相反如果z很小$sigmoid(x)$就会无限接近0。所以当你实现logistic函数时，你要做的是学习参数w和b，所以$\hat{y}$变成了对$y=1$比较好的估计。

在继续之前，我们再讲讲符号约定当我们对神经网络编程时，我们通常会把w和参数b分开（这里b对应一个拦截器）。在其他课程是你可能看过其他不同的表示，在一些符号约定中，定义一个额外的特征向量$x_0=1$所以出现x是$R^{n_x + 1}$维向量；将$\hat{y}$定义为$\sigma(\theta^T x)$在这种符号约定中，你有一个向量参数$\theta= \begin{bmatrix} \theta_0 & \theta_1 & ... & \theta_{n_x} \end{bmatrix}^T$，所以$\theta_0$扮演是的b的角色，这是一个实数而$\theta_1$到$\theta_{n_x}$的作用和w一样。事实上，当你实现神经网络时，将b和w看成单独的参数可能更好，所以对于这门课，我们不会使用这种符号约定。

现在你看到了logistic回归模型长什么样，下一节我们看看参数w和b，需要定义一个成本函数（cost function）。