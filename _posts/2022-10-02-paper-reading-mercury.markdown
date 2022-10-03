---
layout:     post
title:      "论文解读 Mercury: Hybrid Centralized and Distributed Scheduling in Large Shared Cluster "
date:       2022-10-02 10:00:00
author:     "Hawkingrei"
header-img: "img/post-bg-2015.jpg"
catalog: true
tag: [Linux, C] 
---

最近的一段时间里，做了一些资源调度的调研，于是正好找到这篇的 Microsoft 的 paper。就如生物学里的杂交个体，有着更强的生命力一样。结合了 Centralized Scheduling 和 Distributed Scheduling 特征的 Mercury 也有着比前两者，更加的适应性和灵活性。

# Microsoft 遇到的问题

Microsoft 内部有个统一的 share cluster，里面会运行各种类型的任务。例如 Long-running services, production SLA jobs, ad-hoc jobs 等等，而这些任务本身有着从毫秒级到无限大的执行时间。所以如何在集群内部达到最佳的投入产出比，就十分重要了。

传统上调度器其实有两种设计，一种以 Yarn，Mesos，Borg 等等为代表的中心

# Mercury 架构与设计

