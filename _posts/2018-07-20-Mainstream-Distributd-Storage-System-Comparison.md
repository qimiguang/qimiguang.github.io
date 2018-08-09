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

所以，这篇文章的初衷就是：比较不同分布式存储系统对不同问题的具体实现。


