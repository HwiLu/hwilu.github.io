---
layout: post
title: "nodemanager始终无法启动，并提示Caused by: java.io.IOException: Not able to enforce cpu weights; cannot find cgroup for cpu controller in /proc/mounts"
categories: [Yarn]
description: 
keywords: Yarn
---

对于一个新加入的 nodemanager 节点或者刚换好磁盘的节点可能会出现 nodemanager 角色无法启动的现象。

查看 nodemanager 日志可以发现以下报错：

```vim
at org.apache.hadoop.yarn.server.nodemanager.NodeManager.initAndStartNodeManager http://org.apache.hadoop.yarn.server.nodemanager.nodemanager.initandstartnodemanager/NodeManager.java:537 at org.apache.hadoop.yarn.server.nodemanager.NodeManager.main http://org.apache.hadoop.yarn.server.nodemanager.nodemanager.main/ NodeManager.java:585
Caused by: java.io.IOException: http://java.io.ioexception/ Not able to enforce cpu weights; cannot find cgroup for cpu controller in /proc/mounts
```

如以下截图所示：
![nm-error](/images/posts/yarn/nm-error-01.png)


我们查看相应的 `/proc/mount` 文件，发现：

![mounts-error](/images/posts/yarn/nm-error-proc-mounts-error.png)


而正常的节点为这个样子：
![mounts-error](/images/posts/yarn/nm-error-proc-mounts-right.png)

解决方案

以下操作步骤需安装顺序执行。

- 先安装libcgroup
```
yum install libcgroup -y
mount -t cgroup -o cpu none /cgroup/cpu
```

- 改权限

```
# mkdir /cgroup/cpu/hadoop-yarn
# chown -R yarn:hadoop /cgroup/cpu/hadoop-yarn
```

- 重启nm 

[Refer To Hortonworks](https://support.hortonworks.com/s/article/Not-able-to-start-the-Node-Managers-exception)
