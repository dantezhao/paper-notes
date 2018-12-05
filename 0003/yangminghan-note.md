---
layout: post
title: Wide & Deep Learning for Recommender Systems
date: 2018-11-17
categories: blog
tags: [文献阅读]
description: 
---
<script type="text/javascript" async src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML"></script>
# Wide & Deep Learning for Recommender Systems

1. what(是什么): Wide & Deep Learning是谷歌提出的一个推荐系统算法，该算法结合广义线性模型和深度学习模型的优点。
2. why (为什么): 广义线性模型需要较为复杂的特征工程，但是在处理稀疏数据时效果较好，深度学习模型可以生成以前没有推荐，并且不需要复杂的特征工程，该模型希望规避两者的缺点，同时兼顾两者的优点。  
3. how (怎么做): 文章给出了使用TensorFlow实现的方法。

### Abstract
&emsp;&emsp;广义线性模型在大型的回归和分类问题中有着很广泛的应用，其优势在于可以“记忆”(memorization)用户的喜好，但是，该模型需要花费很多的时间在特征工程上。深度神经网络可以通过low-dense embeddings生成以前没有的特征组合，并且不需要花费很长的时间在特征工程上。然而，深度神经网络在数据比较稀疏并且比较集中的时候，会发生过拟合，生成一些相关性比较小的推荐。在这篇文章中，作者提出了一种新的模型 **Wide and Deep Learning**希望可以结合两者的优点，既可以“记忆”用户的喜好，推荐给他们最相关的产品，又可以生成新的推荐。

### 1.Introduction
&emsp;&emsp;一个推荐系统可以看做一个搜索排序系统，输入的是查询是一个用户与文本信息的集合，输出是一个物品的排名。推荐系统是在数据库中选择相关的物品(items)，然后把这些物品根据特定的目标(such as clicks or purchases)进行一个排名。
&emsp;&emsp;推荐系统的一个挑战就是如何能同事做到“记忆”(memorization)和“生成”(generalization)。“记忆”可以简单的理解为学习物品或者特征在历史数据中同时出现的频率和相关性。“生成”则是关注相关的可推广性和探索以前没有过的，新的物品和特征组合。  
&emsp;&emsp;“记忆”在推荐系统的体现为根据用户已经有的行为(更多的基于历史数据)，推荐同一个主题下的物品或者直接与之相关的物品。“生成”则表现为提高推荐物品的多样性。  
&emsp;&emsp;这篇论文，我们将注意力放在Google的App Store的推荐系统上，但是该模型具有较强的泛化能力。这篇文章的主要关注点：   
- The Wide & Deep learning framework for jointly training feed-forward neural networks with embeddings and linear model with feature transformations for generic recommender systems with sparse inputs.   
- The implementation and evaluation of the Wide & Deep recommender system productionized on Google Play, a mobile app store with over one billion active users and over one million apps. 

### 2.Recommender System Overview
&emsp;&emsp;用户发送给推荐系统一个查询请求，推荐系统返回给用户一个推荐的list。通常情况下，在数据库中有着大量的app，所以在这种时候，很难在短时间内给每一个查询的app都有一个精确的评分，所以推荐系统的第一步通常是retrieval，这个步骤类似于粗排，返回一个和查询最匹配的较短的物品的清单。得到这个清单的方式一般是使用机器学习模型和规则进行匹配。然后，再根据这个粗排的list，继续精排，这个分数由ranking system继续给出。那这篇文章提出的W&D模型主要是做精排。  
![Overview_of_the_recommender_system](https://raw.githubusercontent.com/dantezhao/paper-notes/master/0003/bigablecat_Overview_of_the_recommender_system.png)
### 3.Wide & Deep Learning
#### 3.1 The Wide Component
#### cross-product transformation  
&emsp;&emsp;借着看论文的机会，我回去好好复习了一下线性代数的相关概念，线性代数中cross-product翻译成叉乘，表示两个向量叉乘后得到一个新的向量，该向量与原来两个向量张成的平面垂直，长度等于两个向量所围成的平行四边形的面积(行列式的值)。但是在这篇论文中做这项变换的是下面的公式，这个有待进一步对线性代数进行研究。

![cross product transformation](https://raw.githubusercontent.com/dantezhao/paper-notes/master/0003/bigablecat_cross_product_transformation.gif)  
#### 3.2 The Deep Component
&emsp;&emsp;深度这一块，原始模型用的激活函数是Relu。  

![wide_deep_model](https://raw.githubusercontent.com/dantezhao/paper-notes/master/0003/bigablecat_hidden_layer_computation.gif)  
#### 3.3 Joint Traing of Wide & Deep Model
&emsp;&emsp;Joint training和ensemble的区别：在ensemble中，每一个独立的模型在训练的时候是相互独立的，他们的结果是在最后做出推断的时候才被合并起来，而不是在训练的时候进行合并。与ensemble相反，joint training在训练的时候就将wide，deep两部分和这两部分所占的比重同时进行训练。在模型的规模方面，ensemble的每一个独立的模型规模一般都大一点，而W&D的模型规模比较小，这意味着较少规模的叉乘特征转换。

![广度和深度模型图谱](https://raw.githubusercontent.com/dantezhao/paper-notes/master/0003/bigablecat_The_spectrum_of_Wide_And_Deep_models.png)
&emsp;&emsp;最后整个W&D的模型的整个模型结构，经过我们讨论，认为可以把Wide部分和Deep部分都看成是特征工程，Wide部分使用的方式是cross product transform，Deep部分使用的embedding。
![Wide_and_Deep_model_structure.png](https://raw.githubusercontent.com/dantezhao/paper-notes/master/0003/bigablecat_Wide_and_Deep_model_structure.png)  


### Embedding  
&emsp;&emsp;Embedding这一项的操作是将分类变量映射到多维空间，这个我最困惑的问题是映射到多维空间的初始值是怎么设定的？随机设定，就和神经网络的参数一开始随机设定一样，再由BP算法对这些值进行更新就可以了。
 
[Google AI Blog: Wide & Deep Learning: Better Together with TensorFlow ](https://ai.googleblog.com/2016/06/wide-deep-learning-better-together-with.html)  

[3Blue1Brown: Cross products in the light of linear transformations | Essence of linear algebra chapter 11](https://www.youtube.com/watch?v=BaM7OCEm3G0)
