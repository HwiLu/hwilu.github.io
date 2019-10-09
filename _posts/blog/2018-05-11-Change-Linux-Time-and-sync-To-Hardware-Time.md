---
layout: post
title: 如何强行修改服务器时间，并将系统时间写入硬件时间
categories: Linux
description: 强行修改服务器时间，并将系统时间写入硬件时间
keywords: Linux, date
---

介绍在服务器时间不同步时，如何进行校正。

## 强行校正服务器时间

使用以下格式：

```
date -s 14:22:20
```
如果有相应的时钟源，也可以使用`ntpdate`进行同步
```
# ntpdate  0.cn.pool.ntp.org

# date
```
## 同步至硬件时钟

查看硬件时钟：

```
# hwclock --show
```
然后把系统时间写入到硬件：
```
# hwclock -w
# hwclock --show
```