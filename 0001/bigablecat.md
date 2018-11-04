#### [Goods: Organizing Google’s Datasets](https://github.com/dantezhao/paper-notes/blob/master/0001/Goods%20Organizing%20Google%E2%80%99s%20Datasets.pdf)  
##### <阅读笔记>  
```shell  
Google Dataset Search (Goods)是一个以元数据的形式管理数据集的企业内部系统
这篇论文介绍了Goods的设计和用途

带着问题阅读：
1. what: Goods系统是做什么的  
2. why: 为什么需要Goods系统  
3. how: Goods系统是如何设计的
```  
 
**0. 摘要**  

**0.1 文章目标**  
* 探讨实现数据管理所面临的技术挑战 (Challenges)  
```shell  
从亿万级数据源爬取和推断元数据

维持大规模元数据目录的一致性

使用元数据为用户提供服务
```  
* 对于打造大规模企业级数据管理系统的借鉴意义  

**1. 引言**  

**1.1 企业数据管理的两种形式**  
* Enterprise Data Management (EDM)  
```shell  
生成数据集和管理数据集都在一套系统内

系统本身限制了数据的生产和流转
```  
* Data Lake  
```shell  
与EDM不同的是，Data Lake采取事后(post-hoc)模式  

不介入数据的生产和使用

只为已经生成的数据提供一套有效的管理工具

所以这是一种事后(post-hoc)行为

lake的比喻很形象，数据不断生成和累积汇聚成湖，查找数据就像从湖中钓鱼("fish the right data")  
```  

**1.2 Google Dataset Search (Goods)**  
* Goods是一个静默的服务者
```shell  
Goods的全称是Google Dataset Search，但它的功能不限于search

Goods采用了类似Data Lake的形式，对数据进行事后(post-hoc)管理

post-hoc将在文中多次出现，强调Goods是静默的辅助工具

作者还用非侵扰(non-intrusive)表明Goods既不影响数据本身，也不影响数据的生产者和使用者
```  

**1.3 Goods的运作机制**  
![Goods设计概览](https://raw.githubusercontent.com/dantezhao/paper-notes/master/0001/Goods_design.png)  
>Goods设计概览  

* 分层介绍  
```shell  
1) Goods持续地爬取(crawls)各类存储系统和业务线，
从中发现数据集(datasets)并收集数据集的元数据(datasets)信息和使用情况(usage)
(见图最底层)  

2) Goods将元数据聚合为一个中央目录(central catalog)，同时把特定数据集的元数据与其他数据集的信息相互关联
(见图中间层)  

3) Goods用这套目录的信息构建搜索、监控和数据流可视化等工具，
从而为Google工程师提供数据管理服务
(见图最上层)
```  

* 关于元数据(metadata)  
```shell  
1) 元数据信息的一个来源是直接从数据源收集

2) 元数据还可以来源于对数据内容的推断(infer)：
    a. 处理附加的数据源，如日志、数据集所有者和项目信息等
    c. 分析数据集的内容
    d. 收集Goods使用者输入

3) Goods收集的元数据可能包括：
   a. 数据所有者 (owners)
   b. 访问时间 (time of access)
   c. 内容特征 (content features)
   d. 生产管线的访问记录  (accesses by production pipelines)
```  

**1.4 Goods提供的一些工具**  
* dashboard 仪表板  
```shell  
展示所有数据集  

提供多条件检索  

可以横跨不同的存储系统  

可以获得数据集和所有依赖的数据集
```  

* monitor 监控  
```shell  
监控内容特征：如大小、数值的分布、是否可用  

在特征发生意外改变时通知数据所有者
```  

* provenance 溯源  
```shell  
生成某数据集的上游数据
  
依赖某数据集的下游数据  
```  

* search engine 搜索引擎  
```shell  
缩小搜索范围
```  

* Goods API  
```shell  
通过暴露接口，所有的团队都能为元数据目录做贡献，形成一种crowd-sourced(众包)的方式

数据使用者能够共享和交换数据集的信息
```  

**2. Goods系统的技术挑战**  

**2.1 天量数据规模**  
```shell  
Goods系统为超过260亿个数据集建立了索引

260亿这个数字还只包括了对全体google员工开放权限的数据集

如果Goods为更高级别权限的数据集建立索引，同时支持更多存储系统，规模可能在260亿的基础上翻倍

即使每秒处理一个数据集，260亿也需要一千台并行的机器运转300天

对元数据的推断(infer)也因为计算量呈指数级增长而完全不可行
```  

**2.2 多样性**  
```shell  
数据集以各种格式存储，来自各式存储系统，于是多样性成了数据统一管理的技术障碍

Goods需要将各种格式的数据以同一种形式展示给用户


```  