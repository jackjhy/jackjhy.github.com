---
layout: post
title: "Vector Clock,面包房(Lamport)算法和Paxos选举算法"
description: ""
category: ""
tags: []
---
{% include JB/setup %}

还真都是一脉相传的算法，面包房算法和Paxos算法都是Lamport提出的，就是LaTex中间的La，Vector Clock基本上可以算是Lamport算法的改进型。这三个算法都可用于分布式进程（系统）之间的调度协调，区别仅在于前置约束不同，从最简单的面包房算法出发，去除掉进程之间存在同步的原子时钟的前置条件，就需要在发送的协调消息中间为每一个进程维护单独的步进标志位，这就是Vector Clock算法。去除掉消息必然会被送达到目标进程的前置条件，考虑接受方离线，则必然会加上确认机制，以及维护有序的协调流程，这就是Paxos算法。必然，Paxos算法也是没有同步原子时钟的，包含Vector Clock思想在里面也就是理所当然的。


面包房算法
===
思路源于顾客去面包店采购。面包店只能接待一位顾客的采购。已知有n位顾客要进入面包店采购，安排他们按照次序在前台登记一个签到号码。该签到号码逐次加1。根据签到号码的由小到大的顺序依次入店购货。完成购买的顾客在前台把其签到号码归0. 如果完成购买的顾客要再次进店购买，就必须重新排队。这是wiki百科的描述，个人以为说的重点精确。

另外一种网上比较广泛的描述是“顾客在面包店中购买面包时的排队原理. 顾客在进入面包店前, 首先抓一个号, 然后按照号码由小到大的次序依次进入面包店购买面包. 这里, 面包店发放的号码是由小到大的, 但是两个或两个以上的顾客却有可能得到相同的号码(使所抓号码不同需要互斥), 如果多个顾客抓到相同的号码, 则规定按照顾客名字的字典次序进行排序, 这里假定顾客是没有重名的.”个人认为，遗漏了同步原子时钟，已经有了Vector Clock的雏形了。

算法实现重点在于分发进程自己的签号，并各自维护一个签号有序队列，进入可执行状态之后，分发移除签号消息。

伪代码：

```
     // declaration and initial values of global variables
     Entering: array [1..NUM_THREADS] of bool = {false};
     Number: array [1..NUM_THREADS] of integer = {0};
 
  1  lock(integer i) {
  2      Entering[i] = true;
  3      Number[i] = 1 + max(Number[1], ..., Number[NUM_THREADS]);
  4      Entering[i] = false;
  5      for (j = 1; j <= NUM_THREADS; j++) {
  6          // Wait until thread j receives its number:
  7          while (Entering[j]) { /* nothing */ }
  8          // Wait until all threads with smaller numbers or with the same
  9          // number, but with higher priority, finish their work:
 10          while ((Number[j] != 0) && ((Number[j], j) < (Number[i], i))) { /* nothing */ }
 13      }
 14  }
 15  
 16  unlock(integer i) {
 17      Number[i] = 0;
 18  }
 19
 20  Thread(integer i) {
 21      while (true) {
 22          lock(i);
 23          // The critical section goes here...
 24          unlock(i);
 25          // non-critical section...
 26      }
 27  }
```
第三方这里获取当前最大签号，就是我上面所说的同步原子时钟的一种实现，通过全局共享数据实现。在单机环境下，其实原子时钟是更好的选择。

Vector Clock
===
这里就引用一篇了，以为网络上，没比他说得更加清楚的了。作者没找着。

> 向量时钟，最早是用于分布式系统中进程间的时间同步。由于在分布式系统中没有一个直接的全局逻辑时钟。在一个由n个并发进程构成的系统中，每个事件的逻辑时钟均由一个n维向量（n元组）构成，其中第i维（分量）对应于第i个进程的逻辑时钟Vi
> 
> 向量时钟的维护方法：
>
> * 初值
> * 第i个进程在事件发生时，继承上一事件的逻辑时钟并将自身所对应的分量Vi增加一个步长
> * 任何进程Pi在发出任何消息m时都要将自己当前的逻辑时钟分量Vi一起发送出去
> * 如果是接收事件，且发送方为j，则比较自己继承的Vj和收到的逻辑时钟，并取其中较大者为自己的Vj
> 
> 形式化表示如下：
> 在向量时钟方法中，每个进程Pi和一个向量LCi[1...n]相关联，其中
>
> * LCi[i]描述了进程Pi，即自身进程
> * LCi[j]表示Pi关于Pj进程的知识
> * LCi[1...n]组成Pi对于逻辑全局时间的局部视图
> 
> 对每一个进程LCi的更新规则如下
>
> * 规则1：在发生一个事件（一个外部发送或内部事件）之前，我们更新LCi[i]: LCi[i] := LCi[i] + d (d > 0)
> * 规则2：每个消息捎带发送方在发送时的向量时钟。当接收到一个消息（m, LCj, j）时，Pi执行更新： LCi[k] := max(LCi[k], LCj[k]), 1<=k<=n LCi[i] := LCi[i]+d 
> 
> 下图为增量值d=1，初始值init=0时的示例图
> ![示意图](https://raw.github.com/jackjhy/jackjhy.github.com/master/images/vc.png)
> 数据一致性（Quorum） 定义
>
> * N：系统中数据的备份数
> * R：读取操作最小成功数
> * W：写操作最小成功数

> 要求：R+W>N
>  
> 配置
> 
> * R+W>N，保证读操作至少能读取到一个最新数据
> * 读写速度受性能最差的节点影响
> * R和W越小，系统的响应越快
> * W越大，数据的一致性，可用性越强
> * 通常：N=3， R=W=2
> 
> 时钟向量与数据冲突处理
>
> * 将上述向量时钟应用到解决数据一致性的问题上来
> * 向量时钟在分布式环境中表示对象或事件的逻辑关系
> * 分布式系统中每个消息都附加Vector Clock的信息
> * 当某个节点内部处理一个事件时，将其自身对应的counter加1
> * 节点发送一个消息前，将其自身对应的counter加1
> * 节点收到一个消息时，将其自身对应的counter加1；同时，根据消息中的Vector Clock信息更新所有其他counter
> 
> 在Dynamo系统中为了可用性，所以对一致性的约束做了妥协。使得整个系统对“写”一直是可用的，而将数据不一致性的处理部分放到了客户端程序读数据时再进行处理。
> 在客户端使用get获取数据时，coordinator对node返回数据依据时钟向量进行version处理。如果数据一致则将取得的值返回给客户端。如果数据冲突，则将所有数据返回给客户端，由客户端进行解决。


Paxos
===
给个超链接，wiki(http://zh.wikipedia.org/wiki/Paxos%E7%AE%97%E6%B3%95)
另外，有一个关于Paxos的有趣故事：

> Lamport大牛在他Paxos的第一个版本早在1990年就提交给ACM TOCS Jnl.的评审委员会了 但是当时没有人理解他的算法 主编回执他的稿子建议他用数学而不是神话描述他的算法 他们才会考虑接受这篇paper
> 
> Lamport大牛很生气 他有次就在一个会议上说:”为什么搞理论的这群人一点幽默感也没有呢?” 他拒绝修改 而且withdraw了这篇文章
> 
> 1996年微软的Butler Lampson在WDAG96上提出了重新审视这篇文章 因为他读懂了
> 
> 1997年MIT的Nancy Lynch在WDAG97上 根据原文重新改写了这篇文章 叫做 在本文中她们”代替”Lamport用数学形式化的定义并证明了Paxos
> 
> 于是在1998年的ACM TOCS上 这篇迟到了9年的paper终于被接受了
> 
> 后来2001年 Lamport大牛也作出了让步 他用简单的语言而不是神话故事 重述了原文 这就是 但是通篇还是没有数学符号 L大牛甚为固执 据他自己说 他检查过了自己的语言并没有歧义 不需要数学来描述

