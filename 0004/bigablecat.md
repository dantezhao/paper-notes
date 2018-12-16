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

<br>

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

![model_architecture_of_fasttext](https://raw.githubusercontent.com/dantezhao/paper-notes/master/0004/bigablecat_model_architecture_of_fasttext.png)  

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

>x<sub>n</sub>是第n个文本的标准化特征包(normalized bag of features)  

>y<sub>n</sub>表示标签(label)  

>A和B代表权重矩阵(weight matrices)  

>这个模型使用随机梯度下降(stochastic gradient descent)和线性衰减学习速率(linearly decaying learning rate)在多核CPU上进行异步训练  

**2.1 层次softmax函数(Hierarchical softmax)**  

* 改善计算复杂度  

```shell  

当类型的数量非常庞大时，线性分类器的计算代价太高

为了降低运行时间，模型引入了：
基于霍夫曼编码树(Huffman coding tree)的层次softmax函数(Hierarchical softmax)

```  

* 训练(training)：  

>1) 普通线性分类器：  
O(kh)，k是类型数量，h是文本表征的维度  

>2) 层次softmax函数：  
O(hlog<sub>2</sub>(k))  

* 测试(test):  

>1) 普通线性分类器：  
O(kh)，k是类型数量，h是文本表征的维度  

>2) 层次softmax函数：  
O(hlog<sub>2</sub>(k))  

>3) 在层次softmax函数基础上引入二叉堆(binary heap)计算T-top节点：  
O(log(T))  

* 对测试(test)时节点概率的进一步解释  

![node_probability](https://raw.githubusercontent.com/dantezhao/paper-notes/master/0004/bigablecat_node_probability.gif)  

>上面的公式用于表示与节点相关的概率  

>(l + 1) 表示节点深度(depth)  

>n<sub>1</sub>,...,n<sub>l</sub> 表示节点的所有父节点  

>每个节点都与根节点到该节点路径的概率相关  

```shell  

每个节点的概率总是小于其父节点的概率

搜索树时直接从最大概率的叶子节点开始查找

舍弃小概率的叶子节点  

上述操作降低了测试的计算复杂度  

```  

**2.2 N-gram分词特征(N-gram features)**  

```shell  
词袋(bag of words)的词序相对不变

计算时直接使用原词序计算成本高

使用一个包含n-grams分词的词袋作为捕捉词序的附加特征
能够达到与直接使用词序近似的效果

我们使用哈希(hashing trick)来维护一个快速且高效存储的n-grams映射

使用二元分词(bigrams)只需10M的bins存储空间，否则需要100M的空间

```  

<br>

**3. 实验(Experiments)**  

```shell  
本文使用两项任务来衡量fastText

1) 与现有的文本分类器在情感分析问题上比对

2) 在具有大量输出空间的标签预测数据集上测试大数据计算能力

经实践测试，本文使用的模型在两项对比中至少快出2到5倍

```  

**3.1 情感分析(Sentiment analysis)**  

```shell  
情感分析进行了两组实验  

第一组实验将fastText和另外6个模型在8个数据集上进行对比

第二组实验将fastText和另外4个模型在4个数据集上进行对比
```  


* 第一组实验(Table 1)  

```shell  

fastText, h=10  

fastText, h=10, bigram  

其他模型：  

1) BoW  

2) n-grams  

3) TF-IDF(term frequency–inverse document frequency)  

4) char-CNN，字符级卷积神经网络(character level convolutional model)  

5) char-CRNN，字符级循环卷积神经网络(character based convolution recurrent network)  

6) VDCNN，超深卷积神经网络(very deep convolutional network)  

```  

* 第二组实验(Table 3)  

```shell  

fastText

模型：

1) SVM+TF  

2) CNN  

3) Conv-GRNN  

4) LSTM-GRNN  

```  

* 结果  

```shell  

Table 1：

fastText使用10个隐藏层(hidden units)运行5个回合(epochs)  

学习率(learning rate)从验证集{0.05, 0.1, 0.25, 0.5}中获取

添加二元分词(bigram)信息可以将整体表现提升1~4%

整体来看fastText准确率比char-CNN和char-CRNN稍高，略差于VDCNN

使用更多分词可以略微提高准确率，如三元分词(trigrams)可将Sogou数据集上的准确率提升至97.1%


Table 3：

fastText比Table 3中其他几种方法都好

调整验证集中的超参(hyper-parameters)发现n-grams为5时模型效果最好

fastText没有使用预训练的词嵌入(pre-trained word embeddings)可能是1%准确率差异的原因

```  

* 训练时间(training time)  

```shell  
char-CNN和VDCNN都在NVIDIA tESLA K40 GPU上进行训练

fastText在20个线程的CPU上训练


Table 2显示使用卷积的方法比fastText慢了若干个数量级

如果char-CNN使用更新的CUDA卷积实现，速度能够加快10倍

fastText在一分钟内就能在同样的数据集上完成训练

GRNN方法在单线程的CPU上一个回合(epoch)耗费12个小时

相比于神经网络方法，fastText随着数据量增大至少增速15000倍

```  

**3.1 情感分析(Sentiment analysis)**  

```shell  


```  


<br>

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


##### <参考资料>：  

a) [神经网络训练中，Epoch、Batch Size和迭代傻傻分不清](https://cloud.tencent.com/developer/article/1117838)  
b) [Training Hidden Units with Back Propagation](https://web.stanford.edu/group/pdplab/pdphandbook/handbookch6.html)  
