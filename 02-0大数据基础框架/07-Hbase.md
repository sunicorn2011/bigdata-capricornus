[toc]


***
# HBase简介
## 1、HBase定义
- HBase是一种分布式、可扩展、支持海量数据存储的NoSQL数据库
## 1.1 HBase逻辑结构
- 逻辑结构图![image](222ADC63E50246878FCAD27D3564E2C4)
- 
## 1.2 HBase物理存储结构
- 物理结构：![image](158FED0BA3B349BDA267E007617A029A)
-
## 1.3 数据模型
- 1.3.1 Name Space：类似关系数据库中的database，自带两个命名空间：hbase和default，hbase存放Hbase的内置表
- 1.3.2 Table：类似关系数据库中的`表`概念。在定义时候，只需要声明列族即可。往HBase写入数据时，字段可以`动态`、`按需`指定
- 1.3.3 Row：HBase表中的每行数据都由一个RowKey和多个Column（列）组成，数据是按照RowKey的字典顺序存储的，并且查询数据时只能根据RowKey进行检索，所以RowKey的设计十分重要。
- 1.3.4 Column: HBase中的每个列都由Column Family(列族)和Column Qualifier（列限定符）进行限定。建表时，只需指明列族，而列限定符无需预先定义
- 1.3.5 Time Stamp：用于标识数据的不同版本（version）
- 1.3.6 Cell：由`{rowkey, column Family：column Qualifier, time Stamp}` 唯一确定的单元。cell中的数据全部是字节码形式存贮。
- 
## 1.4 HBase基本架构
- 架构图：![image](D309BE4F6F944611BACE9AFD00CE0639)
- Hbase架构角色：
    - 1.4.1 Region Server: Region Server为 Region的管理者，其实现类为HRegionServer，主要作用如下
        - 1、对于数据的操作：get, put, delete
        - 2、对于Region的操作：splitRegion（`切分`）、compactRegion（`压缩`）
        - 
    - 1.4.2 Master: Master是所有Region Server的管理者，其实现类为HMaster,主要作用
        - 1、对于表的操作：create, delete, alter
        - 2、对于RegionServer的操作：分配regions到每个RegionServer，`监控`每个RegionServer的状态，`负载均衡`和`故障转移`。
        - 
    - 1.4.3 Zookeeper: HBase通过Zookeeper来做master的高可用、RegionServer的监控、元数据的入口以及集群配置的维护等工作。
    - 1.4.4 HDFS: HDFS为Hbase提供最终的底层数据存储服务，同时为HBase提供高可用的支持
    - 

***
# 2、HBase进阶
## 2.1 RegionServer 架构
- 架构图 ![image](D8935BA95DF646E696B738DDF51F67FF)
- 
## 2.2 架构角色
- 2.2.1 StoreFile
    - 保存实际数据的物理文件，StoreFile以Hfile的形式存储在HDFS上。每个Store会有一个或多个StoreFile（HFile），数据在每个StoreFile中都是有序的
- 2.2.2 MemStore
    - 写缓存，由于HFile中的数据要求是有序的，所以数据是先存储在MemStore中，排好序后，等到达刷写时机才会刷写到HFile，每次刷写都会形成一个新的HFile。 
- 2.2.3 WAL
    - 由于数据要经MemStore排序后才能刷写到HFile，但把数据保存在内存中会有很高的概率导致数据丢失，为了解决这个问题，数据会先写在一个叫做Write-Ahead logfile的文件中，然后再写入MemStore中。所以在系统出现故障的时候，数据可以通过这个日志文件重建。 
- 2.2.4 BlockCache
    - 读缓存，每次查询出的数据会缓存在BlockCache中，方便下次查询。
    - 
## 2.3 Hbase写流程
- Hbase写流程图 ![image](865B895F5F1847D4AD3B9E837CF6DD02) 
- 
- Hbase具体写流程：
    - 1、Client先访问zookeeper，获取hbase:meta表位于哪个Region Server。
    - 2、访问对应的Region Server，获取hbase:meta表，根据读请求的namespace:table/rowkey，查询出目标数据位于哪个Region Server中的哪个Region中。并将该table的region信息以及meta表的位置信息缓存在客户端的meta cache，方便下次访问。
    - 3、与目标Region Server进行通讯
    - 4、将数据顺序写入（追加）到WAL
    - 5、将数据写入对应的MemStore，数据会在MemStore进行排序
    - 6、向客户端发送ack
    - 7、等达到MemStore的刷写时机后，将数据刷写到HFile。
    - 
## 2.4 MemStore Flush（`刷写`）
- 刷写流程图：![image](719A3E439F494A0DB5EA32C017641135)
- 2.4.1 刷写流程
    - 1）当`memstore`的大小出发一定条件时候，其所在的region的所有memstore就会触发刷写，刷写时，会阻止继续往memstore写入数据：
        - 1、`hbase.hregion.memstore.flush.size（默认值128M）`
        - 2、` hbase.hregion.memstore.block.multiplier（默认值4）`
    - 2）当`region server`中memstore的总大小达到一定的阈值，也会触发region按照其所有的memstore的大小顺序（`从大到小`）依次刷写，直到`region server`中所有的Memstore的总大小到达满足下面两个阈值：
        - 1、`hbase.regionserver.global.memstore.size（默认值0.4）` 
        - 2、`hbase.regionserver.global.memstore.size.lower.limit（默认值0.95）`
        - 
    - 3）触发自动刷写的时间，也会触发memstore flush。`参数配置：`
        - `hbase.regionserver.optionalcacheflushinterval（默认1小时）`
        - 
    - 4）当WAL文件的数量超过 `hbase.regionserver.max.logs`，region会按照时间顺序依次进行刷写
    - 
## 2.5 读流程
- 读流程图一：![image](041A507B7A1C46F192AD19A3D18E3BEA)
- 读流程细节二（`Merge细节`）![image](91BEDBADB0684EA89718EE495D1CF6A5)
- 读流程文字描述：
    - 1、Client先访问zookeeper，获取hbase:meta表位于哪个Region Server。
    - 2、访问对应的`Region Server `，获取hbase:meta表，根据读请求的namespace:table/rowkey，查询出目标数据位于哪个Region Server中的哪个Region中。并将该table的region信息以及meta表的位置信息缓存在客户端的meta cache，方便下次访问
    - 3、与目标Region Server进行通讯
    - 4、分别在MemStore和Store File（HFile）中查询目标数据，并将查到的所有数据进行合并。此处所有数据是指同一条数据的不同版本（time stamp）或者不同的类型（Put/Delete）
    - 5、将查询到的新的数据块（Block，HFile数据存储单元，默认大小为64KB）缓存到Block Cache
    - 6、将合并后的最终结果返回给客户端
    - 
## 2.6  StoreFile Compaction（`HFile（memstore）合并`）
-
-
-
##

***
#
##
##
##
##
##
##
##
##











