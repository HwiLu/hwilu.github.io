---
layout: post
title: "CentOS6账号密码正确可su但无法ssh登陆，messages提示error:Could not get shadow information for user"
categories: [Linux]
description: 
keywords: Linux
---

因机器重启，重启后发现，普通账号无法`ssh 账号名@ip`登陆，但是却可以在本机使用其他账号`su - 账号名`的方式登陆。

查看sshd_config配置并无异常。遂怀疑是selinux未永久关闭所致，查看`/etc/selinux/config `确实发现`SELINUX=enforcing`。 
![ssh error](/images/posts/linux/ssh-error-01.png)

## 解决办法

关闭selinux
先临时关闭
`setenforce 0`
永久关闭
`vim /etc/selinux/config`
修改
`SELINUX=enforcing 为 SELINUX=disabled。`
这样便无需重启机器，使用ssh再次登录发现可以登录了。