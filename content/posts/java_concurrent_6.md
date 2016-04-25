+++
author = "Ryan Zhao"
categories = ["Java","并发编程"]
date = "2016-04-25T11:47:48+08:00"
description = "sync、Lock、CAS等同步机制的性能比较和分析。"
keywords = ["Java并发编程","高性能并发"]
mathjax = false
tags = ["并发","Java"]
title = "Java极致并发：（六）synchronized、Lock、CAS性能对比"

+++

<div>
<span><div>本文设计了简单的microbenchmark来对比各种同步机制的性能。具体代码可参考Github。</div><div>使用不同的同步机制，实现了线程安全的如下接口：</div><div><b><span style="font-size: 11.3pt;"><span style="font-family: Consolas;"><span style="color: rgb(0, 0, 128);"><br/></span></span></span></b></div><div><span style="font-size: 11.3pt;"><span style="font-family: Consolas;"><span style="color:#000080;font-weight:bold;">public interface </span>Counter {</span></span></div><div style="background-color:#ffffff;color:#000000;font-family:&apos;Consolas&apos;;font-size:11.3pt;">    <span style="color:#000080;font-weight:bold;">void </span>increment();<br/><br/>
    <span style="color:#000080;font-weight:bold;">long </span>get();</div><div><span style="font-size: 11.3pt;"><span style="font-family: Consolas;">}</span></span></div><div><span style="font-size: 11.3pt;"><span style="font-family: Consolas;"><br/></span></span></div><div>其中</div><ol><li>SyncCounter使用synchronized保护increment方法</li><li>FairLockCounter使用公平的ReentrantLock锁保护increment方法</li><li>NonFairLockCounter使用非公平的ReentrantLock锁保护increment方法</li><li>CASCounter使用AtomicLong提供的CompareAndSet方法，不断循环，直到increment成功。</li><li>BackOffCASCounter与CASCounter基本一致，但当CompareAndSet方法失败后，挂起该线程1ns。</li><li>AtomicCounter使用AtomicLong提供的incrementAndGet来实现increment方法。</li></ol><div><br/></div><div>测试机器：Intel Core i7-4710MQ</div><div>得到的性能结果如下图：</div><div><span style="font-size: 11.3pt;"><span style="font-family: Consolas;"><br/></span></span></div><div><img src="/images/counter_perf.png" type="image/png" style="height: auto;"/></div><div><br/></div><div>从图中可以观察到</div><div><br/></div><ol><li>FairLockCounter的伸缩性最差，公平的调度是需要付出代价的。</li><li>NonFairLockCounter与SyncCounter性能差不多，在高度竞争环境下，LockCounter的性能稍微好一些（也证明的java.util.concurrent的设计目标）。</li><li>CASCounter 与 AtomicCounter在原理上是一致的，都是轮询CAS操作状态。但AtomicCounter的性能较好。</li><li>BackOffCasCounter几乎不会因为竞争而导致性能受损，因为CAS失败的线程都挂起了，在短时间内不会再次产生竞争。</li><li>从CPU的使用率来看，在8线程情况下，SyncCounter、LockCounter、BackOffCASCounter都不会使CPU到达100%，因为同一时间几乎只有一个线程在运行。而CASCounter、AtomicCounter将会全力运转CPU。此时的CASCounter、AtomicCounter虽然性能较SyncCounter、LockCounter高，但也使用了成倍的CPU资源（8倍）。</li><li>竞争的强弱对吞吐量好像没有明显的影响，2线程下的吞吐量与16线程下的吞吐量基本一致。（这里的描述好像有些问题，尽管只有两个线程，但由于其一直在竞争，因此竞争是相当强烈的。现实环境中很少会碰到这种长时间持续竞争的情况。）</li></ol><div><br/></div><div>当CAS竞争非常激烈时，采用一些退让策略能显著地提高吞吐率。更多的退让策略，可以参考文章：&quot;<a href="http://arxiv.org/pdf/1305.5800v1.pdf">Lightweight Contention Management for Efficient Compare-and-Swap Operations</a>&quot;。但是，有时候很难判断竞争是否会一致持续下去，若只产生一次竞争，因为这次竞争的失败而让该线程休眠若干时间，等于是浪费了休眠的时间。</div><div><br/></div><div>下文将介绍Java中的embarrassing parallel（即各并行任务不相关的场景）的处理方法，以及ThreadLocal变量。</div><div><br/></div></span>
</div>