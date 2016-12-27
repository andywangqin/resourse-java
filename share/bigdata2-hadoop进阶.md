
###2、HDFS
---
####2.1、数据块
HDFS（hadoop distributed filesystem）是一个综合性的文件系统抽象，可以集成在本地文件系统，amazon s3等系统。与文件系统类似，文件系统块大小一般为几千字节（磁盘块一般为512字节），而hdfs块默认为64M，hdfs上的文件也被划分为块大小的多个分块（chunk），作为独立的存储单元。与文件系统不同的是，hdfs上中的小于一个块大小的文件不会占据整个块空间。
```
hadoop fsck / -files -blocks
```

####2.2、namenode和tasknode
namenode管理文件系统的命名空间，它维护着文件系统数及整棵树内所有的文件和目录。这些信息以两个文件形式永久保存在本地磁盘上：命名空间镜像文件和编辑日志文件。namenode也记录着每个文件中各个块所在的数据节点信息，但它并不永久保存块的位置信息，因为这些信息会在系统启动时由数据节点重建。

####2.3、命令行接口
>fs.default.name：hdfs://localhost/
>dfa:replication：3
```
hadoop fs -help(或hdfs dfs)

hadoop fs -copyFromLocal

hadoop fs -copyToLocal

hadoop fs -mkdir

```

####2.4、hadoop文件系统
Hadoop有一个抽象的文件系统概念，HDFS只是其中的一个实现。Java抽象类org.apache.hadoop.fs.FileSystem定义了Hasoop中的一个文件系统接口，并且该抽象类有几个具体的实现：

####2.5、Java接口



####2.6、数据流


###3、MapReduce
---
Hadoop Map/Reduce是一个使用简易的软件框架，基于它写出来的应用程序能够运行在由上千个商用机器组成的大型集群上，并以一种可靠容错的方式并行处理上T级别的数据集。一个Map/Reduce 作业（job） 通常会把输入的数据集切分为若干独立的数据块，由 map任务（task）以完全并行的方式处理它们。框架会对map的输出先进行排序，然后把结果输入给reduce任务。通常作业的输入和输出都会被存储在文件系统中。 整个框架负责任务的调度和监控，以及重新执行已经失败的任务。

通常，Map/Reduce框架和分布式文件系统是运行在一组相同的节点上的，也就是说，计算节点和存储节点通常在一起。这种配置允许框架在那些已经存好数据的节点上高效地调度任务，这可以使整个集群的网络带宽被非常高效地利用。Map/Reduce框架由一个单独的master JobTracker 和每个集群节点一个slave TaskTracker共同组成。master负责调度构成一个作业的所有任务，这些任务分布在不同的slave上，master监控它们的执行，重新执行已经失败的任务。而slave仅负责执行由master指派的任务。

应用程序至少应该指明输入/输出的位置（路径），并通过实现合适的接口或抽象类提供map和reduce函数。再加上其他作业的参数，就构成了作业配置（job configuration）。然后，Hadoop的job client提交作业（jar包/可执行程序等）和配置信息给JobTracker，后者负责分发这些软件和配置信息给slave、调度任务并监控它们的执行，同时提供状态和诊断信息给job-client。

虽然Hadoop框架是用JavaTM实现的，但Map/Reduce应用程序则不一定要用 Java来写 。

- Hadoop Streaming是一种运行作业的实用工具，它允许用户创建和运行任何可执行程序 （例如：Shell工具）来做为mapper和reducer。
- Hadoop Pipes是一个与SWIG兼容的C++ API （没有基于JNITM技术），它也可用于实现Map/Reduce应用程序。

####2.1、输入与输出
Map/Reduce框架运转在<key, value>键值对上，也就是说， 框架把作业的输入看为是一组<key, value> 键值对，同样也产出一组<key, value>键值对做为作业的输出，这两组键值对的类型可能不同。框架需要对key和value的类(classes)进行序列化操作，因此，这些类需要实现 Writable接口。另外，为了方便框架执行排序操作，key类必须实现WritableComparable接口。

一个Map/Reduce 作业的输入和输出类型如下所示：

(input) <k1, v1> -> map -> <k2, v2> -> combine -> <k2, v2> -> reduce -> <k3, v3> (output)

####2.2、combiner
每次map运行之后，会对输出按照key进行排序，然后把输出传递给本地的combiner（按照作业的配置与Reducer一样），进行*本地*聚合。用户可选择通过JobConf.setCombinerClass(Class)指定一个combiner，它负责对中间过程的输出进行本地的聚集，这会有助于降低从Mapper到Reducer数据传输量。

####2.3、Mapper
Map是一类将输入记录集转换为中间格式记录集的独立任务。 这种转换的中间格式记录集不需要与输入记录集的类型一致。一个给定的输入键值对可以映射成0个或多个输出键值对。

Hadoop Map/Reduce框架为每一个InputSplit产生一个map任务，而每个InputSplit是由该作业的InputFormat产生的。概括地说，对Mapper的实现者需要重写JobConfigurable.configure(JobConf)方法，这个方法需要传递一个JobConf参数，目的是完成Mapper的初始化工作。然后，框架为这个任务的InputSplit中每个键值对调用一次 map(WritableComparable, Writable, OutputCollector, Reporter)操作。应用程序可以通过重写Closeable.close()方法来执行相应的清理工作。

Mapper的输出被排序后，就被划分给每个Reducer。分块的总数目和一个作业的reduce任务的数目是一样的。用户可以通过实现自定义的 Partitioner来控制哪个key被分配给哪个 Reducer。

*需要多少个Map？*

Map的数目通常是由输入数据的大小决定的，一般就是所有输入文件的总块（block）数。

Map正常的并行规模大致是每个节点（node）大约10到100个map，对于CPU 消耗较小的map任务可以设到300个左右。由于每个任务初始化需要一定的时间，因此，比较合理的情况是map执行的时间至少超过1分钟。

这样，如果你输入10TB的数据，每个块（block）的大小是128MB，你将需要大约82,000个map来完成任务，除非使用 setNumMapTasks(int)（注意：这里仅仅是对框架进行了一个提示(hint)，实际决定因素见这里）将这个数值设置得更高。

####2.4、Reducer
Reducer将与一个key关联的一组中间数值集归约（reduce）为一个更小的数值集。

用户可以通过 JobConf.setNumReduceTasks(int)设定一个作业中reduce任务的数目。概括地说，对Reducer的实现者需要重写 JobConfigurable.configure(JobConf)方法，这个方法需要传递一个JobConf参数，目的是完成Reducer的初始化工作。然后，框架为成组的输入数据中的每个<key, (list of values)>对调用一次 reduce(WritableComparable, Iterator, OutputCollector, Reporter)方法。之后，应用程序可以通过重写Closeable.close()来执行相应的清理工作。

Reducer有3个主要阶段：shuffle、sort和reduce。

- Shuffle
Reducer的输入就是Mapper已经排好序的输出。在这个阶段，框架通过HTTP为每个Reducer获得所有Mapper输出中与之相关的分块。
- Sort
这个阶段，框架将按照key的值对Reducer的输入进行分组（因为不同mapper的输出中可能会有相同的key）。
Shuffle和Sort两个阶段是同时进行的；map的输出也是一边被取回一边被合并的。
- Secondary Sort
如果需要中间过程对key的分组规则和reduce前对key的分组规则不同，那么可以通过JobConf.setOutputValueGroupingComparator(Class)来指定一个Comparator。再加上 JobConf.setOutputKeyComparatorClass(Class)可用于控制中间过程的key如何被分组，所以结合两者可以实现按值的二次排序。
- Reduce
在这个阶段，框架为已分组的输入数据中的每个<key, (list of values)>对调用一次reduce(WritableComparable, Iterator, OutputCollector, Reporter)方法。

*需要多少个Reduce？*
Reduce的数目建议是0.95或1.75乘以 (<no. of nodes> * mapred.tasktracker.reduce.tasks.maximum)。

用0.95，所有reduce可以在maps一完成时就立刻启动，开始传输map的输出结果。用1.75，速度快的节点可以在完成第一轮reduce任务后，可以开始第二轮，这样可以得到比较好的负载均衡的效果。

增加reduce的数目会增加整个框架的开销，但可以改善负载均衡，降低由于执行失败带来的负面影响。

上述比例因子比整体数目稍小一些是为了给框架中的推测性任务（speculative-tasks） 或失败的任务预留一些reduce的资源。

####2.5、Partitioner
Partitioner用于划分键值空间（key space）。

####2.6、Reporter
Reporter是用于Map/Reduce应用程序报告进度，设定应用级别的状态消息， 更新Counters（计数器）的机制。

####2.7、OutputCollector

####2.8、作业配置
JobConf代表一个Map/Reduce作业的配置。

####2.9、任务的执行和环境

####2.10、作业的提交与监控
JobClient是用户提交的作业与JobTracker交互的主要接口。

####2.11、作业的输入

####2.12、作业的输出

####2.13、其他有用的特性

###4、Hadoop周边
---
####4.1、HBase
源自Google的Bigtable论文，发表于2006年11月，HBase是Google Bigtable克隆版

####4.1、Sqoop
Sqoop是SQL-to-Hadoop的缩写，主要用于传统数据库和Hadoop之前传输数据。数据的导入和导出本质上是Mapreduce程序，充分利用了MR的并行化和容错性。

Sqoop利用数据库技术描述数据架构，用于在关系数据库、数据仓库和Hadoop之间转移数据。

####4.3、Presto
很多朋友可能非常熟悉SQL数据库构建与SQL查询编写工作。这方面专业知识在大数据领域同样适用。Presto是一套开源SQL查询引擎，允许数据科学家运用SQL查询来查询数据库，包括从Hive到专有商业数据库等各类数据库系统不限。像Facebook这类巨头级企业都在利用其进行交互查询，因此我们基本可以将Presto视为一套理想的大规模数据集交互式查询工具。

####4.4、Hive
Hive与Impala将数据库引入Hadoop,大数据世界中，还需要数据库对应结构化数据部分。如果大家希望为Hadoop数据平台加入一些秩序管理，那么Hive则是最佳选项。这是一款基础性结构工具，允许大家在非SQL Hadoop当中执行SQL类操作。Hive适合于长时间的批处理查询分析。

不想用程序语言开发MapReduce的朋友比如DB们，熟悉SQL的朋友可以使用Hive开离线的进行数据处理与分析工作。 
Hive是基于Hadoop的一个数据仓库工具，可以将结构化的数据文件映射为一张数据库表，并提供简单的sql查询功能，可以将sql语句转换为MapReduce任务进行运行。 
注意Hive现在适合在离线下进行数据的操作，就是说不适合在挂在真实的生产环境中进行实时的在线查询或操作，因为一个字“慢”。 
Hive起源于FaceBook，在Hadoop中扮演数据仓库的角色。建立在Hadoop集群的最顶层，对存储在Hadoop群上的数据提供类SQL的接口进行操作。你可以用 HiveQL进行select、join，等等操作。 
如果你有数据仓库的需求并且你擅长写SQL并且不想写MapReduce jobs就可以用Hive代替。

####4.5、ZooKeeper
源自Google的Chubby论文，发表于2006年11月，Zookeeper是Chubby克隆版

解决分布式环境下的数据管理问题：统一命名，状态同步，集群管理，配置同步等。

Hadoop的许多组件依赖于Zookeeper，它运行在计算机集群上面，用于管理Hadoop操作。

####4.6、Pig

###5、Spark
截至目前，我们已经探讨了数据的存储与整理。但是，我们该如何对数据进行实际操作呢?这时候我们就需要一套像Spark一样的分析与处理引擎。Spark也是属于Apache的一个项目，与其他大数据平台不同的特点，主要如下：

Spark也是Apache基金会的开源项目，它由加州大学伯克利分校的实验室开发，是另外一种重要的分布式计算系统。它在Hadoop的基础上进行了一些架构上的改良。Spark与Hadoop最大的不同点在于，Hadoop使用硬盘来存储数据，而Spark使用内存来存储数据，因此Spark可以提供超过Hadoop100倍的运算速度。但是，由于内存断电后会丢失数据，Spark不能用于处理需要长期保存的数据。

###6、Storm
Storm是Twitter主推的分布式计算系统，它由BackType团队开发，是Apache基金会的孵化项目。它在Hadoop的基础上提供了实时运算的特性，可以实时的处理大数据流。不同于Hadoop和Spark，Storm不进行数据的收集和存储工作，它直接通过网络实时的接受数据并且实时的处理数据，然后直接通过网络实时的传回结果。

###7、kafka
Kafka是由LinkedIn开发的一个分布式的消息系统，使用Scala编写，它以可水平扩展和高吞吐率而被广泛使用。目前越来越多的开源分布式处理系统如Cloudera、Apache Storm、Spark都支持与Kafka集成

###9、大数据架构
---

####9.1、酷狗音乐的大数据平台重构
#####9.1.1、问题分析

- 重构难点: 大头的行为流水数据迁移到新平台稳定运行
- 建设周期涉及的生态链包括：数据采集、接入，清洗、存储计算、数据挖掘，可视化等环节，每个环节都可以当做一个复杂的系统来建设
- 原有的大数据平台架构，如下图：

![](image/111.png)

主要基于Hadoop1.x+hive做离线计算(T+1)，基于大数据平台的数据采集、数据接入、数据清洗、作业调度、平台监控几个环节存在的一些问题来列举下。

>数据采集：

1. 数据收集接口众多，且数据格式混乱，基本每个业务都有自己的上报接口
2. 存在较大的重复开发成本
3. 不能汇总上报，消耗客户端资源，以及网络流量
4. 每个接口收集数据项和格式不统一，加大后期数据统计分析难度
5. 各个接口实现质量并不高，存在被刷，泄密等风险

>数据接入:

1. 通过rsync同步文件，很难满足实时流计算的需求
2. 接入数据出现异常后，很难排查及定位问题，需要很高的人力成本排查
3. 业务系统数据通过Kettle每天全量同步到数据中心，同步时间长，导致依赖的作业经常会有延时现象

>数据清洗：

1. ETL集中在作业计算前进行处理
2. 存在重复清洗

>作业调度：

1. 大部分作业通过crontab调度，作业多了后不利于管理
2. 经常出现作业调度冲突

>平台监控：

1. 只有硬件与操作系统级监控
2. 数据平台方面的监控等于空白
3. 基于以上问题，结合在大数据中，数据的时效性越高，数据越有价值(如：实时个性化推荐系统，RTB系统，实时预警系统等)的理念，因此，开始大重构数据平台架构。

#####9.1.2、新一代大数据技术架构
新一代大数据技术架构的数据流架构图：

![](image/114.png)
从这张图中，可以了解到大数据处理过程可以分为数据源、数据接入、数据清洗、数据缓存、存储计算、数据服务、数据消费等环节，每个环节都有具有高可用性、可扩展性等特性，都为下一个节点更好的服务打下基础。整个数据流过程都被数据质量监控系统监控，数据异常自动预警、告警。

新一代大数据整体技术架构如图：

![](image/115.png)
将大数据计算分为实时计算与离线计算，在整个集群中，奔着能实时计算的，一定走实时计算流处理，通过实时计算流来提高数据的时效性及数据价值，同时减轻集群的资源使用率集中现象。

整体架构从下往上解释下每层的作用：

>数据实时采集：

主要用于数据源采集服务，从数据流架构图中，可以知道，数据源分为前端日志，服务端日志，业务系统数据。下面讲解数据是怎么采集接入的。

a.前端日志采集接入：

前端日志采集要求实时，可靠性，高可用性等特性。技术选型时，对开源的数据采集工具flume,scribe,chukwa测试对比，发现基本满足不了我们的业务场景需求。所以，选择基于kafka开发一套数据采集网关，来完成数据采集需求。数据采集网关的开发过程中走了一些弯路，最后采用nginx+lua开发，基于lua实现了kafka生产者协议。有兴趣同学可以去Github上看看，另一同事实现的，现在在github上比较活跃，被一些互联网公司应用于线上环境了。

b.后端日志采集接入：

FileCollect,考虑到很多线上环境的环境变量不能改动，为减少侵入式，目前是采用Go语言实现文件采集，年后也准备重构这块。

前端，服务端的数据采集整体架构如下图：

![](image/116.png)

c.业务数据接入

利用Canal通过MySQL的binlog机制实时同步业务增量数据。

数据统一接入：为了后面数据流环节的处理规范，所有的数据接入数据中心，必须通过数据采集网关转换统一上报给Kafka集群，避免后端多种接入方式的处理问题。

数据实时清洗(ETL)：为了减轻存储计算集群的资源压力及数据可重用性角度考虑，把数据解压、解密、转义，部分简单的补全，异常数据处理等工作前移到数据流中处理，为后面环节的数据重用打下扎实的基础(实时计算与离线计算)。

数据缓存重用：为了避免大量数据流(400+亿条/天)写入HDFS，导致HDFS客户端不稳定现象及数据实时性考虑，把经过数据实时清洗后的数据重新写入Kafka并保留一定周期，离线计算(批处理)通过KG-Camus拉到HDFS(通过作业调度系统配置相应的作业计划)，实时计算基于Storm/JStorm直接从Kafka消费，有很完美的解决方案storm-kafka组件。

离线计算(批处理)：通过spark，spark SQL实现，整体性能比hive提高5—10倍，hive脚本都在转换为Spark/Spark SQL；部分复杂的作业还是通过Hive/Spark的方式实现。在离线计算中大部分公司都会涉及到数据仓库的问题，酷狗音乐也不例外，也有数据仓库的概念，只是我们在做存储分层设计时弱化了数据仓库概念。数据存储分层模型如下图：

![](image/117.png)

大数据平台数据存储模型分为：数据缓冲层Data Cache Layer（DCL）、数据明细层Data Detail Layer（DDL）、公共数据层（Common）、数据汇总层Data Summary Layer（DSL）、数据应用层Data Application Layer（DAL）、数据分析层（Analysis）、临时提数层（Temp）。

1）数据缓冲层(DCL)：存储业务系统或者客户端上报的，经过解码、清洗、转换后的原始数据，为数据过滤做准备。

2)数据明细层（DDL）：存储接口缓冲层数据经过过滤后的明细数据。

3）公共数据层（Common）：主要存储维表数据与外部业务系统数据。

4）数据汇总层（DSL）：存储对明细数据，按业务主题，与公共数据层数据进行管理后的用户行为主题数据、用户行为宽表数据、轻量汇总数据等。为数据应用层统计计算提供基础数据。数据汇总层的数据永久保存在集群中。

5）数据应用层（DAL）：存储运营分析（Operations Analysis ）、指标体系（Metrics System）、线上服务（Online Service）与用户分析（User Analysis）等。需要对外输出的数据都存储在这一层。主要基于热数据部分对外提供服务，通过一定周期的数据还需要到DSL层装载查询。

6）数据分析层（Analysis）：存储对数据明细层、公共数据层、数据汇总层关联后经过算法计算的、为推荐、广告、榜单等数据挖掘需求提供中间结果的数据。

7）临时提数层（Temp）：存储临时提数、数据质量校验等生产的临时数据。

实时计算：基于Storm/JStorm，Drools,Esper。主要应用于实时监控系统、APM、数据实时清洗平台、实时DAU统计等。

HBase/MySQL：用于实时计算，离线计算结果存储服务。

Redis：用于中间计算结果存储或字典数据等。

Elasticsearch：用于明细数据实时查询及HBase的二级索引存储(这块目前在数据中心还没有大规模使用，有兴趣的同学可以加入我们一起玩ES)。

Druid：目前用于支持大数据集的快速即席查询(ad-hoc)。

数据平台监控系统：数据平台监控系统包括基础平台监控系统与数据质量监控系统，数据平台监控系统分为2大方向，宏观层面和微观层面。宏观角度的理解就是进程级别,拓扑结构级别,拿Hadoop举例，如：DataNode，NameNode，JournalNode，ResourceManager，NodeManager，主要就是这5大组件，通过分析这些节点上的监控数据，一般你能够定位到慢节点，可能某台机器的网络出问题了，或者说某台机器执行的时间总是大于正常机器等等这样类似的问题。刚刚说的另一个监控方向，就是微观层面，就是细粒度化的监控，基于user用户级别，基于单个job，单个task级别的监控，像这类监控指标就是另一大方向，这类的监控指标在实际的使用场景中特别重要，一旦你的集群资源是开放给外面的用户使用，用户本身不了解你的这套机制原理，很容易会乱申请资源，造成严重拖垮集群整体运作效率的事情，所以这类监控的指标就是为了防止这样的事情发生。目前我们主要实现了宏观层面的监控。如：数据质量监控系统实现方案如下。

![](image/118.png)

#####9.1.3、大数据平台重构过程中踩过的坑

我们在大数据平台重构过程中踩过的坑，大致可以分为操作系统、架构设计、开源组件三类，下面主要列举些比较典型的，花时间比较长的问题。

1、操作系统级的坑

Hadoop的I/O性能很大程度上依赖于Linux本地文件系统的读写性能。Linux中有多种文件系统可供选择，比如ext3和ext4，不同的文件系统性能有一定的差别。我们主要想利用ext4文件系统的特性，由于之前的操作系统都是CentOS5.9不支持ext4文件格式，所以考虑操作系统升级为CentOS6.3版本，部署Hadoop集群后，作业一启动，就出现CPU内核过高的问题。如下图：

![](image/119.png)

#####9.1.4、后续持续改进

####9.2、Airbnb的大数据平台架构

####9.3、主流大数据采集平台的架构图解

####9.4、大数据架构的典型方法和方式

####9.5、后Hadoop时代的大数据架构

####9.6、bigdataarch.pdf


参考:

- [大数据技术Hadoop入门理论系列之一----hadoop生态圈介绍](http://www.thebigdata.cn/Hadoop/28830.html)