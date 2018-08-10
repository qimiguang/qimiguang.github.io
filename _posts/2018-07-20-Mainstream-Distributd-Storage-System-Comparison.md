---
title: Comparison of mainstream distributed storage systems
date: 2018-07-20 20:10:00
tags:
- distributed system
- distributed storage
- redis
- kafka
- zookeeper
- elasticsearch
---

今天计划将 Kafka, Elasticsearch, Redis, Zookeeper 放到一起来聊一聊。为什么要将这几个 Components 放一起比较呢，因为本质上来说，他们都是存储数据的：
* Kafka 存消息数据
* Redis 存缓存数据
* Elasticsearch 存搜索数据
* Zookeeper 通过 like-file-system 存配置信息

还有关键的一点，他们都是天然支持**分布式存储**的。而提到分布式存储，就不得不提:
* node role
* failure discovery
* leader election
* partition
* replication
* CAP

所以，这篇文章的初衷就是：比较不同分布式存储系统对不同问题的具体实现。接下来，我们梳理一下这篇繁杂文章应有的逻辑：
1. 首先是 node role，它定义的不同节点的不同作用
2. cluster 启动时进行 leader election
3. cluster 正常启动后，进行数据的操作，需要处理 replication
4. failure discovery:
    * leader: leader election   
    * follower: drop or restart 
5. 最后是从整体上分析各个系统的 CAP 选择
6. 部分系统涉及到 partition 

我会尽我所能梳理清楚这些 key point 及其之间的关系。


# Zookeeper

## Node role
zookeeper 中 roles 有:
* leader
    * leader 处理所有的写请求(create, delete, setData)
* follower
    * 只处理 client 发起的读请求（exists, getData, getChildren）
    * 收到写请求后，forward 给 leader，然后从 leader 同步写操作
    * 参与 leader election
* observer
    * 只处理 client 发起的读请求（exists, getData, getChildren）
    * 收到写请求后，forward 给 leader，然后从 leader 同步写操作
    * 不参与 leader election
    * 不参与强一致性同步(不参与 quorum)
    * 单纯为了在不影响写性能的情况下，提高读性能

### Quorum
Quorum 有两个作用:
* cluster state: zk cluster 正常运行所需的最少 server 数。在这里理解为**参与投票的最少可用法定人数**。
* leader election: 选举时最少需要 quorum 个节点认同，节点才能成为 leader。以避免 split brain，如果无法选出，则 cluster 将无法正常服务。
* consistency: replication 中，server 在返回 client success 前，最少写成功的 servers 数。这种意义上来说，和 **kafka 中的 acks** 是一个概念。

quorum 保证：只要当前集群整体正常（至少 quorum 个节点正常），写操作也得到 quorum 个节点 ack。当部分节点失效后，如果集群还整体正常，则一定有一个节点持有所有最新的数据。

## Replication: Zab
Zookeeper 是通过 ZAB 实现 
* data replication 
* leader election

### Transaction
leader 将每一个写请求转换为一个 transaction，并分配 **zxid**。

### ZXID 
zxid 为一个64位 long 整数，分为两部分，每个部分32位：
* 时间戳（epoch）
* 计数器（counter）

zxid 主要有两个作用：
* 通过 zxid 对事务标识，就可以按照 leader 指定的顺序在各个服务器中**顺序执行**
* 服务器之间在进行新的 **leader election** 时交换 zxid 信息，哪个无故障服务器接收了更多的事务，就选举那个作为新的 leader

### EPOCH
epoch 代表了**管理权随时间的变化情况**，epoch 的值在每次进行 leader election 时增加，一个 epoch 表示某个服务器行使管理权的这段时间。在一个 epoch 内，leader 根据 counter 定义每一个 transaction。zxid 可以很容易地与 transaction 被创建 epoch 相关联。


### Zab (ZooKeeper Atomic Broadcast protocol)
Zab 是 Zookeeper 的 Consistency Algorithm，可以用它实现： 
* leader election
* data replication consistency 

Zookeeper 通过 Zab 协议来进行 transaction commit 操作，ZAB 协议和 2PC 有相似之处：
1. 接到 client 的写请求后，follower 将请求转发给 leader
2. leader 向所有 followers 发送一个 proposal 消息，该消息中包含**写事务**
3. 当 follower 接收到 PROPOSAL 后，会响应 leader 一个 ACK 消息，通知 leader 其已接受该提案（proposal）
4. 当 **quorum(包括 leader 自己)** 个以上服务器回复 ack 后，leader 就会给整个集群发 commit 消息
5. 之后 leader response client success


![](/assets/images/zookeeper/regular-message-pattern-to-commit-proposals.jpg)

ZAB 保证:
* 事务在服务器之间的传送**顺序的一致**
* 服务器**不会跳过**任何事务


## Leader election
1. 每个服务器启动后进入 LOOKING 状态
2. 如果 leader 已经存在，其他服务器会通知这个新启动的服务器，告知哪个服务器是 leader，与此同时，新的服务器会与 leader 建立连接，以确保自己的状态与 leader 一致
3. 如果集群中所有的服务器均处于 LOOKING 状态，这些服务器之间广播发送 **leader election notification** 来选举一个 leader。

notification 包括该服务器的投票（vote）信息，投票包含
* 服务器标识符（sid）
* 最近执行的事务的 zxid 信息(epoch & counter)


### 选举过程
1. receiver 自己的 vote: myZxid & mySid
2. receiver 收到的 vote: voteId & voteZxid 
3. If (voteZxid > myZxid) or (voteZxid = myZxid and voteId > mySid), vote = (voteId, voteZxid)
3. else, vote = (mySid, myZxid)

经过多轮选举后，最终选出一个拥有最新 zxid & 被 quorum 的 nodes 同意的 leader。

![leader election](/assets/images/zookeeper/a-leader-election-execution.jpg)

#### 选举过程中的消息延迟
在从服务器 s1 向服务器 s2 传送消息时发生了网络故障导致长时间延迟，与此同时，服务器 s2 选择了服务器 s3 作为 leader 。但是服务器 s1 和服务器 s3 组成了仲裁数量（quorum）的 vote，将忽略服务器 s2。

![](/assets/images/zookeeper/interleaving-of-messages-causes-a-server-to-elect-a-different-leader.jpg)

#### FastLeaderElection
如果让 s2 在进行 leader 选举时多等待一会（无法确定需要等待多长时间），它就能做出正确的判断：
![](/assets/images/zookeeper/longer-delay-in-electing-a-leader.jpg)

现实中，默认 leader election finalizeWait = 200ms

如果此延迟不够长，部分服务器可能错误地选择 leader，导致这轮 election 无法产生符合 quorum 的 leader，导致需要发送更多消息来继续执行 leader election，这会使整个恢复时间更长。

> 200ms 对于现代的数据中心消息延迟来说足够了。


### 选举成功后
1. leader election 成功后，leader 节点告诉集群中的所有节点自己的 leader 身份
2. 其他节点成为 leader 的 followers
3. 新的 leader 同步自己持有的所有之前 epoch 的 transactions 给 followers
    * DIFF（增量同步）: minCommittedLog <= peerLastZxid <= maxCommittedLog，follower 和 leader 差距不大，leader 只发送最近的 transaction log
    * SNAP（全量同步）: peerLastZxid < minCommittedLog，follower 数据过于陈旧，ZooKeeper 会执行完整的 snapshot 传输（增加恢复时间）
4. followers 更新完后，新 leader 才开始广播新的 transaction proposal
5. 新的 transaction 的 zxid 都**具有新的 epoch**
  
> 可以看出，对于如 Zookeeper 一样持有状态(state/data) 的服务，leader election 和 replication 深深的结合在了一起。

## Failure discovery
Zookeeper 可以通过临时节点来被第三方应用（如 Kafka）来实现故障检测，但是它自身的节点是如何通讯 & 实现故障检测的呢？

在每个 zk 的 zoo.cfg 中，配置了如下信息：
* tickTime=2000
* initLimit=5
* syncLimit=2
* server.1=192.168.20.105:2888:3888
* server.2=192.168.20.106:2888:3888
* server.3=192.168.20.107:2888:3888

### tickTime 
tickTime 是一个 tick 的时间，是后面配置的一个基准时间，以 ms 计量。

### initLimit
initLimit: 
* follower 初始连接到 leader 的最大 tick 次数。timeout = **tickTime * initLimit**
* follower 试图连接 leader 的最大时间，如果还连不上，就发起 **leader election**。

> 可以理解为 leader failure discovery 的配置。

### syncLimit 
syncLimit: leader 同步 follower (sending a request and getting an acknowledgement) 的最大 tick 次数。timeout = **tickTime * syncLimit**。

如果 leader 超过 syncLimit 个 ticks 还收不到 ack，就会在 cluster 中抛弃该 follower。
> 可以理解为 follower failure discovery 的配置。





## CAP
Consistency, Availability and Partition tolerance are the the three properties considered in the CAP theorem. The theorem states that a distributed system can only provide two of these three properties. ZooKeeper is a CP system with regard to the CAP theorem. This implies that it sacrifices availabilty in order to achieve consistency and partition tolerance. In other words, if it cannot guarantee correct behaviour it will not respond to queries.




## Partition
Zookeeper 不涉及到 Partition。

 



## Application Leader election by Zookeeper 
初识时 app 通过抢占式的创建临时的 /master 节点来推选自己为主节点（我们称为“主节点竞选”）。如果 /master 已存在，app 在 /master 节点上设置一个监视点。
app 创建一个有序的节点 /node/node-，ZooKeeper 在这个 znode 节点上自动添加一个序列号，成为 /node/node-xxx，其中 xxx 为序列号。当 master 宕机时，可以使用这个序列号来确定哪个 app 成为新的 master。


# Kafka
## Broker controller election
### controller broker function
* 承担普通的 broker 读写功能
* 监控 cluster 中别的 brokers 的健康状态 
* 负责 **partition leaders 的选举**。如果 controller broker 发现有 broker 离开 cluster（通过监听 zk 的相关路径 node）时，那么所有存在于丢失 broker 上的作为 leader 的 partitions replicas 需要新的 leader，controller 负责选择一个 partition 作为 leader，并通知给各个 brokers partitions。新的 partition leaders 明确自己的职责，followers 则明确自己需要同步的 new leader。

### controller failure detection:
* 当 controller broker stop or loses connectivity to zk 时，它创建的 /controller node 会被 zk 删除。
* 其他 broker 监听该 /controller，如果被 zk 删除，其他 brokers 收到通知，开始选举

### election steps：
* 每一个 broker 都配置有自身唯一的 id。并在启动时将此 id 写入到 zk 的临时节点中。当 broker 和 zk 断掉连接后（broker stop / network partition / long garbage-collection pause），broker 启动时创建的 ephemeral node 将自动被 zk 删除。 
* 第一个加入 cluster 的 broker 会在 zk 中创建一个 /controller 的临时节点，表明自己是 cluster 的 controller。
* 其他后加入 cluster 的 brokers 也试图创建 /controller ephemeral node，但是失败，其余的 brokers 会监听 /controller node。
* 当 controller broker stop or loses connectivity to zk 时，它创建的 /controller node 会被 zk 删除。
* **先到先得**的原则: cluster 中别的 brokers 将被 zk 通知 controller 丢失，剩下的 brokers 继续抢占式创建 /controller node，第一个写成功的成为新的 controller。

> 先到先得


## Replication
kafka 将 topic 拆分成 partition，又对 partition 进行 replicas，partition replicas 有两种 roles：
* leader
* follower

### leader function
* leader: 每个 partition replicas 有一个 leader
    * 负责该 partition 的所有读写操作(producer, follower replicas, consumer)
    * leader 维护每个 followers 的同步进度
* follower: followers 不直接承担 client 的读写任务。它们的唯一工作就是通过向 leader 发 fetch request 备份 messages。

### leader failure detection
partition replica leader 的故障发现是通过 broker controller 监控 broker 实现的，当某个 broker crash 时，broker controller 会通过监听 zk 临时节点观察到，并对该 broker 上持有的 partition replica leader 进行选举。 

### election steps
Kafka dynamically maintains a set of in-sync replicas (ISR) that are caught-up to the leader. 

Only members of this set are eligible for election as leader. 
A write to a Kafka partition is not considered committed until all in-sync replicas have received the write. 
This ISR set is persisted to ZooKeeper whenever it changes.



replica fetch messages 时，会把自身已有的最大 offset 带给 leader 来获得准确的 messages。当 partition leader crashes 时，其中一个拥有最多消息（最大 offsets）的 followers 将变成 leader。



所有 Kafka replicas 都有两个重要的属性：log end offset & high watermark。
* log end offset：记录了该副本底层日志(log)中**下一条消息**的位移值。也就是说，如果LEO=10，那么表示该副本保存了10条消息，位移值范围是[0, 9]。
* high watermark：对于同一个副本对象而言，其 HW 值不会大于 LEO 值。小于等于 HW 值的所有消息都被认为是“已备份”的（replicated）。

![](../../mq/kafka/images/HighWater%20&%20LogEndOffset.jpg)

consumer 无法消费未提交消息，准确解释为：consumer 无法获取到 leader partition 的 high water 到 log end offset 之间的的任何消息。

> hw 在新版中被 epoch 替换

### log end offset
follower 的 log end offset 存储在
* 自身 broker 上：
    * 作用：自身更新 high watermark
    * 更新时机：当 follower 发送 FETCH 请求后，leader 将数据返回给 follower，此时 follower 开始向底层 log 写数据，从而自动地更新 LEO 值
* leader broker 上（leader broker 上保存了所有的 follower 的 LEO）
    * 作用：用于 leader 处理自身 high watermark
    * 更新时机：leader 收到 follower 发送的 FETCH 请求，它首先会从自己的 log 中读取相应的数据，但是在给 follower 返回数据之前它先去更新自己 broker 的 follower 的 leo

leader 的 log end offset 存储在：
* 自身 broker 上
* 更新时机：leader 写 log 时就会自动地更新它自己的 LEO 值

### high watermark
follower更新HW发生在其更新LEO之后，一旦follower向log写完数据，它会尝试更新它自己的HW值。具体算法就是比较当前LEO值与FETCH响应中leader的HW值，取两者的小者作为新的HW值。这告诉我们一个事实：如果follower的LEO值超过了leader的HW值，那么follower HW值是不会越过leader HW值的。


leader的HW值就是分区HW值，因此何时更新这个值是我们最关心的，因为它直接影响了分区数据对于consumer的可见性 。以下4种情况下leader会尝试去更新分区HW——切记是尝试，有可能因为不满足条件而不做任何更新：

副本成为leader副本时：当某个副本成为了分区的leader副本，Kafka会尝试去更新分区HW。这是显而易见的道理，毕竟分区leader发生了变更，这个副本的状态是一定要检查的！不过，本文讨论的是当系统稳定后且正常工作时备份机制可能出现的问题，故这个条件不在我们的讨论之列。
broker出现崩溃导致副本被踢出ISR时：若有broker崩溃则必须查看下是否会波及此分区，因此检查下分区HW值是否需要更新是有必要的。本文不对这种情况做深入讨论
producer向leader副本写入消息时：因为写入消息会更新leader的LEO，故有必要再查看下HW值是否也需要修改
leader处理follower FETCH请求时：当leader处理follower的FETCH请求时首先会从底层的log读取数据，之后会尝试更新分区HW值       

> async replication
 



## Consumer group leader
### consumer group leader function
维护组内 consumer assign & reassign partitions。

### leader failure detection

### election steps



# Redis

# Redis Sentinel

### 
### leader election
Sentinel 使用 **Raft** 算法来进行 sentinel cluster 的 leader election。






# Elasticsearch



# MySQL
# RabbitMQ
# etcd


# Summary
## 先到先得
leader 不负责作为 master 写数据 & 同步数据到 replicas，仅仅是维护 cluster 状态时，随便一个 follower 都可以**抢占**为 leader 。

Kafka broker controller
kafka consumer group leader

## 选举
leader 负责作为 master 负责写数据 & 同步数据到 replicas，需要拥有最多最新数据的 node 成为新的 leader。









role
leader election
replication
failure discovery
leader election
cap
partition
