---
title: RESTful总结（二）：HTTP协议
date: 2016-03-02 21:07:26
tags:
---

## 1. 简介
HTTP是一种应用层协议,它定义了客户端与服务器之间的转移操作的表述形式。在此协议中,诸如 GET, POST, PUT 和 DELETE 之类的方法是对资源的操作。有了它,就无须创造 createOrder, getStatus, updateStatus等应用程序特定的操作了。能从 HTTP 基础设施中获得多少收益,主要取决于您把它当做`应用层协议`用得有多好。然而,包括 SOAP 和一些 Ajax Web 框架 在内的不少技术都将 HTTP 作为一种传输信息的协议,这种用法很难充分利用 HTTP 层的基础设施。

### 如何保持交互的可见性
HTTP 通过以下途径来实现可见性:

* HTTP 的交互是无状态的,任何 HTTP 中介都可以推断出给定请求和响应的意义,而无须关联过去或将来的请求和响应。* HTTP 使用一个统一接口,包括有 OPTIONS, GET, HEAD, POST, DELETE 和 TRACE 方法。 接口中的每一个方法操作一个且仅有一个资源。每个方法的语法和含义不会因应用程序 或资源的不同而发生改变。这就是为什么 HTTP 以统一接口而闻名于世了。* HTTP 使用一种与 MIME 类似的信封格式进行表述编码。这种格式明确区分标头和内容。标头是可见的,除了创建、处理消息的部分,软件的其他部分都可以不用关心消息的内容。

保持可见性：
* 使用 HTTP 方法时,其语义要与 HTTP 所规定的语义保持一致,并添加适当的标头来描述请 求和响应
* 使用适当的状态码和状态消息,以便代理、缓存和客户端可以决定 请求的结果。状态码是一个整数,状态消息是文本

## 2. HTTP Verbs

### 2.1 GET
GET 的目的是得到一个资源的表述，它应该是`安全`且`幂等`，意味着无论重复多少次带有相同参数的get操作，结果都是相等的。

### 2.2 POST
以下场合中使用 POST 方法:
* `新增资源`
* 更新资源* 执行需要大数据输入的查询* 在其他 HTTP 方法看上去不合适时，执行不安全或非幂等的操作

所有这些操作都是不安全和非幂等的，所有基于 HTTP 的工具都会这样对待 POST,例如:
* 缓存不会缓存这一方法的响应* 网络爬虫和类似的工具不会自动发起 POST 请求* 大部分通用的 HTTP 工具不会自动重复提交 POST 请求

### 2.3 PUT
PUT 比较正确的定义是 Replace (Update or Create)，根据 URI 判断是否存在指定资源：
* 存在：将资源完整替换为 HTTP request body 中的数据
* 不存在：根据 HTTP request body 中的数据来创建一个新的资源

例如下面请求的意思是：资源 /items/1 如果已经存在就替换，没有就新增。
```code
PUT /items/1
```

### 2.4 PATCH
如果只是为了更新 items/1 的其中一个属性，就需要重传所有 items/1 的属性也太浪费带宽了，所以后来又有新的 PATCH Method 标准，可以用来做部分更新(Partial Update)。它是幂等的。
```code
PATCH /users/123 HTTP/1.1
Content-Type: application/json-patch+json 

[
  { "op": "replace", "path": "/name", "value": "shuanggan" }
]
```

The patch operations supported by JSONPatch are “add”, “remove”, “replace”, “move”, “copy” and “test”. 
详细用法可参考：[What is JSONPatch](http://jsonpatch.com/) [GitHub json-patch](https://github.com/fge/json-patch)

### 2.5 DELETE
删除资源

### 总结
| HTTP method | Safe | Idempotent |
|:-----------|------------:|:------------:|
| GET |	Y|Y|
| POST | N |	N|
| PUT | N |Y|
| PATCH |N	|	N|
| DELETE |N	|Y|

Idempotent 特性影响可否 Retry （重试，反正结果一样）。
Safe 特性会影响是否可以快取。


| Verbs | URI | affect |
|:-----------|------------:|:------------:|
| GET       |    /api/user/ |     returns a list of users |     
| GET     | /api/user/1 |    returns the user with ID 1    
| POST       | /api/user/ |creates a new user with a user object as JSON   
| PUT  |   /api/user/3  |updates user(ID 3) with a user object as JSON
| DELETE       | /api/user/4 |  deletes the user with ID 4    
| DELETE    |  /api/user/  |   deletes all the users
| PATCH | /api/user/1 |update user(ID 1) whit a user object partial params|

## 3. HTTP URI
[参考另一篇post](http://qimiguang.github.io/2016/03/02/RESTful%E6%80%BB%E7%BB%93%EF%BC%88%E4%B8%89%EF%BC%89%EF%BC%9AREST-API/)

## 4. HTTP Headers
很多REST API犯的比较大的一个问题是：不怎么理会request headers。对于 REST API，有一些HTTP headers很重要。

### 4.1 Request Headers
`Accept` : The Accept header tells the server what your client wants in the response. 如果客户端要求返回"application/xml"，而服务器端只能返回"application/json"，那么最好返回status code 406 not acceptable，一个合格的 REST API 需要根据 Accept 头来灵活返回合适的数据。
`Content-Type` : The Content-Type header tells the server what the client sends in the request. 

The server consumes what it receives from the client in body (its format is specified in Content-Type) and it produces what the client accepts (its format is specified in Accept) : 

|Client||Server|
|---|---|---|
|Accept|<--->|Spring annotation : RequestMapping.produces |
|Content-Type|<--->|Spring annotation : RequestMapping.consumes|

`Cache-Control`
`Cookie`

### 4.2 Response Headers

`Content-Type` : The ContentType property specifies the HTTP content type for the response. If no ContentType is specified, the default is text/HTML.

`严格意义上，当客户端发送不带 accept 的请求时，返回错误码 400(Bad Request)。当你从服务器接收到一个不带 content-type 的响应时, 将其视为不正确的响应。`

`Content-Length` : 用于指定表述正文的字节大小。
`Content-Encoding` : 让您的网络库代码来解压那些压缩过的表述。


## 5. Status codes
很多REST API犯下的另一个错误是：返回数据时不遵循 RFC 定义的status code，而是一律200 ok ／ error message。这么做在client + API 都是同一公司所为还凑合可用，但一旦把 API 暴露给第三方，不但贻笑大方，还会留下诸多互操作上的隐患。

Status codes indicate the result of the HTTP request.

* 1XX - informational
* 2XX - success
* 3XX - redirection
* 4XX - client error
* 5XX - server error
