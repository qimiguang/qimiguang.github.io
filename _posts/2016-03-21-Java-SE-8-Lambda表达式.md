---
title: 'Java SE 8: Lambda表达式'
date: 2016-03-21 22:53:52
tags:
---

## 1. Lambda
lambda 是指`带有参数的代码块`。在函数为 first class 的语言里, lambda是一个很自然的语法糖。

lambda 表达式的声明方式比较简单，由`参数`、`方法体`和`自由变量`三部分组成

* 参数：一般情况不需要包含类型声明，可以进行自动推断
* 方法体：表达式或代码块
* 自由变量：不是入参，并且没有在代码块中定义的变量

当你想要代码块在以后的某个时间点执行时，可以使用 lambda 表达式
lambda 表达式可以在闭包作用域中访问 final 变量

## 2. Java8 接口新特性
在 Java8 中，接口不仅可以声明抽象方法，还可以有具体的实现，如 default function / static function.
### 2.1 default function
该特性和 lambda 无直接关系，是 Java8 对 interface 做的增强。

interface default function 的主要目标：

* 解决接口的演化问题。Java 开发中推荐面向接口而不是实现来编程，不过接口本身的演化比较困难。在之前的 jdk 版本中，当接口发生变化时，该接口的所有实现类都需要做出相应的修改。现在，当往一个接口中添加新的方法时，可以提供该方法的默认实现。对于已有的接口使用者来说，代码可以继续运行。新的代码则可以使用该方法，也可以覆写默认的实现。
* 实现行为的多继承。Java语言只允许类之间的单继承关系，但是一个类可以实现多个接口。在默认方法引入之后，接口中不仅可以包含变量和方法声明，还可以包含具体方法实现。

默认方法的一些规则：

如果一个父接口定义了一个默认方法，

* 另一个父类提供了具体的实现，那么接口中具有相同名称和参数的默认方法会被忽略。`类优先策略`，可以保证兼容 Java7 
* 另一个父接口也提供了具有相同名称和参数类型的方法（不管该方法是否是默认方法），子类必须通过覆盖该方法来解决冲突

### 2.2 static function
在 Java8 中，可以为接口添加静态方法，虽然这看起来违反了接口作为一个抽象定义的理念。

能通过将静态工具方法添加到接口中，来帮助减少静态工具类。

### 2.3 functional interface
functional interface(函数式接口)，a.k.a. SAM（Single Abstract Method)，即只包含一个`抽象方法`的接口。
在 Java8 中，为了支持 lambda 表达式，引入了这个概念。

## 3. Java8 中的 Lambda
Java8 之前，不能将代码块到处传递。由于 Java 是面向对象语言，因此不得不构建一个属于某个类的对象，由它的某个方法来包含所需的代码。如 Runnable。

在其他的一些 FP 语言中，函数作为 first class, 是可以直接使用并传递的，这样大大简化了代码的编写。

在 Java8 中，为了更贴近 FP 的风格，作出了一些改变，如 lambda。`我们应该把 lambda 表达式想象成一个函数，而不是一个对象，并记住它可以被转化为 API 中对应的函数式接口。`

Java8 新增了 java.util.function 包，里面定义了很多通用的函数式接口。开发人员也可以创建新的函数式接口，但最好在接口上使用注解 @FunctionalInterface 进行声明，以免团队的其他人员错误地往接口中添加新的抽象方法。

多说无益，上代码：
之前普通 Java 匿名内部类实现 Runnable：

```java
new Thread(new Runnable() {
   public void run() {
       System.out.println("Run!");
   }
}).start();
```
Lambda 表达式实现 Runnable：

```java
new Thread(() -> {
   System.out.println("Run!");
}).start();
```

### 3.1 变量作用域
前文提到，lambda 表达式除了参数和代码块外，还有自由变量。

```java
public static void repeatMessage(String text, int count) {
	Runnable r = () -> {
		for(int i = 0; i < count; i++) {
			System.out.println(text);
			Thread.yield();
		}
	}
	new Thread(r).start();
}
```
text 和 count 没有在 lambda 表达式中被定义，而该表达式可能会在 repeatMessage 返回后才执行，此时参数已经消失了。实际上，这些参数复制后被 lambda 表达式捕获了，同时参数必须是 final 修饰。在 java8 对这个限制做了优化，可以不用显示使用 final 修饰，但是编译器隐式当成 final 来处理。

### 3.2 Closure(闭包)
含有自由变量的函数被称之为闭包（Closure），这个被引用的自由变量将和这个函数一同存在，即使已经离开了创造它的环境。在 Java 中， Lambda表达式就是闭包。事实上，内部类一直都是闭包。

介绍闭包的两篇文章：
[wikipedia : closure](https://zh.wikipedia.org/wiki/%E9%97%AD%E5%8C%85_(%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%A7%91%E5%AD%A6))
[mozilla : closure](https://developer.mozilla.org/cn/docs/Web/JavaScript/Closures)


### 3.3 异常处理
如果 lambda 表达式中可能抛出一个 checked exception,那么该 exception 需要在目标接口的抽象方法中声明。

### 3.4 Method reference
方法引用可以在不调用某个方法的情况下引用一个方法。方法引用是另外一种实现 functional interface 的方法。在某些情况下，方法引用可以进一步简化代码。

* objectName::instanceMethod

```java
// (args) -> expression.instanceMethod(args); 
x -> System.out.println(x)  // lambda 语法
System.out::println  // method reference 语法
```
* ClassName::staticMethod

```java
// (args) -> ClassName.staticMethod(args); 
(x, y) -> Math.max(x,y) // lambda 语法糖
Math::max  // method referenca 语法
```
* ClassName::instanceMethod

```java
// (instance, args) -> instance.instanceMethod(args);
x -> x.toLowerCase() // lambda 语法
String::toLowerCase  // method reference 语法
```
### 3.5 Construct reference
构造方法引用可以在不创建对象的情况下引用一个构造方法

```java
// x -> new BigDecimal(x)
BigDecimal::new
```


