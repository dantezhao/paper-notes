## Wide & Deep Learning for Recommender Systems 阅读笔记


### 0 现有方法的不足

现有wide模型memorization能力强，不具备generalization能力。deep模型具备generalization能力，但在数据稀疏的情况下也存在过于“泛化”，进而推荐不相关物品的情况。

### 1 WDL是如何解决这个问题的

WDL将wide linear model 和deep neural network 结合起来，一起训练。WDL模型的优势在于同时获得了wide模型的memorization（“记忆”）能力，和deep模型的generalization（“泛化”）能力。

### 2 两个问题

什么是“记忆”能力和“泛化”能力？

为什么wide模型的“记忆”能力强？deep模型的“泛化”能力强？

（1）什么是“记忆”能力和“泛化”能力

Memorization can be loosely defined as learning the frequent co-occurrence of items or features and exploiting the correlation available in the historical data.

“记忆”能力是指对历史数据集中已经出现过的特征、特征关联及其与label之间关系进行很好的学习。即对历史数据中已经出现过的模式进行很好的学习。

Generalization, is based on transitivity of correlation and explores new feature combinations that have never or rarely occurred in the past.

“泛化”能力指能将从历史数据集中学到的模式迁移，用于对包含未出现特征组合的样本进行预测。

（2）**为什么wide模型的“记忆”能力强？deep模型的“泛化”能力强？**

wide模型以逻辑回归（LR）模型为例，deep模型以深度神经网络为例。

**主要差异在于两者的特征组合能力不一样。**

LR是属于广义线性模型，本身不具备对特征之间非线性关系进行建模。所以通过人工构建特征组合（交叉特征）的方式来给LR模型增加非线性建模能力。特征组合文中的例子如下：

AND(user_installed_app=netflix, impression_app=pandora"), whose value is 1 if the user installed Netflix and then is later shown Pandora.

可以看出来人工构建组合特征，主要基于历史数据集中已经出现过的特征取值组合，而对于未出现过的特征取值组合没有办法构建组合特征。所以LR模型具备很好的记忆能力，但没有泛化能力。

深度神经网络模型无需人工构建组合特征，有自动做特征组合的能力。而且由于加入非线性的激活函数，神经网络模型具备非线性建模能力。神经网络模型的泛化能力除了因为它自动特征组合的能力，还因为它对类别特征的取值做embedding。这样，由于embedding的特性，即使没出现过的特征取值组合，神经网络模型也可以计算得到组合特征的值。WDL模型的架构图如下：

![](https://raw.githubusercontent.com/dantezhao/paper-notes/master/0003/bigablecat_Wide_and_Deep_model_structure.png)


WDL模型的函数表达如下：

![](https://raw.githubusercontent.com/dantezhao/paper-notes/master/0003/bigablecat_wide_deep_model.gif)


### 3 WDL模型与ensemble的区别

本文将wide模型和deep模型联合训练不同于一般的模型融合（ensemble）。

ensemble时，单个的基模型在训练的时候是彼此独立的。只有在做预测的时候，才将多个基模型的预测结果进行融合得到最终的预测结果。

WDL是将wide模型和deep模型作为一个整体进行训练，意味着同步用反向传播技术更新wide模型和deep模型的参数

### 4 实验结果

实验结果也表明，WDL模型不管是在线下还是线上相比于单独的wide模型和单独的deep模型，效果都有明显提升。

![](https://raw.githubusercontent.com/dantezhao/paper-notes/master/0003/Table_1.gif)
