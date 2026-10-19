---
title: 大数据相关
date: 2021-3-31 16:03:39
categories: 春招复习
tags:
- 2021春招
---

# Hadoop
* 大数据解决方案，分布式系统基础架构，主要包括HDFS和MapReduce。

## HDFS
### NameNode
* Master节点
  * 负责管理HDFS中的文件元信息、例如目录树、各个数据节点的可用空间、每个文件的备份情况等等。
    * 目录树
    * 权限信息
    * 记录HDFS中有哪些块
    * 块到DataNode的映射
  * 负责监控各个DataNode的健康状况，如果发现无法连接到DataNode，会将该DataNode移出HDFS并重新备份该节点相关的数据，以满足集群中文件的备份数要求（默认每个文件每块都要有3个副本）。
* 这些信息以两种形式保存在本地磁盘
  * fsimage: HDFS元数据镜像文件，相当于全量备份
  * editlog: HDFS文件改动日志，相当于增量备份（对执行的修改进行备份）

### Secondary NameNode（Hadoop 1.X的思路）
* 定期合并fsimage和editlog并传输给NameNode，以减小editlog的体积。Secondary NameNode为了减轻NameNode的负担会帮其完成这项工作。
* 可以作为备份节点作为NameNode的元数据进行热备份
  * 云玩家方案：定期备份，但是两次备份期间的修改会被丢失
  * 真实方案：的确是定期备份。。。
    * 每隔1小时会进行一次合并，editlog中的事务条数达到某个数字也会进行一次合并。
#### 合并fsimage和editlog的流程
* 生成一个新文件`edits.new`，用于记录合并过程中产生的日志信息
* Secondary NameNode从NameNode上读取edits文件和fsimage文件，并进行合并操作，生成一个`fsimage.ckpt`文件。
* 将生成的合并后的文件发送到NameNode上，让NameNode替换原本的fsimage
* 用`edits.new`成为新的edits文件。

### Standby NameNode + 共享存储(since Hadoop 2.X)
* 只有主NameNode可以对外提供读写服务，StandBy NameNode作为备份。
* ZKFailOverController：检测到NameNode的健康状况，在主NameNode故障时，借助Zookeeper实现自动的主备选举和切换。
* ZK集群：提供选举支持
* 共享存储系统：保证Active NameNode和Standby NameNode能够实现元数据同步。主备切换时新的主NameNode需要确认元数据能够完全同步。

#### QJM(Quorum Journal Manager)
* HDFS的默认共享存储方案
* ActiveNameNode每次写Editlog时都需要向JournalNode集群的每一个JournalNode发送写请求，让每个JournalNode写入相同的内容，只要大多数JournalNode节点返回成功就认为向集群写入EditLog成功。对于2N + 1台JournalNode，最多可以有N台机器挂掉。剩下的N + 1台机器如果能够保持一致，仍然能够构成“大多数”。
* 向JournalNode提交Editlog是同步阻塞的，但是只需要接收到大多数JournalNode的返回就可以了。
* 如果没有收到大多数的Editlog返回，则认为提交EditLog失败了。（如果借鉴Paxos算法的经验，此处应该有一个版本号，提交失败的时候，说明当前NameNode的数据版本可能落后于JournalNode集群，不应该继续对外提供服务了）此时NameNode会停止服务。
* Editlog的分代和分段同步：每个JournalNode只会接收比自身代数大的Editlog同步请求。
  * 增量同步：存在跨度问题，如果本地和最新版本差了不止一代，只添加一个增量也是错误的EditLog
  * 全量同步：每次都传一个最新版本的EditLog，EditLog文件可能会很大。
  * 分段全量同步：fsimage：当前元信息的完整快照。Active NameNode定期完成fsimage和editlog的合并（借助Secondary NameNode），将旧的editlog截断并只同步最新一段的editlog。Standby NameNode也需要进行fsimage和editlog的合并
* 主备切换
  * ZK方式：让NameNode和ZK维持会话，使ZK能够监控NameNode的健康状况。发生主备切换时，多个StandBy NameNode可能会竞争成为Active NameNode，此时让ZK支持主节点的选举（本质是获得一个分布式锁？）。
  * 只有代数高的节点能够对元信息进行写，但是客户端有可能读到旧的信息。HDFS提供fencing机制，当完成主备切换时，使用一定的机制让原本的Active NameNode停止对外服务。默认是通过ssh的方式发送一条命令，杀掉原本的NameNode进程。

#### StandBy NameNode
* Standby NameNode会从JournalNode定期同步editlog，当需要转换为NameNode时需要将落后的Editlog补回来。

### DataNode
* 完成实际的数据存储，并向NameNode定期发送心跳便于双方感知对方存在。HDFS以固定大小的块存储文件内容。文件会被切分为若干块存储到不同的DataNode中，同时会将同一个块写到3个不同的DataNode上。文件切割需要由客户端完成，但是文件块的复制对用户透明。
* DataNode把每个数据块存在单独的文件中。它不在同一个目录创建所有的文件，而是通过试探的方法确定每个目录的最佳文件数目。
* DataNode的状态报告：本地文件对应HDFS数据块的列表，作为报告发送到NameNode。

### HDFS上传文件的流程
1. Client向NameNode请求数据上传，先告知NameNode自己需要上传的元数据信息（包括大小吗）
2. NameNode对上传文件的请求进行检查。例如重名校验和权限校验等。如果检查通过，NameNode会写Editlog，然后修改内存中的元数据信息。
3. NameNode向客户端响应可以上传文件。
4. Client在本地将文件进行分割，分割成固定大小的块。按顺序向NameNode请求上传块。
5. NameNode向Client返回可以上传的节点信息。节点信息是顺序返回的。
6. Client向返回的DataNode发出传输通道建立请求。第一位的DataNode在收到通道建立请求后也会递归地向下一位DataNode发送通道建立请求。之后DataNode递归地应答通道建立请求，收到所有的通道建立应答后，第一为的DataNode会返回信息示意Client可以开始传输文件。
7. Client会将每个Block上传到通道第一位的DataNode上，通道中的DataNode按顺序完成Block的复制。（这里不清楚是同步的还是异步的，但是肯定会有确认）。
   * 想要异步复制，可以设置`dfs.replication.min`，只让最低限度的DataNode先建立通道并完成复制。然后再让NameNode检测到该块未达到复制因子时决定DataNode怎么复制。
8. 重复以上流程，Client将所有分块传输完毕会向NameNode进行报告。

### HDFS下载文件的流程

## MapReduce
* 同样使用Master-Slave架构：JobTracker - TaskTracker。Slave节点向Master节点发送心跳消息以让双方感知对方存在。
* 只有Map和Reduce函数，更高级的计算范式需要用户手动根据Map和Reduce进行构造。
* 所有的计算中间结果都需要保存在HDFS上，计算过程需要反复地读写磁盘。


# Spark
* 主要提供一个全面、统一的框架来管理有不同性质的数据集和数据源的大数据处理需求。
* 更多的计算范式，不局限于Map和Reduce
* 优化的磁盘使用方式：相比于Hadoop MapReduce，尽可能少地读写磁盘。（只有Shuffle的时候将数据完全存放在磁盘？）
* DAG计算模型：根据RDD的依赖关系先计算得到DAG，如果不涉及与其它节点进行数据交换（Shuffle），Spark可以尽可能地在内存中完成这些操作。（减少不了Shuffle，这个是由计算的依赖关系决定的）
* Spark Core：定义RDD的API，操作。
* Spark Streaming: 允许程序像处理普通RDD一样处理实时数据（流数据）
* Spark GraphX：控制图、并行图操作和计算的一组算法和工具的集合。

## Cluster Manager
* Master节点，控制整个集群，监控Worker
## Worker：控制计算节点，启动Executor或者Driver
  * Driver：运行Application的main函数
  * Executor: 为某个Application运行在Worker Node上的一个进程。

## Spark计算模型
### RDD
#### 特点
* 最小的计算单元
* RDD不会将**所有的**中间结果存放在硬盘上，只会在内存不足时将部分中间结果存放在内存。
* 弹性：
  * 存储的弹性：内存和磁盘可以进行自动切换
  * 容错：数据丢失可以自动回复（Linage？）
  * 计算：计算出错可以重试
  * 分片：根据需要重新分片
* 分布式：数据存储在集群的不同节点
* 封装计算逻辑而不是持有数据
* 不可变：逻辑不可改变，如果需要执行新逻辑需要创建新RDD。（转换...）
* 可分区、并行计算。

#### 核心属性
* 分区列表：用于执行任务时进行并行计算
* 分区计算函数：每个分区的数据不同，但是计算逻辑是一样的。计算逻辑对每个分区的数据进行计算
* 依赖关系：依赖的所有RDD
* 分区器：可选、决定对数据进行分区的方式
* 首选位置：确定计算发送给哪个节点。（让效率最优）
  * 移动计算胜过移动数据

#### 创建
1. 从HDFS/与HDFS兼容的其它持久化存储系统如Hive、Canssandra、HBase输入创建
2. 从本地文件创建
3. 从父RDD转换得到新RDD
4. 通过parallelize或makeRDD将单机数据转换为RDD

#### 算子
* Transformation：延迟计算，等到Action操作才会触发计算。