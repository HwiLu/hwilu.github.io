---
layout: post
title: 什么是孤儿进程和僵尸进程，如何kill它们。
categories: [Linux]
description: 找到linux主机上的孤儿进程和僵尸进程，并杀掉它们。
keywords: Linux
---

# 什么是僵尸进程

在 unix 和 Linux 系统中，已经死掉但是依旧存在进程表中的进程称之为僵尸进程，通常是因为bugs或者编码错误所致。一个僵尸进程在其父进程断定其退出状态（exit status）不再需要前将一直保留在操作系统内。

## 何时进程将变为僵尸进程
通常，当一个进程完成运行，它将向其父进程汇报其退出状态。直到父进程决定其子进程的退出状态不再需要保留，子进程转变为僵尸进程（defunct or zombie process）。僵尸进程不会消耗资源同时也不能调度运行。有时候父进程保留子进程为僵尸状态以确保之后的子进程不会使用相同的 PID。

## 如何找到一个僵尸进程
可以使用 `ps aux | grep Z` 命令来找到僵尸进程。在状态位带有 Z 关键字的进程代表其为僵尸进程。

我们可使用 `top` 命令查看系统中是否有僵尸进程存在。

![zombie-process](/images/posts/linux/zombie-process.png)

## 如何 kill 一个僵尸进程
要杀掉一个僵尸进程，需找到其父进程号（PPID）并给它发送SIGCHLD (17) 信号：kill -17 ppid。可使用以下命令去找到PPID
```vim
ps -p PID -o ppid
```
e.g.
```
$  ps -p 20736 -o ppid

PPID

20735

$ kill -17 20735
```
**注意**：如果 kill 掉僵尸进程的父进程，僵尸进程本身也将死掉。

如果一个僵尸进程的父进程号为 1 ，即其父进程为 init 进程，那只有通过重启的方式来杀掉该进程。


# 什么是孤儿进程

当其父进程死掉(终止)时，该进程被称为孤儿进程。父进程死掉的进程将会被初始进程（init process）接管。

## 一个进程何时会变为孤儿进程

一个父进程退出，而它的一个或多个子进程还在运行，那么那些子进程将成为孤儿进程。用户还可以通过将其与终端分离来创建一个孤儿进程。

## 如何找到孤儿进程

以下命令除了孤儿进程，还会显示所有具有PPID 1（即将init进程作为其父进程）的进程。

```
$ ps -elf | awk '{if ($5 == 1){print $4" "$5" "$15}}'

298 1 upstart-udev-bridge

302 1 udevd

438 1 /usr/sbin/sshd

[...]
```

孤儿进程使用大量资源，因此可以通过 `top` 或 `htop` 轻松找到它们。 要杀死孤儿进程，可使用命令：kill -9 PID。



refer： 
- [https://linux.cn/article-8451-1.html](https://linux.cn/article-8451-1.html)

- [](http://linuxg.net/what-are-zombie-and-orphan-processes-and-how-to-kill-them/)

















