---
title: Comparison of mainstream distributed storage systems-Redis vs Elasticsearch
date: 2018-08-01 20:10:00
tags:
- distributed system
- distributed storage
- redis
- kafka
- zookeeper
- elasticsearch
---
上一篇文章总计了 Zookeeper vs Kafka 对于分布式系统存在的问题的处理方式，今天来聊聊 Redis 和 Elasticsearch。
# Redis
## Node Role
Redis 中 roles 有:
* Master
    * 负责数据的读写
    * 对 slave 的健康监听是通过 sentinel 实现的
* Slave
    * slave 启动时，slaveof host port，即会去连接该 master 服务器
    * slave 可以配置是否参与写，一般建议只参与读


## Replication
> master-slave async replication，master 写成功后直接返回 client success，不会等待 slaves ack。如果 slaves  network fault/ crash/ latency 等原因，master 会继续接受 write，可能造成**数据不一致性**问题。

复制数据分为：
* 全量复制(Full Resync)
* 部分复制(Partial Resync)
* 异步复制

### Full Resync
> slave 向 master 发起: psync masterRunId -1

触发场景：
* slave 第一次 slaveof master 时
* slave 与 master 的数据差距过大到无法使用**复制积压缓冲区**中的 backlog 时

![Redis replication startup process](/assets/images/distributed-storage/what-happens-when-a-slave-connects-to-a-master.jpg)

全量复制的开销大：
* 生成 rdb 文件的时间
* 网络传输 rdb 的时间
* slave 清空本地历史 rdb 的时间
* slave 加载 rdb 的时间
* 对主从节点和网络有很大压力

所以，除了在 slave 第一次连接上 master 时使用全量复制，其他时候都建议使用部分复制。

### Partial Resync
> slave 向 master 发起: psync masterRunId offset

触发场景：
* 当主从复制过程中如果出现网络闪断等原因，超过 repl-timeout 没连上。这段时间 master 正常接受写请求，并记录在**复制积压缓冲区（默认最大 1MB）**。
* 网络恢复后，slave 主动向 master 要求补发丢失数据，带上自己已复制完的最大的 offset。
* master 检查 offset 之后的数据是否在自己的复制积压缓冲区，如果在，则执行部分复制。
* 如果该区域没有 slave 请求的 offset，则部分复制退化为全量复制。

### Async
触发场景：
* 主从服务架构稳定后，master 执行完写请求后，异步发送给 slaves

redis 是完全的异步复制机制。



## Failure Discovery
谈到 redis 的 Failure discovery & leader election，需要关注两个组件：
* redis master/slaves 节点本身
* redis sentinel cluster

Redis 通过 redis sentinel 自动完成对 redis master & redis slave & redis sentinel 的 fault discovery(故障发现) & failover(故障转移)。


### Redis
* sentinel 会每隔 1s 向 redis master & slaves & other sentinels 发送 ping 命令来维持 heartbeat
* 若 timeout 后 master 没响应，则认为 master **主观下线**
* 发现 master 不可用的 sentinel 和别的 sentinels 交互，当 quorum 个 sentinels 都认为 master 不可用时，标记 master **客观下线**（防止误判）

### Redis Sentinel
* 每个 sentinel 除了定期 ping redis master & slaves 外，还定期 ping 其他 sentinels，来检查其他 sentinels 是否正常。

### Quorum
quorum 的作用：
* 至少要有 quorum 个 sentinels 参与 sentinel leader election 过程
* 至少要有 quorum 个 sentinels 认可 master 宕机，客观下线


## Leader Election
### Sentinel
Sentinel 使用 **Raft** 算法来进行 sentinel cluster 的 leader election。

### Redis
* 故障转移的工作只需要一个 sentinel 就可以完成，所以会先选举出一个 sentinel leader
* sentinel leader 进行 failover 工作，选举出新的 redis master
    * 过滤不可达的 slaves
    * 选择 slave-priority 最高的 slave
    * 选择 offset 最大的
    * 选择 runid 最小的
* 将新的 master 通知所有的 redis servers & redis clients
* redis slaves follow 新的 master
* 原 master 恢复后，follow 新的 master
* sentinel 通知 redis client 更新本地的 master & slaves 配置信息


## Partition
Redis partition 是通过 redis cluster 实现的，这块目前暂时没有足够了解和应用，后续有实际经验后，再来更新文档。

## CAP
[Redis Cluster and limiting divergences](http://antirez.com/news/70)

