---
layout: post
title: HBase 自带性能测试工具的使用
categories: HBase
description: HBase test 
keywords: HBase
---
和HDFS一样，HBase也提供了自带的benchmark测试工具供用户测试HBase性能，本文主要记录HBase benchmark工具--PerformanceEvaluation 的用法。PerformanceEvaluation  简称为 pe。

我们可以使用 `hbase pe` 查看其相应的参数：


```
$ hbase pe
Usage: java org.apache.hadoop.hbase.PerformanceEvaluation
  <OPTIONS> [-D<property=value>]* <command> <nclients>

Options:
 nomapred        Run multiple clients using threads (rather than use mapreduce)
 rows            Rows each client runs. Default: One million
 size            Total size in GiB. Mutually exclusive with --rows. Default: 1.0.
 sampleRate      Execute test on a sample of total rows. Only supported by randomRead. Default: 1.0
 traceRate       Enable HTrace spans. Initiate tracing every N rows. Default: 0
 table           Alternate table name. Default: 'TestTable'
 multiGet        If >0, when doing RandomRead, perform multiple gets instead of single gets. Default: 0
 compress        Compression type to use (GZ, LZO, ...). Default: 'NONE'
 flushCommits    Used to determine if the test should flush the table. Default: false
 writeToWAL      Set writeToWAL on puts. Default: True
 autoFlush       Set autoFlush on htable. Default: False
 oneCon          all the threads share the same connection. Default: False
 presplit        Create presplit table. Recommended for accurate perf analysis (see guide).  Default: disabled
 inmemory        Tries to keep the HFiles of the CF inmemory as far as possible. Not guaranteed that reads are always served from memory.  Default: false
 usetags         Writes tags along with KVs. Use with HFile V3. Default: false
 numoftags       Specify the no of tags that would be needed. This works only if usetags is true.
 filterAll       Helps to filter out all the rows on the server side there by not returning any thing back to the client.  Helps to check the server side performance.  Uses FilterAllFilter internally. 
 latency         Set to report operation latencies. Default: False
 bloomFilter      Bloom filter type, one of [NONE, ROW, ROWCOL]
 valueSize       Pass value size to use: Default: 1024
 valueRandom     Set if we should vary value size between 0 and 'valueSize'; set on read for stats on size: Default: Not set.
 valueZipf       Set if we should vary value size between 0 and 'valueSize' in zipf form: Default: Not set.
 period          Report every 'period' rows: Default: opts.perClientRunRows / 10
 multiGet        Batch gets together into groups of N. Only supported by randomRead. Default: disabled
 addColumns      Adds columns to scans/gets explicitly. Default: true
 replicas        Enable region replica testing. Defaults: 1.
 splitPolicy     Specify a custom RegionSplitPolicy for the table.
 randomSleep     Do a random sleep before each get between 0 and entered value. Defaults: 0
 columns         Columns to write per row. Default: 1
 caching         Scan caching to use. Default: 30

 Note: -D properties will be applied to the conf used. 
  For example: 
   -Dmapreduce.output.fileoutputformat.compress=true
   -Dmapreduce.task.timeout=60000

Command:
 append          Append on each row; clients overlap on keyspace so some concurrent operations
 checkAndDelete  CheckAndDelete on each row; clients overlap on keyspace so some concurrent operations
 checkAndMutate  CheckAndMutate on each row; clients overlap on keyspace so some concurrent operations
 checkAndPut     CheckAndPut on each row; clients overlap on keyspace so some concurrent operations
 filterScan      Run scan test using a filter to find a specific row based on it's value (make sure to use --rows=20)
 increment       Increment on each row; clients overlap on keyspace so some concurrent operations
 randomRead      Run random read test
 randomSeekScan  Run random seek and scan 100 test
 randomWrite     Run random write test
 scan            Run scan test (read every row)
 scanRange10     Run random seek scan with both start and stop row (max 10 rows)
 scanRange100    Run random seek scan with both start and stop row (max 100 rows)
 scanRange1000   Run random seek scan with both start and stop row (max 1000 rows)
 scanRange10000  Run random seek scan with both start and stop row (max 10000 rows)
 sequentialRead  Run sequential read test
 sequentialWrite Run sequential write test

Args:
 nclients        Integer. Required. Total number of clients (and HRegionServers)
                 running: 1 <= value <= 500
Examples:
 To run a single client doing the default 1M sequentialWrites:
 $ bin/hbase org.apache.hadoop.hbase.PerformanceEvaluation sequentialWrite 1
 To run 10 clients doing increments over ten rows:
 $ bin/hbase org.apache.hadoop.hbase.PerformanceEvaluation --rows=10 --nomapred increment 10
```

 介绍一个用例：

​	1.随机写测试 RandomWriteTest 

```shell
hbase pe --nomapred --oneCon=true --valueSize=100 --compress=SNAPPY --rows=150000 --autoFlush=true --presplit=60 randomWrite 64
```

 --rows=150000 代表每个线程会写入150000行数据 ；

 autoFlush 默认为false，即PE默认用的是BufferedMutator，BufferedMutator会把数据攒在内存里，达到一定的大小再向服务器发送，如果想明确测单行Put的写入性能，建议设置为true；

 presplit 表的预分裂region个数，在做性能测试时一定要设置region个数，不然所有的读写会落在一个region上，严重影响性能 ；

64 代表起了64个线程来做写入 。

​	2.顺序读：sequentialRead

例如，顺序读1千万条数据：
```
hbase org.apache.hadoop.hbase.PerformanceEvaluation --nomapred --rows=1000000 sequentialRead 10 
```
1000000 * 10 = 10,000,000

​	3.顺序写

```
 hbase/bin/hbase org.apache.hadoop.hbase.PerformanceEvaluation sequentialWrite 1
```
​	4.随机读

```
hbase/bin/hbase org.apache.hadoop.hbase.PerformanceEvaluation randomRead 1
```
建议测试随机读时使用顺序写来准备数据。

参考：[segmentfault文章-较为详细](https://segmentfault.com/a/1190000015276482)