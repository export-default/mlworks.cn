+++
author = "Ryan Zhao"
categories = ["Java","并发编程"]
date = "2016-04-18T17:53:46+08:00"
description = "并发编程的基本问题：可见性与原子性，以及Java中的解决方案。"
keywords = ["Java并发编程","高性能并发"]
mathjax = false
tags = ["并发","Java"]
title = "Java极致并发：（一）并发编程基础"

+++


<div>
<span><div>随着CPU主频趋于稳定，CPU性能的提升趋向于在芯片上集成更多的处理核心。单纯地等待更强的CPU来提升程序性能的时代已经过去了，如何有效地利用多核进行高性能的并发运算变得越来越重要。本系列文章是我在学习高性能并发编程中的一些心得，与诸君分享。</div><div><br/></div><div>本篇主要讨论并发编程中的两个基本问题，其产生的原因以及解决方法。</div><div><br/></div><div><b>一、操作的原子性</b></div><div><br/></div><div>一个操作是原子性的，如果它对系统的其它部分来讲，是即时发生的。我们来考虑下面这个常见例子，假设两个线程A、B同时执行下面的increment方法：</div><div><br/></div><div style="background-color:#ffffff;color:#000000;font-family:&apos;Consolas&apos;;font-size:11.3pt;"><span style="color:#000080;font-weight:bold;">public class </span>Example_1 {<br/>
    <span style="color:#000080;font-weight:bold;">private volatile int </span><span style="color:#660e7a;font-weight:bold;">i </span>= <span style="color:#0000ff;">0</span>;<br/><br/>
    <span style="color:#000080;font-weight:bold;">void </span>increment() {<br/>
        <span style="color:#660e7a;font-weight:bold;">i</span>++;<br/>
    }</div><div><span style="font-size: 11.3pt;"><span style="font-family: Consolas;">}</span></span></div><div><span style="font-size: 11.3pt;"><span style="font-family: Consolas;"><br/></span></span></div><div>我们知道 i++ 并不是一个原子操作，其大致相当与如下代码</div><div><br/></div><div><span style="font-size: 11.3pt;"><span style="font-family: Consolas;"><span style="color:#000080;font-weight:bold;">1：int </span>temp = <span style="color:#660e7a;font-weight:bold;">i</span>;</span></span></div><div style="background-color:#ffffff;color:#000000;font-family:&apos;Consolas&apos;;font-size:11.3pt;"><b><span style="color: rgb(65, 0, 125);">2：</span></b>temp = temp +<span style="color:#0000ff;">1</span>;</div><div><span style="font-size: 11.3pt;"><span style="font-family: Consolas;"><span style="color:#660e7a;font-weight:bold;"><span style="color: rgb(79, 0, 154);">3：</span>i </span>= temp;</span></span></div><div><span style="font-size: 11.3pt;"><span style="font-family: Consolas;"><br/></span></span></div><div>若线程A执行到第二行时，线程A由于某种原因中断执行，与此同时线程B一直在运行。则当线程A被唤醒后继续执行时，在第三行将会对i赋予错误的值。</div><div><br/></div><div>需要注意的是，原子性之是要求操作对该系统的其余部分是即时发生的，并不代表该操作不会被其他底层事件中断。</div><div><br/></div><div>如何来保证操作的原子性呢？多余大部分应用场景来说，解决方案是使用互斥锁。互斥锁保证了该系统中，同时只能有一个线程执行该操作，因此满足了上述的原子性要求。</div><div><br/></div><div><b>二、共享变量的可见性</b></div><div><b><br/></b></div><div>共享变量的可见性问题相对与原子性来说，较为隐晦一些。原因在于这并不是由于并发编程的本质特性而带来的问题，而是由于性能优化带来的副作用。我们知道，CPU会乱序执行指令、编译器会优化代码，而这些性能优化仅保证在单线程情况下执行结果和我们写的代码执行结果一致。</div><div>这给编译器作者和CPU设计者带来了巨大的优化空间，却为并发程序设计者带来了一些问题。试想，对于上述的Example_1来说，如果每个核心将变量i放入自己的寄存器中，会产生什么样的结果。</div><div><br/></div><div>因此，必须有一种机制来确保共享变量的可见性。在CPU级别上，通过添加内存屏障指令，而在语言级别上，通过添加关键字来使编译器不对共享变量做相关优化。</div><div><br/></div><div>关于内存屏障的问题，强烈推荐阅读文章“<b>Memory Barriers: a Hardware View for Software Hackers</b> ”，其详细讲述了为什么会产生可见性问题、以及其解决方法。</div><div><br/></div><div>在Java语言中，Java内存模型中有这样的保证：</div><div><br/></div><div><img src="/images/jmm_happens_before.png" type="image/png"/></div><div><br/></div><div>如果操作A happens-before 操作B，则操作A对B是可见的，即不论A、B是否在同一个核心运行，A对内存的操作对B都是可见的。上述的保证也可以理解为：</div><ol><li>当获得锁之后，上一个锁的拥有者执行的所有操作，对当起的锁拥有者都可见。</li><li>对volatile字段的读操作，永远能够读到最新的值。（Java中的volatile与C++中的有明显不同，C++中的volatile并没有可见性的保证）</li><li>调用start方法启动线程的线程中的所有操作，对启动的线程均可见。</li><li>当A线程中线程B.join返回后，线程B中的所有操作，对A线程可见。</li><li>所有对象的默认初始化操作（变量的默认值）对其他操作均可见。</li></ol><div><br/></div><div><b>三、总结</b></div><div><b><br/></b></div><div>从上述讨论中可以看出，可见性保证是并发编程的基础，若没有可见性保证，对共享内存操作的原子性保证是没有意义的。从JMM的规范中也可以看出，锁既可以保证原子性，也可以保证可见性。</div><div>而volatile关键字，仅仅保证可见性。</div><div><br/></div><div>下篇文章将讨论Java语言为并发编程提供的基本工具。</div></span>
</div>