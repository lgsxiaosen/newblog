---
title: 【转】简洁又快速地处理集合——java8 Stream（下）
date: 2018-08-19 10:26:04
tags:
- java
- stream
- 转载
category:
- 技术文章
---

> 作者：Howie_Y

> 主页：www.jianshu.com/u/79638e5f0743

上一篇文章我讲解 Stream 流的基本原理，以及它的基本方法使用，本篇文章我们继续讲解流的其他操作 

*值得注意的是：学习 Stream 之前必须先学习 lambda 的相关知识。本文也假设读者已经掌握 lambda 的相关知识。*

**本篇文章主要内容：**

1. 一种特化形式的流——数值流
2. Optional 类
3. 如何构建一个流
4. collect 方法
5. 并行流相关问题

**一. 数值流**

前面介绍的如

```java
int sum = list.stream().map(Person::getAge).reduce(0, Integer::sum);
```

计算元素总和的方法其中暗含了装箱成本，map(Person::getAge) 方法过后流变成了 Stream类型，而每个 Integer 都要拆箱成一个原始类型再进行 sum 方法求和，这样大大影响了效率。

针对这个问题 Java 8 有良心地引入了数值流 IntStream, DoubleStream, LongStream，这种流中的元素都是原始数据类型，分别是 int，double，long

**1. 流与数值流的转换**

**流转换为数值流**

- mapToInt(T -> int) : return IntStream
- mapToDouble(T -> double) : return DoubleStream
- mapToLong(T -> long) : return LongStream

```java
IntStream intStream = list.stream().mapToInt(Person::getAge);
```

当然如果是下面这样便会出错 

```java
LongStream longStream = list.stream().mapToInt(Person::getAge);
```

因为 getAge 方法返回的是 int 类型（返回的如果是 Integer，一样可以转换为 IntStream）

**数值流转换为流**

很简单，就一个 boxed

```jav
Stream<Integer> stream = intStream.boxed();
```

**2. 数值流方法**

下面这些方法作用不用多说，看名字就知道：

- sum()
- max()
- min()
- average() 等...

**3. 数值范围**

IntStream 与 LongStream 拥有 range 和 rangeClosed 方法用于数值范围处理

- IntStream ： rangeClosed(int, int) / range(int, int)
- LongStream ： rangeClosed(long, long) / range(long, long)

这两个方法的区别在于一个是闭区间，一个是半开半闭区间：

- rangeClosed(1, 100) ：[1, 100]
- range(1, 100) ：[1, 100)

我们可以利用 IntStream.rangeClosed(1, 100) 生成 1 到 100 的数值流

```java
//求 1 到 10 的数值总和：
IntStream intStream = IntStream.rangeClosed(1, 10);
int sum = intStream.sum();
```

**二. Optional 类**

NullPointerException 可以说是每一个 Java 程序员都非常讨厌看到的一个词，针对这个问题， Java 8 引入了一个新的容器类 Optional，可以代表一个值存在或不存在，这样就不用返回容易出问题的 null。之前文章的代码中就经常出现这个类，也是针对这个问题进行的改进。

Optional 类比较常用的几个方法有：

- isPresent() ：值存在时返回 true，反之 flase
- get() ：返回当前值，若值不存在会抛出异常
- orElse(T) ：值存在时返回该值，否则返回 T 的值

Optional 类还有三个特化版本 OptionalInt，OptionalLong，OptionalDouble，刚刚讲到的数值流中的 max 方法返回的类型便是这个

Optional 类其中其实还有很多学问，讲解它说不定也要开一篇文章，这里先讲那么多，先知道基本怎么用就可以。

**三. 构建流**

之前我们得到一个流是通过一个原始数据源转换而来，其实我们还可以直接构建得到流。

**1. 值创建流**

Stream.of(T...) ： Stream.of("aa", "bb") 生成流

```java
//生成一个字符串流
Stream<String> stream = Stream.of("aaa", "bbb", "ccc");
```

Stream.empty() : 生成空流

**2. 数组创建流**

根据参数的数组类型创建对应的流：

- Arrays.stream(T[ ])
- Arrays.stream(int[ ])
- Arrays.stream(double[ ])
- Arrays.stream(long[ ])

值得注意的是，还可以规定只取数组的某部分，用到的是Arrays.stream(T[], int, int)

```java
//只取索引第 1 到第 2 位的：
int[] a = {1, 2, 3, 4};
Arrays.stream(a, 1, 3).forEach(System.out :: println);
//打印 2 ，3
```

**3. 文件生成流** 

```java
Stream<String> stream = Files.lines(Paths.get("data.txt"));
```

每个元素是给定文件的其中一行

**4. 函数生成流**

两个方法：

- iterate ： 依次对每个新生成的值应用函数
- generate ：接受一个函数，生成一个新的值

```java
Stream.iterate(0, n -> n + 2)
//生成流，首元素为 0，之后依次加 2
Stream.generate(Math :: random)
//生成流，为 0 到 1 的随机双精度数
Stream.generate(() -> 1)
//生成流，元素全为 1
```

**四. collect 收集数据**

coollect 方法作为终端操作，接受的是一个 Collector 接口参数，能对数据进行一些收集归总操作

**1. 收集**

最常用的方法，把流中所有元素收集到一个 List, Set 或 Collection 中

- toList
- toSet
- toCollection
- toMap

```java
List newlist = list.stream.collect(toList());
//如果 Map 的 Key 重复了，可是会报错的哦
Map<Integer, Person> map = list.stream().collect(toMap(Person::getAge, p -> p));
```

**2. 汇总**

**（1）counting**

用于计算总和：

```java
long l = list.stream().collect(counting());
```

没错，你应该想到了，下面这样也可以： 

```java
long l = list.stream().count();
```

推荐第二种

**（2）summingInt ，summingLong ，summingDouble**

summing，没错，也是计算总和，不过这里需要一个函数参数

计算 Person 年龄总和：

```java
int sum = list.stream().collect(summingInt(Person::getAge));
```

当然，这个可以也简化为： 

```java
int sum = list.stream().mapToInt(Person::getAge).sum();
```

除了上面两种，其实还可以： 

```java
int sum = list.stream().map(Person::getAge).reduce(Interger::sum).get();
```

推荐第二种

由此可见，函数式编程通常提供了多种方式来完成同一种操作

**（3）averagingInt，averagingLong，averagingDouble**

看名字就知道，求平均数

```java
Double average = list.stream().collect(averagingInt(Person::getAge));
```

当然也可以这样写 

```java
OptionalDouble average = list.stream().mapToInt(Person::getAge).average();
```

不过要注意的是，这两种返回的值是不同类型的

**（4）summarizingInt，summarizingLong，summarizingDouble**

这三个方法比较特殊，比如 summarizingInt 会返回 IntSummaryStatistics 类型

```java
IntSummaryStatistics l = list.stream().collect(summarizingInt(Person::getAge));
```

IntSummaryStatistics 包含了计算出来的平均值，总数，总和，最值，可以通过下面这些方法获得相应的数据 

```java
getAverage()                                         double
getCount()                                           long
getMax()										 int
getMin()										 int
getSum()										 long
```

**3. 取最值**

maxBy，minBy 两个方法，需要一个 Comparator 接口作为参数

```java
Optional<Person> optional = list.stream().collect(maxBy(comparing(Person::getAge)));
```

我们也可以直接使用 max 方法获得同样的结果

```java
Optional<Person> optional = list.stream().max(comparing(Person::getAge));
```

**4. joining 连接字符串**

也是一个比较常用的方法，对流里面的字符串元素进行连接，其底层实现用的是专门用于字符串连接的 StringBuilder

```java
String s = list.stream().map(Person::getName).collect(Collectors.joining());
//结果：jackmiketom
String s = list.stream().map(Person::getName).collect(Collectors.joining(","));
//结果：jack,mike,tom
```

joining 还有一个比较特别的重载方法：

```java
String s = list.stream().map(Person::getName).collect(Collectors.joining(" and ", "Today ", " play games."));
//结果：Today jack and mike and tom play games.
```

即 Today 放开头，play games. 放结尾，and 在中间连接各个字符串

**5. groupingBy 分组**

groupingBy 用于将数据分组，最终返回一个 Map 类型

```java
Map<Integer, List<Person>> map = list.stream().collect(groupingBy(Person::getAge));
```

例子中我们按照年龄 age 分组，每一个 Person 对象中年龄相同的归为一组

另外可以看出，Person::getAge 决定 Map 的键（Integer 类型），list 类型决定 Map 的值（List类型）

**多级分组**

groupingBy 可以接受一个第二参数实现多级分组：

```java
Map<Integer, Map<T, List<Person>>> map = list.stream().collect(groupingBy(Person::getAge, groupingBy(...)));
```

其中返回的 Map 键为 Integer 类型，值为 Map<t, list

**按组收集数据**

```java
Map<Integer, Integer> map = list.stream().collect(groupingBy(Person::getAge, summingInt(Person::getAge)));
```

该例子中，我们通过年龄进行分组，然后 summingInt(Person::getAge)) 分别计算每一组的年龄总和（Integer），最终返回一个 Map

根据这个方法，我们可以知道，前面我们写的：

```java
groupingBy(Person::getAge)
```

其实等同于：

```java
groupingBy(Person::getAge, toList())
```

**6. partitioningBy 分区**

分区与分组的区别在于，分区是按照 true 和 false 来分的，因此partitioningBy 接受的参数的 lambda 也是 T -> boolean

```java
//根据年龄是否小于等于20来分区
Map<Boolean, List<Person>> map = list.stream()
                                    .collect(partitioningBy(p -> p.getAge() <= 20));
//打印输出
{
   false=[Person{name='mike', age=25}, Person{name='tom', age=30}], 
   true=[Person{name='jack', age=20}]
}
```

同样地 partitioningBy 也可以添加一个收集器作为第二参数，进行类似 groupBy 的多重分区等等操作。

**五. 并行**

我们通过 list.stream() 将 List 类型转换为流类型，我们还可以通过 list.parallelStream() 转换为并行流。因此你通常可以使用 parallelStream 来代替 stream 方法

并行流就是把内容分成多个数据块，使用不同的线程分别处理每个数据块的流。这也是流的一大特点，要知道，在 Java 7 之前，并行处理数据集合是非常麻烦的，你得自己去将数据分割开，自己去分配线程，必要时还要确保同步避免竞争。

Stream 让程序员能够比较轻易地实现对数据集合的并行处理，但要注意的是，不是所有情况的适合，有些时候并行甚至比顺序进行效率更低，而有时候因为线程安全问题，还可能导致数据的处理错误，这些我会在下一篇文章中讲解。

比方说下面这个例子

```java
int i = Stream.iterate(1, a -> a + 1).limit(100).parallel().reduce(0, Integer::sum);
```

我们通过这样一行代码来计算 1 到 100 的所有数的和，我们使用了 parallel 来实现并行。

但实际上是，这样的计算，效率是非常低的，比不使用并行还低！一方面是因为装箱问题，这个前面也提到过，就不再赘述，还有一方面就是 iterate 方法很难把这些数分成多个独立块来并行执行，因此无形之中降低了效率。

**流的可分解性**

这就说到流的可分解性问题了，使用并行的时候，我们要注意流背后的数据结构是否易于分解。比如众所周知的 ArrayList 和 LinkedList，明显前者在分解方面占优。

**我们来看看一些数据源的可分解性情况**

数据源可分解性ArrayList极佳LinkedList差IntStream.range极佳Stream.iterate差HashSet好TreeSet好顺序性。

除了可分解性，和刚刚提到的装箱问题，还有一点值得注意的是一些操作本身在并行流上的性能就比顺序流要差，比如：limit，findFirst，因为这两个方法会考虑元素的顺序性，而并行本身就是违背顺序性的，也是因为如此 findAny 一般比 findFirst 的效率要高。

**六. 效率**

最后再来谈谈效率问题，很多人可能听说过有关 Stream 效率低下的问题。其实，对于一些简单的操作，比如单纯的遍历，查找最值等等，Stream 的性能的确会低于传统的循环或者迭代器实现，甚至会低很多。

但是对于复杂的操作，比如一些复杂的对象归约，Stream 的性能是可以和手动实现的性能匹敌的，在某些情况下使用并行流，效率可能还远超手动实现。好钢用在刀刃上，在适合的场景下使用，才能发挥其最大的用处。

函数式接口的出现主要是为了提高编码开发效率以及增强代码可读性；与此同时，在实际的开发中，并非总是要求非常高的性能，因此 Stream 与 lambda 的出现意义还是非常大的。