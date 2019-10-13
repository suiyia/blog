title: JVM学习——运行时数据区
author: Answer
tags:
  - JVM
categories:
  - JVM
date: 2019-10-11 22:37:00
---
# 介绍

> Java虚拟机在执行 Java 程序的过程中会把它所管理的内存划分为若干个不同的数据区域。这些区域都有各自的用途，已经创建和销毁时间，有的区域随着虚拟机进程的启动而创建，有些区域则依赖用户线程的启动和结束而创建和销毁。

Java 文件中定义的方法、变量、常量等进入内存后，存放的区域以及对应的变化。

Java 虚拟机内存空间就是一块普通的内存空间，只是这部分处理 Java 程序。

Java 虚拟机内存结构按线程数据是否共享分为两部分：

- 线程共享
  - 堆
  - 方法区
  - 常量池

- 线程私有
  - PC 寄存器
  - 本地方法栈
  - Java 虚拟机栈
  

# 各个分区的细节

基于 JDK 1.8

## 堆：存放对象的地方



![upload successful](\images\java8heap.png)

Dog dog = new Dog();   new Dog() 这个对象就在堆上

堆进一步可分为：年轻代、老年代。年轻代对象大多数存活时间短，很快会被垃圾回收。年轻代存活久的会进入老年代，当然有些对象比较大，会直接进入老年代。

年轻代内存分区进一步可分为 Eden、survivor0、survivor1 三个部分，内存默认大小比例为 8:1:1。

垃圾回收的时候， Eden 和其中一个 surivor 内的对象大部分会被清除，而没清除的放入另外一个 surivor 中（垃圾回收算法——复制算法）。

JDK 1.8 之前，堆中有永久代（Perm GEN）这个概念，JDK 1.8 开始，去掉永久代，取而代之的是元空间（MetaSpace），元空间不属于堆，元空间的大小仅受限于机器的内存大小。

当 Perm GEN 属于堆的时候，有时候由于堆内存大小不足，报 "java.lang.OutOfMemoryError: PermGen space" 错误，现在这个错误将不复存在；当然对应的 -XX:MaxPermSize 也将不起作用

[One important change in Memory Management in Java 8](http://karunsubramanian.com/websphere/one-important-change-in-memory-management-in-java-8/)

[Java 8: From PermGen to Metaspace](https://dzone.com/articles/java-8-permgen-metaspace)

[Java8内存模型—永久代(PermGen)和元空间(Metaspace)](https://www.cnblogs.com/paddix/p/5309550.html)


## 方法区
存放类的结构信息：运行时常量池、方法、构造方法

方法区是一个接口概念，是 Java 虚拟机定义的一个规范；而永久代、元空间则被认为是方法区这个规范的实现，并且永久代 HotSpot 虚拟机才有的概念，其它虚拟机没有。

JDK 1.7 时，永久代包含类的元信息、静态变量、常量池（Constant Pool Table）；

JDK 1.8 开始元空间（元空间不属于堆，在机器的本地内存中）存储类的元信息。静态变量、常量池并入堆中。

常量池：用于存放编译期生成的各种字面量和符号引用，这部分内容将在类加载后存放到方法区的运行时常量池中。

[jdk8之后永久代去哪了？ - Mr Zeng的回答 - 知乎](
https://www.zhihu.com/question/39990490/answer/369690291)


## 常量池

[JVM常量池浅析](https://www.jianshu.com/p/cf78e68e3a99)

- Class 文件常量池

class 文件中有定义，包括字面量和符号引用
	 
     - 字面量 指数据的值
     
     - 符号引用 包含类和接口的全限定名，字段的名称和描述符，方法的名称和描述符

- 运行时常量池，JDK1.7 及之后版本的 JVM 已经将运行时常量池从方法区中移了出来，在 Java 堆（Heap）中开辟了一块区域存放运行时常量池。同时在 jdk 1.8中移除整个永久代，取而代之的是一个叫元空间（Metaspace）的区域

- 全局字符串常量池

- 基本类型包装类对象常量池


## PC（Program Counter） 程序计数器

指的是保存线程当前正在执行的方法。唯一一个无 OOM 的区域

如果这个方法不是 native 方法，那么 PC 寄存器就保存 Java 虚拟机正在执行的字节码指令地址。如果是 native 方法，那么 PC 寄存器保存的值是 undefined。

任意时刻，一条 Java 虚拟机线程只会执行一个方法的代码，而这个被线程执行的方法称为该线程的当前方法，其地址被存在 PC 寄存器中。


## 本地方法栈

「当 Java 虚拟机使用其他语言（例如 C 语言）来实现指令集解释器时，也会使用到本地方法栈。如果 Java 虚拟机不支持 natvie 方法，并且自己也不依赖传统栈的话，可以无需支持本地方法栈。」


## Java 虚拟机栈

一个线程就包含一个虚拟机栈，与线程共存亡。

虚拟机栈有大小，如果栈的深度大于 JVM 所允许的范围，会抛出 StackOverflowError；如果申请不到额外空间，会抛出 OutOfMemoryError，这两种错误如果要捕获，需使用 Throwable 进行捕获。

线程执行一个方法时，虚拟机栈就会创建一个栈帧，栈帧内包含局部变量表，操作数栈。方法执行完退出，该栈帧就会清除。

栈帧内容：

- 局部变量表：基本数据类型、对象引用（reference 类型，它不同于对象本身，可能是一个指向对象起始地址的引用指针，也可能是指向一个代表对象的句柄或其他与此对象相关的位置）

- 操作数栈

- 方法出口


### 博客参考

[JVM系列第6讲：Java 虚拟机内存结构](https://www.cnblogs.com/chanshuyi/p/jvm_serial_06_jvm_memory_model.html)