---
layout: post
title: HBase 如何下线一个 regionserver 节点
categories: HBase
description: HBase decommission a rs node
keywords: HBase
---

原理不多说，直接说操作步骤。
## 在0.90.2之前
在需要下线的节点上执行
```shell
./bin/hbase-daemon.sh stop regionserver
```
这条语句执行后，该RegionServer首先关闭其负责的所有Region而后关闭自己。在关闭时，RegionServer在ZooKeeper中的“Ephemeral Node”会失效。此时，Master检测到RegionServer挂掉并把它作为一个宕机节点，并将该RegionServer上的Region重新分配到其他RegionServer。
在使用此方法前，一定要关闭HBase Load Balancer。关闭方法：
```shell
hbase(main):001:0> balance_switch false
true
```
## 在0.90.2之后
HBase添加了一个新的方法，即“graceful_stop”,只需要在HBase Master节点执行：

```shell
./bin/graceful_stop.sh rs主机名
```
该命令会自动关闭Load Balancer，然后Assigned Region，之后会将该节点关闭。

** graceful stop的用法 **

```
$ ./bin/graceful_stop.sh
Usage: graceful_stop.sh [--config &conf-dir>] [--restart] [--reload] [--thrift] [--rest] &hostname>
 thrift      If we should stop/start thrift before/after the hbase stop/start
 rest        If we should stop/start rest before/after the hbase stop/start
 restart     If we should restart after graceful stop
 reload      Move offloaded regions back on to the stopped server
 debug       Move offloaded regions back on to the stopped server
 hostname    Hostname of server we are to stop
```
 在关闭之后，我们在HBase webui上可以看到对应的rs已经是dead的状态了，再查看rs在zk里的信息`get /hbase/rs`，发现该节点也已经被删除了。
但是，我们在hbase内还是可以看到这个rs是dead的这条信息，这是因为，HMaster从zk拿到rs的信息中有一个是dead的，所以依旧会显示有节点死了，我们需要重新HMaster来去掉这条信息。
最后，手动打开 load balancer

```
hbase(main):001:0> balance_switch true
false
0 row(s) in 0.3590 seconds
```



