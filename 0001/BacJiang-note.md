论文阅读第一期的文章《Goods：Organizing Google’s Datasets》讲的是关于谷歌在海量元数据管理方面的实践。本篇总结主要从3个方面进行展开：1.什么是元数据；2.如何管理元数据；3.启发与总结
## 1.什么是元数据
元数据被称之为描述数据的数据，记录的是文件的特征，包括数据属性、拥有者、权限、数据块等信息。无论是mysql、oracle这样的关系型数据库，还是Hive、HBase以及图数据库，都需要管理组织元数据，用户才能顺利地获取并使用相关的数据及文件，足以看出元数据管理的重要性。
![元数据的作用](https://img-blog.csdnimg.cn/20181029214753715.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTI5MjI4Mzg=,size_16,color_FFFFFF,t_70)
## 2.如何管理元数据
元数据的组织和管理十分重要，但随着企业的发展，不同的生产系统产生了成千上万甚至几十亿的数据集，如何有效地管理这海量的元数据便成了一个挑战。Google在[Goods](https://github.com/dantezhao/paper-notes/blob/master/0001/Goods%20Organizing%20Google%E2%80%99s%20Datasets.pdf)这篇文章介绍相关理论和实践。LinkedIn也开源了元数据管理系统[WhereHows](https://github.com/linkedin/WhereHows)

 - Goods：Google Dataset Search
 
Google构建了一个*元数据目录*来管理几十亿数据集的元数据，以供工程师们了解Google有哪些数据，哪些数据比较常用（数据排名，类似网页的PageRank）。其结构如下图所示：整个系统分为3层，底层以无侵扰的方式获取不同数据库和文件系统的元数据，中间为存储在Bigtable中形成元数据目录，上层则是提供的各类服务，包括搜索、监控、血缘可视化、增加备注等功能。

比较重要的两个点是：
**1. 元数据的获取是事后型无侵扰的，不会干扰原有业务系统的正常运转；
2. 数据搜素：搜索功能能有效地用户筛选数据并提高利用率，搜素是Google的强项，但文中说数据重要性排名，他们也没处理好还存在一些挑战。**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181029215221881.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTI5MjI4Mzg=,size_16,color_FFFFFF,t_70)
 - WhereHows
 WhereHows是LinkedIn开源的一个元数据管理系统，一直都有在更新维护。WhereHows： *WHERE is the data, and HOW is it produced/consumed.数据在哪里，数据是生成和消费的*。 它也提供后端元数据抽取工具，以及前端的搜素、分析工具。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20181029221903590.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTI5MjI4Mzg=,size_16,color_FFFFFF,t_70)
和Google类似，也是通过一个高可读权限的账户爬取各类文件系统的元数据，组织存储在MySQL中以供上层使用。
### 3.启发和总结
以往认为在构建数据仓库或者数据管理平台的时候，首先是要把所有的汇聚到一起，然后再做上层的各类应用，但往往在实践过程会发现自己有哪些数据，要哪些数据，能做什么。如果能先构建出一个元数据管理系统，既能增加对数据的熟悉度，也能增加数据管理的有效性。
