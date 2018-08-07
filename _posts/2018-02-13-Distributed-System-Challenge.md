title: Distributed System challenge
date: 2018-02-13 20:10:00
categories:
- Distributed System
tags:
- Distributed System
---

> No Silver Bullet
> **监控是一切的前提**，没有好的监控系统，我们将无法进行自动化运维和资源调度。

[](../images/distributed%20system.jpg)
分布式系统其实从整体上来说，分为分布式应用系统 & 分布式存储系统。下文主要介绍分布式应用系统，分布式存储系统参见：distributed storage。 

![Monolithic system vs Distributed system trade-off](images/trade-off.png)

![challenge](images/challenge.png)

这个世界上不存在完美的技术方案，都是有得有失，都是 trade-off。

分布式系统(微服务)的出现使得开发速度变得更快，部署快，隔离性高，系统的扩展度也很好，解决了“单点”和“性能容量”的问题。
但它又是一个成本巨高无比的技术栈，在系统设计、集成测试、应用监控、自动化运维和服务治理等方面带来了很多其他的问题和挑战，需要不断地用各式各样的技术和手段来解决。

好在，通过 Docker 及其衍生出的 Kubernetes, PaaS 平台等方案的支撑，大大地降低了做上面很多事情的门槛：
* 像 Spring Cloud 一样需要提供各种配置服务、服务发现、智能路由、控制总线
* 像 Docker & Kubernetes 提供的各式各样的部署和调度方式


# 分布式系统带来的挑战
* architecture design（service dependency & **distributed transactions** and so on）。
* deployment: 部署单个服务会比较快，但是如果一次部署需要多个服务，部署会变得复杂。
* performance: 系统的吞吐量会变大，但是性能会变差， response time / latency 会变长(由于 geographic distance increases & communication increase)。
* ops: 运维复杂度会因为服务变多而变得很复杂，需要自动化运维。
* learning curve: 架构复杂导致学习曲线变大。
* testing: 测试和查错的复杂度增大，需要引入集成测试。
* maintain: 技术可以很多样，这会带来维护和运维的复杂度。
* scheduling: 管理服务和流量调度
* monitor: 大量的服务需要自动化监控、报警及后续的处理
* transaction: 事务处理变得复杂
* failure: increases the **probability of failure **in a system (reducing availability and increasing administrative costs)
* latency/timeout: 带来响应时长的增加、甚至单体应用没有的超时问题
* protocol: 整体架构间使用标准的协议，以便可以被调度、编排，且互相之间可以通信 

## exception
单体应用的**故障影响面很大**，而分布式系统中，虽然故障的影响面可以被隔离，但因为机器和服务多，**出故障的频率也比单体应用大**。出现故障不可怕：
* **恢复时间过长**才可怕
* **影响面积过大**才可怕

分布式系统设计开发时，要谨记：异常时时刻刻都存在，磁盘会损坏、网络会中断、会断电、会停机，所以需要在设计或运维系统时都要为这些故障考虑，即所谓 Design for Failure。

## timeout
异常已经够可怕，但是还有一个更痛苦的问题：timeout。
单体应用是不存在 timeout 这个问题的，每次调用的结果都是明确的成功／失败。
分布式请求结果存在三态：成功、失败、超时。

但是在分布式系统（multiprocess）中，timeout 是很常见的，并且很容易导致 client 无所适从，不清楚发生了什么：
* 请求没有发成功
* 请求发成功了，server 处理成功了，但是由于网络等原因，没有把 response 返回
* 请求发成功了，server 处理失败了，但是由于网络等原因，没有把 response 返回

所以，client 不能简单的认为服务处理失败。所以，我们倾向于把 server 的接口设计成幂等的，这样保证多次调用的执行结果相同，当 timeout/failure 时，可重试。

## 多层架构的运维复杂度更大
可以把系统分成四层：基础层、平台层、应用层和接入层:
* **基础层**: 我们的机器、网络和存储设备等。
* **平台层**: 我们的中间件层，Tomcat、MySQL、Redis、Kafka 之类的软件。
* **应用层**: 我们的业务软件，比如，各种功能的服务。
* **接入层**: 接入用户请求的 Nginx、网关、负载均衡或是 CDN、DNS 这样的东西。

对于这四层，我们需要知道：
* 任何一层的问题都会导致整体的问题
* 没有统一的视图和管理，导致运维被割裂开来，造成更大的复杂度


# 解决方案
* 需要有完善的**监控系统**，以便对服务运行状态有全面的了解。
* 设计服务时要分析其依赖链；当非关键服务故障时，其他服务要自动降级功能，避免调用该服务。
* 自动构建服务的依赖地图，并引入好的处理流程，让团队能以最快速度定位和恢复故障。
* 重构老的软件，使其能被服务化；可以参考 SOA 和微服务的设计方式，目标是微服务化；使用 Docker 和 Kubernetes 来调度服务。
* 为老的服务编写接口逻辑来使用标准协议，或在必要时重构老的服务以使得它们有这些功能。
* 使用一个 API Gateway，它具备服务流向控制、流量控制和管理的功能。
* 事务处理建议在存储层实现；根据业务需求，或者降级使用更简单、吞吐量更大的最终一致性方案，或者通过二阶段提交、Paxos、Raft、NWR 等方案之一，使用吞吐量小的强一致性方案。
* 通过更真实地模拟生产环境，乃至在生产环境中做灰度发布，从而增加测试强度；同时做充分的单元测试和集成测试以发现和消除缺陷。
* 通过异步调用来减少对短响应时间的依赖；对关键服务提供专属硬件资源，并优化软件逻辑以缩短响应时间。

> 别的问题会在稍后具体分析讲解
