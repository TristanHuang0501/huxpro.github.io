---
layout:     post
title:      分布式原理（3）
subtitle:   BASE 定理
date:       2018-11-28
author:     "Tristan"
header-img: "img/post-bg-androidstudio.jpg"
catalog:    true
tags:
- Distribution System
---

> eBay的架构师Dan Pritchett源于对大规模分布式系统的实践总结，在ACM上发表文章提出BASE理论，BASE理论是对CAP理论的延伸，核心思想是即使无法做到强一致性（Strong Consistency，CAP的一致性就是强一致性），但应用可以采用适合的方式达到最终一致性（Eventual Consitency）。

BASE 是 **Basically Available(基本可用)**、**Soft State(软状态)** 和 **Eventually Consistent(最终一致性)** 三个短语的简写。BASE 是对 CAP 中一致性和可用性权衡的结果，其来源于对大规模互联网系统分布式实践的总结，是**基于 CAP 定理逐步演化**而来的，其核心思想是即使无法做到强一致性，但每个应用都可以根据自身的业务特点，采用适当的方法来使系统达到**最终一致性**。接下来，我们着重对 BASE 中的三要素进行讲解。

### 1. Basically Available(基本可用)

基本可用是指分布式系统在出现不可预知故障的时候，允许损失部分可用性，保证核心可用——但请注意，这绝不等价于系统不可用。一下就是两个"基本可用"的例子:

- **响应时间上的损失**：正常情况下，一个在线搜索引擎需要在0.5秒之内返回给用户相应的查询结果，但由于出现故障（比如系统部分机房发生断电或断网故障），查询结果的响应时间增加到了1~2秒。
- **功能上的损失**：电商大促时，为了应对访问量激增，部分用户可能会被引导到降级页面，服务层也可能只提供降级服务。这就是损失部分可用性的体现，如下图：
![](https://ws1.sinaimg.cn/large/006tNbRwly1fxnulq4xe9j30820eimx9.jpg)

### 2. Soft State(软状态)
软状态是指允许系统中的数据存在**中间状态**，并认为该中间状态的存在不会影响系统的整体可用性，即允许系统在不同的数据副本之间进行数据同步的过程**存在延时**。分布式存储中一般一份数据至少会有三个副本，允许不同节点间副本同步的延时就是软状态的体现。


### 3. Eventually Consistent(最终一致性)
最终一致性强调的是系统中所有的数据副本，在经过一段时间的同步后，**最终能够达到一个一致的状态**。因此，最终一致性的本质是需要系统保证最终数据能够达到一致，而不需要实时保证系统数据的强一致性。

最终一致性是一种**特殊的弱一致性**：系统能够保证在没有其他新的更新操作的情况下，数据最终一定能够达到一致的状态，因此所有客户端对系统的数据访问都能够获取到最新的值。同时，在没有发生故障的前提下，数据到达一致状态的时间延迟，取决于网络延迟、系统负载和数据复制方案设计等因素。

在实际工程实践中，最终一致性存在以下五类主要变种:

 - 因果一致性(Causal consistency)
 - 读己之所写(Read your writes)
 - 会话一致性(Session consistency)
 - 单调读一致性(Monotonic read consistency)
 - 单调写一致性(Monotonic write consistency)

以上就是最终一致性的五种常见的变种，在实际系统实践中，可以将其中的若干个变种互相结合起来，以构建一个具有最终一致性特性的分布式系统。事实上，最终一致性并不是只有那些大型分布式系统才涉及的特性，许多**现代的关系型数据库**都采用了最终一致性模型。在现代关系型数据库中（比如MySQL和PostgreSQL），大多都会采用**同步或异步**方式来实现**主备数据复制**技术。在同步方式中，数据的复制过程通常是更新事务的一部分，因此在事务完成后，主备数据库的数据就会达到一致。而在异步方式中，备库的更新往往会存在延时，这取决于事务日志在主备数据库之间传输的时间长短。如果传输时间过长或者甚至在日志传输过程中出现异常导致无法及时将事务应用到备库上，那么很显然，从备库中读取的数据将是旧的，因此就出现了数据不一致的情况。当然，无论是采用多次重试还是人为数据订正，关系型数据库还是能够保证最终数据达到一致，这就是系统提供最终一致性保证的经典案例。

### BASE 和 ACID 的比较

| ACID                   | BASE        |
|:-----------------------|:------------|
| 强一致性                   | 弱一致性        |
| 隔离性（一个事务的执行不能被其他事务所干扰） | 可用性优先       |
| 采用悲观、保守方法              | 采用乐观方法      |
| 难以变化                   | 适应变化、更简单、更快 |

ACID和BASE代表了两种截然相反的设计哲学。ACID是传统数据库常用的设计理念，追求强一致性模型。BASE支持的是大型分布式系统，提出通过牺牲强一致性获得高可用性。在分布式系统设计的场景中，系统组件对一致性要求是不同的，因此ACID和BASE又会结合使用。

### 参考资料

 - [分布式系列文章——从ACID到CAP/BASE](https://mp.weixin.qq.com/s/epksv_GNCTrZ5-3cGlxewA?)


