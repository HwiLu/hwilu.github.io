---
layout: post
title: KafkaOffsetMonitor测试与使用，并解决内网页面显示异常问题
categories: [Kafka]
description: KafkaOffsetMonitor测试与使用
keywords: Kafka
---

This is an app to monitor your kafka consumers and their position (offset) in the queue.

## 安装

开源地址：[KafkaOffsetMonitor](https://github.com/quantifind/KafkaOffsetMonitor)

点击一下地址下载：[download](https://github.com/quantifind/KafkaOffsetMonitor/releases/download/v0.2.1/KafkaOffsetMonitor-assembly-0.2.1.jar)

该github 仓库上已经明确说明了部署方法，非常简单。

1. 将KafkaOffsetMonitor-assembly-0.2.1.jar上传到一台服务器（该服务器需已经安装jdk，并配置好环境变量）上，制作以下脚本：


```shell
   # cat kafkaOffsetMonitor.sh 
   #!/bin/bash
   java -cp KafkaOffsetMonitor-assembly-0.2.1.jar \
        com.quantifind.kafka.offsetapp.OffsetGetterWeb \
        --zk zk01.site.com:2181,zk02.site.com:2181,zk03.site.com:2181 \
        --port 8089 \
        --refresh 10.seconds \
        --retain 2.days
   
   #     --offsetStorage kafka
```

   将脚本与jar包放在同一个目录之下。

2. 运行脚本

```shell
nohup sh kafkaOffsetMonitor.sh  &
```

打开相应的ui地址：http://ip:8089

## 测试

使用kafka生产和消费数据，属于kafka的知识，不详细展开。

在kafka节点。

### 创建topic

```shell
bin/kafka-console-producer.sh --broker-list kafka01.site.com:6667,kafka02.site.com:6667,kafka03.site.com:6667 --topic test01 
```

### 创建producer

```shell
bin/kafka-console-producer.sh --broker-list kafka01.site.com:6667,kafka02.site.com:6667,kafka03.site.com:6667 --topic test01 
```
### 创建consumer
```shell
bin/kafka-console-consumer.sh --zookeeper zk01.site.com:2181,zk02.site.com:2181,zk03.site.com:2181 --topic test01 --from-beginning
```

## 生产大批量数据测试

### 创建topic

```shell
bin/kafka-topics.sh --create --zookeeper zk01.site.com:2181,zk02.site.com:2181,zk03.site.com:2181 --replication-factor 3  --partition 3  --topic test_perf_topic
```
### 创建producer

```shell
./kafka-producer-perf-test.sh  --topic  test_perf_topic --num-records 1000000  --throughput  10 --record-size 100000 --producer-props   bootstrap.servers=kafka01.site.com:6667
```
### 创建consumer

```shell
/kafka-consumer-perf-test.sh --zookeeper zk01.site.com:2181,zk02.site.com:2181,zk03.site.com:2181 --topic test_perf_topic --messages 10000000000
```

![kafkaMonitorOffset](/images/posts/kafka/offset.png)



**注:**

- Topic：创建Topic名称
- Partition：分区编号
- Offset：表示该Parition已经消费了多少Message
- LogSize：表示该Partition生产了多少Message
- Lag：表示有多少条Message未被消费
- Owner：表示消费者
- Created：表示该Partition创建时间
- Last Seen：表示消费状态刷新最新时间

## 内网访问问题

当我们使用公司内网访问时，会发现这个网页只显示了文字，真正的ui并没有正常显示，网上已经给出了一些解决办法，即更换 angular 的三个js文件。参考：[KafkaOffsetMonitor-assembly-0.2.1.jar使用遇到的问题](https://blog.csdn.net/feinifi/article/details/83015492)

然后发现还是不行，问题显然不止这一个，于是使用 `F12` 查看该页面的调试信息，发现：
![显示异常](/images/posts/kafka/kafkaMonitorUiError.png)
我们可以看到，有部分js和css文件无法加载出来，于是我们从 offsetapp/index.html 找到这些css和js文件的地址。全部下下之后放在 offsetapp/css 目录之下，然后修改该5个文件的地址，如下图所示：

![bootstrap](/images/posts/kafka/kafkaMonitorBootstrap.png)

全部替换完之后，重新打jar包，发现可以正常访问了。

![kafkaMonitor](/images/posts/kafka/kafkaMonitorUiRight.png)

下载的文件我上传在[这](https://github.com/HwiLu/blog-comments/tree/master/kafkaOffsetMonitor)了。

## jar包解压及压缩
记录一下

### 解压到指定位置

```
unzip KafkaOffsetMonitor-assembly-0.2.1.jar -d web
```
### 重新打包
```shell
cd web/;
jar cvfm KafkaOffsetMonitor-assembly-0.2.1.jar ./META-INF/MANIFEST.MF ./
```