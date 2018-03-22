---
layout: post
title: Java并发 -JMM之happens-before
date:   2016-4-10
categories: "java并发"
tags: [java, jvm,并发]
author: Mark
comment: false
---

> 在JMM中，如果一个操作的执行结果需要对另一个操作可见，那么这两个操作之间必须存在happens-before关系。

happens-before原则非常重要，它是判断数据是否存在竞争、线程是否安全的主要依据。依靠这个原则，我们解决在并发环境下两操作之间是否可能存在冲突的所有问题。

#### 下面是Java内存模型下一些“天然的”happens-before关系，这些happens-before关系无需任何同步器协助就已经存在，可以在编码中直接使用。如果两个操作之间的关系不在此列，并且无法从下列规则推导出来的话，它们就没有顺序性保障，虚拟机可以对它们随意地进行重排序。

 1. **程序次序规则**： 一个线程内，按照程序代码顺序，书写在前面的操作先行发生于书写在后面的操作。即一段代码在单线程中执行结果是有序的，尽管虚拟机，处理器会对指令重排序，并不影响执行结果，在对线程环境下无法保证正确性。
 2. **锁定规则**：一个unlock操作先行发生于后面对同一个锁的lock操作。这里指的是同一个锁，而“后面”是时间上的先后顺序。
 3. **volatitle变量规则**：对于一个volatile变量的写操作先行发生于后面对这个变量的读操作，这里“后面”同样是指时间上的先后顺序。
 4. **线程启动规则**：Thread对象的start()方法先行发生于此线程的每一个动作。
 5. **线程终止规则**：线程中所有的操作都先行发生于对此线程的终止检测，我们通过Thread.join()方法结束，Thread.isAlive()方法的返回值等手段检测线程已经终止执行。
 6. **线程中断规则**：对线程interrupt()方法的调用先行发生于被中断线程代码检测到中断事件的发生，可以通过interrupted()方法检测到是否有中断发生。
 7. **对象终结规则**：一个对象的初始化完成（构造函数执行结束）先行发生于它的finalize()方法的开始。
 8. **传递性**：如果操作A先行发生于操作B，操作B先行发生于操作C，则操作A先行发生于操作C。

#### 上面八条是原生Java满足Happens-before关系的规则，但是我们可以对他们进行推导出其他满足happens-before的规则：

 1. 将一个元素放入一个线程安全的队列的操作Happens-Before从队列中取出这个元素的操作。
 2. 将一个元素放入一个线程安全容器的操作Happens-Before从容器中取出这个元素的操作。
 3. 在CountDownLatch上的倒数操作Happens-Before CountDownLatch#await()操作。
 4. 释放Semaphore许可的操作Happens-Before获得许可操作 Future表示的任务的所有操作Happens-Before Future#get()操作。
 5. 向Executor提交一个Runnable或Callable的操作Happens-Before任务开始执行操作。

#### 如果两个操作不存在上述（前面8条 + 后面6条）任一一个happens-before规则，那么这两个操作就没有顺序的保障，JVM可以对这两个操作进行重排序。如果操作A happens-before操作B，那么操作A在内存上所做的操作对操作B都是可见的。