#### [Wide & Deep Learning for Recommender Systems](https://github.com/dantezhao/paper-notes/blob/master/0003/Wide%20%26%20Deep%20Learning%20for%20Recommender%20Systems.pdf)  

#### <阅读笔记>  

```shell  
本文讲解了广度和深度学习相结合的方法，并以Google Play为例，介绍了该方法在推荐系统实践中的应用

带着问题阅读：
1. what: Wide & Deep Learning的两部分，分别是什么  
2. why: 为什么将二者相结合  
3. how: 广度和深度相结合的具体实现
```  

**0. 摘要**  

```shell  
对已知特征，广度线性模型(wide linear models)有很好的记忆(Memorization)能力

对低维未知特征，深度神经网络(deep netural network)具有很好的泛化(Generalization)能力

本文将广度线性模型的记忆能力和深度神经网络的泛化能力相结合

这套方法被应用于Google Play，10亿活跃用户，1百万apps的手机应用市场

相比仅用广度和仅用深度，广度深度相结合的方法明显改善了app的获得率

广度和深度学习(wide & deep learning)方法的实现已经开源到TensorFlow中
```  

<br>

**1. 导言**  

* 推荐系统也是一种搜索排序系统  

```shell  
推荐系统可以视为一个搜索排序(search ranking)系统

在这个搜索排序系统中

输入查询可以看作用户和上下文信息的集合

输出可以看作物品的排序列表

```  

* 记忆(Memorization)和泛化(Generalization)  

```shell  
推荐系统和搜索排序系统，都要实现记忆和泛化

记忆可以粗略地解释为，学习物品或特征共现的频率以及从历史数据发掘可用的关联关系

泛化则是基于关联关系的可传递性，发掘历史数据中从未出现或极少出现的特征组合

基于记忆的推荐系统更为"主题相关"，直接关联被用户已发生行为所作用的物品

泛化则用于改善推荐物品的多样性

```  

* 线性模型  

```shell  
广义线性模型(generalized linear models)简单、可扩展、可解释

被广泛应用于工业级的大规模在线推荐和排序系统

广义线性模型通常是基于二进制化的稀疏特征训练的

这些特征使用了独热编码(one-hot encoding)

例如:
user_installed_app=netflix值为1
当出现AND(user_installed_app=netflix, impression_app=pandora)时
如果用户安装了netflix，那么这个组合的值为1，推荐系统就会显示pandora给用户

向量叉积变换(cross-product transformation)的局限之一是
无法泛化出训练数据中没有出现过的"查询-物品"特征组合(query-item feature pair)

```  

* 泛化  

```shell  
泛化的一种形式是加入粒度更粗的特征

比如上面的例子
将AND(user_installed_app=netflix, impression_app=pandora)
变为AND(user_installed_category=video, impression_category=music)

通过学习每个查询(query)和每个物品(item)特征的低维密集嵌入向量(low-dimensional dense embedding vecotr)

嵌入式模型(Embedding-based models)，如分解机(factorization machines)或深度神经网络(deep neural networks)
可以泛化出之前没见过的"查询-物品"特征组合

如果潜在的"查询-物品"矩阵过于稀疏或高阶，就很难从这类数据中学习到有效的"查询-商品"低维组合

比如有特殊偏好的用户或者仅有少数拥趸的小众商品

"查询-物品"组合之间没有相关性，而密集嵌入又会为所有的组合泛化出预测结果

这种情况带来了过度泛化和相关性很弱的推荐

而具有向量叉积变换的线性模型，就能记住这些参数极少的异常规则(exception rules)
```  

* 广度和深度学习框架  

![广度和深度模型图谱](https://raw.githubusercontent.com/dantezhao/paper-notes/master/0003/bigablecat_The_spectrum_of_Wide_And_Deep_models.png)  

>Figure 1：广度和深度模型图谱

```shell  
综上所述，本文对一个线性模型模块和一个神经网络模块进行联合训练(joint training)

将记忆和泛化融入一个模型
```  

* 本文的主要贡献  

```shell  
1) 开发了一套广度和深度相结合的学习框架，用于稀疏输入的推荐系统
这套框架对"带嵌入的前馈神经网络"和"特征变换线性模型"进行联合训练(joint trainning)

2) 实现并评测了广度和深度推荐系统在Google Play市场的产品化

3) 以API的形式在TensorFlow上开源了广度和深度学习系统的具体实现代码

广度和深度学习框架的思路简单，在满足训练和服务速度的前提下，显著改善了移动应用市场的app获得率
```  

<br>

**2. 推荐系统概述**  

![Overview_of_the_recommender_system](https://raw.githubusercontent.com/dantezhao/paper-notes/master/0003/bigablecat_Overview_of_the_recommender_system.png)  

>Figure 2 展示了手机应用推荐系统的基本架构

* 查询(Query)和用户行为(User Action)  

```shell  
当用户访问app商店时，查询(Query)就产生了，查询可以包括各类用户和文本特征

推荐系统首先展示一个应用列表(也被称为 展示impression)

用户可以点击或者购买应用列表上的应用

用户的查询(queries)和展示(impressions)被记入日志，作为学习器的训练数据
```  

* 召回和排序(Retrieval and Ranking)  

```shell  
推荐系统有一个服务延迟上限(serving latency requirements)，通常是O(10)毫秒

在时限内为每个查询穷尽百万app计算每个app的分数是不可能的

这种情况下，推荐系统在收到查询的第一步是召回(retrieval)

召回系统为用户返回一个与查询匹配度最佳的简短物品列表

返回这个列表的依据通常是机器学习模型或者人工定义的规则

缩小了备选池(candidate pool)的规模之后，排序(ranking)系统为备选列表中的所有物品排序

分数通常是P(y|X)，代表给定特征x时，用户行为标签y的概率

特征x包括：
a) 用户特征：国籍、语言、人口统计学
b) 文本特征：设备、当前时间
c) 展示特征(impression feature)：app年龄，app历史数据

本文将广度和深度学习框架应用于上述第二个步骤排序系统中

```  

<br>

**3. 广度和深度学习**  

**3.1 广度部分**  

```shell  
广度部分是广义线性模型，如图Figure 1 左侧所示
```  

* 广义线性模型公式  
>y = w<sup>T</sup>x + b  
>
>y表示预测值  
x=[x<sub>1</sub>, x<sub>2</sub>,..., x<sub>d</sub>] 是d个特征的向量  
w=[w<sub>1</sub>, w<sub>2</sub>,..., w<sub>d</sub>] 是模型的参数  
b 是bias(偏差)  

```shell  
特征集包括原始特征输入和变换后的特征

其中最重要的一个变换就是向量叉积变换
```  

* 向量叉积变换(cross product transformation)公式  
![cross product transformation](https://raw.githubusercontent.com/dantezhao/paper-notes/master/0003/bigablecat_cross_product_transformation.gif)  

>其中C<sub>ki</sub>是一个布尔变量  
如果第i个特征属于第k个变换&Oslash;	<sub>k</sub>  
那么C<sub>ki</sub>的值为1，否则为0  

```shell  
对于二进制特征，当且仅当组成特征都为1，它的向量叉积变换才是1，否则向量叉积为0

通过这种方式捕获二进制特征之间的交互，为广义线性模型增加非线性
```  

**3.2 深度部分**  

```shell  
深度部分是图Figure 1右侧的前馈神经网络

分类特征的原始输入是特征字符串(如"language=en")

每一个稀疏高维的分类特征都被转为低维密集的真值向量，通常被称为嵌入向量

嵌入的维度大约在O(10)到O(100)的范围内

嵌入向量先被随机初始化，然后它们的值在模型训练期间被用于最小化最终损失函数

这些低维密集嵌入向量在前馈传递中被送入神经网络的隐藏层
```  

* 隐藏层使用的计算公式  
![wide_deep_model](https://raw.githubusercontent.com/dantezhao/paper-notes/master/0003/bigablecat_hidden_layer_computation.gif)  

>其中l是层数  
f是激活函数，通常是一个线性整流函数ReLUs(rectified linear units)  
a<sup>(l)</sup>是第l层的激活值(activations)  
b<sup>(l)</sup>是第l层的偏差(bias)  
W<sup>(l)</sup>的第l层的模型权重  

**3.3 广度和深度模型的联合训练(joint training)**  

* 联合训练(Joint Training)  

```shell  
广度部分和深度部分相结合时，使用了输出(output)对数几率(log odds)的加权和作为预测 

这个加权和被送入一个通用的逻辑损失函数(logistic loss function)进行联合训练  
```  

* 联合训练(Joint Training)和集成训练(Ensemble)  

```shell  
1) 集成训练中

每个模型都是独立训练，模型产生的预测在推导阶段整合而不是在训练阶段

因为独立训练，模型的规模更大(更多特征和变换)以达到合理的准确性

2) 联合训练中

在训练阶段就同时优化了所有参数，包括广度部分和深度部分以及它们的加权和

广度部分只需少量向量叉积特征变换补充深度部分即可，无需动用整个广度模型

```  

* FTRL算法  

>联合训练中，使用了Follow-the-regularized-leader(FTRL)算法  
模型广度部分使用L<sub>1</sub>范数(regularization)作为优化器  
模型深度部分使用了AdaGrad  
模型图示在图Figure 1的中间部分

* 模型的计算公式  
![wide_deep_model](https://raw.githubusercontent.com/dantezhao/paper-notes/master/0003/bigablecat_wide_deep_model.gif)  

>Y表示二进制分类标签  
&sigma;(·)表示反曲函数  
&Oslash;(X)表示原始特征x的向量叉积变换  
b表示偏差(bias)项  
W<sub>wide</sub>表示所有广度模型权重的向量  
W<sub>deep</sub>表示应用于最终激活值a<sup>(l<sub>f</sub>)</sup>的权重  

<br>  

**4. 推荐系统实现**  

```shell  
实现推荐系统包括三个步骤(如图Figure 3所示)：
1) 数据生成(data generation)

2) 模型训练(model Training)

3) 模型提供服务(model serving)
```  

* 应用推荐系统管线概览(Figure 3)  

![Apps_recommendation_pipeline_overview](https://raw.githubusercontent.com/dantezhao/paper-notes/master/0003/bigablecat_Apps_recommendation_pipeline_overview.png)  

**4.1 数据生成(data generation)**  
* 训练数据(training data)

```shell  
用户和应用在一段时间内的展示数据(impression data)用于生成训练数据

每个样本都对应一个展示(impression)

这个展示的标签就是"应用获取率(app acquisition)"：
如果展示应用被安装，数值为1，否则数值为0
```  

* 词汇数据(vocabularies)  

```shell  
词汇数据实际上是数据表
这些表将分类特征字符串(categorical feature strings)映射为整型ID(integer IDs)

所有出现超过一定次数的字符串特征值，系统都会计算它们的ID空间(ID space)

```  

>通过映射特征值x到它的累计分布函数P(X≤x)  
再分为n<sub>q</sub>个分位数  
连续真值特征会被归一化为[0,1] 
在第i个分位数的值被归一化为 (i-1)/(n<sub>q</sub>-1)  
分位数的边界是在数据生成阶段计算出来的  


**4.2 模型训练(Model Training)**  

![Wide_and_Deep_model_structure.png](https://raw.githubusercontent.com/dantezhao/paper-notes/master/0003/bigablecat_Wide_and_Deep_model_structure.png)  

```shell  
模型结构如图Figure 4所示

1) 输入层(input layer)  
训练阶段，输入层获取训练数据和词汇数据并生成带标签的稀疏或密集特征

2) 广度部分(wide component)  

模型的广度部分包括用户已安装应用和展示应用的向量叉积变换

3) 深度部分(deep component)  

模型的深度部分，从每个分类特征学习得到一个对应的32维嵌入向量

模型将所有嵌入及密集特征合并到一起，生成一个大约1200维的密集向量

合并所得的向量先被传入3个线性整流函数层，最终到达逻辑输出单元

4) 模型训练的效率问题  

广度和深度组合模型在5000亿个样本上训练

每加入一个新的训练数据集，模型都需要重新训练

重新训练计算成本高昂且延迟了从加入新数据到更新模型的时间

5) 热启动系统(warm-starting system)  

为解决重新训练的效率问题，模型使用了一种热启动模式

热启动模式在每次重新训练时都初始化一个新模型

新模型包括了新数据嵌入和之前模型的线性模型权重

6) 试运行(dry run)  

模型加载到模型服务器前，会进行一次试运行(dry run)，以确保线上实时场景不会出问题

上线前通过对比之前的模型，检验新模型运转正常

```  

**4.2 模型提供服务(Model Serving)**  

```shell  
1) 模型训练完成并验证功能后，将加载到模型服务器

2) 模型为推荐系统提供服务的流程如下：

    a) 每一次请求，服务都会通过应用召回系统和用户特征获取一个备选应用列表，并为其中的应用打分

    b) 应用按照分数从高到低排列并展示给用户

3) 分数计算

系统给应用的评分，是通过在广度和深度组合模型上运行一个正向推断(forward inference)计算而来的

4) 提升服务时效

为了将每次请求响应控制在10毫秒内，系统通过多线程并行对性能进行了优化

之前在一个批次内计算所有备选应用的分数

优化后并行计算更小批次里的备选应用的分数

```  

<br>  

**5. 实验结果**  

```shell  
本文主要从app获取率和服务性能两方面验证广度和深度学习模型的效果
```  

**5.1 应用获取率(app aquisitions)**  

```shell  
本文采用A/B测试进行了为期3周的在线实验

分别随机选取了三组各1%的用户作为随机实验对象

三组对象分别应用了三类模型的推荐系统：
1) 高度优化的广度逻辑回归模型，具有丰富的向量叉积特征变换(cross-product feature transformations)
2) 具有相同特征和深度神经网络的深度模型
3) 具有相同特征的深度广度结合模型

其中第一组作为控制组

实验结果表明，在应用获得率(app aquisitions)上：
广度和深度相结合比仅用广度高出3.9%
广度和深度相结合比仅用深度高出1%

除了在线实验，本文使用留出法(hold-out)测试了与线上数据互斥的线下数据集

广度和深度相结合的方法相比仅用广度和仅用深度，只有略微偏高的AUC*值

广度和深度相结合在线上比线下更为显著

原因可能是线下系统的展示(impression)和标签的数据是固定的
线上系统可以混合记忆与泛化生成新的推荐
线上系统也会从新的用户回馈中持续学习

*AUC备注
AUC(Area Under Receiver Operator Characteristic Curve)
ROC全称是"受试者工作特征"(Receiver Operating Characteristic)
ROC曲线的面积就是AUC(Area Under the Curve)
AUC用于衡量"二分类问题"机器学习算法性能(泛化能力)
```  

**5.2 服务性能(serving performance)**  

```shell  
在商用应用商店，需要同时满足高吞吐和低延迟

峰值流量时，推荐系统服务器需要每秒为上百万个应用评分

单线程处理单个批次的全部备选项需要31毫秒

拆分批次为体量更小的批次，同时使用多线程处理，能够将客户端延迟有效降低到14毫秒

```  

<br>  

**6. 相关研究**  

* 分解机(factorization machines)  

```shell  
使用因式分解的方式泛化线性模型，即将2个变量间的关联转成两个低维嵌入向量之间的点积

本文通过学习神经网络嵌入之间的高度非线性关联，替代点积，从而扩展了模型的能力
```  

* 语言模型中的RNN  

```shell  
对
递归神经网络RNNs(Recurrent Neural Networks)和
带语言模型特征的最大熵模型(Maximum entopy models with n-gram features)
进行联合训练(joint training)

学习输入和输出的直接权重，从而显著降低RNN的复杂度
```  

* 计算机视觉  

```shell  
1) 深度残差网络
	a) 降低训练更深度模型的难度
	b) 通过跳跃传递(shortcut connections)来提高准确性

2) 对带图像模型的神经网络进行联合训练(joint training)可以从图片中识别人体姿势
```  

* 推荐系统  

```shell  
1) 内容信息深度学习和协同过滤评分矩阵合二为一，组成协同深度学习(collaborative deep learning)

2) app推荐系统AppJoy，对用户使用记录使用协同过滤
```  

* 本文所用方法  

```shell  
本文对前馈神经网络和线性模型进行联合训练(joint training)
将稀疏特征和输出单元直接关联
用于稀疏输入数据的推荐和排序问题

之前类似研究使用的方法基于协同过滤或基于内容
本文在用户和展示数据(impression data)的基础上训练广度和深度相结合的模型

```  

<br>  

**7. 总结**  

```shell  
记忆和泛化是推荐系统重要的两个方面

广度线性模型借助向量叉积变换，能够有效记忆稀疏特征的关联

深度神经网络可以通过低维嵌入泛化出之前未见的特征关联

广度和深度结合的学习框架(Wide & Deep learning framework)将两种方式的优点

本文将该方法在Google Play产品化，并评估了它的效果

在线实现结果显示，相较于单纯的广度或深度模型，广度和深度结合的模型显著改善了app获得率

```  

<br>  

##### <参考来源>：  
a) [TensorFlow Wide And Deep 模型详解与应用（作者：汪剑）](https://blog.csdn.net/heyc861221/article/details/80131369?_blank)  
b) [TensorFlow Wide And Deep 模型详解与应用（二）（作者：汪剑）](https://blog.csdn.net/heyc861221/article/details/80131373?_blank)  
c) [ROC与AUC的定义与使用详解](https://blog.csdn.net/shenxiaoming77/article/details/72627882?_blank)  
