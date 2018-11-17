---
layout: post
title: The Wisdom of the Few
date: 2018-11-14
categories: blog
tags: [文献阅读]
description: 
---
<script type="text/javascript" async src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML"></script>
# The Wisdom of the Few  
## A Collaborative Filtering Approach Based on expert Opinions from the Web
### 1.Introduction

&emsp;&emsp; 协同过滤算法是现在推荐系统的主流方法，但是协同过滤算法存在着一些问题：数据稀疏，数据噪声，冷启动问题，可扩展性问题。这篇论文，提供了一种新的推荐方法：基于专家观点的推荐系统。这篇论文不希望单纯的探索如何提高推荐系统的准确率，而是希望尝试： 
1. 如何使用较小的数据集预测大量人群；
2. 如果使用独立并且不相关的数据集来产生推荐；
3. 分析专业人士是否是普通用户很好的预测者；
4. 探索新的方法可不可以避开传统协同过滤的缺陷。

### 2.Mining the web for expert ratings

&emsp;&emsp;这篇论文通过爬取烂番茄网站，得到了两组数据，第一组数据是用户对于电影的评价，选取了17,700部电影中的8000部电影，第二组数据是专家对于这8000部电影的评价，这里论文给出了一个threshold，只有评价超过250部电影的专家才被纳入到数据集中，最后1750个专家中有169个被纳入到数据集中。

#### 2.1 Dateset analysis: User and Experts

&emsp;&emsp;将专家数据和普通用户数据集进行比较我们可以发现，普通用户数据集要比专家数据集存在着很大的不同：
1. 普通用户数据更稀疏：(sparsity coefficient:the user data 0.01,the expert set 0.07)；
2. 专家对于所有电影都存在打分，没有了普通用户只对那些流行的电影打分的偏倚；
3. 专家似乎在一些好的电影上得出了相当一致的结论，他们比普通的用户更容易得出同样的结论。

### 3.Expert nearest-neighbors
&emsp;&emsp;传统的CF使用KNN算法来预测用户的排名，度量距离的方法是余弦相似度。这篇论文提供了一种新的思路，即，寻找和用户相似度大于δ的专家，用来预测用户对于特定物品的排名。
### 4.Results
&emsp;&emsp;最终的结果是新方法在准确度和覆盖率上都超过了传统方法的协同过滤系统。
### 5.User Study
### 6.Discussion

&emsp;&emsp;基于专家数据的协同过滤算法有效的解决了以下传统算法的缺点：
1. 数据稀疏：普通用户数据要比专家数据更稀疏，所以解决数据稀疏的问题。
2. 噪声和恶意打分：专家在评分的时候，更加一致和稳定，因此可以避免这样的问题。
3. 冷启动问题：在传统的CF系统中，新的物品缺少评价数据因此不会被推荐，但是这个领域的专家会主动给新的物品打分，这样就可以解决冷启动问题。
4. 可扩展性：基于专家数据的系统计算量小，因此可扩展性好。
5. 隐私问题

