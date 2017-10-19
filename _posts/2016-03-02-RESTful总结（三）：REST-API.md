---
title: RESTful总结（三）：RESTful API
date: 2016-03-02 21:36:21
tags: RESTful
---

[twitter api design](https://dev.twitter.com/rest/public)

URI 是跨越 Web 的资源描述符，一个 URI 由以下内容组成:

* 协议，例如 http 和 https 
* 主机，例如 www.example.org
* 端口号
* 后面紧跟一段或多段路径，例如 /user/1234
* 查询字符串

在本章中，我们将关注为 RESTful Web 服务设计 URI。

## 1. API URI design
API URI 设计最重要的一个原则：nouns (not verbs!)，名词（而不是动词）

通过跨域、子域和路径来对服务器的 URI 进行分区，这会带来负载分配(distribution)、 监控、路由和安全方面的操作灵活性。针对本地化、分布式、强化多种监控及安全策略等方面的需求,可以使用域及子域对资源进行合理的分组或划分：
* 在 URI 的路径部分使用斜杠分隔符(/)来表示资源之间的层次关系。 
* 在 URI 的路径部分使用逗号(,)和分号(;)来表示非层次元素。* 使用连字符(-)和下划线(_)来改善长路径中名称的可读性。* 在 URI 的查询部分使用"&"来分隔参数。
* 在 URI 中避免出现文件扩展名(例如.php,.aspx 和.jsp)。


## 2. CRUD 简单 API 的设计:
操作用户：

|URI|作用|
|----|---|
|GET /users|获取用户列表|
| GET /users/1 |获取 Id 为 1 的用户|
|POST /users|创建一个用户|
|PUT /users/1|替换/创建 Id 为 1 的用户|
|PATCH /users/1|修改 Id 为 1 的用户|
|DELETE /users/1|删除 Id 为 1 的用户|

上面是对某一种资源进行操作的 URI，如果是级联的资源该如何设计呢？ 

## 3. 级联资源 URI 的设计
级联的资源之间存在关联，设计 URI 时最好不要打破这种关联，如：给消费者创建一个订单：
```code
// bad design, thought it could work
POST http://www.example.com/orders

// good design
POST http://www.example.com/customers/33245/orders
```

然而，级联关系也不宜太深，如：给消费者的订单创建一个条目：
```code
// bad design, thought it could work
POST http://www.example.com/customers/33245/orders/8769/lineitems

// good design : line items don't make sense only in customer context or also make sense outside the context of a customer

POST http://www.example.com/orders/8769/lineitems
```

操作某一用户下的产品：

|URI| 作用|
|---|---|
|GET /users/1/products|获取 Id 为 1 用户下的产品列表|
|GET /users/1/products/2|获取 Id 为 1 用户下 Id 为 2 的产品|
| POST /users/1/products |在 Id 为 1 用户下，创建一个产品|
|PUT /users/1/products/2|在 Id 为 1 用户下，替换/创建 Id 为 2 的产品|
|PATCH /users/1/products/2|修改 Id 为 1 的用户下 Id 为 2 的产品|
|DELETE /users/1/products/2|删除 Id 为 1 的用户下 Id 为 2 的产品|

## 4. 分页 & 排序
在设计 API 的时候，会进行一些查询操作，比如分页和排序等。如：
分页查询 id 为 1 的用户的购物记录列表：
```code
GET /users/1/goods?page=1&limit=10&sort=buytime
```

查询 id 为 1 的用户的购票信息，并按 priority 降序排列、buytime 生序排列：
```
GET /users/1/tickets?sort=-priority,buytime
```

## 5. Version
当 API 出现 backwards-incompatible（向后不兼容） changes 时，我们需要发布新的版本。version 可以帮助你快速迭代 & 平滑的从老 API 过度到新 API。

两种方式：
* 将 version 信息放到 URI 中
* 将 version 信息放到 request headers 中（学术意义上推荐这种做法）

```code
// URI
/api/v1/article/1234

// request headers
GET /api/article/1234 HTTP/1.1
Accept: application/vnd.api.article+xml; version=1.0
```

网上一种比较好的建议：
`the URL has a major version number (v1), but the API has date based sub-versions which can be chosen using a custom HTTP request header. In this case, the major version provides structural stability of the API as a whole while the sub-versions accounts for smaller changes (field deprecations, endpoint changes, etc).`

