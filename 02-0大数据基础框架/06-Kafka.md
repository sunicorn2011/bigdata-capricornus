[toc]

# 概念
    - 是什么？
    - 有啥用？
    - 怎么做？
    - 
# Kafka概述
## 定义
- Kafka是一个分布式的基于发布/订阅模式的消息队列（Message Queue），主要应用于大数据实时处理领域。
## 消息队列
- 消息队列优势：
    - 解耦
    - 消峰填谷
    - 异步通信
    - 缓冲/灵活性 & 峰值处理能力
    - 
- 消息队列模式
    - 1 VS 1 点对点模式
    - 发布 / 订阅模式（1对N）
    - 
## Kafka基础架构
- 基础架构图
![image](C459F136A4B840DD9957939AB2AA1211) 
- `系统角色说明:`
    - `基础角色`：   
        - Producer ：消息生产者，就是向kafka broker发消息的客户端
        - Consumer ：消息消费者，向kafka broker取消息的客户端
        - Broker ：一台kafka服务器就是一个broker。一个集群由多个broker组成。一个broker可以容纳多个topic。
        - Topic ：可以理解为一个队列，生产者和消费者面向的都是一个topic
    - `保证高可用(高可用读和高可用写)分类：`
        - Consumer Group （CG）：消费者组，由多个consumer组成。消费者组内每个消费者负责消费不同分区的数据，一个分区只能由一个组内消费者消费；消费者组之间互不影响。所有的消费者都属于某个消费者组，即消费者组是逻辑上的一个订阅者
        - Partition：为了实现扩展性，一个非常大的topic可以分布到多个broker（即服务器）上，一个topic可以分为多个partition，每个partition是一个有序的队列
        - Replica：副本，为保证集群中的某个节点发生故障时，该节点上的partition数据不丢失，且kafka仍然能够继续工作，kafka提供了副本机制，一个topic的每个分区都有若干个副本，一个leader和若干个follower
    - `按照从属关系分类：`
        - leader：每个分区多个副本的“主”，生产者发送数据的对象，以及消费者消费数据的对象都是leader
        - follower：每个分区多个副本中的“从”，实时从leader中同步数据，保持和leader数据的同步。leader发生故障时，某个follower会成为新的leader
        - 
##
# Kafka架构深入
## Kafka工作流程及文件存储机制
- Kafka工作流程结构图
- ![image](2B6FBE329B714C1E9E6006EF535D5442)
-
- 工作流程涉及角色：（存储文件： `__consumer_offsets_N`）
    - Producer: 数据生产者 
    - Topic：kafka中消息都是以Topic进行分类，Topic是个逻辑概念
        - 每个Topic下的每个partition都单独维护一个offset偏移量，所以分发到不同分区中的数据是不一样的
        - 消费者的分区是靠 一个`消费组`一个`Topic`一个`分区Partition`去维护一个offset
        - `<GTP, offset>` 
        - `重点记住`： 一个分组中一条消息值能够被消费一次
        - 
    - Partion：partition是物理上的概念，每个partition对应于一个log文件
        - 存储格式： `TopicName-PartitionN`即 `first-0`
    - log：
        -
- 工作关系
    -  
- 自定义分区器（`重点`）：
    - 1、人工直接指定分区器
    - 2、通过使用序列化后的`key`计算分区器
    - 3、使用默认的粘性分区器
    - 
- `分区器代码：`
-
-
## Kafka文件存储机制
- KafKa文件存储流程图
- ![image](B34C9EB0E4DA44E6B79440A7D0332877)
- `总概述`
    - 为防止log文件过大导致数据定位效率低下，Kafka采取了 `分片` 和 `索引` 机制,将每个partition分为多个segment。每个segment对应两个文件——“.index”文件和“.log”文件，这些文件位于一个文件夹下，该文件夹（即`Partition`）的命名规则为：topic名称+分区序号
    - `.index` 文件存储大量的索引信息，索引信息按照数组的逻辑排列，`.log` 文件存储大量的数据，数据直接紧密排列，索引文件中的元数据指向对应数据文件中message的物理偏移地址。
    - 
- 工作流程涉及角色：
    - `.index` 和 `.log`数据格式： 
    -![image](AF38B68E93D44A32AC1A08A5F5C4E349) 
    - Topic：逻辑分区
    - Partion：物理分区
    - Segment: 逻辑分区,包含有 `.index` 和 `.log`
        - `.index`：文件存储大量的索引信息，存储格式: <K, V>
        - `.log`：文件存储大量的数据，数据直接紧密排列,存储格式：[message-0,message-1,...Message-N]
        - 
    - 
## Kafka生产者(`写`)
- `消息发送流程图`
- ![image](D2D31CD8AD304B689EA5B4AEB840063D)
- 
- 消息发送流程：
    - 1、`main` 线程将消息发送给 `RecordAccumulator`
    - 2、`Sender` 线程不断从 `RecordAccumulator` 中拉取消息
    - 3、满足出发条件之后 `Sender`发送到`Kafka broker`
        - `满足条件`：（一般都是 `定时` 和 `满足更新的操作数量`）
            - batch.size：只有数据积累到batch.size之后，sender才会发送数据。
            - linger.ms：如果数据迟迟未达到batch.size，sender等待linger.time之后就会发送数据
            - 
- `涉及角色`：
    - `main`线程: 
    - `Sender`线程:  
    - `RecordAccumulator`: 线程共享变量
    - 
- `在生产者端 如何做到数据可靠性保证？`（面试重点）
    - 一、数据生产端主要涉及两个阶段： 
        - 1、生产者发送数据到topic partition的可靠性保证，即Producer发送到Kafka Broker阶段
        - 2、Topic partition存储数据的可靠性保证，即Kafka Broker保存数据阶段
        - 
    - 二、如何做到可靠性？
        - 遵从原则：`使用 ACK 确认机制·` 
        - 1、合适发送ACK响应？
            - 确保有follower与leader完成同步之后，才发送ack，这样即使leader挂掉，follower也能够选举出新的Leader
            - 
        - 2、什么样场景条件下可以发送ACK响应？
            - 1）半数以上follower完成，即可发送ack响应（半同步）
            - 2）全部完成，才可发送ack响应（全同步）
            -
        - 3、acks参数配置：（`这部分需要和 Leader选举机制模式配合使用` 重点）
            - 0：partition的Leader收到消息后，还没有写入磁盘就已经返回ACK，Leader有故障时，可能造成数据有丢失。（`数据在内存中`）
            - 1：这意味着领导者会将记录写入其本地日志，但会在不等待所有Follower(即`其他分区的Leader`)的完全确认的情况下做出响应。在这种情况下，如果领导者在确认记录后立即失败，但在追随者复制它之前，则记录将丢失。 (`重点`) (`数据从内存刚落磁盘, follower还没同步成功`)
            -1（all）：partition的leader和follower全部落盘成功后才返回ack.但是如果在follower同步完成后，broker发送ack之前，leader发生故障，那么会造成数据重复(因为客户端会有重传机制。)
            - 
- 什么是`ISR`?（`kafka是全副本同步机制，为了解决在数据同步时候，部分节点不能够及时反馈的情况，增加的一种优化机制`）
    - 定义：Leader维护了一个动态的in-sync replica set (ISR)，意为和leader保持同步的follower集合。
    - 同步过程：
        - 当ISR中的follower完成数据的同步之后，leader就会给producer发送ack。如果follower长时间未向leader同步数据，则该follower将被踢出ISR。Leader发生故障之后，就会从ISR中选举新的leader。
        - 
    - `时间阈值`参数设定：
        - `replica.lag.time.max.ms` 
        - 
- leader和 follower故障处理细节？
    - 系统图
    ![image](F73E760CEC0F4C438A61DAF8C969DB3A)
    -  
    - 基础性概念：
        - LEO：指的是每个副本最大的offset
        - HW：指的是消费者能见到的最大的offset，ISR队列中最小的LEO
        - 
    - 故障处理：
        - 1、follower故障
            - follower发生故障后会被临时踢出ISR，待该follower恢复后，follower会读取本地磁盘记录的上次的HW，并将log文件高于HW的部分截取掉，从HW开始向leader进行同步。等该follower的LEO大于等于该Partition的HW，即follower追上leader之后，就可以重新加入ISR了 
        - 2、leader故障
            - leader发生故障之后，会从ISR中选出一个新的leader，之后，为保证多个副本之间的数据一致性，其余的follower会先将各自的log文件高于HW的部分截掉，然后从新的leader同步数据 
        - `注意细节`：
            - 这只能保证副本之间的数据一致性，并不能保证数据不丢失或者不重复
            - 
- Exactly Once语义（幂等性）
    - 基础概念：
        - At Least Once可以保证数据不丢失，但是`不能保证数据不重复`，ack设置为 all
        - At Most Once可以保证数据不重复，但是`不能保证数据不丢失`，ack设置为 0
        - 
    - At Least Once + 幂等性 = Exactly Once
    - 设置参数：在Producer的参数中设置： `enable.idempotence = true`
    - 保证幂等过程：
        - 把原来需要下游去重的操作放到数据上有来做，Producer在初始化的时候都会被分配一个PID（`Producer ID`），发往同一个Partition的消息会附带一个Sequence Number，而Broker端会对<PID, Partition, SeqNumber>做缓存。当具有相同主键的消息提交时，Broker只会持久化一条。
        - ｀注意：｀
            -　PID重启就会变化，同时不同的Partition也具有不同主键，所以幂等性无法保证跨分区跨会话的Exactly Once
            -　
- Producer事务(`解决不同Partion数据重复的问题`)
    - `Producer事务`详解：
        - 链接：
        - 
    - 为了实现跨分区跨会话的事务，需要引入一个全局唯一的Transaction ID，并将Producer获得的PID（`Producer ID`）和Transaction ID绑定。这样当Producer重启后就可以通过正在进行的Transaction ID获得原来的PID。
    - 为了管理Transaction，Kafka引入了一个新的组件Transaction Coordinator（`事务协调器`）。Producer就是通过和Transaction Coordinator交互获得Transaction ID对应的任务状态。Transaction Coordinator还负责将事务所有写入Kafka的一个内部Topic，这样即使整个服务重启，由于事务状态得到保存，进行中的事务状态可以得到恢复，从而继续进行。
    - 
## Kafka消费者
### 消息常用消费方式
- consumer采用pull（拉）模式从broker中读取数据。
- push（推）模式很难适应消费速率不同的消费者，因为消息发送速率是由broker决定的。
### 分区分配策略（`重点`）
- 常用分区策略：
    - RoundRobin（轮询）
    - Range ：( `Partitions / GroupsN `）（1、首先等分  2、剩下的一次分配）
    - 
    - `参数配置`：`partition.assignment.strategy`
## offset的维护
- 问题的出现：
    - 一般由于不可控外力的出现，导致Consumer挂掉，Consumer重启之后，要能够找到自己offset继续去消费？
- 引入的解决方案：
    - 在0.9之前是使用的zookeepr保存offset，在之后会把offset保存到kafka之的一个内部Topic中 `__consumer_offset` 
    - 
- 消息提交（`Commit`）、更新 `offset` 方案：
    - 1、自动提交offset
        - 需要开启配置参数：
            - `enable.auto.commit`：是否开启自动提交offset功能
            - `auto.commit.interval.ms`：自动提交offset的时间间隔
        - 引发问题：
            - 1、消息丢失
            - 
    - 2、重置Offset
        - 需要开启配置参数：
            - `auto.offset.reset`: 参数值
                - `earliest`：自动将偏移量重置为最早的偏移量
                - `latest(默认值)`：自动将偏移量重置为最新偏移量
                - `none`：如果未找到消费者组的先前偏移量，则向消费者抛出异常
                - 
        - 引发问题：
            - 1、重复消费
            - 
    - 3、手动提交offset（`poll拉取消费消息`）
        - 1、两种模式：`commitSync（同步提交）` 和 `commitAsync（异步提交）`
            - 共同点：
                - 都会将本次poll的一批数据最高的偏移量提交
                - 
            - 不同点：
                - `commitSync（同步提交）`：阻塞当前线程，有自动重试机制。
                - `commitAsync（异步提交）`：没有重试机制。
-
- Consumer事务（精准一次性消费）`这个问题和上面Producer对应`
    - `Consumer事务`详解：
        - 链接：
        - 
    - `问题引出`：数据漏消费和重复消费分析
        - 无论是同步提交还是异步提交offset，都有可能会造成数据的漏消费或者重复消费
        - 先提交offset后消费，有可能造成数据的`漏消费`
        - 先消费后提交offset，有可能会造成数据的`重复消费`
    
    - `解决方案`：
        - 1、由于Consumer可以通过offset访问任意信息，而且不同的Segment File生命周期不同，同一事务的消息可能会出现重启后被删除的情况。 
        - 2、如果想完成Consumer端的精准一次性消费，那么需要kafka消费端将消费过程和提交offset过程做原子绑定。此时我们需要将kafka的offset保存到支持事务的自定义介质（比如mysql、Redis）
    - 
##
## Kafka 高效读写数据
- 参考链接：https://blog.csdn.net/yunduo1/article/details/108714939
- 
## 1、顺序写磁盘
## 2、应用页缓存技术
## 3、零复制技术
- 零拷贝结构图
- 链接：
- 传统拷贝过程：
- ![image](88BA1FDB42424F4E99640B0E17C3DECC)
- 
- 零拷贝过程：
- ![image](BA5F8EBB44F345F59434846F2F743E7E)
-
## Zookeeper在Kafka中的作用
- Kafka的Leader选举机制
    - 参考链接：
        - 链接：http://note.youdao.com/noteshare?id=e4595f1949436d3f9657227dbda3cf2a&sub=93F8E998FFE04885AAD1275166282FDF
-
# Kafka面试题
##
##
##
##
##
##
##

























