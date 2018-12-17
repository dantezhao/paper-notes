这篇文章提出的fastText既是一种词向量训练工具又是一种文本分类工具。它同时具备这两项功能是因为它在训练分类器的同时也训练了词向量。我们知道，词向量训练的思路无非就是让模型在做有监督的任务过程中，学习得到词向量的表达。即使Word2Vec号称是利用无标签文本做无监督学习，其实在训练的时候也是在无标签文本上面构建了有监督学习的任务——利用上下文单词预测中间单词（CBOW）。fastText训练词向量也是这个思路，让模型在做有监督的文本分类任务过程中学习词向量。

## fastText模型结构

相比于其它深度的分类模型，fastText模型结构非常浅，非常简单，如下图所示。

![](https://raw.githubusercontent.com/dantezhao/paper-notes/master/0004/lilong_fastText.png)

可以看到，fastText模型只包含输入层、一层隐含层、输出层，总共三层。输入层就是某条文本中的单词序列w1,w2,...wn-1,wn。当然这里不是单词的符号，而是单词的词向量，其实前面还有一层embedding层，可以看做词向量的lookup table，输入单词符号或索引就可以找到对应的词向量。当然训练开始前，lookup table 里面的词向量是随机初始化的。中间的隐层也很简单，并没有多余的参数，就是把所有单词的词向量相加，得到的向量可以看做是这个文本的embedding，然后加一个softmax激活函数。输出层就是文本属于每个类别的概率，这里的类别是事先定义好的，训练文本也是有类别标签的。

## N-gram feature

fastText值得一说的一个改进在于N-gram 特征。fastText采用字符级别的N-gram。以trigram为例，单词<apple>就可以分为<ap，app，ppl，ple，le>。那么在输入层就要做修改，lookup table 里随机初始化的是trigram的向量，由于字符的个数很有限（26个字母加几个特殊字符），所以其实trigram总的数量并不会很多。以单词apple为例，有了trigram向量，就可以将拆分的所有trigram向量SUM起来就得到了apple词向量。用N-gram feature的好处是能让词向量能学习到单词的morphology（词形）信息。比如change和changed其实是一个意思，如果用word2vec训练，它两的词向量可能很不一样，但是用fastText训练它两的词向量就很接近。另外的好处就是有了N-gram向量也能生成未登录词（out-of-vocabulary）的词向量。

## 层次化softmax

层次化softmax是fastText从Word2Vec借鉴过来的，主要作用是降低softmax的复杂度。如果用一般的softmax，由于类别太多，每个类别都要计算概率。采用层次化softmax时，由于Hoffman树特有的性质，只要计算几次可能就能分到正确的类别。

## fastText、Word2Vec、Glove对比

fastText的模型结构跟Word2Vec中的CBOW模型非常像。如下图所示，唯一的差别是CBOW是利用上下文单词预测中间单词，而fastText是根据文本中全部单词预测它的分类。另外就是CBOW没有引入N-gram特征。fastText的训练同时得到了一个文本分类器。

![](https://raw.githubusercontent.com/dantezhao/paper-notes/master/0004/lilong_CBOW.png)

还有一个比较常用的词向量训练工具是Glove。相比于Word2Vec，Glove更加简洁。GLove主要利用单词在文本中的“co-occurance”信息来构建一个加权最小二乘回归模型，通过最优化这个最小二乘回归模型，就得到了每个单词的词向量。构建的最小二乘回归模型如下：

![](https://raw.githubusercontent.com/dantezhao/paper-notes/master/0004/lilong_Glove.png)

其中f(Xij)是权重函数，V是词汇集，wi是单词i的词向量，Xij是单词i和单词j的共现概率。

所以，可以把Word2Vec和fastText是“predictive”模型，而Glove是“count-based”模型。

显然在训练速度上，GLove比Word2Vec和fastText都更快。

一般认为在词向量的性能上CBOW < skip-Gram == Glove < fastText。
