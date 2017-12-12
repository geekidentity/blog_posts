在运行XGboost之前，我们必须设置三种类型的参数：通用参数、booster参数和任务参数。

* 通用参数与我涉及到我们要使用哪个booster进行boosting运算，通常选择树（tree）或线性（linear）模型。
* Booster参数取决于你选择的booster。
* 决定学习场景的学习任务参数，例如：进行排序的回归任务可以使用不同的参数。
* 与xgboost的CLI版本相关的命令行参数