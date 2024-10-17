---
title: Spark抽象和架构
date: 2020-05-24 21:58:35
categories:
- spark
tags:
- 大数据
---

# Spark抽象和架构

## 一、Spark架构

Spark 往往作为统一资源管理平台的用户，向统一资源管理平台提交作业，作业提交成功后，Spark 的作业会被调度成计算任务，在资源管理系统的容器中运行。在集群运行中的 Spark 架构是典型的主从架构，如下面这张图所示。

<img src="/Spark抽象和架构/spark架构.png" style="zoom:33%;" />

<!-- more -->

我们先来看看 Spark 架构，在运行时，Driver 是主节点，而 Executor 是从节点，这 3 个 Executor 分别运行在资源管理系统中的 3 个容器中。

在 Spark 的架构中，Driver 主要负责作业调度工作，Executor 主要负责执行具体的作业计算任务，Driver 中的 SparkSession 组件，是 Spark 2.0 引入的一个新的组件，曾经我们熟悉的 SparkContext、SqlContext、HiveContext 都是 SparkSession 的成员变量。

因此，用户编写的 Spark 代码是从新建 SparkSession 开始的。其中 **SparkContext 的作用是连接用户编写的代码与运行作业调度以及任务分发的代码**。当用户提交作业启动一个 Driver 时，会通过 SparkContext 向集群发送命令，Executor 会遵照指令执行任务。一旦整个执行过程完成，Driver 就会结束整个作业。

<img src="/Spark抽象和架构/spark架构2.png" alt="spark架构2" style="zoom:33%;" />

上图更详细地描述了 Driver 与 Executor 之间的运行过程。

- 首先，Driver 会根据用户编写的代码生成一个计算任务的有向无环图（Directed Acyclic Graph，DAG），这个有向无环图是 Spark 区别 Hadoop MapReduce 的重要特征；

- 接着，DAG 会根据 RDD（弹性分布式数据集，图中第 1 根虚线和第 2 根虚线中间的圆角方框）之间的依赖关系被 DAG Scheduler 切分成由 Task 组成的 Stage，这里的 Task 就是我们所说的计算任务，注意这个 Stage 不要翻译为阶段，这是一个专有名词，它表示的是一个计算任务的集合；

- 最后 TaskScheduler 会通过 ClusterManager 将 Task 调度到 Executor 上执行。

可以看到，Spark 并不会直接执行用户编写的代码，而用户代码的作用只是告诉 Spark 要做什么，也就是一种“声明”。

## 二、Spark抽象

当用户编写好代码向集群提交时，一个作业就产生了，作业的英文是 job，在 YARN 中，则喜欢把作业叫 application。Driver 会根据用户的代码生成一个有向无环图，下面这张图就是根据用户逻辑生成的一个有向无环图。

<img src="/Spark抽象和架构/spark抽象.png" alt="spark抽象" style="zoom:33%;" />

通过这张图，可以大概看出计算逻辑：A 和 C 都是两张表，在分别进行分组聚合和筛选的操作后，做了一次 join 操作。

在上图中，灰色小方框就是我们所说的分区（partition），它和计算任务是一一对应的，也就是说，有多少个分区，就有多少个计算任务，显然的，一个作业，会有多个计算任务，这也是分布式计算的意义所在，我们可以通过设置分区数量来控制每个计算任务的计算量。在 DAG 中，每个计算任务的输入就是一个分区，一些相关的计算任务所构成的任务集合可以被看成一个 Stage，这里"相关"指的是某个标准，我们后面会讲到。RDD 则是分区的集合（图中 A、B、C、D、E），用户只需要操作 RDD 就可以构建出整个 DAG。

一个 Executor 同时只能执行一个计算任务，但一个 Worker（物理节点）上可以同时运行多个 Executor。Executor 的数量决定了同时处理任务的数量，一般来说，分区数远大于 Executor 数量才是合理的。所以同一个作业，在计算逻辑不变的情况下，分区数和 Executor 的数量很大程度上决定了作业运行的时间。





