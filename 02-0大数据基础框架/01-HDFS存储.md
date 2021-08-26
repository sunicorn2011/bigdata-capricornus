[toc]

# HDFS概述
## HDFS定义
- 定义：hdfs是文件存储系统，通过目录树来定位，其次是分布式
- 适用场景：
    - 适合一次写入，多次读出，且不支持文件修改。
    - hdfs只能够是单线程写入
## HDFS组织架构
- 架构图
    - ![image](1CCE1DF858CD47078BCCBB3FBE7B291A)
## 架构图角色
- NameNode（NN）
    - 系统Master角色
    - 作用
        - 管理hdfs的命名空间
        - 配置副本策略
        - 管理数据库（Block映射信息）
        - 处理客户端请求
- DataNode（DN）
    - 数据存储的实际执行者
    - 作用
        - 存储实际的数据块
        - 执行数据的读写操作
- Client端
    - 作用
        - 文件切片
        - 与NN交互，获取文件存储位置
        - 与DN交互，进行数据的读写操作
        - 提供一些命令来访问更新hdfs。格式化、增删改查等
- Secondary NameNode
    - 非NN的热备角色。当NN挂掉，他并不能马上提供服务
    - 作用：
        - 分担NN的工作，比如定期的合并fsimage和edit编辑日志，生成最新的fsimage推送给NN
        - 紧急情况下，可以辅助回复NameNode
        - 
## HDFS文件块大小
- hdfs物理分块（block）大小是128M，默认通过dfs.blocksize来配置
- 代码中实现
```
    max(minSize, min(maxSize, default_size(128M)))
```
- HDFS中块大小的主要设置取决于磁盘传输效率
    - 原理：
        - 1、如果数据寻址时间为10ms，即查找目标块（block）的时间10ms
        - 2、原则上寻址时间为传输时间的 `1%`为最佳状态，则传输时间为 ` 10ms / 1% = 1000ms`
        - 3、而目前磁盘的普遍传输效率是 100MB/s
        - 4、所以Block大小 = 1s * 100MB/s = 100MB
## HDFS优缺点
- 优点：
    - 高容错性
        - 数据有多个副本
    - 适合处理大数据
        - 数据规模和文件规模 
    - 可构建在廉价机器上面，通过多副本机制，提高可靠性
- `缺点`：
    - 不适合低延迟访问
    - 无法高效的对大量小文件进行存储
        - 存储大量小文件的话，会占用NameNode大量的内存来存储文件目录和块信息，这样不可取
        - 小文件存储的寻址时间会超过读取时间，他违反了HDFS的设计目标(`寻址时间是读取时间的 1% 最优`)
        - 
    - 不支持并发写入和文件的随机修改
        - 一个文件只能有一个写，不允许多线程写入
        - 仅支持数据append，不支持文件随机修改
# HDFS数据流（写入/读取）
## 写入
- 数据流图
    - ![image](96C7BC1427FA40C6A2456E4D1642BACE) 
- 写入流程:
    - 客户端通过Distributed FileSystem模块向NameNode请求上传文件，NameNode检查目标文件是否已存在，父目录是否存在。
    - NameNode返回是否可以上传。
    - 客户端请求第一个 Block上传到哪几个DataNode服务器上。
    - NameNode返回3个DataNode节点，分别为dn1、dn2、dn3。
    - 客户端通过FSDataOutputStream模块请求dn1上传数据，dn1收到请求会继续调用dn2，然后dn2调用dn3，将这个通信管道建立完成
    - dn1、dn2、dn3逐级应答客户端。
    - 客户端开始往dn1上传第一个Block（先从磁盘读取数据放到一个本地内存缓存），以Packet为单位，dn1收到一个Packet就会传给dn2，dn2传给dn3；dn1每传一个packet会放入一个应答队列等待应答。
        - `重点`：此处会生成两个消息队列：packet消息队列和ACK消息队列
    - 当一个Block传输完成之后，客户端再次请求NameNode上传第二个Block的服务器。（重复执行3-7步）。
        - `参考源码`：
        ```
        org.apache.hadoop.hdfs.DFSOutputStream
        
        ```
- 网络拓扑-节点距离计算
    - 在HDFS写数据的过程中，NameNode会选择距离待上传数据最近距离的DataNode接收数据。
    - 计算选择的原则：
        - 原则：遵循`机架感知（副本存储节点选择） `
        - `节点距离：两个节点到达最近的共同祖先的距离总和。` 
    - `副本节点选择原则`
        - 图示![image](315508D7B39C48EA919763B2E4AE7FA5)
        - 选择如下：
            - 第一个副本在Client所处的节点上，如果客户端在集群外，则随机一个
            - 第二个副本在另外一个随机机架上
            - 第三个副本在第二个机架所在的随机节点
## 读取
- 数据流图
    - ![image](CC0AA535B0EE442CA941723A076A8107)
- 读取流程:
    - 客户端通过DistributedFileSystem向NameNode请求下载文件，NameNode通过查询元数据，找到文件块所在的DataNode地址。
    - 挑选一台DataNode（就近原则，然后随机）服务器，请求读取数据。
    - DataNode开始传输数据给客户端（从磁盘里面读取数据输入流，以Packet为单位来做校验）。
    - 客户端以Packet为单位接收，先在本地缓存，然后写入目标文件。
    - 
# NameNode和SecondaryNameNode关系
## NN和2NN工作机制
    - 原理：
        - 引入一个新的节点SecondaryNamenode，专门用于FsImage和Edits的合并。 
## NameNode的工作原理
- 结构图 
    ![image](6964A8A916804581B5BCED498D85141B)
- 流程：
        - 第一阶段：NameNode启动
            - 第一次启动NameNode格式化后，创建Fsimage和Edits文件。如果不是第一次启动，直接加载编辑日志和镜像文件到内存。
            - 客户端对元数据进行增删改的请求。
            - NameNode记录操作日志，更新滚动日志。
            - NameNode在内存中对元数据进行增删改。
            - 
        - 第二阶段：Secondary NameNode工作
            - Secondary NameNode询问NameNode是否需要CheckPoint。直接带回NameNode是否检查结果。
            - 如果需要，则Secondary NameNode请求执行CheckPoint。
            - NameNode滚动正在写的Edits日志。即生成新的Edit日志
            - 将滚动前的编辑日志和镜像文件拷贝到Secondary NameNode。
            - Secondary NameNode加载编辑日志和镜像文件到内存，并合并。
            - 生成新的镜像文件fsimage.chkpoint。
            - 拷贝fsimage.chkpoint到NameNode。
            - NameNode将fsimage.chkpoint重新命名成fsimage。
## CheckPoint时间设置
- 通常情况下，SecondaryNameNode每隔一小时执行一次
        - 文件：`hdfs-default.xml` 
        ```
        <property>
          <name>dfs.namenode.checkpoint.period</name>
          <value>3600s</value>
        </property>
        ```
    - 一分钟检查一次操作次数，当操作次数达到1百万时，SecondaryNameNode执行一次
        - 文件: 
        ```
        <property>
          <name>dfs.namenode.checkpoint.txns</name>
          <value>1000000</value>
        <description>操作动作次数</description>
        </property>
        
        <property>
          <name>dfs.namenode.checkpoint.check.period</name>
          <value>60s</value>
        <description> 1分钟检查一次操作次数</description>
        </property >
        ```
-
## NameNode故障处理（扩展）
- 方法
    - 方法一：将SecondaryNameNode中数据拷贝到NameNode存储数据的目录
        - kill -9 NameNode进程
        - 删除NameNode存储的数据（/opt/module/hadoop-3.1.3/data/tmp/dfs/name）
        - 拷贝SecondaryNameNode中数据到原NameNode存储数据目录
        - 重新启动NameNode
    - 方法二：
        - 使用-importCheckpoint选项启动NameNode守护进程，从而将SecondaryNameNode中数据拷贝到NameNode目录中。
    - 
# 集群安全模式
- 结构图
![image](DB8F4BEE378C4639A1D257EBC8ACF5F0)
- 基本语法
```
（1）bin/hdfs dfsadmin -safemode get	（功能描述：查看安全模式状态）
（2）bin/hdfs dfsadmin -safemode enter  （功能描述：进入安全模式状态）
（3）bin/hdfs dfsadmin -safemode leave	（功能描述：离开安全模式状态）
（4）bin/hdfs dfsadmin -safemode wait	（功能描述：等待安全模式状态）
```
# DataNode
## DataNode工作机制
- 流程图
- ![image](A9A1AE6EFE334E99B4D3AA94DA82FF81)
- DataNode工作具体流程：
    - 一个数据块在DataNode上以文件形式存储在磁盘上，包括两个文件，一个是数据本身，一个是元数据包括数据块的长度，块数据的校验和，以及时间戳。
    - DataNode启动后向NameNode注册，通过后，周期性（1小时）的向NameNode上报所有的块信息。
    - 心跳是每3秒一次，心跳返回结果带有NameNode给该DataNode的命令如复制块数据到另一台机器，或删除某个数据块。如果超过10分钟30秒没有收到某个DataNode的心跳，则认为该节点不可用。
    - 集群运行中可以安全加入和退出一些机器。
- DataNode数据完整性校验方法
    - 1、当DataNode读取Block的时候，它会计算CheckSum。
    - 2、如果计算后的CheckSum，与Block创建时值不一样，说明Block已经损坏。
    - 3、然后Client去读取其他DataNode上的Block。
    - 4、常见的校验算法 crc（32），md5（128），sha1（160）
    - 5、DataNode在其文件创建后周期验证CheckSum。
- 掉线时限参数设置
    - 流程：
    - ![image](6E21C77B02AB408B81E5A8CD6B7E6B83)
    - 设置超时时间：
        - 时长计算：TimeOUT = 2*recheck-interval + 10*interval
        - 文件：`hdfs-site.xml`
        ```
        <property>
            <name>dfs.namenode.heartbeat.recheck-interval</name>
            <value>300000</value ---- 毫秒
        </property>
        <property>
            <name>dfs.heartbeat.interval</name>
            <value>3</value>        ---- 秒
        </property>
        ```
- 服役新数据节点
    - 服役步骤：
        - 1、克隆一台DataNode节点机器 
        - 2、直接启动DataNode，即可关联到集群
            - 1启动命令
            ```
                hdfs --daemon start datanode
                yarn --daemon start nodemanager
            ```
        - 3、在hadoop105上上传文件
        - 4、如果数据不均衡，可以用命令实现集群的再平衡
            - 命令
            ```
                ./start-balancer.sh
            ```
- 退役旧数据节点
    - 添加白名单和黑名单
        - 白名单和黑名单是hadoop管理集群主机的一种机制。
        - 添加到白名单的主机节点，都允许访问NameNode，不在白名单的主机节点，都会被退出。添加到黑名单的主机节点，不允许访问NameNode，会在数据迁移后退出。
        - 白名单用于确定允许访问NameNode的DataNode节点，内容配置一般与workers文件内容一致。 黑名单用于在集群运行过程中退役DataNode节点。
    - 添加`白名单`处理步骤：(`黑名单类似`)
        - 1、在NameNode节点的/opt/module/hadoop-3.1.3/etc/hadoop目录下分别创建whitelist 和blacklist文件
            - whitelist中的内容为
            ```
            hadoop102
            hadoop103
            hadoop104
            hadoop105
            ```
            - blacklist暂时为空（``）
        - 2、在hdfs-site.xml配置文件中增加dfs.hosts和 dfs.hosts.exclude配置参数
        ```
        <!-- 白名单 -->
        <property>
            <name>dfs.hosts</name>
            <value>/opt/module/hadoop-3.1.3/etc/hadoop/whitelist</value>
        </property>
        <!-- 黑名单 -->
        <property>
            <name>dfs.hosts.exclude</name>
            <value>/opt/module/hadoop-3.1.3/etc/hadoop/blacklist</value>
        </property>
        ```
        - 3、分发配置文件
        - 4、集群重启
##

























































