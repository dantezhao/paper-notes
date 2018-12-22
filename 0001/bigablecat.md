#### [&lt;Goods: Organizing Google’s Datasets&gt;](https://github.com/dantezhao/paper-notes/blob/master/0001/Goods%20Organizing%20Google%E2%80%99s%20Datasets.pdf)  
##### 阅读笔记  

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
>图：Goods设计概览  

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

* provenance 数据血缘关系  

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

260亿这个数字仅包括对全体google员工开放权限的数据集

如果Goods为更高级别权限的数据集建立索引，同时支持更多存储系统，规模可能在260亿的基础上翻倍

即使每秒处理一个数据集，260亿也需要一千台并行的机器运转300天

对元数据的推断(infer)也因为计算量呈指数级增长而完全不可行
```  

**2.2 多样性**  

```shell  
数据集以各种格式存储，来自各式存储系统，多样性成为数据统一管理问题之一

Goods需要将各种格式的数据以同一种形式展示给用户

数据集的类型和大小不同或者元数据的类型不同，抽离元数据的成本也会不同

Goods系统在推断元数据的时候，需要考量元数据的成本和收益

多样性也体现在数据集的相互关系中，而数据集的相互关系反过来影响元数据在Goods目录的计算和存储
```  

**2.3 目录条目(Catalog Entries)的变动**  

```shell  
Goods目录条目中，每天约有5%左右的数据集被删除

优先计算哪些数据集的元数据，将哪些数据集加入目录，都会受到新旧数据集交替的影响

生存周期time-to-live(TTL)长的数据集相对比较重要

短周期数据集可能关联长周期数据集，所以Goods系统不可能排除短周期数据集
```  

**2.4 元数据的不确定性**  

```shell  
Goods系统采取事后(post-hoc)非侵入(non-invasive)的方式，无法全程把握数据集的产生

Goods根据已有数据推断和计算出的元数据，在一定程度并不精准
```  

**2.5 计算数据集重要性**  

```shell  
推算数据集对使用者的重要性和在网页检索中推算网页的重要性不同

网页用于排序和检索的许多特性（如锚文本anchor text），数据集并不具备

不过数据集可以提供结构性的上下文，这是网页所没有的

数据集之间的唯一联系就是来源关联，而这种关联并不能完全决定数据集的重要性

为哪些数据集优先计算元数据，也可以作为数据集重要性的一项参考
```  

**2.6 复原数据集语义**  

```shell  
数据集的语义可以通俗地理解为数据集的某项内容代表了什么

文中举的例子是，从一个数据集中推断出一批整型数值代表了地理坐标的ID

这样的数据集语义，在用户检索地理数据时就能排上用场

将无意义的原始数据升华为包含意义的内容对推断元数据非常有用

但是发现数据集语义本身也是一项非常困难的工作
```  

**3. Goods系统的目录(Catalog)**  

* 综述  
 
```  
Google内部每个存储系统都可能维护着自己的catalog，每个catalog还会有自己的metadata

数据在不同的数据集和存储系统之间自由流转是常态

Goods所做的就是为所有存储系统和数据集建立统一的目录

原则上Goods为每个爬取的数据集建立条目(entry)

而出现一些特征相似度高的集群时，Goods会将它们归并为一个集群(cluster)，建立单个条目

集群的一个典型例子就是相同数据集的不同版本
```  

**3.1 元数据(Metadata)**  

* 概要  

```shell  
终于讲到了Goods系统的关键要素————元数据(metadata)

Goods系统在爬取数据集的时候，会顺带获取一些元数据，如数据集的大小、所有者、访问权限等

但是数据集并没有保存所有的元数据信息，比如生成数据集的作业(jobs)，数据集的访问者等

不能从数据集直接获取的元数据往往存在于日志中

综上所述，Goods除了爬取获得元数据外，还会通过推断(inference)获取元数据
```  

![Metadata_Table](https://raw.githubusercontent.com/dantezhao/paper-notes/master/0001/bigablecat_Metadata.png)  
>Table2: 元数据(Metadata)和元数据组(Metadata Group)  

* 基础元数据(Basic Metata)  

```shell  
包括时间戳、文件格式、所有者、访问权限等

基础元数据一般由Goods系统爬取存储系统直接获得，无需推断

Goods的其他模块通常将基础元数据作为行为依据之一
```  

* 数据血缘/数据谱系(Provenance)  

```shell  
Goods中的元数据血缘关系来自数据集的生产和消费过程、数据集上下游依赖

Goods通过分析生产日志来确定元数据的血缘信息

Goods将时间信息用于决定依赖关系，即晚发生的依赖于早发生的

在计算血缘关系时，Goods为了效率可能会牺牲信息的完整性
```  

* 结构信息(Schema)  

```shell  
Google内几乎所有结构化数据集都是基于serialized protocol buffer编码的

推断数据集使用了哪种形式的protocol buffer编码会产生多个结论

Goods系统把所有可能的protocol buffer形式都记录在元数据中
```  

* Content summary  

```shell  
Content summary按照字面直译为内容摘要反而不容易理解

实际上，元数据记录的content summary可以看成一套数据的关键字合集

文中举了三个summary的例子：
a) 抽样(sampling)产生的frequent token
b) 分析字段(fields)得出的键(key for data)
c) 有校验和(checksums)的fingerprints

Goods通过summary来判断来自不同数据集的内容或者字段是否相似和相等，
```  

* 用户注释(User-provided annotations)  

```shell  
一般用户做注释都是为了明确告知数据集的使用者有必要知晓的信息

Goods的元数据通过分析注释来优化排序或者规避数据隐私
```  

* 语义学信息(Semantics)  

```shell  
数据集的语义学信息可以帮助理解数据集

如果数据集使用了特定的protocol buffer，Goods可以分析源码提取有用的备注(comment)信息

本段中举了个例子，比如数据集中一个名为"mpn"的字段，通过分析备注，发现mpn是"//Model Product Number"的缩写
这就是获取备注(comment)信息在语义学方面的作用

Google的知识图谱可以作为一个资源库，Goods系统将数据集内容与知识图谱匹配，识别不同字段中包含什么样的条目信息(如位置信息，业务信息)
```  


* 其他  

```shell  
除上述类型的元数据外，Goods系统还会将以下信息作为元数据的内容：
a) 获取一个标识，通过该标识可以确认拥有数据集的团队(team)
b) 数据集所属项目的描述
c) 数据集元数据的变更历史

此外，Goods允许团队在目录添加自定义的元数据，从而为所有使用者提供统一管理元数据的平台
```  

**3.2 数据集群(cluster)**  

* 数据集重复问题  

```shell  
Goods目录上的260亿个数据集并非完全独立，很多数据集存在内容重复，比如：

a) 相同数据集的不同版本  

b) 相同数据集复制到不同的数据中心  

c) 大数据集被切分为小数据集  

...   
```  

* 数据集集群化的好处  

```shell  

针对上述问题，将类似数据集归纳为集群有如下好处：

a) 为用户提供合乎逻辑的数据集分组

b) 只需计算集群中的少量数据集的元数据，节约计算成本


```  

* 集群化的技术考量  

```shell  
将数据集集群化的计算成本要足够低，才值得使用

生成集群的计算成本不能高于重复计算相似数据集的成本

```  

* 根据路径(path)层级(hierarchies)生成集群  

```shell  
数据集的路径可以提供划分集群的思路  

某个数据集路径  
dataset/2015-10-10/daily_scan

按天分类  
dataset/2015-10-<day>/daily_scan

按月份和日期分类  
dataset/2015-<month>-<day>/daily_scan  
```  

* 多粒度的半网格结构(granularity semi-lattice)  

![Metadata_Table](https://raw.githubusercontent.com/dantezhao/paper-notes/master/0001/bigablecat_semi_lattice.png)  
>Figure 2展示了两种层次的集群划分：按天划分和按版本号划分  

![Metadata_Table](https://raw.githubusercontent.com/dantezhao/paper-notes/master/0001/bigablecat_path_dimensions.png)  
>Table 3展示了构建集群所依赖的数据集维度  

```shell  
从数据集的路径抽离出不同维度(dimensions，如日期、版本号)
为每个数据集构建一个半网格(semi-lattice)结构  

在Figure 2所示的半网格结构中，非叶子节点代表了数据集的分组依据  
```  

* 集群的日常更新和重复映射问题  

```shell  
如果分组情况每天计算和更新，可能造成用户每天都看到不同的集群  

为了解决这个问题，Goods只为每个半网格的顶层元素创建条目(entry)  

如Figure 2，目录将只有顶层的条目/dataset/<date>/<version>，
对应网格底层的三个数据集  

采用这种方式可以保证每个数据集只映射到一个集群，
总的集群条目数量也会下降
```  

* 集群的元数据  

![Metadata_Table](https://raw.githubusercontent.com/dantezhao/paper-notes/master/0001/bigablecat_propagation_metadata.png)  
> Figure 4 将3个数据集的元数据汇总成集群元数据  

```shell  
Goods系统通过汇总集群中各个数据集的元数据，生成该集群的元数据  

集群的元数据以实时计算的方式产生，用于区分通过分析推断得出的元数据  
```  

* 数据集在集群中的分布  

![Metadata_Table](https://raw.githubusercontent.com/dantezhao/paper-notes/master/0001/bigablecat_distribution_cluster.png)  
> Figure 3 用柱状图的方式展示了数据集在集群中的分布情况  

```shell  
集群化可以将"物理"集群压缩为"逻辑"集群

极大地降低了元数据的计算成本  

用户可以通过集群更方便地查阅Goods系统的目录  
```  

<br>  

**4 后端实现(Backend Implementation)**  

```shell  
本节主要讨论：

Goods系统目录(catalog)的物理结构  

向目录中不断增加新模块的方法  

目录数据的一致性  

目录的容错机制
```  

**4.1 目录存储(Catalog storage)**  

* Bigtable的行存储  

```shell  
a) 目录使用Bigtable作为存储中介

Bigtable是一种可伸缩的，键值对存储系统

在目录中，Bigtable中的一行代表一个数据集或一个集群

Bigtable提供了单行事务一致性(per-row transactional consistency)  

数据集路径或者集群路径作为这一行的键  

通过这种方式，数据集对应单行，无需查找多余信息

b) Goods系统中与单行单个数据集处理方式相左的特性

数据集提取半网格结构时，将多行信息整合到逻辑数据集中

累计同个集群中不同数据集的元数据  

不过这种元数据累加并不需要强一致性
```  

* Bigtable的列族(column families)  

```shell  
一张Bigtable表格包含多个独立的列族  

Goods的Bigtable中还设置了一些独立列族专门进行批量处理(高压缩率，非内存驻留)  

只能通过批处理任务访问的数据就存在这些列族中  

比如Goods系统中最大的列族，就包含了用于计算血缘图谱的原始血缘数据

这些血缘数据内容不直接面向前端，仅为部分集群提供服务，因而可以高度压缩
```  

* 元数据在Bigtable中的存储  

```shell  
在目录的Bigtable中，每行存储两种元数据信息： 

a) 数据集的元数据 

b) 状态元数据(status metadata)：对某个数据集处理后生成的结果

```  

* 状态元数据(status metadata)  

```sehll  
状态元数据列出了用于处理条目的(entry)每一个模块

状态元数据可能包含时间戳，成功状态，错误信息等内容

Goods使用状态元数据协调模块的执行

状态元数据也可以检测系统，比如通过成功状态观察数据集，收集出现次数最多的错误码

与Bigtable的临时数据模型配合，状态元数据还能用于调试(debugging)  

通过配置Bigtable，保留若干代的状态元数据

这些代际数据能够揭示模块的历史表现
```  

**8 结论和展望(conclusions and future work)**  

* 总结  

```shell  
本文介绍的数据管理系统，可访问企业内部数以十亿计的数据集的元数据
```  

* 挑战  

```shell  
1) 对数据集全排列，鉴别重要数据集，数据集的排序方法是否和网页排序方法类似

2) 完善元数据需要依赖各种资源和信息

3) 理解隐含在数据集中的语义信息能为用户提供更有效的搜索服务，
而对语义的理解也需要借助其他资源比如Google的知识图谱(Knowledge Graph)

4) 数据集产生的时候就在Goods上注册能够保持目录更新，
而这种方式需要修改现有的存储系统，与Goods非侵入式的理念冲突，
如何结合非侵入式和侵入式仍然有待解决

```  

* 期望  

```shell  
希望类似Goods的数据管理系统能够助推数据导向型公司培养一种数据文化

企业将数据当作企业核心资产，发展出类似"代码规约"的"数据规约(data discipline)"

```  