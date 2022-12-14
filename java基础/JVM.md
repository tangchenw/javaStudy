# 基本概念

JVM是可运行java代码的假想计算机，其组成包括了一套字节码指令集、一组寄存器、一个栈、一个堆、方法回收和一个存储方法域，jvm运行在操作系统上，它和硬件没有任何的交互，都是操作系统去和硬件进行交互。

java代码的执行：先通过编译器将.java文件编译成.class字节码文件，然后通过jvm中的解释器编译成特定机器上的机器码。

一个程序运行会产生虚拟机实例，多个程序启动就会产生多个虚拟机实例。程序退出或者关闭程序虚拟机实例就会消亡，多个虚拟机实例之间数据不共享。

JVM中的jvm线程和原生操作系统中的线程有直接的映射关系。当线程本地存储、缓冲区分配、同步对象、栈、程序计数器等准备好之后，就会创建一个操作系统原生线程。

当原生线程初始化完毕之后，就会调用java线程的run方法。当线程结束之后，会释放原生线程和java线程的全部资源。

## JVM线程

- 虚拟机线程：出现时间：JVM达到安全点操作。这些操作必须在独立的线程中运行，因为当堆修改无法进行时，线程都需要jvm处于安全点。安全点操作有：stop-the-world垃圾回收、线程栈dump、线程暂停、线程偏向锁解除。
- 周期性任务线程：该线程负责定时器事件（中断），用来调度周期性操作的执行。
- GC线程：这些线程支持JVM不同的垃圾回收活动。
- 编译器线程：这些线程运行时将字节码文件动态编译成本地平台相关的机器码。
- 信号分发线程：这个线程接收发送到jvm的信号并调度适当的JVM方法处理。



## IDEA调试OOM

在VM Options 中使用参数-Xms1024m -Xmx1024m -XX:+PrintGCDetails

**产生OOM异常时，Dump下来文件**

-Xms1m -Xmx8m -XX:+HeapDumpOnOutOfMemoryErrors

## JMM

1.什么是JMM？

JMM：（java memory model的缩写）

2.它干嘛的？

作用：缓存一致性协议，用于定义数据读写的规则（遵守，找到这个规则）

JMM定义了线程和主内存之间的抽象关系：线程之间的共享变量存储在主内存中，每个线程都有一个私有的本地内存

解决共享对象可见性这个问题：volilate

3.他该如何学习

# JVM内存区域

## 	线程私有

**程序计数器PC**

指向虚拟机字节码指令的位置；唯一一个无OOM的区域。

**虚拟机栈（方法栈，比如main方法）**

虚拟机栈和线程的生命周期相同

一个线程中，每调用一个方法创建一个栈帧

栈帧的结构：本地变量表、操作数栈、对运行时常量池的引用

异常：线程请求的栈深度大于JVM所允许的深度（StackOverFlowError)

若JVM允许动态扩展，但是无法申请到足够的内存（OutOfMemoryError)

**本地方法栈（本地方法也就是navite c++程序的方法？）：**

异常同虚拟机的异常

## 线程共享

**方法区（永久代）——运行时常量池**

**类实例区（java堆）**

新生代：eden、from survivor、to survivor

老年代

异常：OutOfMemoryError

## 直接内存——不受JVM GC管理

**线程周期：**

**线程私有**

线程私有数据区域生命周期和线程相同，依赖用户程序线程的启动/结束而创建/销毁。

**线程共享**

线程共享区域随虚拟机的启动/关闭而创建/销毁

**直接内存**

直接内存不是JVM运行时数据区的一部分，JDK1.4之后引入的基于Channel和Buffer的IO方式会使用到navite函数库直接分配堆外内存，然后使用DirectByteBuffer对象作为这块内存的引用进行操作，避免了在java堆和Native堆中来回复制数据，在某些场景中可以显著提高性能。

## 内存区域介绍

### 程序计数器（线程私有）

一块较小的内存空间，是当前线程所执行的字节码的行号指示器，每条线程都要有一个独立的程序计数器。正在执行java方法的话，计数器记录的是虚拟机字节码指令的地址（当前指令的地址）。如果是native方法的话，则为空。这块内存区域是唯一没有指定任何OutOfMemoryError情况的区域。

### 虚拟机栈（线程私有）

是描述java方法执行的内存模型，每个java方法在执行的同时都会创建一个栈帧，用于存储局部变量表、操作数栈、动态链接、方法出口等信息。每个方法从调用到执行完成的过程，就对应着栈帧在虚拟机栈中入栈到出栈的过程。

栈帧：用来存储数据和部分过程结果的数据结构，同时也被用来处理动态链接、方法返回值和异常分派。栈帧随着方法调用而创建，随着方法结束而销毁——无论方法是正常完成还是异常完成（抛出了在方法内未被捕获的异常）都算结束。

**本地方法区（线程私有）**

本地方法区和java stack作用类似，两者的区别是虚拟机栈为java方法服务，而本地方法区为native方法服务

**堆（线程共享）——运行时数据区**

是一块被线程共享的一块内存区域，创建的对象和数组都保存在java堆内存中，也是垃圾收集器进行垃圾收集的最重要区域。现代VM采用分代收集算法，因此java堆从GC的角度还可以细分为：新生代（Eden区、From Survivor区和To Survivor区）和老年代

**方法区/永久代（线程共享）**

用于存储被JVM加载的类信息、常量、静态变量、即时编译器编译后的代码等数据。VM把GC分代收集扩展至方法区，即使用java堆的永久代来实现方法区，这样HotSpot的垃圾收集器就可以像管理java堆一样管理这块内存，而不用为方法区开发专门的内存管理器。

运行时常量池：是方法区的一部分，Class文件中除了有类的版本、字段、方法、接口等描述信息之外还有一部分是常量池。用于存放编译期生成的各种字面量和符号引用，这部分内容在类加载后存放到方法区的运行时常量池当中。

## JVM运行时内存

java堆从GC的角度还可以细分为：新生代和老年代

### 新生代

用来存放新生的对象。一般占据堆的1/3空间。由于频繁创建对象，所以新生代会频繁触发GC进行垃圾回收。新生代又分为Eden 区、 ServivorFrom、 ServivorTo 三个区。

**Eden区：**

java新对象的出生地，如果新创建的对象占用空间内存很大，则直接分配到老年代（新生代内存不够用）。当Eden区内存不够的时候就会触发MinorGC,对新生代区进行一次垃圾回收。因此新生代空间占有率越高，Minor GC越频繁。

**ServivorFrom**：

上一次GC的幸存者，作为这一次被GC扫描的对象。

**ServivorTo ：**

保留了一次MinorGC过程中的幸存者。

### MinorGC 的过程（复制->清空->互换）

MinorGC采用复制算法

1.eden、servivorFrom复制到servivorTo,年龄+1

首先，将eden、servivorFrom区域中存活的对象复制到ServivorTo区域（如果有对象的年龄达到了老年的标准，则赋值到老年代区），同时把这些对象的年龄+1（如果ServivorTo不够位置了就放到老年区）

2.清空eden、servivorFrom

清空eden、servivorFrom中的对象；

3.ServivorTo和servivorFrom互换

最后，把ServivorTo和ServivorFrom互换，原ServivorTo成为下一次GC时的ServivorFrom

### 老年代

主要存放应用程序中生命周期较长的内存对象

老年代的对象比较稳定，所以MajorGC不会频繁执行。在进行MajorGC前一般都先进行了一次MinorGC，使得有新生代的对象晋升入老年代，导致老年代空间不够从而触发MajorGC.当无法找到足够大的连续空间分配给新创建的较大对象时也会提前触发一次Major进行垃圾回收腾出空间。

**MajorGC：**

majorGC采用标记清除算法：首先扫描一次所有老年代，标记出存活的对象，然后回收没有标记的对象。MajorGC的耗时比较长，因为要扫描然后回收。Major会产生内存碎片，为了减少内存损耗，一般需要进行合并或者标记出来方便下次直接分配。当老年代也满了装不下的时候就会抛出OOM异常。

### 永久代

指内存的永久保存区域，主要存放Class和Meta（元数据）的信息，Class在被加载的时候被放入永久区域，它和存放实例的区域不同，GC不会在主程序运行期间对永久区域进行清理。所以这也导致了永久的区域会随着加载的Class的增多而胀满，最终抛出OOM异常。

**Java8与元数据**

Java8中，永久代已经被移除了，被一个称为“元数据区”（元空间）的区域所取代。元空间的本质和永久代类似，元空间和永久代之间最大的区别在于：元空间并不在虚拟机中，而是使用了本地内存。因此，默认情况下，元空间的大小仅受本地内存限制。类的元数据放入navite memeory,字符串池和类的静态变量放入java堆中，这样可以加载多少类的元数据就不再由MaxPermSize控制，而由系统中实际可用空间控制。