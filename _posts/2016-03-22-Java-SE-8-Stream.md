---
title: 'Java SE 8: Stream'
date: 2016-03-22 00:05:43
tags: Java 
---

## stream 
本来想自己写一篇关于 Stream 的博客的，看过 [ifeve博客](http://ifeve.com/stream/comment-page-1/#comment-26789) 上的一篇文章后，觉得没必要了，已经写的足够通俗易懂，足够生动了。下文仅就[ifeve 博客](http://ifeve.com/stream/comment-page-1/#comment-26789)中遗漏的部分进行补充。

## 为什么需要 Stream
Stream API 引入的目的在于弥补 Java 函数式编程的缺陷。对于很多支持函数式编程的语言，map()、reduce()基本上都内置到语言的标准库中了，不过，Java 8的Stream API总体来讲仍然是非常完善和强大，足以用很少的代码完成许多复杂的功能。

Stream 可以说是加强版的 Iterable。为什么不在集合类实现这些操作，而是定义了全新的Stream API？Oracle官方给出了几个重要原因：

* 一是集合类持有的所有元素都是存储在内存中的，非常巨大的集合类会占用大量的内存，而Stream的元素却是在访问的时候才被计算出来，这种“延迟计算”的特性有点类似Clojure的lazy-seq，占用内存很少。
* 二是集合类的迭代逻辑是调用者负责，通常是for循环，而Stream的迭代是隐含在对Stream的各种操作中，例如map()。

stream 相比 for 循环的优势：
1. Stream 支持并行运算(Collection.parallelStream()来创建)，传统的 for 循环不支持。
2. Stream 比 for 循环可读性更好。

### 子流 & 组合流
子流：stream.skip / stream.limit，ifeve 原文中有详细介绍
组合流：Stream.concat(Stream a, Stream b)：使用 Stream 的静态方法 concat 将两个流连接在一起。

### 状态转化
* 无状态转换 : stream.filter、stream.map、stream.flatMap 等都是无状态的，从一个filter／map 的 stream 中获取某个元素时，结果并不依赖于之前的元素。

* 有状态转换 : distinct 方法会根据原始流中的元素返回一个具有相同顺序、抑制重复元素的新流，这意味着该流必须记住之前已读取的元素。包括 sorted ，也是有状态转换。

### 收集结果
stream.toArray()：收集到 Object[] 中
stream.collect(Collectors.toSet());
stream.collect(Collectors.toList());
stream.collect(Collectors.toCollection(TreeSet::new));指定 collection 的类型
stream.collect(Collectors.joining()):将流中的所有字符串连接起来
stream.collect(Collectors.joining(",")):将流中的所有字符串连接起来，并按, 分隔
stream.collect(Collectors.toMap(Function keyMapper, Function valueMapper));

### 原始类型流
可以创建 Stream<Integer>，但是这种效率不高，需要包装。Stream 提供了 IntStream、LongStream、DoubleStream，直接存储原始类型。

### 并行执行
默认情况下创建的流是串行流。collection.parallelStream()可创建一个并行流。 stream.parallel() 可将串行流转化为并行流。

只要在终止方法执行时，流处于并行模式，那么所有延迟执行的流操作都会被并行执行。

需要确保传递给并行流操作的函数都是线程安全的。

在合适的场景下使用并行计算，可以大大提高运行效率。

### 终止流
stream.forEach()、stream.forEachOrdered()：都是终止流操作。
stream.peek() 不会终止流。

## 函数式编程
[CoolShell的一篇文章](http://coolshell.cn/articles/10822.html)

