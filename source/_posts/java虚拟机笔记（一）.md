---
title: '''jvm'''
date: 2018-07-28 08:43:53
tags:
- java
- jvm
- 虚拟机
- 读书笔记
category:
- 技术文章
---

### JDK和JRE区别

​	1、一般把Java程序设计语言，java虚拟机，java API类库这三部分统称为JDK（Java Development Kit），JDK是用于支持Java程序开发的最小环境。

​	2、Java API类库中Java SE API子集和Java虚拟机这两部分统称为JRE（Java Runtime Environment），JRE是支持Java程序运行的标准环境。
![java技术体系所包含的内容](https://raw.githubusercontent.com/lgsdaredevil/newblog/resource-newblog/source/favicons/article/java%E6%8A%80%E6%9C%AF%E4%BD%93%E7%B3%BB%E6%89%80%E5%8C%85%E5%90%AB%E7%9A%84%E5%86%85%E5%AE%B9.png)

## Java虚拟机内存区域

#### 一、程序计数器

​	可以看做当前线程所执行的字节码的行号指示器。为线程私有的内存。是在Java虚拟机内存区域唯一一个不会内存溢出的区域。

#### 二、Java虚拟机栈

​	Java虚拟机栈描述的是Java方法执行的内存模型：每个方法在执行的同时都会创建一个栈帧，用于存储局部变量表、操作数栈、动态链栈、方法出口等信息。Java虚拟机栈也是线程私有，生命周期和线程相同。

​	Java虚拟机栈中的局部变量表：存放编译期可知的各种基本数据类型、引用类型（reference）和runAddress类型。局部变量表所需要的内存空间在编译期间完成内存分配，在方法运行期间不会改变局部变量表的大小。

​	异常：

1）StackOverflowError：线程请求的栈的深度大于虚拟机所允许的深度。

2）OutOfMemoryError：扩展时无法申请到足够的内存。

#### 三、本地方法栈

​	与Java虚拟机栈类似，线程私有，会抛出StackOverflowError和OutOfMemoryError异常。

​	Java虚拟机栈区别是本地方法栈为虚拟机使用的Native方法服务，Java虚拟机栈为虚拟机执行Java方法服务。

#### 四、Java堆

​	存放对象实例，被所有线程共享，虚拟机启动时创建。Java堆又称GC堆，垃圾收集器管理的主要区域。

​	Java堆可以细分为：新生代和老年代。也可以分为Eden空间、From Survivor空间、To Survivor空间等。

如果在堆中没有内存完成实例分配，并且也无法再扩展，会抛出OutOfMemoryError异常

#### 五、方法区

​	用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。是线程共享的区域。

​	与java堆一样不需要连续的内存和可以选择的固定大小并且可以扩展。不同点是可以选择不实现垃圾收集，这个区域主要的内存回收目标是针对常量池的回收和对类型的卸载。

当方法区无法满足内存分配需求时，将抛出OutOfMemoryError异常

#### 六、运行时常量池

​	是方法区的一部分，用于存放编译期生成的各种字面量和符号引用，这部分内容将在类加载后进入方法区的运行时常量池中存放。具备动态特性，运行时也能将新的常量存放入池。同样在无法申请内存时会抛出OutOfMemoryError异常。

#### 七、直接内存

​	不是虚拟机运行数据区的一部分。NIO类可以使用Native函数库直接分配堆外内存，然后通过存储在Java堆中的DirectByteBuffer对象作为这块内存的引用进行操作，避免了在Java堆和Native堆中来回复制数据。也会抛OutOfMemoryErrory异常

