---
layout: post
title: Capacity-Scheduler如何使用node-labels
categories: [Yarn]
description: Capacity-Scheduler调度器
keywords: Yarn, Hadoop
---

## 什么是Node-label
实际的环境部署中，经常会出现不同的机器类型，比如有些机器是计算型的，有些则是内存型；另一种场景是在大集群中，有时候需要指定有些机器预留给特定的用户用，从而避免其它用户的任务对其造成影响；node label节点标签就是解决这类问题的一种好的方式。运维人员可以根据节点的特性将其分为不同的分区来满足业务多维度的使用需求。Yarn的Node-label功能将很好的试用于异构集群中，可以更好地管理和调度混合类型的应用程序。
## Node-label特性
 - 只适用于Capacity Scheduler，不支持Fair Scheduler
 - 一个Node Manager节点只能属于一个label，如果一个资源节点没有配置label，则其属于一个不存在的DEFAULT分区【即没有被设置label的节点】，没有指定队列的作业将会被放置到这些节点上运行，如果所有节点被设置label，没有指定队列的将会停止在accepted节点，后续将因为得不到资源而failed
- 用户可以为每个队列配置可以访问的分区【label】，默认是只可以访问DEFAULT分区
- 如果队列配置为对所有label可访问，该队列上的作业可以使用所有label节点上的资源
- 可以设置每个队列访问特定分区的资源比率
- node label以及队列和node label的相关配置支持动态更新----即可使用`yarn rmadmin –replaceLabelsOnNode` 修改节点标签

#如何开启Node-label
默认情况下系统时没有开启node label标签功能的，可以在yarn-site.xml中修改下列配置来开启label特性。
```python
<property>
       <name>yarn.node-labels.enabled</name>
       <value>true</value>
</property>
<property>
    <name>yarn.node-labels.manager-class</name>
<value>
org.apache.hadoop.yarn.server.resourcemanager.nodelabels.RMNodeLabelsManager
</value>
</property>
<property>
   <name>yarn.node-labels.fs-store.root-dir</name>
   <value>hdfs://Host0:8020/yarn/node-labels</value>
   <description>
标签数据在HDFS上的存储位置，该目录需要提前创建
</description>
</property>
```
   关于为什么要设置这个HDFS的node-labels存储目录，是因为label信息默认是保存在内存中的，如果将label信息存于hdfs上，重启resourcemanager之后label信息不会因此丢失。
并，将该目录设置为yarn所有。
##配置步骤：
1. 先按上述要求修改`yarn-site.xml`
2. 添加集群便签
`yarn rmadmin -addToClusterNodeLabels label01,label02`
3.	添加节点标签
`yarn rmadmin -replaceLabelsOnNode node1:45454,label01`
`yarn rmadmin -replaceLabelsOnNode node2:45454,label02`
需一个一个添加，节点较多时建议写成脚本执行
4.	查看标签
`yarn node -status node1:45454`
也可以通过Yarn管理页面查看`Node Label`
node-label webUI
http://RM-Address:port/cluster/nodelabels
## 与node-labels有关的重要配置项
- `yarn.scheduler.capacity.<queue-path>.default-node-label-expression ：` 队列默认的访问label，如果请求中未设置label，则设置为该值；默认为空，表现允许访问无label的节点
- `yarn.scheduler.capacity.<queue-path>.accessible-node-labels：`队列可以访问的label列表,如`"label01,label02"`,通过逗号分隔，另外队列均可以访问没有标签的node;默认继承父队列的`accessible labels`；如果只允许访问无标签的node，配置为一个空
-	`yarn.scheduler.capacity.<queue-path>.accessible-node-labels.<label>.capacity：`队列对某个label的容量设置，对于同一个label属于同一个父队列下面的capacity总和必须为100
-	`yarn.scheduler.capacity.<queue-path>.accessible-node-labels.<label>.maximum-capacity：`队列对某个label资源的最大访问容量，默认是100

## 配置实例

配置文件较长，以下贴出node-labes配置片段：
```vim
<!-- node-label配置 -->
  <property>
    <name>yarn.scheduler.capacity.root.default.accessible-node-labels</name>
    <value>*</value>
  </property>
  <property>
    <name>yarn.scheduler.capacity.root.default.accessible-node-labels</name>
    <value>*</value>
  </property>
  <property>
    <name>yarn.scheduler.capacity.root.queue1.accessible-node-labels</name>
    <value>*</value>
  </property>
<!--value值为空时，代表对任何label都无权限-->
  <property>
    <name>yarn.scheduler.capacity.root.queue2.accessible-node-labels</name>
    <value>label02</value>
  </property>
```