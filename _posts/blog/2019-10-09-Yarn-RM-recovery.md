---
layout: post
title: Yarn Active ResourceManager 重启作业不中断配置
categories: [Yarn]
description: Yarn  ResourceManager recovery配置
keywords: Yarn,HA
---

在默认情况下，即使在RM已经配置了HA（即已具备active/standby 两个RM）的前提下，active 的 RM 重启会被停止均会导致正在 Yarn 上运行的作业失败，并丢失所有作业信息。

如图所示：



![yarn 作业在RM切换之后，进展终止](/images/posts/yarn/job-failed-after-rm-failover.png)

![Yarn ui上找不到任何作业信息](/images/posts/yarn/job-failed-after-rm-failover2.png)

这个因为YARN未开启recovery机制所致。

涉及到的配置。

| key                                                          | value                                                        | 备注                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| yarn.resourcemanager.recovery.enabled                        | true                                                         | 是否启用recovery机制                                         |
| yarn.resourcemanager.store.class                             | org.apache.hadoop.yarn.server.<br />resourcemanager.recovery.ZKRMStateStore | 有三种StateStore，分别是基于zookeeper, HDFS, leveldb, HA高可用集群必须用ZKRMStateStore |
| yarn.resourcemanager.work-preserving-recovery.scheduling-wait-ms | 10000                                                        | 默认10000，用默认值即可                                      |




