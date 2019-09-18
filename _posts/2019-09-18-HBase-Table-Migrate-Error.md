---
layout: post
title: HBase表【HDFS文件】迁移报错分析
categories: HBase Hadoop HDFS
description: 使用hdfs的distcp工具对HBase表迁移报错，抛出长度不匹配异常。
keywords: HBase Hadoop HDFS
---

# 问题现象
今天使用HDFS工具distcp对HBase表进行迁移备份时，发现distcp的MR人物最后几个Map Task执行失败了，导致整个MR任务失败。于是查询MR application的作业的发现1400多个map失败了11个，点进去看Map Task查看日志

**failed 的Map Task的日志**

```
2019-09-18 14:44:02,472 ERROR [main] org.apache.hadoop.tools.util.RetriableCommand: Failure in Retriable command: Copying hdfs://192.168.52.11/apps/hbase/data/data/default/weibo_user_201907/39cd5924d0551e010254a67086550c82/.tmp/5fc158e18769449cb04a1c0663b2d63b to hdfs://Beijing/OriginalData/HBase/Backup/weibo_user_201907/39cd5924d0551e010254a67086550c82/.tmp/5fc158e18769449cb04a1c0663b2d63b
java.io.IOException: Mismatch in length of source:hdfs://192.168.52.11/apps/hbase/data/data/default/weibo_user_201907/39cd5924d0551e010254a67086550c82/.tmp/5fc158e18769449cb04a1c0663b2d63b (12884901888) and target:hdfs://Beijing/OriginalData/HBase/Backup/.distcp.tmp.attempt_1566477700032_952407_m_000180_0 (13154949632)
	at org.apache.hadoop.tools.mapred.RetriableFileCopyCommand.compareFileLengths(RetriableFileCopyCommand.java:194)
	at org.apache.hadoop.tools.mapred.RetriableFileCopyCommand.doCopy(RetriableFileCopyCommand.java:126)
	at org.apache.hadoop.tools.mapred.RetriableFileCopyCommand.doExecute(RetriableFileCopyCommand.java:99)
	at org.apache.hadoop.tools.util.RetriableCommand.execute(RetriableCommand.java:87)
	at org.apache.hadoop.tools.mapred.CopyMapper.copyFileWithRetry(CopyMapper.java:282)
	at org.apache.hadoop.tools.mapred.CopyMapper.map(CopyMapper.java:253)
	at org.apache.hadoop.tools.mapred.CopyMapper.map(CopyMapper.java:50)
	at org.apache.hadoop.mapreduce.Mapper.run(Mapper.java:146)
	at org.apache.hadoop.mapred.MapTask.runNewMapper(MapTask.java:787)
	at org.apache.hadoop.mapred.MapTask.run(MapTask.java:341)
	at org.apache.hadoop.mapred.YarnChild$2.run(YarnChild.java:175)
	at java.security.AccessController.doPrivileged(Native Method)
	at javax.security.auth.Subject.doAs(Subject.java:422)
	at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1961)
	at org.apache.hadoop.mapred.YarnChild.main(YarnChild.java:169)
2019-09-18 14:44:06,609 INFO [main] org.apache.hadoop.tools.mapred.RetriableFileCopyCommand: Creating temp file: hdfs://Beijing/OriginalData/HBase/Backup/.distcp.tmp.attempt_1566477700032_952407_m_000180_0
2019-09-18 14:46:18,976 INFO [ResponseProcessor for block BP-581842485-192.168.4.18-1521440332837:blk_1624621819_551166390] org.apache.hadoop.hdfs.DataStreamer: Slow ReadProcessor read fields for block BP-581842485-192.168.4.18-1521440332837:blk_1624621819_551166390 took 47134ms (threshold=30000ms); ack: seqno: 49954 reply: SUCCESS reply: SUCCESS reply: SUCCESS downstreamAckTimeNanos: 20982562 flag: 0 flag: 0 flag: 0, targets: [DatanodeInfoWithStorage[192.168.9.169:1019,DS-37489985-d63b-4fe5-b35c-2fa6e67cb1a9,DISK], DatanodeInfoWithStorage[192.168.11.84:1019,DS-d0b7e852-56b5-4343-9e99-62ca9dd47ce1,DISK], DatanodeInfoWithStorage[192.168.6.215:1019,DS-7d286b5b-b701-4493-864d-3e1e5e675c65,DISK]]
2019-09-18 14:50:27,808 ERROR [main] org.apache.hadoop.tools.util.RetriableCommand: Failure in Retriable command: Copying hdfs://192.168.52.11/apps/hbase/data/data/default/weibo_user_201907/39cd5924d0551e010254a67086550c82/.tmp/5fc158e18769449cb04a1c0663b2d63b to hdfs://Beijing/OriginalData/HBase/Backup/weibo_user_201907/39cd5924d0551e010254a67086550c82/.tmp/5fc158e18769449cb04a1c0663b2d63b
java.io.IOException: Mismatch in length of source:hdfs://192.168.52.11/apps/hbase/data/data/default/weibo_user_201907/39cd5924d0551e010254a67086550c82/.tmp/5fc158e18769449cb04a1c0663b2d63b (12884901888) and target:hdfs://Beijing/OriginalData/HBase/Backup/.distcp.tmp.attempt_1566477700032_952407_m_000180_0 (13154949632)
	at org.apache.hadoop.tools.mapred.RetriableFileCopyCommand.compareFileLengths(RetriableFileCopyCommand.java:194)
	at org.apache.hadoop.tools.mapred.RetriableFileCopyCommand.doCopy(RetriableFileCopyCommand.java:126)
	at org.apache.hadoop.tools.mapred.RetriableFileCopyCommand.doExecute(RetriableFileCopyCommand.java:99)
	at org.apache.hadoop.tools.util.RetriableCommand.execute(RetriableCommand.java:87)
	at org.apache.hadoop.tools.mapred.CopyMapper.copyFileWithRetry(CopyMapper.java:282)
	at org.apache.hadoop.tools.mapred.CopyMapper.map(CopyMapper.java:253)
	at org.apache.hadoop.tools.mapred.CopyMapper.map(CopyMapper.java:50)
	at org.apache.hadoop.mapreduce.Mapper.run(Mapper.java:146)
	at org.apache.hadoop.mapred.MapTask.runNewMapper(MapTask.java:787)
	at org.apache.hadoop.mapred.MapTask.run(MapTask.java:341)
	at org.apache.hadoop.mapred.YarnChild$2.run(YarnChild.java:175)
	at java.security.AccessController.doPrivileged(Native Method)
	at javax.security.auth.Subject.doAs(Subject.java:422)
	at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1961)
	at org.apache.hadoop.mapred.YarnChild.main(YarnChild.java:169)
2019-09-18 14:50:30,623 INFO [main] org.apache.hadoop.tools.mapred.RetriableFileCopyCommand: Creating temp file: hdfs://Beijing/OriginalData/HBase/Backup/.distcp.tmp.attempt_1566477700032_952407_m_000180_0
2019-09-18 14:57:04,939 ERROR [main] org.apache.hadoop.tools.util.RetriableCommand: Failure in Retriable command: Copying hdfs://192.168.52.11/apps/hbase/data/data/default/weibo_user_201907/39cd5924d0551e010254a67086550c82/.tmp/5fc158e18769449cb04a1c0663b2d63b to hdfs://Beijing/OriginalData/HBase/Backup/weibo_user_201907/39cd5924d0551e010254a67086550c82/.tmp/5fc158e18769449cb04a1c0663b2d63b
java.io.IOException: Mismatch in length of source:hdfs://192.168.52.11/apps/hbase/data/data/default/weibo_user_201907/39cd5924d0551e010254a67086550c82/.tmp/5fc158e18769449cb04a1c0663b2d63b (12884901888) and target:hdfs://Beijing/OriginalData/HBase/Backup/.distcp.tmp.attempt_1566477700032_952407_m_000180_0 (13154949632)
	at org.apache.hadoop.tools.mapred.RetriableFileCopyCommand.compareFileLengths(RetriableFileCopyCommand.java:194)
	at org.apache.hadoop.tools.mapred.RetriableFileCopyCommand.doCopy(RetriableFileCopyCommand.java:126)
	at org.apache.hadoop.tools.mapred.RetriableFileCopyCommand.doExecute(RetriableFileCopyCommand.java:99)
	at org.apache.hadoop.tools.util.RetriableCommand.execute(RetriableCommand.java:87)
	at org.apache.hadoop.tools.mapred.CopyMapper.copyFileWithRetry(CopyMapper.java:282)
	at org.apache.hadoop.tools.mapred.CopyMapper.map(CopyMapper.java:253)
	at org.apache.hadoop.tools.mapred.CopyMapper.map(CopyMapper.java:50)
	at org.apache.hadoop.mapreduce.Mapper.run(Mapper.java:146)
	at org.apache.hadoop.mapred.MapTask.runNewMapper(MapTask.java:787)
	at org.apache.hadoop.mapred.MapTask.run(MapTask.java:341)
	at org.apache.hadoop.mapred.YarnChild$2.run(YarnChild.java:175)
	at java.security.AccessController.doPrivileged(Native Method)
	at javax.security.auth.Subject.doAs(Subject.java:422)
	at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1961)
	at org.apache.hadoop.mapred.YarnChild.main(YarnChild.java:169)
2019-09-18 14:57:04,941 ERROR [main] org.apache.hadoop.tools.mapred.CopyMapper: Failure in copying hdfs://192.168.52.11/apps/hbase/data/data/default/weibo_user_201907/39cd5924d0551e010254a67086550c82/.tmp/5fc158e18769449cb04a1c0663b2d63b to hdfs://Beijing/OriginalData/HBase/Backup/weibo_user_201907/39cd5924d0551e010254a67086550c82/.tmp/5fc158e18769449cb04a1c0663b2d63b
java.io.IOException: File copy failed: hdfs://192.168.52.11/apps/hbase/data/data/default/weibo_user_201907/39cd5924d0551e010254a67086550c82/.tmp/5fc158e18769449cb04a1c0663b2d63b --> hdfs://Beijing/OriginalData/HBase/Backup/weibo_user_201907/39cd5924d0551e010254a67086550c82/.tmp/5fc158e18769449cb04a1c0663b2d63b
	at org.apache.hadoop.tools.mapred.CopyMapper.copyFileWithRetry(CopyMapper.java:285)
	at org.apache.hadoop.tools.mapred.CopyMapper.map(CopyMapper.java:253)
	at org.apache.hadoop.tools.mapred.CopyMapper.map(CopyMapper.java:50)
	at org.apache.hadoop.mapreduce.Mapper.run(Mapper.java:146)
	at org.apache.hadoop.mapred.MapTask.runNewMapper(MapTask.java:787)
	at org.apache.hadoop.mapred.MapTask.run(MapTask.java:341)
	at org.apache.hadoop.mapred.YarnChild$2.run(YarnChild.java:175)
	at java.security.AccessController.doPrivileged(Native Method)
	at javax.security.auth.Subject.doAs(Subject.java:422)
	at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1961)
	at org.apache.hadoop.mapred.YarnChild.main(YarnChild.java:169)
Caused by: java.io.IOException: Couldn't run retriable-command: Copying hdfs://192.168.52.11/apps/hbase/data/data/default/weibo_user_201907/39cd5924d0551e010254a67086550c82/.tmp/5fc158e18769449cb04a1c0663b2d63b to hdfs://Beijing/OriginalData/HBase/Backup/weibo_user_201907/39cd5924d0551e010254a67086550c82/.tmp/5fc158e18769449cb04a1c0663b2d63b
	at org.apache.hadoop.tools.util.RetriableCommand.execute(RetriableCommand.java:101)
	at org.apache.hadoop.tools.mapred.CopyMapper.copyFileWithRetry(CopyMapper.java:282)
	... 10 more
Caused by: java.io.IOException: Mismatch in length of source:hdfs://192.168.52.11/apps/hbase/data/data/default/weibo_user_201907/39cd5924d0551e010254a67086550c82/.tmp/5fc158e18769449cb04a1c0663b2d63b (12884901888) and target:hdfs://Beijing/OriginalData/HBase/Backup/.distcp.tmp.attempt_1566477700032_952407_m_000180_0 (13154949632)
	at org.apache.hadoop.tools.mapred.RetriableFileCopyCommand.compareFileLengths(RetriableFileCopyCommand.java:194)
	at org.apache.hadoop.tools.mapred.RetriableFileCopyCommand.doCopy(RetriableFileCopyCommand.java:126)
	at org.apache.hadoop.tools.mapred.RetriableFileCopyCommand.doExecute(RetriableFileCopyCommand.java:99)
	at org.apache.hadoop.tools.util.RetriableCommand.execute(RetriableCommand.java:87)
	... 11 more
2019-09-18 14:57:04,954 WARN [main] org.apache.hadoop.mapred.YarnChild: Exception running child : java.io.IOException: File copy failed: hdfs://192.168.52.11/apps/hbase/data/data/default/weibo_user_201907/39cd5924d0551e010254a67086550c82/.tmp/5fc158e18769449cb04a1c0663b2d63b --> hdfs://Beijing/OriginalData/HBase/Backup/weibo_user_201907/39cd5924d0551e010254a67086550c82/.tmp/5fc158e18769449cb04a1c0663b2d63b
	at org.apache.hadoop.tools.mapred.CopyMapper.copyFileWithRetry(CopyMapper.java:285)
	at org.apache.hadoop.tools.mapred.CopyMapper.map(CopyMapper.java:253)
	at org.apache.hadoop.tools.mapred.CopyMapper.map(CopyMapper.java:50)
	at org.apache.hadoop.mapreduce.Mapper.run(Mapper.java:146)
	at org.apache.hadoop.mapred.MapTask.runNewMapper(MapTask.java:787)
	at org.apache.hadoop.mapred.MapTask.run(MapTask.java:341)
	at org.apache.hadoop.mapred.YarnChild$2.run(YarnChild.java:175)
	at java.security.AccessController.doPrivileged(Native Method)
	at javax.security.auth.Subject.doAs(Subject.java:422)
	at org.apache.hadoop.security.UserGroupInformation.doAs(UserGroupInformation.java:1961)
	at org.apache.hadoop.mapred.YarnChild.main(YarnChild.java:169)
Caused by: java.io.IOException: Couldn't run retriable-command: Copying hdfs://192.168.52.11/apps/hbase/data/data/default/weibo_user_201907/39cd5924d0551e010254a67086550c82/.tmp/5fc158e18769449cb04a1c0663b2d63b to hdfs://Beijing/OriginalData/HBase/Backup/weibo_user_201907/39cd5924d0551e010254a67086550c82/.tmp/5fc158e18769449cb04a1c0663b2d63b
	at org.apache.hadoop.tools.util.RetriableCommand.execute(RetriableCommand.java:101)
	at org.apache.hadoop.tools.mapred.CopyMapper.copyFileWithRetry(CopyMapper.java:282)
	... 10 more
Caused by: java.io.IOException: Mismatch in length of source:hdfs://192.168.52.11/apps/hbase/data/data/default/weibo_user_201907/39cd5924d0551e010254a67086550c82/.tmp/5fc158e18769449cb04a1c0663b2d63b (12884901888) and target:hdfs://Beijing/OriginalData/HBase/Backup/.distcp.tmp.attempt_1566477700032_952407_m_000180_0 (13154949632)
	at org.apache.hadoop.tools.mapred.RetriableFileCopyCommand.compareFileLengths(RetriableFileCopyCommand.java:194)
	at org.apache.hadoop.tools.mapred.RetriableFileCopyCommand.doCopy(RetriableFileCopyCommand.java:126)
	at org.apache.hadoop.tools.mapred.RetriableFileCopyCommand.doExecute(RetriableFileCopyCommand.java:99)
	at org.apache.hadoop.tools.util.RetriableCommand.execute(RetriableCommand.java:87)
	... 11 more
```
日志提示源端文件与目标文件不一致，于是检查源文件的文件状态

**hdfs fsck结果**
`hadoop fsck hdfs://192.168.52.11/apps/hbase/data/data/default/weibo_user_201907`

```
....................................................................................................
...................................................................................Status: HEALTHY
 Total size:    1059681892872431 B (Total open files size: 12884901888 B)
 Total dirs:    79904
 Total files:   355983
 Total symlinks:                0 (Files currently being written: 1)
 Total blocks (validated):      2196898 (avg. block size 482353706 B) (Total open file blocks (not validated): 24)
 Minimally replicated blocks:   2196898 (100.0 %)
 Over-replicated blocks:        0 (0.0 %)
 Under-replicated blocks:       0 (0.0 %)
 Mis-replicated blocks:         0 (0.0 %)
 Default replication factor:    3
 Average block replication:     3.0
 Corrupt blocks:                0
 Missing replicas:              0 (0.0 %)
 Number of data-nodes:          222
 Number of racks:               1
FSCK ended at Wed Sep 18 17:01:02 CST 2019 in 45655 milliseconds
```
发现`(Total open file blocks (not validated): 24)`有文件被打开。

找到被打开文件
```
> cat fsck.log  | grep OPENFORWRITE
............/apps/hbase/data/data/default/weibo_user_201907/39cd5924d0551e010254a67086550c82/.tmp/5fc158e18769449cb04a1c0663b2d63b 12884901888 bytes, 24 block(s), OPENFORWRITE: .......................................................................................
```
我们发现这个.tmp/目录是在region文件夹下面，所以怀疑是region正在compaction产生的新文件，而在迁移之前我们已经将该表disable了，所以该临时文件删除不影响数据的完整性。

# 解决方案
现在有两种方案进行解决
1. 重新enable表，让HBase自己处理该临时文件（这里原本是觉得它会将compaction过程走完，然后tmp文件变成合并后的文件，后面想想可能HBase会直接将该文件删除，这里还没有确定，不太肯定），再disable该表。事实证明，该方法是有效的，再重新disable之后，发现fsck没有了被打开的文件。

**正常的hdfs fsck 结果**
```
....................................................................................................
......Status: HEALTHY
 Total size:    1058696103638004 B
 Total dirs:    60003
 Total files:   354278
 Total symlinks:                0
 Total blocks (validated):      2194007 (avg. block size 482539984 B)
 Minimally replicated blocks:   2194007 (100.0 %)
 Over-replicated blocks:        0 (0.0 %)
 Under-replicated blocks:       0 (0.0 %)
 Mis-replicated blocks:         0 (0.0 %)
 Default replication factor:    3
 Average block replication:     3.0
 Corrupt blocks:                0
 Missing replicas:              0 (0.0 %)
 Number of data-nodes:          222
 Number of racks:               1
FSCK ended at Wed Sep 18 18:24:39 CST 2019 in 42928 milliseconds


The filesystem under path '/apps/hbase/data/data/default/weibo_user_201907' is HEALTHY
```
然后再迁移，即可成功。

2. 直接删除./tmp目录，然后重新迁移
因为正常disable表成功之后，这些临时文件可以删除不影响数据的完整性，这样便可以处理源端与目的端的不一致，这种办法被证明有效。

网上还提到有以下方法，不过有人说无效[distcp-mismatch-in-length-of-source](https://stackoverflow.com/questions/41542844/distcp-mismatch-in-length-of-source)

**加上`-skipcrccheck`参数**
`-skipcrccheck`参数的解释为

> - skipcrcchech
Whether to skip CRC checks between source and target paths

关于`CRC check`的解释在：[CRC check](https://baike.baidu.com/item/%E5%BE%AA%E7%8E%AF%E5%86%97%E4%BD%99%E6%A0%A1%E9%AA%8C%E7%A0%81/10168758?fromtitle=CRC%E6%A0%A1%E9%AA%8C&fromid=3439037)

即使用`hadoop distcp -skipcrccheck -update hdfs://ip1/xxxxxxxxxx/xxxxx hdfs:///xxxxxxxxxxxx/ `重新迁移。	
