[toc]



***
# 二、Zookeeper内部原理
## 2.1 节点类型
![image](816F0C6C223F4CFBB4FE515C17AF8682)
- Persistent节点 （持久化节点）
- Ephemeral节点（临时节点）
- 
## 2.2  Stat结构体
- 2.2.1 czxid-创建节点的事务zxid
    - 每次修改ZooKeeper状态都会收到一个zxid形式的时间戳，也就是ZooKeeper事务ID。
    - 事务ID是ZooKeeper中所有修改总的次序。每个修改都有唯一的zxid，如果zxid1小于zxid2，那么zxid1在zxid2之前发生。
    - dataversion - znode数据变化号
    - dataLength- znode的数据长度
    - numChildren - znode子节点数量
    - 
## 2.2 监听器原理（面试重点）
- ![image](6DF49D7426CF4545885AF3D7BBAB1CB9) 
- 
### 2.2.1 监听器原理流程
- 1、首先客户端有个main()主线程
- 2、哎main线程中创建zookeeper客户端，这个时候会穿件两个线程，一个负责通信（connect线程），一个负责监听（listener）
- 3、通过connect线程将注册的监听事件发送给zookeeper
- 4、在zookeeper的注册监听列表中将新注册的监听事件添加到列表中
- 5、zookeeper监听到有数据或者路径变化时，就会将这个消息发送给listener线程
- 6、listener线程内部调用process（）方法
- 
### 2.2.2 常见的监听
- 监听节点数据的变化
    - get path [watch] 
    - 
- 监听子节点增减的变化
    - ls path [watch] 
    - 

### 2.2 选举机制（面试重点）
- 1) 半数机制：集群中半数以上机器存活，集群可用。所以Zookeeper适合安装奇数台服务器。
- 2) Zookeeper虽然在配置文件中并没有指定Master和Slave。但是，Zookeeper工作时，是有一个节点为Leader，其他则为Follower，Leader是通过内部的选举机制临时产生的。

-
#### 2.3.1 三个核心选举原则：
- 1) Zookeeper集群中只有超过半数以上的服务器启动，集群才能正常工作；
- 2) 在集群正常工作之前，myid小的服务器给myid大的服务器投票，直到集群正常工作，选出Leader；
- 3) 选出Leader之后，之前的服务器状态由Looking改变为Following，以后的服务器都是Follower。
-
#### 2.3.2 主从选举流程（比较`id`, `初始化的选举过程`）
- ![image](2D9240D877B14D6B828D10397059B4A6)
- 
- 1）服务器1启动，发起一次选举。服务器1投自己一票。此时服务器1票数一票，不够半数以上（3票），选举无法完成，服务器1状态保持为LOOKING；
- 2）服务器2启动，再发起一次选举。服务器1和2分别投自己一票并交换选票信息：此时服务器1发现服务器2的ID比自己目前投票推举的（服务器1）大，更改选票为推举服务器2。此时服务器1票数0票，服务器2票数2票，没有半数以上结果，选举无法完成，服务器1，2状态保持LOOKING
- 3）服务器3启动，发起一次选举。此时服务器1和2都会更改选票为服务器3。此次投票结果：服务器1为0票，服务器2为0票，服务器3为3票。此时服务器3的票数已经超过半数，服务器3当选Leader。服务器1，2更改状态为FOLLOWING，服务器3更改状态为LEADING；
- 4）服务器4启动，发起一次选举。此时服务器1，2，3已经不是LOOKING状态，不会更改选票信息。交换选票信息结果：服务器3为3票，服务器4为1票。此时服务器4服从多数，更改选票信息为服务器3，并更改状态为FOLLOWING；
- 5） 服务器5启动，同4一样当小弟。
- 
#### 2.3.3 Zookeeper崩溃恢复过程(Leader选举)
- 1) 参考链接：`https://www.cnblogs.com/cgy-home/p/11836745.html`
- 2) 服务器角色状态：
    - LOOKING 系统刚启动或Leader崩溃后选举状态，认为当前集群中没有leader，因此要进入选举流程
    - FOLLOWING 跟随者状态，角色是Follower
    - LEADING 领导者状态，leader
    - OBSERVING 观察者状态(只支持查询，不参与选举)，Observer
    - 
- 3) ZAB协议定义的三种节点状态
    - Looking-选举状态
    - Following-从节点
    - Leader-主节点
    - 
- 4) ZAB的崩溃恢复选举过程
    - zookeeper 当前的主节点挂掉了，集群会进行崩溃恢复，分三个阶段
        - 1) Leader election 选举状态
            - 1、此时集群中的节点处于Looking状态，向其他节点发起投票，投票中携带自己的服务器ID和最新事务ID（ZXID）
            - 2、节点会用自身的ZXID和从其他节点接收到的ZXID做比较，如果发现别人家的ZXID比自己的大，也就是数据比自己新，那么久重新发起投票，投票给目前已知最大的ZXID所属节点。
            - 3、每次投票后，服务器都会统计投票数量，判断是否有某个节点但得到半数以上的投票，如果存在这样的节点，该节点将会成为准Leader，状态变为Leader,其他系欸但状态变为Following。
            - 
        - 2) Discover 发现阶段
            - 1、用于从节点中发现最新的ZXID和事务日志，问：既然Leader被选为主节点，已经是集群里最新数据了，为什么还要从节点中寻找最新事务呢？（`脑裂问题`）
            - 2、为了防止某些意外情况，比如因网络原因再上一阶段产生多个Leader的情况
            - 3、该阶段。Leader接受所有Follower发来的各自最新的epoch值，Leader从中选出最大的epoch，基于此值加1，生成新的epoch分发给各个Follower。各个Follower收到全新的epoch后，返回ACK给Leader，带上各自最大的ZXID和历史事务日志。Leader选出最大的ZXID，并更新自身历史日志。
            - 
        - 3) Sysnchronization 同步阶段
            - 1、同步阶段，把Leader 刚才收集得到的最新历史事务日志，同步给集群中所有的Follower，只有当半数Follower同步成功，这个准Leader才能成为正式的Leader，故障恢复正式完成。 
        - 
        - 
### 2.4 写数据流程(`流程一`)
- ![image](40CEB0FD0CC642D48115AA391DC1E09D)
- 
- 2.4.1 写数据流程
    - 1) client想zk的server1发送一个写请求
    - 2) 如果server1不是Leader,那么他会把接收到的请求转发给Leader（集群中Leader是唯一），这个Leader会把写请求广播给各个Server，各个Server会将写请求加入到`写队列`中，并想Leader发送成功信息
    - 3）当Leader收到半数以上Server的成功信息后，说明该`写操作`已经执行，Leader会向各个Server发送提交信息，各个Server收到消息后会落实队列里面的写请求，此时写成功
    - 4）Server1会进一步通知Client 数据写成功，此时认为数据写操作成功。
    - 
    - 
### 2.5 ZAB数据写入（`流程二`）（`同上`）
- 流程图![image](2F21EDCC8E344BBFA76276945323293C)
- 
- 0、Broadcast，Zookeeper常规情况下更新数据的时候，由Leader广播所有的Follower
- 1、客户端发出写入的数据请求给任意Follower
- 2、Follower把写入数据请求转发给Leader
- 3、Leader采用二阶段提交方式，先发送Propose广播给Follower
- 4、Follower接收到Propose消息，写入日志成功后，返回ACK消息给Leader。
- 5、Leader接到半数以上ACK消息，返回成功给客户端，并且广播Commit请求给Follower
-
## 三、企业面试
- 1、ZooKeeper的部署方式有哪几种？集群中的角色有哪些？集群最少需要几台机器？
    - 部署方式单机模式、集群模式
    - 角色：Leader和Follower
    - 集群最少需要机器数：3
    - 
- 2、ZooKeeper的常用命令
    - ls create get delete set 
    - 
##






























