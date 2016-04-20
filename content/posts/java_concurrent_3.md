+++
author = "Ryan Zhao"
categories = ["Java","并发编程"]
date = "2016-04-19T20:44:55+08:00"
description = "JDK 1.5之后引入的java.concurrent包中提供的若干同步工具的实现机制。"
keywords = ["Java并发编程","高性能并发"]
mathjax = false
tags = ["并发","Java"]
title = "Java极致并发：（三）java.util.concurrent.locks"

+++

<div>
<span><div>相对于语义较为原始的synchronized、wait、notify等同步工具，JDK1.5（JSR166 ）之后引入了新的middle-level的同步工具java.util.concurrent，其内部包含了诸如信号量、Latch、屏障等高级语义的同步工具。</div><div><br/></div><div>这些同步器都是基于AbstractQueuedSynchronizer而实现的，关于具体细节请参考论文“The java.util.concurrent Synchronizer Framework”。</div><div><br/></div><div>关于优化synchronized的讨论有很多，但大都集中在降低synchronized在单线程中的负载，JVM的主要优化策略集中在优化无竞争条件下的性能，而并不是高度竞争条件下的synchronized性能。而java.util.concurrent中提供的同步工具的主要目标是解决可伸缩性，即强烈竞争环境下的程序性能。本系列文章后面我们会对各种同步工具的性能进行比较，到时候可以发现，在强竞争的条件下，Lock相对于synchronized会有较为明显的性能优势。</div><div><br/></div><div>朴素来讲，同步器的可以如下方法实现。</div><div><br/></div><div>acquire操作：</div><div><img src="/images/acquire.png" type="image/png" style="height: auto;"/></div><div><br/></div><div>release操作：</div><div><img src="/images/release.png" type="image/png" style="height: auto;"/></div><div><br/></div><div>上面的两种操作非常简单，但需要如下三种工具支持</div><div><br/></div><ol><li>同步状态的原子性管理</li><li>线程的阻塞和唤醒</li><li>等待队列的管理</li></ol><div><br/></div><div>一、同步状态的原子性管理</div><div><br/></div><div>同步状态即目前acquire同步器的线程个数，AbstractQueuedSynchronizer使用volative int类型来保证get、set同步状态操作的原子性。同时提供了compareAndSetState操作，来提供加、减操作的原子性（关于如何使用CAS操作提供原子性，会在下文中讲到）。</div><div><br/></div><div>二、线程的阻塞和唤醒</div><div><br/></div><div>如何阻塞和唤醒一个线程？Thread.suspend &amp; Thread.resume 是一个解决方案，但是这个方案会导致一个竞争问题。考虑当线程A已经加入阻塞队列，但尚未被阻塞。另一个线程B release时调用A.resume，而后A线程调用A.suspend被阻塞。此时A将永久被阻塞。</div><div><br/></div><div>为了解决这个问题，JSR166引入了LockSupport类，其提供park和unpark操作（内部通过调用Unsafe.park &amp; unpark native method）。park对阻塞当前线程，unpark唤醒当前线程。若在一个线程上首先执行unpark，然后执行park时，线程并不会被阻塞（即先前的unpark起到了作用），若再次执行park，此时线程会被阻塞（先前的unpark操作已经被消耗掉）。</div><div><br/></div><div>三、等待队列的管理</div><div><br/></div><div>整个同步器框架的核心是等待队列的管理。AbstractQueuedSynchronizer内部使用一个FIFO队列，而不是优先级队列。此外，该队列是一个no-blocking数据结构，其内部并不使用锁来实现。</div><div><br/></div><div>关于该队列的实现方式，后文会有单独的篇章来分析，有兴趣的同学可以自己看看代码，也可以参考<a href="http://javarticles.com/2012/10/abstractqueuedsynchronizer-aqs.html">这篇文章</a>。</div><div><br/></div><div><br/></div><div><br/></div><div><br/></div><div><br/></div></span>
</div>