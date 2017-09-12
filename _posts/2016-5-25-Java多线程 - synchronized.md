---
layout: post
title: Java多线程 - synchronized
date:   2016-5-25
categories: "java多线程"
tags: [java, jvm]
author: Mark
comment: false
---

> 在Java中，使用synchronized关键字是最基本的互斥同步手段，synchronized可用于普通方法，静态方法，代码块，分别锁的是当前实例对象，当前类的class对象和括号中的实例对象，底层实现方法除了细节差异外基本一致，都是使用监视器monitor。

首先使用javap查看下面代码的class文件信息：

``` javas
public class Test{
	public void test(){
		synchronized (this){
			System.out.println("Hello");
		}
	}
}
```
javap –c Test，反汇编信息如下图：

![ ][1]
　　上图第一处红框中执行monitorenter指令，尝试获取对象的锁，如果对象没有被锁定，或者当前线程已经拥有该对象的锁，把锁的计算器加1。相应的，第二处红框中在执行monitorexit指令时会将锁计数器减1，当计算器为0时，锁就被释放。JVM需要保证每一个monitorenter都有一个monitorexit与之相对应。任何对象都有一个monitor与之相关联，当且一个monitor被持有之后，他将处于锁定状态。这里需要有两点需要注意：<br>

　　1. synchronized同步块对同一个线程来说可以重入；<br> 
　　2. 同步块在已进入的线程执行完之前，会阻塞后面其他线程进入；<br>

　　上述monitorenter和monitorexit字节码依赖于底层的操作系统来实现的，阻塞或者唤醒一个线程也需要依赖操作系统，这就需要从用户态切换到内核态来执行，十分的消耗CPU时间，这可能比执行用户代码的时间更长，所以synchronized是java中一个重量级（HeavyWeight）的操作。而在JDK1.6中针对synchronized锁做了大量优化，下面做下简单介绍：
#### 自旋锁
　　频繁的阻塞和唤醒对CPU来说是一件负担很重的工作，势必会给系统的并发性能带来很大的压力。很多时候，对象锁状态只会持续很短时间，为了这段很短的时间挂起和恢复线程并不值得，因此引入了自旋锁。所谓自旋锁，就是让该线程等待一段时间，不会被立即挂起，看持有锁的线程是否会很快释放锁。如何等待？我们只需让线程执行一段忙循环（自旋）即可。JDK1.6之后，默认开启自旋锁，自旋次数默认为10次，可以通过参数-XX:preBlockSpin来调整。<br>
　　然而自旋等待不能代替阻塞，虽然避免了线程切换的开销，但无意义的循环也要占用处理器时间。这要取决于锁被占用的时间，如果时间很短，自旋等待的效果就会非常好，反之，如果锁被占用的时间较长，自旋结束不久就释放锁，或者还未释放，那只会白白消耗处理器资源，带来性能的浪费，于是又引入了自适应自旋锁。

  
  

  [1]: http://wx3.sinaimg.cn/mw690/907499d8gy1fjh3dstq72j20hs0fdwen.jpg