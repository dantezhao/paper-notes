#### [Bag of Tricks for Efficient Text Classification](https://github.com/dantezhao/paper-notes/blob/master/0004/Bag%20of%20Tricks%20for%20Efficient%20Text%20Classification.pdf)  

#### <阅读笔记>  

```shell  
本文介绍了一款基于线性模型的文本分类神器fastText

带着问题阅读：
1. how: fastText是如何实现高效文本分类的
```  

**0. 摘要**  

```shell  

fastText是一个简单高效的文本分类器

在准确度方面可与深度学习分类器比肩  

在训练和评估方面甚至比深度学习快了多个数量级

具体有多快呢？

使用一个标准的多核CPU，fastText能够

1) 在10分钟内训练10亿个单词的文本  

2) 在一分钟内从31万2000个类型中，为50万个语句分类

```  

<br>

**1. 导言**  
* 简单介绍  

```shell  

1) 文本分类(Text classification)是自然语言处理的重要应用，
常用于网络搜索，信息检索，排序和文件分类  

2) 基于神经网络的文本分类器效果很好，
但是在训练和测试时间上相对较慢，
限制了它们在超大数据集上的应用

3) 线性分类器虽然简单，
但是如果特征应用得当，
可以取得非常好的效果，
在超大文本上也能使用。

```  

* 本文内容  

```shell  

本文探讨如何在超大输出空间的超大文本上实现文本分类

在线性模型使用rank constraint和fast loss approximation

能够实现10分钟训练10亿级的单词量，并取得与前沿技术等同的效果

文章通过标签预测和情感分析这两个任务来评估fastText的效果

```  

**2. 模型设计(Model architecture)**  

* 原有线性模型文本分类器的一些问题  

```shell  

句子分类最简单直接的一种方式：
使用词袋模型(bag of words, BoW)并训练一个线性分类器，如逻辑回归或者支持向量机(SVM)

线性分类器的一个问题是无法在特征和类型之间共享参数

这个问题有可能限制了线性模型在超大输出空间的文本上进行泛化，
因为超大空间中有一些类型样本极少

通常的解决方案有：
a) 将线性模型因式分解为低秩矩阵(low rank matrices)
b) 或者使用多层神经网络

```  

* 秩约束矩阵  

![model_architecture_of_fasttext](https://raw.githubusercontent.com/dantezhao/paper-notes/master/0004/bigablecat_model_architecture_of_fasttext.gif)  

>Figure 1 秩约束矩阵

```shell  

第一权矩阵A可以看做一个查词表

词汇表征被平均到文本表征，文本表征又被用于线性分类器

文本表征是能够被重用的隐含变量

我们使用柔性最大函数f(softmax function f)计算预定义类型的概率分布

对于N个文件的集合，柔性最大函数能将类型的负对数似然估计最小化

```  

* 柔性最大函数softmax fuction  

![softmax function](https://raw.githubusercontent.com/dantezhao/paper-notes/master/0004/bigablecat_softmax_function.gif)  

>x<sub>n</sub>是进行了标准化的第n个文本的特征包(bag of features)  
y<sub>n</sub>表示标签  
A和B代表权重矩阵  
这个模型使用随机梯度下降和线性衰减学习速率在多核CPU上进行异步训练


**3. 实验(Experiments)**  

**4. 探讨和结论**  

```shell  

本文展示了一个文本分类的简单方法

与word2vec所得的非监督训练词向量不同的是，
该方法的词特征可以进一步合成整句特征

fastText最终效果比肩深度学习方法且速度更快

理论上深度神经网络比浅层模型表现力更强，
但是在处理类似情感分析这样的简单文本分类问题时，
深度神经网络并不一定是最佳选择

```  


<br>  
