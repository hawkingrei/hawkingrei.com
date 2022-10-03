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


