---
layout:     post
title:      "论文解读 Mercury: Hybrid Centralized and Distributed Scheduling in Large Shared Cluster"
date:       2022-10-02 10:00:00
author:     "Hawkingrei"
header-img: "img/post-bg-2015.jpg"
catalog: true
tag: [Linux, C] 
---

最近的一段时间里，做了一些资源调度的调研，于是正好找到这篇的 Microsoft 的 paper。就如生物学里的杂交个体，有着更强的生命力一样。结合了 Centralized Scheduling 和 Distributed Scheduling 特征的 Mercury 也有着比前两者，更加的适应性和灵活性。

# Microsoft 遇到的问题

Microsoft 内部有个统一的 share cluster，里面会运行各种类型的任务。例如 Long-running services, production SLA jobs, ad-hoc jobs 等等，而这些任务本身有着从毫秒级到无限大的执行时间。所以如何在集群内部达到最佳的投入产出比，就十分重要了。

传统上调度器其实有两种设计，一种以 Yarn，Mesos，Borg 等等为代表的中心的 Centralized Resource Management System。所有的调度决定都需要通过中心化的调度器来进行。并且由调度器来解决资源请求中的冲突问题，并保证给到每个任务请求的资源。其优势就在于有一个中心化的节点之后，可以以全局视角获取机器的负载、健康情况等等维度的信息，来进行调度。但是这些也阻碍了扩展性和低延时。对于大规模的集群来说，各个 node 会将心跳信息上传到 RM，这势必会造成大量的网络开销。而且在资源调度的时候，也需要 RM 来进行全局的调度，这也会造成一定的延时。同时当大量的调度任务展开时，延迟问题将更加的严重。而另外一方面中心化的调度器，设计时会给予其动态扩展调度逻辑的特色，比如说 K8S 的 CRD，所以非常的灵活。

另一种以 Apoll、 等等为代表的分布式的 Distributed Resource Management System(这里的 Distributed 我们应该理解为去中心化的, 而不是分布式，scheduler 无论哪个方案，其都是一个分布式系统)。其解决了 Centralized Resource Management System 的扩展性和延迟问题，但是由于其架构的原因导致其没法解决资源冲突问题。所以其并不适用于 production SLA service 和 long-running job 类型的任务。并且很难在一个去中心化的架构中，实现调度逻辑的扩展性，所以其灵活性也很欠佳。

所以我们是否可以结合 Centralized Resource Management System 和 Distributed Resource Management System 的优点，来设计一个 Hybrid Resource Management System。将 short job 交给 Distributed Resource Management System 来实现低延时高吞吐的调度，而 long job 交给 Centralized Resource Management System 来实现资源冲突的解决和调度逻辑的扩展性，同时满足 long job 对稳定性的要求。

# Mercury 架构与设计

在介绍架构前，需要先引入 Mercury 定义的两个概念：GUARANTEED containers 和 QUEUEABLE containers。

GUARANTEED containers 特指不会产生排队延迟。任务抵达节点，就马上启动，并且他不会被中途暂时，会被 kill。

QUEUEABLE containers 是指在调度时，会被放入队列中等待的任务。任务下达后，不会马上启动。Mercury 同时也不保证任务的排队延迟，也不保证容器将运行完成还是被抢占。

Mercury 由两个组件组成：

Mercury Runtime:

运行在集群内的每个工作节点，负责和应用通信，用于在每个节点上执行 execution policies。

Mercury Resource Management Framework:

这就是一个子系统，包括一个在专用节点上运行的中央调度器, 以及一组分布式调度器运行在各个worker节点上。它们通过一个Mercury Coordinator进行松散的协调。这种调度器的组合在集群范围内对应用程序进行资源分配。

总结就是其架构就包含了两种调度器，一个是中心化的调度器，一个是去中心化的调度器。中心化的节点可以收集到这个节点的资源使用情况，做出准确的判断, 而其代价就是较差的扩展性，和较高的延迟。而去中心化调度正好与之相反，调度速度很快，但其准确性不好。所以在 Mercury 内，GUARANTEED 会通过中心化调度来完成， QUEUEABLE 会通过去中心化调度来完成。


