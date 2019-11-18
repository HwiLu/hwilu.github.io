---
layout: post
title: "CentOS6账号密码正确但是su和ssh均无法登陆，messages提示error:Could not get shadow information for user"
categories: [Linux]
description: 
keywords: Linux
---

因机器重启，**重启后发现**，除了可以免密登陆这台主机之外（之前配置过一台节点可以免密ssh登陆到这台主机）所有账号无法`ssh 账号名@ip`登陆，发现呈现以下错误。



![ssh-error](/images/posts/linux/ssh-error-01.png)

同时，适用su到其他用户输入正确的密码也无法登陆成功，提示 ` su:incorrect password`。于是我认为这并不是 ssh 的问题，应该是密码认证的问题。

于是查看 `/var/log/secure` 日志时也发现上述图片一样的报错。



便查看了 sshd_config 配置文件的 `UsePAM` 参数，发现是 `YES` 。

查看sshd_config配置并无异常，查看 `/etc/shadow` 文件权限也正常。遂怀疑是selinux未永久关闭所致，查看`/etc/selinux/config `确实发现`SELINUX=enforcing`。 


## 解决办法

**关闭selinux**

先临时关闭

`setenforce 0`

永久关闭

`vim /etc/selinux/config`

修改

`SELINUX=enforcing 为 SELINUX=disabled。`

这样便无需重启机器，使用ssh再次登录发现可以登录了。

