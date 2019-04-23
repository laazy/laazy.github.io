---
layout: post
title: java 内存管理机制
tags: java note
---
# Java Java虚拟机学习之 内存管理机制

## 内存模型

Java的内存分为几个部分，分别是：
- 程序计数器
- 虚拟机栈
- 本地方法区
- 方法区
- 堆

### 程序计数器

这是一块较小的内存空间，保存每个线程的字节码行号，此区域为线程私有。
值得注意的是，如果执行一个Java方法，则其中保存虚拟机字节码指令地址，若执行一个Native方法，则为空。

此外，此区域为唯一没有规定任何`OutOfMemoryError`的区域。

### Java虚拟机栈

Java虚拟机栈也是线程私有的，生命周期与线程相同。
每个方法在执行时都会在虚拟机栈中创建一个**栈帧**，其中保存**局部变量表、操作数栈、动态链接、方法出口等信息**，方法执行对应栈帧入栈到出栈。

局部变量表放置编译时可知的基本数据类型和对象引用，该空间可在编译时确定并分配。

当线程请求的栈深度大于虚拟机所允许深度，抛出`StackOverflowError`，若虚拟机栈可动态扩展，则无法扩展时抛出`OutOfMemoryError`。

### 本地方法栈

本地方法栈和虚拟机栈类似，不过仅为Native方法服务。
虚拟机规范对本地方法栈没有强制规定，因此具体的虚拟机实现不同。

本地方法栈也会抛出`StackOverflowError`和`OutOfMemoryError`。

### Java堆

Java堆是Java虚拟机管理的最大的一块内存（对绝大多数应用而言），该区域被所有线程共享，几乎所有对象实例都在此处分配内存。
但随着逃逸分析、栈上分配、标量替换等技术的发展，对象也可分配在栈上了。

Java堆是垃圾收集的主要区域，由于采用分带收集算法，所以Java堆中可细分为：老年代和新生代。Java堆可出于物理上不连续的内存空间，可为固定大小，也可为可拓展。主流实现为可拓展。

若堆中无法完成实例分配，且无法拓展时，会抛出`OutOfMemoryError`。

### 方法区

方法区也是各个线程共享的内存区域，用于存储被加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。

该区域的垃圾收集效果不好，但垃圾回收是必要的。Sun公司的BUG列表中有多个严重的BUG就是因为低版本的HotSpot虚拟机无法对此区域完全回收而内存泄漏。

此区域无法满足内存分配时，抛出`OutOfMemoryError`。

#### 运行时常量池

此区域是方法区的一部分，Class文件中的常量池（存放编译时生成的字面量和符号引用），在类加载后放入此区域。

Class文件的格式被虚拟机规范严格规定，但此区域没有细节要求。
此区域同时具备动态性，运行时的常量也可放入此区域，例如`String`类的`intern()`方法。

当此区域无法分配内存抛出`OutOfMemoryError`。

### 直接内存

直接内存不是虚拟机运行时数据区的一部分，也不在虚拟机规范中，当时可能被Native方法在Java虚拟机之外申请和访问。

## 对象访问

对于`Object obj = new Object()`而言，本地变量表中会有一个`reference`类型的数据`obj`，即`Object`类的一个实例的引用，而该实例保存在堆中。同时，堆中还需要保存能够查询到该类型的数据的信息，这部分信息被放置在方法区中。

虚拟机规范中只规定`reference`为指向对象的引用，所以不同实现不同，主流的有两种：**句柄访问**和**直接访问**。

### 句柄访问

此种方式会在Java堆中分配一块内存来作为句柄池，`reference`保存句柄地址，句柄中保存对象实例和类型数据的具体地址信息。

此种方式当对象移动时，只需修改句柄中实例地址即可。但由于是间接访问，速度会慢。

### 直接指针访问

此方式的`reference`会保存Java堆中的一个实例的地址，在堆中的实例需要保存在方法区中的该类型数据的地址。

此方式由于节省一次指针访问，所以速度快。但是若对象移动，需要修改所有引用的数据。






